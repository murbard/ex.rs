---
layout: post
title: "A few directions to improve Tezos"
author: arthurb
date: 2018-10-22 00:00:00 +0000
categories: tezos
image: assets/images/yay_baking.jpeg
---

In which I outline, for different categories, what seems to me as reasonable, immediate, directions for improving Tezos. The emphasis is on relatively low hanging fruits, nothing earth shattering, just pragmatic, steady, incremental improvements.

Tooling, higher-level languages, second layer scaling solutions, and other elements living outside of the protocol are undoubtedly important, but out of scope for the sake of this post.

## Node

Users running nodes improve the security of the network by making it harder for a malicious coalition to create social consensus around a departure from the protocol. Thus, it’s important that the requirement for running a node remain reasonable. At a minimum, they should be bounded, which means nodes should implement garbage collection for past states and past blocks. Keeping a full history of blocks is laudable, some people should do it, some people will doubtlessly do it, but it doesn’t have to be done by everyone. It certainly shouldn’t be held up as a sine qua non that would deter people from running nodes.

![Another kind of node](/assets/images/other_node.jpeg)*Another kind of node*

The node should also be able to synchronize directly by downloading the context from a known point. Implementing a light client that relies on a known context hash but requires Merkle proofs when it queries parts of the context would also be of interest, but I’m departing from protocol related matters here.

## Consensus algorithm

The “liquid” delegation model with bonding seems like a successful recipe so far, I think it’s important to preserve it. Most of the proof-of-stake machinery in Tezos is roll-tracking and randomness generation. This ends up producing periodic random seeds and a set of 80,000 or so rolls. The combination of the two is a blank canvas for consensus.

![Yay, baking!](/assets/images/yay_baking.jpeg)*Yay, baking!*

### Better randomness

As I’ve suggested before the entropy generation should be hardened. There’s an implementation of PVSS in Tezos which can do that, by requiring a large coalition of malicious users to affect randomness. It’s here, I think it’s worth including. Another technique, which is both more powerful and lighter weight is to use a VDF to sufficiently “delay” the generation of the random seed as to make manipulation impossible. Chia network recently championed two elegant (and surprisingly elementary) techniques for instantiating VDF. The downside is that these require computation in a group of unknown order.

Chia proposes two ways of building such groups: using the residuals modulo a product of two unknown primes or using a more exotic group of binary quadratic form. The former is a well-studied problem but requires a trusted setup, the latter doesn’t require a trusted setup but hasn’t been studied as thoroughly.

I’d for one recommend using both and hashing the concatenated outputs to form a seed. For the RSA group, using the existing 2048 bits RSA Challenge number saves one the trouble of running an MPC ceremony. If this VDF is implemented in Tezos, the use of PVSS or even a commit-reveal scheme becomes technically unnecessary but, in the tradition of belts and suspenders, I would say it can’t hurt.

### Better consensus

Synchronicity assumptions are icky (unless the times are on the order of hours) and using a set number of validators doesn’t work well with the liquid POS model. This leaves essentially one option, which many seem to be converging upon: sortition and BFT agreement (e.g. DFinity, Algorand though both intend to do cleverer stuff in addition). Note that BFT agreement is a criterion *for accepting blocks, *namely that 2/3+ of the endorsers endorse a block. For endorsers to succeed in doing so, they need to reach Byzantine agreement. The *strategy *they use can vary!

Tendermint’s a fine algorithm I think it’s likely the best starting point. Note however that Avalanche — or any other agreement BFT agreement strategy — could be used instead, without changing the Tezos protocol itself.

From the outside, the protocol would look very similar to the current one: block slots with priority lists, endorsers (a few hundred endorsers per block instead of 32). The difference is that the protocol would require the bakers to (probabilistically) achieve Byzantine fault tolerance on each block before proceeding. This seems like a Good Thing and preferable to over-thinking a chain-based approach.

## Privacy

![Sealed enveloppes are better than postcards](/assets/images/sealed_enveloppe.jpeg)*Sealed enveloppes are better than postcards*

Crypto-currencies typically require transactions to be broadcast publicly to the entire world, which is obviously problematic for privacy. No business wants the entirety of its transactions known to its competitors (and of course privacy doesn’t dispense you — or prevent you — from having to open your books should you be required to).

