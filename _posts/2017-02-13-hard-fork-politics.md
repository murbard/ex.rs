---
layout: post
title: "Hard-Fork Politics"
author: arthurb
date: 2017-02-13 00:00:00 +0000
categories: tezos
image: assets/images/mammoth_mud.jpeg
---

Like an idol with feet of clay, the oft-touted mathematical immutability of blockchains stands on a weak foundation so long as hard forks remain a possibility. Indeed, the rules of these protocols only bind those who choose to follow them.

This simple observation is what inspired the development of [Tezos](https://tezos.com), a third-generation decentralized ledger and smart-contract platform designed to solve the governance issues that plague first and second-generation blockchains. In August 2014, the Tezos position paper argued the de facto primacy of social consensus in cryptocurrencies, a controversial position at the time. It also identified the potential risk of core development teams exercising a centralized influence on protocols.

While hard fork politics were almost non existent at the time, they have been thrust into the limelight by the block size debate and the Ethereum fork. The belief that the operating rules of these ledgers, including their monetary policy, rested on ironclad mathematical guarantees has become untenable.

A recent[ blog post](http://unenumerated.blogspot.com/2017/02/money-blockchains-and-social-scalability.html) by Nick Szabo presents similar concerns:
> Because of this fraction, and because of the (hopefully very rare) need to update software in a manner that renders prior blocks invalid — an even riskier situation called a hard fork — blockchains also need a human governance layer that is vulnerable to fork politics.

For Szabo, Bitcoin’s governance derives its strength from strong cultural norms:
> The most successful blockchain, Bitcoin, has maintained its immutable integrity via decentralized decision-making among experts in the technology combined with a strong dogma of immutability, under which only the most important and rare bug fixes and design improvements, that cannot be made any other way, justify a hard fork.

Szabo is correct to point out that fork politics represent a vulnerability — indeed, this is one of our core beliefs — however we find his conclusions about Bitcoin prematurely optimistic.

First of all, cultural norms shift over time. His post’s title alludes to “social scalability”, but cultural norms scale very poorly as communities grow. For Bitcoin to succeed as a widely adopted technology, it will need to reach a much larger and far more diverse set of people, people who may have different views about immutability, or more likely will simply not care. Part of the success of the United States as a society comes from *explicitly* embedding these norms into a written constitution and creating the institutional framework to enforce it, and not merely relying on perpetuating mores. Cultural norms do play an important part, but as a society grows larger, so does the importance of institutions and codification. This does not preclude decentralization, but it does preclude structurelessness.

Second, he assumes away the importance of software updates describing them as “hopefully very rare” and seeks to limit design improvements to “only the most important” ones. Unfortunately, this sounds like wishful thinking at best and sour grapes at worst when considering the stagnating state of Bitcoin. This is no fault of the talented developers and scholars working to innovate and improve the protocol; it is an institutional failure. Simple upgrades which would clearly improve the state of Bitcoin (such as Schnorr signatures, confidential transactions, segregated witness, and side chains) fail to make it, prisoners of a glacial political process.

![Dramatization](/assets/images/mammoth_mud.jpeg)*Dramatization*

The Scylla to this Charybdis is the temptation to resort to political centralization of core development for the sake of agility and expediency. Centralization issues are brushed aside by suggesting that users are still free to accept forks, or reject them by continuing to use a previous version. Unfortunately, this choice is illusory. For users or miners, hard forks are [Keynesian beauty contests](https://en.wikipedia.org/wiki/Keynesian_beauty_contest). It doesn’t matter which branch they prefer individually, but which branch they expect will come to be perceived as legitimate and carry with it the value of its token. On this side of the spectrum lies Ethereum, an exciting project which nonetheless takes, in my view, too cavalier an attitude towards political centralization.

Tezos attempts to strike a balance, allowing it to retain true decentralized decision making while avoiding political deadlock by folding the governance algorithm into the blockchain itself. It also differs substantially from some governance efforts like Dash that only address a narrow aspect of governance such as funding. Tezos doesn’t have a “governance model” — rather, it is a technology that incorporates and enforces governance rules on a ledger. The original [position paper](https://tezos.com/pdf/position_paper.pdf) describes this concept in detail.

Tezos’ strategy to prevent hard forks isn’t to shackle itself in a political deadlock. Stasis might make sense at some point in the future when the safety it provides begins to outweigh the benefits of innovation, but we are still far from such a state, if we ever reach it at all. Instead, our approach is to provide an explicit, sound, and efficient process for upgrades and evolution. Not only does it provide a mechanism for new ideas to flourish, as we’ll see we also contend it serves to prevent hard forks.

In his “[Discourse on voluntary servitude](http://www.constitution.org/la_boetie/serv_vol.htm)”, Etienne de la Boétie wondered how sovereigns could enforce tyrannical rules on a population with numerical superiority. Those who wield political power almost never do so from their own, direct, physical strength, but by controlling game-theoretical equilibria. Etienne de la Boétie’s paradox is solved by considering that obedience to the tyrant is a [Schelling point](https://en.wikipedia.org/wiki/Focal_point_(game_theory)). Obedience is the default, and unilateral departure from the default is costly (even deadly) to individual subjects. Likewise, the outcome of a hard fork depends primarily not on the opinions of the users, but on the perceived legitimacy of each branch. By providing clear and decentralized procedures for upgrades we also delegitimize attempts to hard fork the protocol which come to be seen as suspicious.

A common criticism is that Tezos’ flexibility is a weakness. If the rules of the ledger can change, how can it be trusted at all? This criticism falls prey to the Nirvana fallacy. The sad reality is that no blockchain is free from the lurking political risk of a hard fork. We prefer to create a predictable and explicit conduit for evolution to happen rather than to try and prevent it only to have it pop up in unexpected and disruptive ways.

A second criticism is that creating a legitimate process for upgrades isn’t sufficient to meaningfully quell hard forks. Oftentimes, it is then suggested that a clever technological contraption might be applied instead, or in addition, to prevent hard forks. This usually takes the form of poison pills or economic sanctions against hard forkers. The second part of this blog post will present an impossibility result relating to the levy of economic sanctions against forks, while cautioning the reader against “impossibility results” in system design in general.
