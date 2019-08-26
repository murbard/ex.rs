---
layout: post
title: "Potential design for a simple, and evolvable, on-chain treasury."
author: arthurb
date: 2019-06-13 00:00:00 +0000
categories: tezos
image: assets/images/treasury_delphi.jpg
---

I have been compiling some thought from various community members regarding the design of an on-chain treasury for Tezos chains, and what that might look like. Of course, there is already a way of funding various causes through protocol upgrade proposals that slightly inflate their supply. However, this mechanism may not be appropriate for all the type of endeavor that Tezos communities might want to fund. For instance, they may be interested in funding computer science research, open source software development, or even participate in charitable causes such as medical research. Non-profit entities, such as the Tezos Foundation, the Tezos Common Foundation, Tezos Japan, Tezos Southeast Asia, Tezos Korea (and I probably forget some) already provide funding to such efforts, but it is nonetheless interesting to contemplate how this could be done on a Tezos chain.

![**Treasury of Athens at Delphi**, source [Wikipedia](https://en.wikipedia.org/wiki/Treasury#/media/File:Treasury_of_Athens_at_Delphi.jpg), photo by Sam Korn.](/assets/images/treasury_delphi.jpg)***Treasury of Athens at Delphi**, source [Wikipedia](https://en.wikipedia.org/wiki/Treasury#/media/File:Treasury_of_Athens_at_Delphi.jpg), photo by Sam Korn.*

The protocol upgrade mechanism is so generic that it could, in principle, be used for this purpose. However, for security reasons, this process is deliberately slow and requires the participation of a very large fraction of the network. The ideal mechanism would have the following properties

1. A minimum viable product is easy to implement and deploy

1. The mechanism can be upgraded easily

I’m presenting here a design that is simple to understand and to implement and is fairly generic. It’s the result of discussions with several people in the community and I’m sharing it for comments.

First off, all inflation should come from protocol upgrades. This is because a coin’s supply is a sensitive issue, and one would like to keep as many protections as possible around it. Therefore, a decision to fund an onchain treasury has to come from within a protocol upgrade. Second, this on-chain treasury is represented by a special smart-contract, the treasury contract. This smart-contract behaves in many ways like any other smart-contract* except *for the fact that its code may be rewritten by a decision of the bakers.

In every block, bakers would signal their support for a treasury *policy* by including the hash of a piece of Michelson code in their block. That code represents the proposed body of the treasury contract. If over a period of 3 months, more than 80% of the blocks signal their preference for a new treasury policy, then anyone may submit that code to the treasury contract, and it will update itself.

What might that piece of code look like? A MVP would be for bakers to signal support for a 3 out of 5 multisig contract, where the keys are held by reputable parties. An even better MVP, which is not particularly more complex to implement would be a multisig which lets the key holders upgrade the contract directly by treating the contract code as a stored lambda expression. Note that this is *not* the same as the treasury contract upgrade triggered by the bakers that we described previously. This is just a standard Michelson contract rewriting its storage. The ability for bakers to overwrite this contract still remains, as a sanity check.

Progressively, this treasury contract can evolve into a more fully fledged system with many participants and where proposals are made on-chain. This is the approach taken on Ethereum by the Moloch DAO. To fully unlock the potential of these approaches, another protocol upgrade might allow a contract to query the number of rolls of a baker interacting with it as of a previous snapshot. Although very complex systems are possible, including mixing voting, prediction markets, and more, I believe that a multisig MVP tied with an off-chain discussion platform could provide a lot of mileage fairly quickly.

The trickiest part, in terms of implementation in the protocol, is finding out which hash (if any) might be present in 80% of the blocks over the previous months. I’m pretty sure there is an online algorithm that does it in constant space, but I can’t find a reference (it’s not Boyer-Moore, but that’s the spirit).

In conclusion, there is probably a better way to design a treasury, but I think the simplicity of this approach is likely to get the ball rolling the fastest.
