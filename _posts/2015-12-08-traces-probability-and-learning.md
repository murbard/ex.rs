---
layout: post
title: "Traces, probability, and learning"
author: arthurb
date: 2015-12-08 00:00:00 +0000
categories: math probability
---

This is a collection of thoughts I’ve had over the concept of program traces. The draft has resisted my attempts to clean it up and edit it for clarity for the past six months. I have chosen to post it as is rather than let it rot in my folders, please forgive its rough edges. I found it useful to write, perhaps someone will find it useful to read.

Programmers typically only encounter program traces in the context of debugging. However, looking at them under the lenses of probabilistic programming and recursive neural networks shows that they are of great conceptual importance.

A program trace is a sequence of state transitions that a program undergoes at runtime. You could think of these states as the content of a computer’s memory, but they could also be the different stages in the reduction of a lambda expression. In the following, we assume that the program is a state machine, but the arguments presented hold for other paradigms of computation.

## I. Probabilistic programming & particle filters

### I.A. Probabilistic programs

Probabilistic models are mathematical expressions containing random variables. Their goal is to describe observations as the byproduct of latent (or “hidden”) variables. They are often called generative model because the emphasis is on describing how the observations originated, or graphical models because the expression can be parsed as a graph, generally a DAG.

One typically sets a prior over the latent variables and attempts to estimate their posterior distribution given the observations. This estimate can be formed by deriving the posterior distribution analytically, by sampling (using Monte-Carlo techniques) or — in variational approaches — by deriving analytical approximations to the posterior.

Probabilistic programs are a generalization of probabilistic models replacing expressions with Turing-complete programs. They also generalize programs by enriching their semantic with two primitives:

* draw: which draws random latent variables

* observe: which observes the values of certain variables at runtime

Such programs are particularly well suited to represent Bayesian non-parametric models which generally involve processes (e.g. the Chinese restaurant process, the Dirichlet process, etc.)

A key difference between probabilistic models and probabilistic programs is that — in the latter — the number and type of the random variables are not known before runtime, and not all random variables may be instantiated in a given execution.

Over what space is the posterior distribution then defined? In general, it is defined over program traces. The probabilistic program defines a prior over traces, and the goal is to sample traces from the posterior distribution given the observations.

### I.B the particle filter

(You may skip this if you know how particle filters work. If you do not know how they work, this is hardly detailed enough to be a good introduction, but it contains just enough background for our purpose.)

A common approach to sample from these probabilistic programs uses particle filters. To get a better intuition of the motivation behind the algorithm, let’s start with a naive filter and build from there. We’ll set aside concerns about program termination: for all practical purposes, time bounds are known or can be imposed without major consequences.

A naive filter would run many instances of the program (or “particles”) in parallel, making random draws when needed. Whenever an observation is encountered, all the particles in disagreement with it are terminated while the others are allowed to continue their execution.

Unfortunately, in all but the simplest cases, no particle will happen to be in agreement with all the observations. This is particularly true when the observed variables are floating point numbers.

In most cases, though, observations are noisy. They might take the form:

    e ~ P(E) # draw a corrupting noise
    observe(x+e) # observe variable x corrupted with e

A naive filter would have every particle sample a different value of e, and retain only those particles for which x+e exactly matches the observation x’. A more astute approach consists in maintaining weights for each particle. Initially, those weights are all equal and sum to one but, when we encounter such an observation, we multiply each particle’s weight by P’s probability density function at point (x’-x). We then renormalize the particle’s weights and carry on.

So for instance, if e is a standard normal variable, we would multiply each particle’s weight by

(2 pi)^(-1/2) exp(-1/2 (x-x’)^2)

and then renormalize the weights to make sure they sum to one. Instances which predicted a value of x close to the observed value x’ will have their weight increased relative to the instances which predicted a value of x far from x’.

The log sum of the weights before renormalization is a likelihood estimate of making this observation conditional on past observations.

We’ve cheated a bit. We’ve marginalized over e by peeking into the future and using algebra. The equivalent, unmarginalized, version would have stumped our filter.

