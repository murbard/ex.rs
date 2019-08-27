---
layout: post
title: "Smart contracts: Turing completeness & reality"
author: arthurb
date: 2016-10-03 00:00:00 +0000
categories: tezos
image: assets/images/alan_turing.png
---

The meltdown of “The DAO” and, more recently DDOS attacks on Ethereum have spurred a debate on the wisdom of “Turing completeness” in smart contract languages. Tezos, a cryptographic ledger and smart contract platform offers Turing complete smart contracts in a formally specified language (with strong typing and static type checking). We seek to clarify some misconceptions about the relevance of “Turing completeness” in the context of smart contracts.

## The theory

![Alan Turing](/assets/images/alan_turing.png)*Alan Turing*

Turing-completeness is a property indicating that certain (typically, most) programming languages are fully general. According to the [Church-Turing thesis](https://en.wikipedia.org/wiki/Church%E2%80%93Turing_thesis) (a physics principle with substantial empirical evidence), every computation which can be physically carried out, even in principle, can be expressed and implemented in a Turing-complete language.

It does not matter which language. Any Turing-complete language is general enough to emulate all the others (albeit with a potential slowdown), making them equivalent in terms of the computations they can carry out. Turing-complete languages are so pervasive that they even appear in deceptively simple looking systems such as [cellular automata](https://en.wikipedia.org/wiki/Cellular_automaton) or [tiling systems](https://en.wikipedia.org/wiki/Wang_tile).

![Alonzo Church](/assets/images/alonzo_church.png)*Alonzo Church*

Unfortunately, with power comes complexity. [Rice’s theorem](https://en.wikipedia.org/wiki/Rice%27s_theorem) tells us that, in principle, no *general* procedure (human or mechanical) can decide *for all* programs whether they will terminate. In fact, it may not even predict any nontrivial property about their behavior.

One might be tempted to conclude that since we cannot predict what these programs will do, we should stay clear of such languages. In fact, isn’t this clearly what happened with The DAO? The contract participants weren’t able to predict the behavior of the contract, and catastrophe ensued.

This misconception is surprisingly common . Clearly, at least one person, the attacker, was able to predict and understand the contract — better so than its creators in fact.

Rice’s theorem asserts that there is no *general* procedure for deciding if the properties about the behavior of a program hold true. These might be properties such as: “does this program halt”, “does it reach an error condition”, “can a certain output ever be produced”.

The theorem implies the existence of programs which *do* terminate, and *do* behave as expected, but not provably so. We cannot really know what they are, but we know they exist through clever mathematical arguments (courtesy of Turing, Church, and Gödel). Importantly, there may be general procedures which can compute the behavior of *most* programs, just not *all*.

![Kurt Gödel](/assets/images/kurt_godel.jpg)*Kurt Gödel*

Some languages forego Turing completeness and their programs *can* be shown always to halt. Such is the case for the Bitcoin script for instance. A folk belief seems to be that the converse of Rice’s theorem is true. Namely, the misconception goes:

“Since the halting problem in Turing complete languages implies that we cannot predict the behavior of programs in general, then, as long as we write our programs in a language that guarantees termination, we’ll be able to predict all the interesting properties we’d like.”

This is not true, not in theory, not in practice. There are languages for which programs are guaranteed to halt for all input, but where you may not decide if any given input will cause the program to reach an error condition. *(For the theoretically inclined: take for instance the language consisting of a single program f which checks if its input x is a proof of the inconsistency of ZFC. f always halt, but whether or not it ever returns “true” is undecidable in ZFC)*

## The practice

In practice, Turing completeness is an idealization. Concretely, computers have a finite amount of memory and will only run for a finite amount of time before being turned off.

Tezos (or Ethereum) smart contracts are bounded in time by a limit set per block and per transaction. In some trivial way, they are not quite Turing-complete. Better, since the number of steps in the execution of a contract is bounded, it is theoretically possible to predict the behavior of a given program for any possible input.

Since parsing an input takes an amount of time at least commensurate with the length of the input, the cap on the number of steps in the execution bounds the maximal size of any input that the contract may read. The following algorithm can thus decide the behavior of any contract:

    for(all inputs < max size) {
       run program for max time;
       check property;
    }

Of course, this algorithm is astronomically unpractical. If the input size were restricted to a measly kilobyte, there may still not be enough free energy in the universe to carry out the computation.

My point here isn’t to show that we can always prove all the true properties of time bounded smart contracts. Quite the opposite, my point is that the existence of a general algorithm and the existence of a *practical* algorithm are two very different things.

Does such an efficient algorithm exist for Bitcoin script? The answer is no as well!

Consider the following Bitcoin script

    scriptPubKey: OP_HASH256 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef OP_EQUAL

This is a hash puzzle, as described on the [bitcoin wiki](https://en.bitcoin.it/wiki/Script#Transaction_puzzle). Notice that the hash I picked is clearly made up. The script clearly halts for all input, but we might be interested to know if this output is spendable. It all depends if there exists a preimage to the hash I made up, and that is absolutely unclear. We would have to brute force the hash function for billions of years to find out.

Bitcoin’s script lack of Turing completeness is not a magic bullet. It is of no help in deciding a fundamental property of this very simple script.

## What really matters

As we’ve said before, Rice’s theorem essentially tells us that there exist programs which *do* terminate, and *do* behave as expected, but not provably so.

However, almost all *correct* programs of interest are going to be essentially *provably correct*. Moreover, the proof of their correctness will be relatively straightforward. This is especially true in the context of smart contract on cryptographic ledgers which generally implement simple and straightforward business logic (it is of course less true if you’re dealing with, say, computational algebra).

A correctness proof might rely on admitting certain hard to prove conjectures. For instance, we may accept from heuristic arguments that hash functions are indeed hard to invert, and use it as a lemma in our proof, as is generally done in cryptography papers.

But overall, whenever a programmer writes a program, they generally keep a vague outline of a proof of correctness in their head. Sometimes that proof is incomplete or wrong — humans are fallible — but it always guides the writing. The programmer constantly reasons about he behavior of their program, to make sure it does what is intended.

It just isn’t the case that a programmer sits as their desk and writes, in good faith, a swap contract that destroys all of its pledged collateral if and only if the [Collatz conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture) is true, or [ZFC](https://en.wikipedia.org/wiki/Zermelo%E2%80%93Fraenkel_set_theory) is inconsistent.

Whether there exists a fully general procedure for deciding if *arbitrary *programs written in a given language are correct doesn’t really matter. Rather, what matters is how *straightforward* it is to construct programs which are *easy* to reason about.

In that respect, it *is* generally straightforward to reason about the behavior of Bitcoin scripts. They do not carry much surprise. Is there a preimage t0 0x123… ? Perhaps, perhaps not, but it’s astronomically unlikely that anyone can find out, so for all intent and purposes, the transaction is not spendable.

## The tradeoff

![The efficient frontier, and the new frontier created by automated theorem proving and formal verification](/assets/images/efficient_frontier.png)*The efficient frontier, and the new frontier created by automated theorem proving and formal verification*

Programming languages exist below an expressiveness / ease to reason about frontier.

*Expressive* languages make it easy to describe certain desired behavior. While, in principle, all Turing languages can express the same type of computations, in practice, it can be very hard to express certain computations if the language is inadequate. A humorous example is the [Malbolge](https://en.wikipedia.org/wiki/Malbolge) language. While Malbolge is Turing complete, it is extremely hard to write programs in this language that do what they are intended to do.

Like expressiveness, *ease to reason about* is not a binary variable. Turing completeness has important theoretical and practical implications for the general possibility of reasoning about a language, but as we’ve seen earlier, it does not tell the whole story.

Very expressive languages tend to be harder to reason about, while some restricted languages (such as finite state machines) can be very easy to reason about at the cost of expressiveness. There is fortunately a way to push that frontier: formal verification.

Formal verification is a rising computer science discipline which produces provably correct programs. You might think of it as creating a machine checkable proof that a program is correct, though, oftentimes, the proof itself *is* a program. Smart contracts have few lines of code, but a lot at stake, making them a perfect candidate for this technique.

In particular, assisted, or even automated, theorem proving lets the programmer specify the properties they want to prove and lets the computer do most of the tedious work of coming up with a formal proof of correctness. This does not affect all languages equally.

While it is true that a provably correct program is still provably correct when compiled down to some assembly, it is much harder to derive proofs from the compiled version. This is true for humans, this is true for automated theorem provers. The compilation step throws away vital information about the structure of the program.

## What we’re doing about it

Since Tezos started in the summer of 2014, a core tenet of the project has been that smart-contract languages are a perfect target for formal verification. We wanted a language that would play nicely with theorem provers, but also that could also be easily reasoned about on its own. This led to the following choices:

1. We started with a [formal specification](https://tezos.com/tech.html) of the language, to determine precisely and exactly what the program *meant*.

1. We made the language strongly typed, with algebraic datatypes and static type checking. Types are a powerful way to reason about the behavior of a program. They can ensure for instance that we do not mistakenly add apples and oranges, but they can be pushed further to enforce certain properties of the contract.

1. We made the language high-level. The contract that participants enter into isn’t some obfuscated assembly, but the language in which the contracts are written. This allows us to introduce useful high level primitives, such as functional maps, and functional sets. Our goal isn’t to build some arbitrary code execution engine, but an engine tailored to the needs of smart-contract writers.

All these properties facilitate proving facts about the contracts using automated theorem provers such as the [Coq proof assistant](https://coq.inria.fr/), but they also make it easier to manually check the correctness of the programs.

## Conclusion

Turing completeness was framed! There are programs for which properties are easy to understand and prove, and some for which they are difficult. There are programming languages which facilitate such proofs, and some which obscure them. It’s about trade-offs, not about some bright line around the halting problem.
