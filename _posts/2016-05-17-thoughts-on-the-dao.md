---
layout: post
title: "Thoughts on “the DAO”"
author: arthurb
date: 2016-05-17 00:00:00 +0000
categories: tezos
---

An exciting glimpse into the future that ultimately falls a bit short

(Disclosure: I own Bitcoins and Ethers, and I am the team leader on the [Tezos](http://www.tezos.com/) project. I do not own tokens from “the DAO”)

With over $120 million worth of ether (Ethereum’s native token) under its control, it’s hard to ignore “[the DAO](https://daohub.org/)”. While we’re thrilled that this project strives to achieve some of the objectives of [Tezos](http://www.tezos.com/), we find it comes short in several important way.

**What is a DAO?**

DAO stands for “decentralized autonomous organization”. The idea behind a DAO is to use the consensus forming abilities and programmable transaction logic of blockchains to let a set of participants (who may not trust each other) jointly undertake economic activities. In some sense, Bitcoin itself is a DAO where the participants, miners, are rewarded by the creation of new bitcoins and transaction fees for maintaining a proof-of-work consensus. Generally speaking, the range of activities a DAO can undertake is potentially much wider.

A DAO participant might propose, for instance, to develop and market a piece of software. One way to go about this would be to set up a smart‑contract designed to crowdfund the development. Upon successful funding, the contract would automatically release a fraction of the funds to the DAO, as profits, and another to a set of trusted third parties tasked with hiring developers. Alternatively, the specification of the software might be so precise that a smart contract itself could evaluate the suitability of the implementation, thus dispensing with third parties.

In essence, a DAO is a set of programmatic rules allowing the participants to control the release of funds and direct them to certain endeavors. There also lies an important limitation. The contracts can ultimately only monitor the release of funds based on inputs they receive from the participants or from trusted third parties. They can take no other action. Thus, the more “real world” oriented the activities of the DAO, the more it depends on trusted third parties and the less “decentralized” and “autonomous” it becomes.

**What is “the DAO”?**

“the DAO” is a particular DAO (with the clever idea of occupying the name) which was launched on the Ethereum blockchain and has attracted a tremendous amount of interest. The participants have been invited to contribute ethers to “the DAO” and have received tokens in exchange at a 1:1 parity. It is important to understand that, although the amount raised represents a resounding success, it should not be compared to the amounts raised by projects such as Ethereum in their inception. The funds raised are still controlled by the people who contributed them, albeit through the organization. In investor parlance, at this point, the “pre-money” valuation of “the DAO” is effectively zero. Of course, that is likely to change very soon as the DAO raises the price of its tokens.

On its website, “the DAO” indicates its interest in funding a few “Slock.it” and “Mobotiq”, applications in the “Internet of Thing” and “Peer to peer mobility” respectively. It is not immediately clear what the advantages of funding such proposals through a DAO are given. As professor Emin Gün Sirer puts it “The DAO is an excellent solution to the non-existent problem of Kickstarter absconding with the funds.”.

<iframe src="https://medium.com/media/37372aac915a59f305ad569fd2de0cdf" frameborder=0></iframe>

It may also reflect the difficulty that small investors have in participating in early stage technological projects, despite the passage of the JOBS act more than four years ago. Quoting Gün again: “The DAO is an indication of a bigger malaise: there are too few investment opportunities, esp for cryptocurrencies but also for fiat.”

<iframe src="https://medium.com/media/246c3e097dc9ddf0a3d10cbfba3a9be8" frameborder=0></iframe>

**Where does it come short?**

Setting aside the question of its purpose, “the DAO” comes a bit short on a few points. In particular, its governance mechanism shows a few weaknesses. In order to avoid majority stakeholders abusing the minority, the project relies on trusted curators who whitelist potential “contractors” who will receive payments. Those curators can be revoked by vote, and, the voting minority is allowed to split off the DAO keeping its tokens in proportion.

However, if “the DAO” is to make successful investments, a substantial fraction of its value will be in the form of intangible assets, customer relationships, contracts, know-how, goodwill, etc. It is not clear how such assets would be split.

A sophisticated governance system ought to provide better protections for the minority shareholder. Crucially, it should be able to be securely amended to adapt to the changing requirements created by the complex activities undertaken by the organization. Thus, in its current form, the DAO acts a sort of limited purpose investment fund and not as a living and breathing complex organization.

In addition, “the DAO” depends on the stability of Ethereum to survive, and Ethereum has not solved the governance problem which has plagued Bitcoin. At best one could claim that its benevolent dictators are more in tune with their community, but the governance remains ad-hoc and centralized.

**Why [Tezos](http://www.tezos.com) is different?**

Disclosure: I’m the team leader on the Tezos project.

Tezos was created first and foremost to address a governance issue in decentralized cryptographic ledgers. It does so by allowing stakeholders to approve updates to the protocol, including the consensus algorithm, and the governance rule themselves. Unlike “the DAO” which merely transfers funds to some development contracts, Tezos can transform itself in incredibly flexible ways.

It is possible in Tezos to go from a simple democratic system of organization to one displaying robust mechanisms inspired by successful governance models such as the Republic of Venice. An even more radical model that can be adopted is to rely on prediction markets for making automated decisions. Updates to the protocol can be required to carry formal proofs indicating that they respect agreed core principles agreed upon by the stakeholders. All of this can be achieved directly, on the blockchain.

**P.S. is a DAO the same as proof-of-stake?**

I have seen a few people confuse a DAO and proof‑of‑stake. The two are distinct ideas, though both depend on the concept of “stakeholders”. Proof‑of‑stake is a consensus algorithm for blockchains which relies on cryptographic signatures from token holders to secure itself against malicious forks. Unlike proof‑of‑work, the consensus algorithm introduced by Satoshi Nakamoto in Bitcoin, it does not rely on mining. It does not consume a very large amount of energy, the network scales better and the incentives of different protocol participants are more aligned. This is particularly important for a DAO! If its interest run contrary to those of the miners (for instance they may wish to make important security updates to the consensus algorithm that could render expensive mining equipment worthless), the miners will win.

A downside of proof‑of‑stake is that it is considerably trickier to implement correctly than proof‑of‑work. Early concepts for proof‑of‑stake also suffered from a severe flaw dubbed the “nothing at stake” problem. Tezos uses a variant called proof‑of‑stake-at-stake, or bonded proof‑of‑stake in which participants place bonds as collateral to ensure that they will not cheat the network and endorse multiple forks which solves the “nothing at stake problem”.