Though the particles may not fully disappear anymore, a single fortuitous particle will often end up bearing all the probability mass of the sample, which greatly increases the bias of the filter. Thus, a second trick is used: the particles are regularly resampled. Periodically, the set of particles is replaced by performing a random draw with replacement (also known as a bootstrap sample) from the set of weighted particles. This technique, known as sequential importance resampling, has the effect of pruning some of the very low weight particles and duplicating successful ones, without introducing bias in the limit.

### I.C The PMCMC algorithm

As such, this algorithm performs filtering, which means that at each point in time, the particles represent an approximate sample from the forward, or filtering, distribution. That is, the distribution of program states, at this point in the execution, given all of the past observations. In contrast, the smoothing distribution would incorporate future observations as well.

Interestingly, the fraction of the probability mass lost after conditioning on each observation gives us an estimate of how surprising that observation is. Their product across every execution step in the program gives an unbiased estimate of the likelihood of the observed data under the model.

The set of instances reaching the end of the program represents the leaves of a forest of program traces, the roots of which form a subset of the initial particles. In fact, for most problems, this forest will typically consist of a single tree, rooted in one of the most fortuitous initial choices.

By randomly sampling one of the leaves and following the trace back to the initial state, one can obtain a proposal for a Markov-Chain Monte Carlo algorithm, where the acceptance ratio is given by the ratio between the (noisy) estimate of the likelihood of the observations in the forward pass where we sampled the candidate trace and the (noisy) estimate of the likelihood of the observations in the forward pass where the current trace was sampled. This is the heart of the breakthrough PMCMC algorithm discovered by Andrieu, Doucet, and Holenstein. This variant is generally dubbed PIMH.

An alternative to this algorithm called Particle Gibbs consists in sampling a trace and performing the next filtering pass conditional on that trace. That is, one introduces an instance that merely replays the previously sampled trace, and resampling is performed at every step. This has the effect of “guiding” the sampling procedure around the previously sampled trace.

### I.D Where things go wrong

**Sticky chains**

So far, this looks great — and it is, provided that the traces do not have strong autocorrelation. This is what happens if many random decisions are made early on in the program execution, the consequences of which are only observed much later. We’ve already described one such case: if we restricted ourselves to sharp observations, most particles would sample the corrupting noise variable blindly and would never exactly hit the right observation. Marginalizing over this error is a nice trick that allows the filter to peek a little bit ahead. Unfortunately, there are many cases where this is not enough. Consider for instance the trivial program:

    for i = 1 to 64:
       x[i] ~ bernoulli(0.5)
    for i = 1 to 64:
       # the bernoulli(0.01) introduces a corrupting noise
       observe(x[i] or bernoulli(0.01))

While we can easily show that the posterior factorizes as x[i] ~ bernoulli(100/101) for all i, the filter will not be able to see that. It will try to sample random 64 bit vectors, and hopelessly realize that they do not fit observations very well. Ironically, the equivalent program:

    for i = 1 to 64:
       x[i] ~ bernoulli(0.5)
       observe(x[i] or bernoulli(0.01))

is trivial to sample from. The key difference is that in the second program, the observations come as we sample the x[i], thus constraining our choices progressively. The particle filter chokes when it is forced to maintain a representation of a high dimensional joint distribution, even though it is factorial in practice. The most obvious trick around this would be to use lazy evaluation of the random variables. However, consider now:

    for i = 1 to 64:
       x[i] ~ bernoulli(0.5)
       y = y or x[i]
    observe(y)
    for i = 1 to 64:
       observe(x[i])

Once again, it is trivial when looking at the program to see that all the x[i] must be true, but lazy evaluation will not save the filter here! Indeed, the observation of y implies that at least one of the x[i] must be true, and it forces the filter to sample potential values of x.

How do these pathologies present themselves when we sample from the full posterior using PIMH or particle Gibbs?

In PIMH, we’ll get sticky proposals. That is, one particle, during one pass will happen to sample more true values than any other has before, and this proposal will be accepted with high probability. However, that candidate will remain undethrowned for a very long time, and it cannot be used as a starting point to find a better candidate. Every pass will require blind luck, and our chain will have a very low acceptance probability.

The situation is somewhat better with particle Gibbs. In this case, we are likely to sample candidates which are closer and closer to the observation, as the sampled trace branches out into other possibilities. However, very often, a successful trace will be found which starts with an erroneous choice. This initial erroneous choice is unlikely to be corrected, as an alternative trace branching out early is unlikely to randomly rediscover the rest of the path. Particle Gibbs with ancestral sampling attempts to remedy this problem, but it is less straightforward to apply to program traces.