Several solutions have been proposed and implemented, each with different trade-offs in terms of complexity, scalability, privacy, etc. With the new Sapling release, I lean towards Zcash currently offering the best trade-off. The transactions are concise and provide the maximum amount of privacy achievable. The necessity of a trusted setup is a sore point, but it is mitigated by the large number of participants in the ceremony, and the possibility of “turn-stiling” the private notes by preventing more coins from coming out than coins coming in. The easiest path is binding to Sapling’s Rust library.

In Zcash coins are turned into blinded commitments placed in a Merkle tree (which I insist should be called a dazzle). These commitments can be transacted using zero knowledge proofs. While it would be possible, in principle, to have these hidden commitments participate in proof-of-stake, it seems hard to achieve and it’s unlikely to be fruitful as an *immediate* area of focus. Note that, unfortunately, both “turn-stiling” the commitments and not allowing them to participate in the proof-of-stake inflation are detrimental to the size of the anonymity set since they incentivize getting funds out quickly. Also bear in mind that it would be possible to offer this circuit as a primitive in smart-contracts as well, so that other assets may benefit from the same level of privacy.

There are many exciting developments in the space of zero-knowledge proofs (STARKs, Zexe) but this exists today.

## Governance

![Althing in Thingvellir](/assets/images/althing.jpeg)*Althing in Thingvellir*

### Vote

Currently, delegate votes are counted based on their roll ownership at the beginning of the voting period. It makes a lot more sense to base it on their roll ownership at the *end* of the voting period. This means that delegates can set up an alternate address where they vote for the other side of an issue, and let their delegators move their delegation to that address if it is their preference. It puts the vote back in the hands of the token holders, but defaults to the delegator’s choice in case of apathy. That’s a very low hanging fruit.

### Constitutionalism

It’s a good idea to start building up constitutionalism early on. By constitutionalism I mean protocol rules regarding the *content* of protocol upgrades. A refactoring of the code could ensure that every creation and destruction of token has to happen through a single OCaml module. This can be enforced via the type system. Further assume that this module programmatically limits the yearly issuance of tez. It would be straightforward to hold protocol amendments modifying this module to a higher standard than protocol amendments that don’t. For instance, modifying this module might require a stronger majority, or several confirmation votes over a longer period.

The Tezos position paper mentions doing this by embedding a proof-checker in the protocol, this is the most general approach but it requires some heavy machinery. A lot of low hanging fruits can be plucked by merely expressing rules about which files can or cannot be modified. Here’s another example of what can be achieved through this technique: a module could consist of a series of filters that blocks have to go through before being considered by the protocol. An amendment which only *adds* filter would be the rough equivalent of a Bitcoin “*soft-fork*”. Differentiating between soft-“forks” and hard-“forks” can be done relatively easily from within the protocol.

## Mempool

*(no I’m not putting a stock picture of a pool there, enough already)*

An easy 2x gain in throughput would come from reworking the mempool. Specifically, the possibility of including transactions in a block should be fully determined without computing their effects. This is achieved if the mempool can cheaply ensure that the fee can be paid (invalid transactions are then included and treated as nops, as is already the case).

One way to do this is to make it so that all fees are paid by the manager’s implicit account, even if the source is some other account. The mempool can then ensure that the managers have a sufficient balance to pay the fees they promise to pay, *as of the previous block*.

This is a very minor restriction in terms of the blocks that can be created, but it has a large implication for the baker’s strategy. A baker can just grab the highest fee operations, shove them in a block, publish it, and only then take the time to analyze the effects of the operation. As part of this change, it would make sense to also merge “accounts” and “implicit accounts” which have no strong reason for remaining separate.

## Smart-contracts

The gas costs in Michelson need tuning in order to raise the gas limit in each block. Beyond that, and from a language perspective, it would be reasonable to add closures to the lambdas (with type annotations) and offer more polymorphism where possible. I don’t think I have a whole lot to contribute here given how strongly most Tezos developers gravitate towards expertise in programming language theory.

Multiple entry points in contracts would be a nice standard to add. Offering cryptographic primitives in Michelson like elliptic curve point manipulation could yield some interesting use cases. Most of the work here is in the tooling layer which is explicitly out of scope for this post.

## Conclusion

There is no pretense of originality in the above (though I think the governance parts are novel). I believe Tezos’ strength will come from its ability to make steady, incremental, improvements based no only on its own community of developers and researcher, but also by tapping* with discernment* across the entire industry.
