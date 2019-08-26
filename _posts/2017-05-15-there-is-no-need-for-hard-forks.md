---
layout: post
title: "“There is no need for hard forks”"
author: arthurb
date: 2017-05-15 00:00:00 +0000
categories: tezos
image: assets/images/fork_in_road.jpeg
---

But there will be hard-forks nonetheless

One of the most salient aspects of the Tezos ledger is its ability to self-govern by accepting and deploying protocol upgrades, on chain, without hard-forking. Participants in a blockchain protocol typically follow a fixed set of rules. A hard-fork is a procedure in which these participants break some of those rules and start following a different rule set. Hard-forks have been suggested as a way to introduce new features and keep up with innovations in cryptographic ledgers. However, this sudden change of rules potentially creates governance issues. Without a clear procedure for converging on a specific course of action during a hard fork, participants are left guessing as to which rule set other participants will pick. This creates the dynamics of a[ Keynesian beauty contest](https://en.wikipedia.org/wiki/Keynesian_beauty_contest) in which the winner ends up being selected on the basis of social expectations rather than merit.

The Tezos rule set provides for the self-amendment of its own rules. As a result, innovation and change are possible without the need for a hard fork. You can see me in [this video](https://www.youtube.com/watch?v=7m7EU4JWI88) saying “there is no need for hard-forks”. And indeed, there is no need for hard-forks to incorporate new features into Tezos.

However, **there will almost certainly be hard-forks in the life of Tezos**. Allow me to explain. There are circumstances in which changes to the ledger through the governance model aren’t practical and do not present any advantage over a simple hard-fork. In those scenarios, the desired change is typically uncontroversial and widely desired among the participants. Here are a few examples:

![Photo  by Billie Grace Ward](/assets/images/fork_in_road.jpeg)*[Photo](https://www.flickr.com/photos/wwward0/23862250382) by Billie Grace Ward*

In 2011, it was found that the Bitcoin code contained an integer overflow bug which let anyone generate very large amounts of Bitcoins out of nothing. The logic of the code was checking every output individually but did not guard the total against an overflow: 0 btc were equal to 92,000,000,000 btc + 92,000,000,000 btc. Fortunately for Bitcoin, this was promptly fixed thanks to the diligence of the Bitcoin developers, and it could even be fixed as a soft-fork, a variant of a fork which only restricts the set of valid blocks and can be enforced by node validators (miners in this case), alone.

The Tezos code base strives to root as many bugs as possible (for instance, all tez amounts are typed and encapsulated in a module which prevents accidental overflow). However, bugs can and do happen. A bug of this nature cannot easily be resolved by a governance procedure which is slow and deliberative in nature. The discovery that billions of coins can be created out of this air is an issue that requires immediate action. It cannot wait for a scheduled vote to be held.

Fortunately, there is no legitimate constituency of token holders who honestly believe that rendering the network unusable by letting anyone create billions of coins is the *preferable *course of action. There can be a blurry line between technological and political decisions on a decentralized blockchain, but this scenario falls squarely in the “technological decision” category.

Should Tezos ever be plagued by a bug of this nature, it will, of course, experience a hard fork. This isn’t just me being prescriptive (though I would of course support such a fork), it’s a mere statement of fact. A hard fork would take place in such an instance, and it would be regarded as the legitimate branch.
> Emergency bug fixes are a perfectly good reason to hard fork (or soft-fork if possible given the circumstances).

There are other circumstances in which I could see the Tezos blockchain hard forking. Suppose for instance that a new protocol upgrade wishes to make use of pairing cryptography. One possibility would be to implement a pairing cryptography library in OCaml and include it in the update. Another would be to rely on an existing library like[ Relic](https://github.com/relic-toolkit/relic) or[ PBC](https://crypto.stanford.edu/pbc/). For instance, our current protocol relies on the external[ libsodium](https://download.libsodium.org/doc/) to provide most cryptographic primitives. We might also want to include the [Zcash-Bellman](https://z.cash/blog/bellman-zksnarks-in-rust.html) library. Strictly speaking, adding an external dependency should be considered a hard fork. However, if participants do agree to an upgrade which depends on that library, it isn’t a stretch to expect that they will also install the dependency rather than stick to a “classic” chain who cannot produce any new block by its own rules.
> Occasionally adding useful external dependencies, particularly for cryptographic libraries, is a perfectly good reason to hard fork. It barely even registers as one.

A hard-fork is not “needed”. We could re-implement pairing cryptography in OCaml (OCaml excels for writing program dealing with abstract algebra, but its GC makes it somewhat less suitable for avoiding side channel attacks). We could also embed an LLVM interpreter in the protocol and pass the compiled sources of the external library as a string. Push come to shove, we wouldn’t be stuck. We don’t *have* to depend on a hard-fork, which means we don’t have to rely on any centralized party having enough of a following to push it through. But if a hard-fork is the lowest hanging fruit and the issue is menial (installing the dependency required by an upgrade which stakeholders have already approved) it’s quite alright to use one.

We wish to avoid hard-forks because, as a mechanism for introducing innovative but contentious changes, they tend to be centralizing. We do not do it out of some weird intrinsic rejection of hard-forks . We value pragmatism.

A few potential scenario for governance failure in Tezos often come up: “What if a malicious party bribes a large number of stakeholders to make them vote for a decision that destroys the network?” or “What if a large clique of Proof-of-Stake delegates decide to harm the chain?”. Tezos has several built-in protections against such scenarios, but it does also have the possibility to hard-fork. Our[ principles and values](https://medium.com/tezos/tezos-philosophy-and-values-9297f308ae21) suggest that we follow principles to their natural conclusion but not to their absurd conclusion. The goal of Tezos isn’t to run a formal game disconnected from reality. If the network were harmed in some catastrophic failure of the governance mechanism, of course it would hard-fork.
> If buggy or malicious protocol upgrade passes through the various defenses of the governance procedure, not letting it permanently cripple the network is a perfectly valid reason to hard fork.

Hard forks don’t scale well to large communities and they can have a centralizing effect. We do not see them as a good and reliable tool for routinely upgrading the ledger but, like revolutions, they do occasionally have their place in the toolbox, and their possibility serves to keep many actors honest. Let it be known that Tezos was designed to provide a reliable smart-contract platform with a dynamic governance model that lets it keep up with innovation, not to make a theoretical point about hard-forks.