**Pathological cases are the rule, not the exception.**

The motivation for probabilistic programming is often to encode existing knowledge about a generating process and update this belief based on available data. In such case, we do not really collect any information during the execution of the program, we only observe an output at the very end. This is the worst case scenario for a filter, as these perform well when information is distilled over the course of the execution, not in one chunk, at the end. It means the filter has to make a lot of lucky guesses for its particles not to lose all of their mass by the end.

There are some ways one can work around these problems, by incorporating extra MCMC steps, or by exploiting the topological structure of the problem. In general, though, things would clearly be much easier if we could make observations along the way, and not only at the end. In fact, the more observations, the better. In the limit, we would observe every variable exactly and have nothing to do.

**Side channel attacks, when things go right (but shouldn’t)**

A common class of attacks against cryptographic systems are side channel attacks. They allow attackers to break cryptographic systems by using information leaked during the execution of the algorithm. This information could be the blinking of an LED, the time taken by the execution, voltage patterns, or the size of compressed packets.

This should be no surprise given what we’ve discussed so far. The secret in the algorithm acts as high dimensional random variable initialized early on, and it is probably impossible for a simple forward filter to reverse it better than brute force. However, the side channel gives our filter observations about the trace of the program, thus allowing us to progressively condition our distribution before the output is produced. The side channel is what guide our filtered traces towards the posterior distribution.

### I.E Probabilistic interpreters

A very useful probabilistic program would be an interpreter for a randomly generated deterministic program. Here, our only assumption would be that the observed data was produced by some algorithm, with some prior on the complexity of the algorithm.

This is, in fact, the basis of Solomonoff induction which argues that the prior on our experience of reality should the set of all possible programs, weighted by the inverse of the exponential of their binary encoding length. The biggest advantage of this prior is that, up to a constant, it does not depend on your model of computation. That is, in the limit, as you collect more data, you’ll reach the same posterior regardless of what model of computation you started with.

In practice, such a prior is uncomputable, but we’d like to do something similar… given some data, wouldn’t it be awesome to sample from a set of concise programs that can produce this data? Such programs would likely display good generalization properties, and help us make predictions — after all, understanding the world is all about describing the latent mechanisms at play.

Unfortunately, by this point, it is clear why this seems hopeless if we solely rely on a forward filtering approach. We would end up sampling programs and evaluating their outputs like monkeys throwing typewriters at a dartboard.

In general, my (pessimistic) intuition is that you cannot sample your way out of this problem because sampling is too myopic. You have to form theories about the data and about your program. A known manifestation of a similar problem arises in constraint satisfaction. Given n holes, can you put n+1 pigeons into the holes, so that no two pigeons share the same hole? The obvious answer is no, but if you express this problem as a Boolean circuit and restrict yourself to first order logic, you will need a proof of exponential size to show it. To solve the problem efficiently, you need to form a theory of integers and use that powerful theory to solve the problem. The problem expressed as a Boolean circuit cannot efficiently be solved from *within*. MCMC sampling is typically such a “within” approach, it does not see symmetries in the model, it does not see independence or algebraic marginalizations.

The recent successes of SMT solvers can be attributed to the fact that instead of throwing away the structure of the program by encoding everything in binary, as previous approaches had tried, they preserve as much of the structure as possible. Successful generic probabilistic programming approaches will need to do something similar, and detect and exploit any mathematical structure they can find in the programs.

## II. Recurrent neural networks

Recent advances in deep learning — and deep recurrent neural networks in particular — have made for spectacular demonstrations. Such networks provide state of the art speech recognition, natural language modelling, automated translation, image recognition, etc. An interesting approach, recently revived by Deepmind, consists in building recurrent networks that embody algorithms, and not merely expressions. I’ll draw examples from two of their papers, the neural Turing machine (or NTM) and the DRAW network.

The idea behind the Neural Turing Machine is to build a recurrent neural network that can access a memory. The memory is accessible both by index and by content using reading and writing “heads”. A key aspect of the design is that this machine is end to end differentiable. By that, it is meant that at every step in time, the state of the machine is a differentiable function of the previous state of the machine. Thus, the output will be a differentiable function of the input — albeit possibly a highly non linear function. The main subtlety is to define differentiable reading and writing operations in the memory.

