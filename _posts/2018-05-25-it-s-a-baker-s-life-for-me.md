---
layout: post
title: "It’s a baker’s life for me: being a Tezos validator"
author: arthurb
date: 2018-05-25 00:00:00 +0000
categories: tezos baking
image: assets/images/fresh_bread.jpeg
---

## What is baking?
Bitcoin has mining, Tezos has baking. In Bitcoin, miners compete to publish blocks containing a proof-of-work stamp by repeatedly hashing block headers. In Tezos, block creation is done by bakers. Rather than deriving the right to create a block by finding the solution to a proof-of-work problem, bakers obtain that right when a Tezos token (or rather a roll, see below) they own (or that is delegated to them) is randomly selected to create a block. Since not everyone holding tokens is interested in being a baker, tokens can be “delegated” to another party. The delegate does not own or control the tokens in any way. In particular, it cannot spend them. However, if and when one of these tokens is randomly selected to bake a block, that right will belong to the delegate.

A baker becomes aware of its right to bake blocks a few weeks in advance. When it does so, it is expected to create a safety deposit that will be held for a few weeks. This safety deposit is referred to in the white paper and in various places as a “bond”. In a few other proof-of-stake systems, this deposit is a single static set amount staked by a delegate. In contrast, in Tezos, the deposit dynamically changes depending on the number of blocks a delegate is set to create.

This safety deposit ensures the honesty of the baker during the period around the block creation. If the baker cheats and attempts to propagate blocks on different branches, it is punished by losing its deposit. If a baker creates blocks on a losing branch, it merely forfeits its reward. A baker needs to explicitly try to create blocks on two different branches to be punished. If the baker successfully creates and propagates a valid block, it is rewarded with a block reward as well as some fees paid for by the transactions included in the block.

## What is a roll?
To speed up computations for deciding which delegates are selected to bake, the Tezos ledger keeps track of groups of tokens called “rolls”. Rolls are aggregated at the delegate level, which means a baker’s baking power is proportional to the amount of tokens delegated to them, rounded down to the nearest roll. Rolls are set at 10,000 ꜩ.

## What is endorsing?
Endorsing is a lot like baking, but instead of creating an entire block, bakers are sometimes called upon to endorse a block, which means to witness that they saw a block and checked that it was valid. Each block has many randomly selected endorsers. Endorsers also put up safety deposits and receive rewards.

All in all, every baker also participates in endorsing and vice versa. Baking and endorsing are part of the same activity and sometimes, “baking” can be used as a shorthand to describe both.

## Why bake/endorse?

Bakers/endorsers earn rewards for doing their job in the form of new tokens emitted by the protocol. Those rewards are calibrated so that the number of Tezos tokens grows at roughly 5.5% per year and so that the safety deposits required for baking represent about 8.25% of the tokens delegated to a baker.

In practice: suppose 100% of tokens are baking or delegated to active bakers, and suppose a baker has received delegation for 1,000,000 ꜩ. They would need to hold about around 80,250 ꜩ, at all times, in safety deposits. They would receive about 55,000 ꜩ per year in block and endorsement rewards, not to mention transaction fees.

However, if only 40% of the tokens are actively creating blocks, this baker would create 2.5x more blocks. Thus, they would be needing around 200,625 ꜩ in deposits but would be receiving around 137,500 ꜩ.

These are rough back of the envelope calculations which do not take into account many factors like the cost of operating servers, the risk of not baking properly due to bugs or network disruptions, the possibility of baking blocks with lower priorities if some bakers are only partially active, the potential cost of attracting delegators, and other yet unforeseen costs and risks. Bakers should rely entirely on their own analysis of the Tezos protocol and network to undertake this activity and not on this high-level blog post.

Stake delegation isn’t just about baking though! Voting in the Tezos network is also done by delegates, so it is their responsibility to analyze proposals (or weigh several opinions) in order to select them. That said, going forward, these two rights could be unbundled.

## What does it take to bake?

### Bandwidth, CPU, and memory

Bakers have a responsibility to validate the chain and create blocks. They should have a good, stable Internet connection and high-availability servers. The emphasis upon launch is on correctness, not throughput, therefore, the requirements should be rather light at the onset. Given the experience gleamed in the test-network, at least 8 gigabytes of RAM would be advisable.

### Security

Bakers have a responsibility, and a pecuniary incentive, to secure their operations, which should be done in several ways, including:

1. DDOS resistance: malicious parties might try to DDOS a baker at the time when they are trying to bake a block. A smart baker covertly listens to the network, and inject their blocks from a variety of IP addresses that may be hard to attribute.

1. Key protection: a malicious actor stealing a baker’s private key could not only get control of all the baker’s security deposit but also harm the network if they preferred wrecking havoc to earning tokens. A smart Baker will use a hardware security module, SGX enclave or hardware wallets to protect their keys for baking. There is already some support for doing this using the Ledger Nano S device.

1. Preventing and mitigating intrusion: the Tezos node, the Tezos baker, and the Tezos endorser should probably run on a machine isolated from the network by at least another well secured machine which is patched regularly. Ideally, there would be no remote shell access to the machine running the baker.

### Know-how

When the Tezos network launches, some kinks will likely remain in the baker code. Many developers are working hard to make sure there are as few as possible, but some of them will undoubtedly reveal themselves only at scale. Typically, the baker might get stuck in a bad state and stop doing its job.

A good baker needs to understand the baking process inside out (or, for larger operations, have one person on staff who does) and be ready to get their hands dirty by examining the process and troubleshooting it.

A good baker also needs to be in touch with developers to make sure they are ready to apply security patches quickly if the needs arise. In the early days of the network, baking will not be a passive activity but will require active involvement in Tezos and a good working knowledge of its underpinnings.

Bakers should experiment with the zeronet and engage with the rest of the community on the [matrix_channel](http://riot.im/app/#/room/#tezos:matrix.org).

### Tokens

A baker needs to own some tokens of their own in order to serve as delegates. On average they need to own about 8.25% of the number of tokens delegated to them at any time, divided by the fraction of the network that is actively baking or delegating to an active baker. If they don’t, they will be given the opportunity to create blocks but will not be able to place enough bonds to do so.

Since the right to bake blocks is known a few weeks in advance, it’s possible for a baker to try and buy (or borrow) tokens in anticipation of the required deposit. This possibility is key for bakers with a small amount of stake who only bake a block once in a blue moon and for whom the deposit may represent a lot more than 8.25% of their delegated stake.

To help bakers get started, the protocol initially ramps up deposits linearly from 0% to 8.25% over a period of six months. This ramp would give enough time to popular bakers to acquire the additional tokens they might need in order to bake all the blocks that they are allowed to bake. Delegates should monitor their treasury and be ready to react if they suddenly experience a surge in popularity.

For more information on baking and Tezos in general, check out the resources below:

* An [explainer video](https://www.youtube.com/watch?v=andKrImAfVE&feature=youtu.be) on baking and the economics of it by Tezzigator

* A list of known delegation services on [Tezos.rocks](https://tezos.rocks).

* For technical discussions about Tezos, ask about the technical channel on the [matrix chat](https://riot.im/app/#/room/#tezos:matrix.org).

* [GitLab](https://gitlab.com/tezos/tezos)

* [Developer Resources](http://doc.tzalpha.net/index.html#)

* [White Paper](https://tezos.com/static/papers/white_paper.pdf)

* [Position Paper](https://tezos.com/static/papers/position_paper.pdf)
