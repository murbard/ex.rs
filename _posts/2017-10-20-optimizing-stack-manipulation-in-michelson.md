---
layout: post
title: "Optimizing Stack manipulation in Michelson"
author: arthurb
date: 2017-10-20 00:00:00 +0000
categories: tezos michelson
image: assets/images/rubiks_cube.jpeg
---

This post describes a light compilation step to help stack manipulation in Michelson. It’s not critical for the launch, so we aren’t focusing on it in the near term, but I am sure this problem will nerd-snipe a few people, and I’d love to see someone pick it up.

![Rubik’s cube — photo by keqs](/assets/images/rubiks_cube.jpeg)*Rubik’s cube — photo by keqs*

When writing contracts in Michelson, Tezos’ smart contract language, stack manipulation can often get in the way. To get a sense, check out Milo’s [contract examples](https://www.michelson-lang.com/). Splitting pairs, reforming them, re-ordering elements of the stack is time consuming for the programmer, and reduces code readability.

There are, fortunately, three workarounds:

1. Use a Michelson IDE, a prototype of which can be seen [michelson-mode](https://github.com/tezos/tezos/blob/master/emacs/michelson-mode.el), for Emacs. This mode gives you a symbolic representation of the state of your stack at the cursor. This will be improved with the introduction of annotations, letting you “tag” values on the stack with an identifier.

1. Program in a higher-level language that compiles down to Michelson, such as OCamlPro’s [liquidity](https://github.com/OCamlPro/liquidity).

1. Use a light compilation step, only for stack manipulation

## The JUMBLE macro

I have been thinking about 3 for a while. I call this the “JUMBLE” instruction, and it would work like this. Suppose you have a stack with the following form:

    (a,b):c:(d,(e,f))

and would like a stack of the form

    (a,f):(c,d)

You could write

    CAR; DIP { SWAP; DUP; CAR; SWAP; CDR; CDR;}; PAIR; DIP {SWAP; PAIR; PUSH;}

Which isn’t very explict… Or you could write

    JUMBLE (a,b):c:(d,(e,f)) => (a,f):(c,d)

Which is a lot clearer! Of course, the JUMBLE operation needs to be compiled — we will get to that in a second — but note that it can also be decompiled easily.

Any code sequence of the form (DUP|DROP|SWAP|CDR|CAR|DIP|PAIR)* can be compressed into a JUMBLE instruction.

## Compiling JUMBLE

Writing down a correct implementation of JUMBLE isn’t trivial, but it’s still relatively straightforward. A simple way to do this is to start by flattening all the pairs in both the initial stack and the target stack. Flattening the pairs of the initial stack is done by repeatedly applying an UNPAIR macro of the form DUP; CDR; SWAP; CAR. The flattened initial stack is then sorted and variables are duplicated to match the flattened target stack. The pairs are then reformed to match the target stack.

This isn’t a very efficient way of doing things. Some pairs may be present in both the initial and target stack, and it may be inefficient to break them and reform them. In addition, sorting elements with only the ability to SWAP nearby entries is necessarily quadratic in the worst case.

It’s possible to start from this correct procedure, and tack on a series of optimization, by finding common patterns and avoiding unnecessary stack manipulation.

Another approach is to directly search for an optimal solution. I have implemented a prototype of that compilation, and recently pushed it on Github: [https://github.com/tezos/optimal_stack](https://github.com/tezos/optimal_stack)

It’s an implementation of [A*](https://en.wikipedia.org/wiki/A*_search_algorithm) with a few heuristics to cut down on the search space. It hasn’t been tuned for performance by any means, so it won’t be helpful for large stack manipulations, but it will find non-trivial patterns.

I recommend trying out in [utop](https://opam.ocaml.org/blog/about-utop/), which can be done like this

    $ ocamlc -c *.ml *.mli
    utop
    #require "heap";;
    #require "optimal_stack";;
    open Optimal_stack;;
    example ()

Example optimizes the following problem

    JUMBLE (a, b):c => (c,a):b

which can be programmed like this

    **let** a **=** **Var** **"a"** **and** b **=** **Var** **"b"** **and** c **=** **Var** **"c"** **in** 
    **let** start  **=** **Stack** **([Pair** **(**a, b**);** c**]**, **[])** **and
        **target **=** **Stack** **([Pair** **(**c,a**);** b**]**, **[])**
     **in
    **optimize start target

The given solution is

    DUP; POP; CAR; SWAP; PAIR; PUSH; CDR; SWAP

You will notice that the stack type is a pair of lists. This lets us represent the DIP operation as PUSH and POP operations onto an auxiliary stack. Unless you’re inside of a DIP, the auxiliary stack should be empty.

## Next steps

My focus is on launching the network, and we don’t need JUMBLE for that. However, I think it’s cool and I’d love to see someone pick up the preliminary work to make it faster and able to tackle larger optimization problems.

My suggestions on how to take it further would be:

1. Find more “theorems” about the cost of moving from one stack to another which can be used in the A* heuristic.

1. Use sub-optimal search. If you can guarantee bounds on the error of your theorems, you can guarantee bounds on how close to optimal the solution is.

1. Plug the problem into traditional constrained optimization solver.

1. Instead of blindly looking for a solution, start with a known solution (following the approach outlined earlier) and search for provably correct improvements to that solution.

1. Mathematically formalize the problem, and derive an efficient algorithm for computing the optimal solution by reasoning algebraically about it, not unlike the mathematization of the Rubik’s cube through group theory. I give this a 10% chance of panning out, but it’s high reward if it does.