Since a universal Turing machine can be expressed by such an RNN, Turing completeness follows. Less obvious is the ability of the machine to learn an algorithm using gradient descent only.

Merely having a gradient is no panacea. It is trivial for instance to turn any Boolean SAT problem into a differentiable problem by mapping the Booleans to the reals. Replace, for example, every Boolean term by a real number in [0,1] and use the differentiable NAND gate f(x,y) = 1-xy. Or, if you want an unbounded domain, use f(x,y) = Log(e^-x + e^-y + e^(-x-y)). However, there is no guarantee that applying gradient descent to this formula will yield correct solutions or even useful approximations.

The NTM paper does demonstrate positive results involving sorting and copying, but it is not clear whether more complex algorithms can be learned.

Another interesting paper is the DRAW paper. In this architecture, a recurrent neural network learns to recognize hand written digits by simultaneously learning to draw them. It does so using a recurrent attention model. Instead of learning to generate a whole hand written digit at once, it learns to focus its gaze to one part of the intput, and draw or erase one part of the output.

Such an architecture has in fact much in common with the neural Turing machine. The memory has been replaced with a canvas, and the writing operations are performed by drawing on this canvas. The fovea is the reading head and the paintbrush the writing head. Much as humans can carry arbitrary computations with pen an paper, so can the DRAW network.

Suppose we wanted to teach the DRAW network to perform long multiplication. One way to do this would be to use a form of supervised learning where a teacher literally performs long multiplication (using a graphic tablet for instance) , and the network learns to predict the strokes of the teacher on the canvas. At first, the network would learn a model of drawing digits, as they explain much of what the teacher is doing. Then, it would learn that these digits are written in particular places, generally from right to left and from top to bottom. It would learn that the value of a digit depends on the value of other digits in certain positions. It would learn to write the carry as superscript and to use it.

In addition to writing on the canvas, the teacher could also point to important or relevant parts. It might demonstrate the operation of summing two digits and a carry by pointing successively at all three. This would make it far easier for the network to train its attention model to focus on the correct parts at the correct time.

In the end the network would likely learn an algorithm that performs long multiplication. It would likely generalize well to arbitrary number of digits, since it only needs a fixed number of registers to carry on the execution.

Consider now presenting the same network with fully formed long multiplications, in pixel space. This time, there is no teacher drawing the numbers in order or pointing out which parts to pay attention to. This is a much harder problem, and I suspect one that may not be solvable by mere gradient descent. (I do plan to try at some point, but there are so many hours in a day…)

In both cases, we are relying on supervised learning, but in the first case, the information given to the network is almost a full trace of the program. Learning an algorithm from outputs is hard, learning it from traces is easier. This is also what happens when the network becomes privy to the instructor’s gaze; the gaze correlates with some latent states in the algorithm and thus make it easier for the network to learn to produce such states.

As a thought experiment, consider a recurrent neural network attempting to learn the game of Go by learning to predict the moves of an expert player. If all this armchair machine learning is correct, such an algorithm would learn to make better predictions faster if it were given access to the expert’s player gaze. The gaze correlates with the internal algorithm happening inside the expert’s head. Instead of merely telling the learner which piece it should move, we’re providing it with some insight into how to reach the right conclusion.

In fact, consider how a master might teach a human student. What would he do? He would point at the board, and say things like “See this piece? It’s important because of that other piece here”. He would provide a dump of his internal trace. What is the accessory most typically associated with a teacher? A pointer.

In the limit, the learner would observe the state of every neuron of the teacher over time. The learning task would then boil down to modelling the behavior of neurons in general, a much easier task.

It would not necessarily be practical to do so, and it would require considerable computing power, but as a pure learning problem, it is conceptually much easier than learning the entire Go playing algorithm by merely observing its inputs and outputs.

A testable prediction of all this armchair machine learning is that intelligence is taught. This means that for connectionist AI to go beyond animal intelligence and learn abstract, general, reasoning skills, it will need to learn an algorithm. I suspect that such an algorithm is not accessible by gradient descent but was discovered by evolution and is being transmitted through language, which is is a great way to explicitly expose internal mental states and their progression.
