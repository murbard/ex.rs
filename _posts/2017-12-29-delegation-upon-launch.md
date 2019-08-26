---
layout: post
title: "Delegation upon launch"
author: arthurb
date: 2017-12-29 00:00:00 +0000
categories: tezos baking
image: assets/images/althing.jpeg
---

Tezos is a proof-of-stake blockchain with a delegation mechanism. How delegation is handled at the launch of the network could have enduring consequences for the network.

In Tezos’ proof-of-stake mechanism, the keys granting the right to sign blocks (“bake”) are different from the keys granting the right to move tokens. This is a security measure which lets users keep keys in cold storage while participating in the proof-of-stake algorithm. It also enables delegation, whereby the right to create blocks (“bake”) is transferred to a third party, the holder of the delegation key.

How should delegate keys be handled at the launch of the network? Different strategies could be adopted, each of which comes with benefits and drawbacks. Of course, users can change their delegates later on, but a certain stickiness is to be expected.

![The Icelandic Alþingi](/assets/images/althing.jpeg)*The Icelandic Alþingi*

## First off…

Before diving into ways in which delegation can be initially set up, I’d like to remark that people consistently reach out expressing their interest and intent to bake themselves. This is important to maintain the decentralized nature of the ledger and I love being part of a community with this spirit.

To that end, when a user with a balance in the genesis block first joins the network, being their own baker should be the preferred option. That said, a few checks could be useful to ensure that the user understands the responsibility that baking entails. For instance, they could be asked to first run a baker on their computer.

The process should be made as easy as possible, and I’ve heard of two community driven projects to ease “home baking”. Ultimately, what I’d love to see, in the medium term, is a small appliance that requires no maintenance and can be plugged and forgotten (without the electric bill of an ASIC).

## À la carte

In this model a user who does not want to bake for themselves would be offered to select a baker from a list (presented in randomized order) upon joining the network.

The list would be compiled from entities across the world having been audited and graded on their security practices. Think of these as the rough equivalent of Bitcoin mining pools. A few people have already started setting up these pools.

Benefits: the randomization would provide a good amount of decentralization from the get go, probably greater than the typical distribution of hashing power in proof-of-work cryptocurrencies. It would also ensure a high-level of participation in the proof-of-stake algorithm, increasing the security of the network.

Drawbacks: though there are a few people working on pools, such a list should have dozens of options with a wide geographic distribution. Outreach would help, as would providing technological help and support.

## Opt-in

This model defaults every account not to participate in block creation, unless they explicitly choose to.

Benefits: simple to set up, strongly motivates pools to step up and convince users to give them their baking rights.

Drawbacks: the fastest movers could get a disproportionately large advantage in the network, reducing the effective decentralization of the network.

## Opt-out

This model defaults every account to being their own baker. If someone does not want to bake, they should proactively set a different delegate key.

Benefits: simple to set up, theoretically provides a lot of decentralization.

Drawbacks: some users may not be ready or willing to run full nodes, participation in the proof-of-stake algorithm would be low. After a while, non participating delegates do not receive baking rights, so the opt-out model ends up looking a lot like the opt-in model.

## Default delegate

This model would default the initial delegation of block creation to a third party such as a foundation. I’ll start with the drawbacks on this one because they are most obvious.

Drawbacks: this would immediately introduce a great deal of centralization. While that centralization would fade over time, as users pick different delegates, it could stick for a while. Moreover, even assuming 100% honest baking, key compromise would be a serious concern.

Benefits: the number one risk the network faces when launching is consensus breakdown. This would likely neuter the risk of consensus breakdown for a long time, allowing the network to be released earlier and mature progressively by going through deeper audits as delegation progressively disperses.

## Conclusion

There are several ways to set up baking rights in the genesis blocks. Each of which comes with benefits and drawbacks. The “À la carte” option was the plan being pursued in late August, and it’s the one I personally like best.
