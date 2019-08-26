---
layout: post
title: "A quick tour of the Tezos code base, and the state of its development"
author: arthurb
date: 2017-05-19 00:00:00 +0000
categories: tezos
image: assets/images/manhattan_bridge.jpeg
---

This post is an attempt to give a tour of the Tezos code base and its state of development. All of the functionality described in the whitepaper has been implemented to this date, except for gas metering. Most of the remaining work consists in finishing a security addition that we made to the network shell to increase its resilience to DDOS, optimizing smart-contract storage, and — most importantly — testing our network on a large scale and performing external security audits.

![The Manhattan bridge under construction.](/assets/images/manhattan_bridge.jpeg)*The Manhattan bridge under construction.*

We give a high-level overview, with a low-level emphasis on the remaining development to launch the Tezos network(highlighted like this). This approach is both more transparent and more objective than giving timeline estimates about the completion of the project. Most links in this document point to directories in our Github repo where those features are implemented.

## Services

Broadly speaking, we have four different services, the node, the client, the baker, and the evil client.

The main program is the Tezos **[node](https://github.com/tezos/tezos/tree/master/src/node)**. It connects to the network, relays blocks and transactions, validates transactions, and maintains a locally accessible view of the globally consistent public ledger. Think of it as the equivalent of bitcoind with generate set to off.

The **[client](https://github.com/tezos/tezos/tree/master/src/client)** interfaces with the node through its JSON HTTP API to provide a more convenient command line interface to access the ledger. In particular, it handles keys, remembers contract addresses, etc. You might call it our wallet.

The **[baker](https://github.com/tezos/tezos/tree/master/src/client/embedded/alpha/baker)** (which is technically split between the baker and the endorser and started from the client) monitors the network and creates blocks (or attests to the validity of other blocks through a signature) when the proof-of-stake algorithm calls upon it to do so.

The **[attacker](https://github.com/tezos/tezos/tree/master/src/attacker)** is an integration test tool. Its goal is to implement various attacks (sending a large amount of data, fake data, trying to trick a node into doing several reorganizations or validating a very long chain). It is philosophically inspired by Netflix’s [Chaos Monkey](https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey).

We also have a [webclient](https://github.com/tezos/tezos/tree/master/src/client/embedded/alpha/webclient), which contains a rudimentary version of the client that can be accessed locally in the browser, but we’re rethinking this in favor of putting more logic on the client side, following Ethereum’s approach with web3, so I won’t really cover it in this post. We do have [UX / UI wireframes](https://docs.google.com/document/d/15OYU4PVAG7cObGp7YajYtIb3atfnwTe0IC6zEnMfszo/edit?usp=sharing) for it too.

The **[alpha protocol](https://github.com/tezos/tezos/tree/master/src/proto/alpha)**, i.e. the protocol Tezos will initially ship with, is compiled within the node, but since it is extremely modular and concerns itself with problematic which are largely orthogonal to the concerns of the node, we give it its own section. When we talk about a protocol in Tezos, we are not referring to a wire protocol, but to a blockchain protocol, defined by its transaction semantic and its consensus algorithm as described in the white paper.

## 1. [Node](https://github.com/tezos/tezos/tree/master/src/node)

The node connects to the rest of the Tezos network through a peer to peer network layer. It validates and relays blocks and transactions, maintains the state of the ledger, and interfaces with the current protocol to interpret the semantic of the various operations it encounters.
Both the peer-to-peer layer and the network shell export their [RPC](https://github.com/tezos/tezos/blob/master/src/node/shell/node_rpc.ml) which is useful for building block explorers or debugging network connectivity issues.

## 1.1. [Peer-to-peer layer](https://github.com/tezos/tezos/tree/master/src/node/net)

The peer to peer layer is the outermost layer of a Tezos node. It communicates with other Tezos nodes through a peer-to-peer gossip network. Beyond its basic functionality, the current peer to peer layer implements a few additional features:

* All communications between all nodes are encrypted and authenticated

* Peers are scored, monitored and placed in gray lists or blacklists if they misbehave (e.g. relay invalid information) or use too many resources. Peers can also be whitelisted.

* Peers regularly exchange known good peers to increase the overall connectivity of the network.

* Each peer’s public key comes with a proof-of-work stamp to ensure a minimal cost for creating new identities in the network.

* We performed extensive fuzz-testing on the peer to peer layer in the past to identify possible risks of remotely triggering exceptions (OCaml is memory safe so remote code execution is extremely unlikely, barring a bug in the OCaml code or in some of the C libraries we use such as libsodium).

Here’s some of the remaining development and testing we would like to do on the peer to peer layer:

* Perform additional fuzz-testing. Our nodes are programmed to be very conservative and shut down at the slightest exception. This is great for debugging, but not always a good behavior to display on a real life system. It’s been awhile since we ran our fuzz tests and the peer to peer layer has evolved quite a bit since then, so we should give it another round.

* Run a bigger, messier, testnet with many more machines located all around the world. It’s one thing for the peer to peer layer to work in a small environment, but we want to make sure it scales to a much larger number of nodes and higher latencies. This is something we’ll learn as we grow our testnet.

## 1.2 [Context Database](https://github.com/tezos/tezos/tree/master/src/node/db)

This is the part of the node that stores data related to the state of the ledger. We use Irmin as a backend which has several benefits. It gives us a functional, append-only, view of the storage, and it encodes the state of the ledger in a hash tree which simplifies building simple verification clients.

There isn’t much to say here other than we’d like to run scalability tests to see how Irmin performs with billions of internal node. If it turns out that the backend is to slow to handle a tree of this size, we would look to replace it with key-value database such as leveldb. This doesn’t mean replacing Irmin itself, but rather adding a new backed to Irmin. This change would be completely transparent to the rest of the code.

In addition to the context database, the node also stores blocks and operations. Currently those are being stored inefficiently on disk. Moving them to leveldb or Irmin will help preserve inodes on the file system.

## 1.3 [Network Shell](https://github.com/tezos/tezos/tree/master/src/node/shell)

The network shell sits at the interface between the network and the protocol. It takes blocks sent to it by its peers and validates them against the current protocol.

This is, in my opinion, one of the most delicate parts of the code and hard to get right (other parts of the code are more security sensitive, but easier to get right). Proof-of-work consensus algorithms are very resilient to spam and denial of service attacks. If a Bitcoin node tries to trick you into validating a bad reorganization, you can either tell it is lying very quickly (by examining its headers), or it is impractically expensive for them to trick you. By design, block creation in proof-of-stake is cheap, which makes it harder to defend against denial of service attacks.

In January, we started re-architecting the validation logic to be more defensive. The white paper only specified a one pass validator checking every transaction to accept a reorganization. We replaced it with a multi-pass validator which checks increasingly large subsets of the blocks to accept a reorganization. First, we check the timestamps and a very small proof-of-work stamp; then we verify the signatures of the block creator and the endorsers. Finally, we verify the full blocks. This combined with a peer scoring and blacklisting mechanism provides a fair amount of protection against denial of service attacks. Note that, in a blockchain, denial of service attacks aren’t just a quality of service issue. They open the door for an attacker to create more than 51% of the blocks and create reorganizations, though this state of affair can be detected. The risk is greater if state channels (as used by the lightning network or the Raiden network) are used.

We have mostly completed this multipass validator. We started integrating this code to the master branch [in April](https://github.com/tezos/tezos/commit/1d7de77f6e96924e000e6e8057f0347f5addc336), but it hasn’t been fully integrated yet and needs more stress testing. Particularly, we haven’t yes asserted how gracefully it handles being flooded with a very large number of small reorganizations as opposed to a few long invalid ones.

Also, we need to implement checkpointing and fast chain synchronization. It’s important for users who haven’t been online in a long time to provide the software with a checkpoint. This checkpoint should also allow fast chain synchronization by letting you download the state of the ledger at this point in time from peers rather than revalidating the entire blockchain. Our entire storage is already in a Merkle tree which greatly this type of synchronization, but it remains unimplemented.

## 1.4 [Updater](https://github.com/tezos/tezos/tree/master/src/node/updater)

The updater grabs the source of a new protocol (a “protocol upgrade”), [compiles them](https://github.com/tezos/tezos/tree/master/src/compiler) on the fly and replaces the testnet with this new protocol. After a period of time, it may promote the test protocol to replace the main protocol, completing our default, two-phase, protocol upgrade mechanism.

We have been able to push and upgrade new protocols so this is basically working, but it hasn’t been tested extensively. It needs a test suite to double-check that it handles some corner cases properly (is the tar file invalid, did the compilation fail, etc.).

## 2. [Client](https://github.com/tezos/tezos/tree/master/src/client)

The client provides a higher level and nice interface to the node and the protocol than the RPCs do. It can

* store keys and track contracts

* send transactions

* typecheck and trace smart contracts

* deploy smart contracts

* trigger contract execution with data

It isn’t encrypting saved private keys at the moment because it doesn’t particularly matter for a testnet, but we obviously need it for the mainnet. It’s a trivial fix given that we already have bindings to libsodium, but we’ve been putting it off.

In general, I favor building web3 like wrappers around our JSON HTTP API in a variety of languages — typically, python, javascript, or go — and make it really easy for third parties to build all kinds of wallets. We shouldn’t have bindings for OCaml because we’d rather have OCaml programmers contribute to the core :)

Strictly speaking, such a client isn’t needed for launching the network since the RPC form a rudimentary client, but it is needed in practice for interacting with the testnet and evaluating it in the period prior to the launch. We need to cleanup the client, polish it, implement some of the functionality offered in the RPC but not in the client, and also cleanup the RPC interface, which is a mix of very low-level calls useful for debugging and high-level primitives.

## 3. [Baker](https://github.com/tezos/tezos/tree/master/src/client/embedded/alpha/baker)

The baker creates and validates blocks, manages its bond deposits and endorsers other blocks when called upon by the proof of stake algorithm. We call it baking for lack of a better word for what a proof-of-stake “miner” does. Mining suggests proof-of-work, forging sounds like forgery, minting implies coin creation. We went with baking to highlight the french roots of the projects, and because breads somewhat look like blocks. The baker also handles denouncing double endorsements (slashing), but denunciations still need to be tested.

In addition, the current baking client is a bit naive and sometimes get stuck in the testnet because it doesn’t use all of its available accounts to put up bonds when needed. This should be fixed.

We are also looking at more substantive improvements to the baker:

* Separate cryptographic signing into its own process.

Currently, the proof-of-stake client sign blocks and endorsements directly using the libsodium. We should replace this with an IPC to give ourselves flexibility as to how the signature is obtained (for instance, through a hardware wallet like the [ledger nano](https://www.ledgerwallet.com/products/ledger-nano-s), or a remote machine).

* Provide a safe remote control for the proof-of-stake client to withdraw funds and upgrade the software.

A node running proof-of-stake signing is sensitive and shouldn’t be easily accessed. However, it does need to be upgraded from time to time, and the tokens collected through block rewards may need to be withdrawn. We should provide a clean, authenticated, and minimalistic RPC for doing so, rather than relying on users SSHing into the machine and having broad permissions.

## 4. [Attacker](https://github.com/tezos/tezos/tree/master/src/attacker)

The attacker is currently fairly minimal. It mostly focuses on logical denial of service attacks by submitting corrupted data and trying to trick nodes into performing large amounts of useless validation.

We’d like to scale up the attacker and introduce more subtle attacking strategies such as selfish-mining, endorsing the wrong blocks, and other strategic attacks on the consensus mechanism.

## 5. [Protocol](https://github.com/tezos/tezos/tree/master/src/proto/alpha)

## 5.1 Transactions, proof-of-stake protocol, governance voting

This is a largely mature part of the code and seems to work as intended with no surprise. It is also well covered by unit tests. A great overview of the protocol can be obtained by looking at [apply.ml](https://github.com/tezos/tezos/blob/master/src/proto/alpha/apply.ml) which is largely readable, even to non-OCaml programmers. It also highlights the construction of our protocol logic, building upon layers of modules progressively abstracting the state of the ledger, from raw storage to high-level concepts like “contracts”.

## 5.2 [Smart contract capabilities](https://github.com/tezos/tezos/search?utf8=%E2%9C%93&q=script+in%3Apath&type=)

The Michelson smart contract language is operational, and users have successfully used and deployed Michelson contracts on the testnet. [We’ve also formally verified a few Michelson contracts](https://github.com/tezos/tezoscoq/tree/new_map_implementation) in Coq, including a [multisig contract](https://github.com/tezos/tezoscoq/blob/new_map_implementation/multisig.v). The Michelson typechecker can prove its correctness based on the correctness of the OCaml GADT.

There are a few tweaks remaining in Michelson. A small, but cool one, is that we’re replacing all the integer types with arbitrary precision integers. We already have a dependency on Zarith and libGMP, this should be rather straightforward, but it requires a slightly more careful gas accounting. The overhead when dealing with small integers is minimal, as libGMP will default back to the processor’s multiplication and it makes the formal analysis of contracts much easier. We are also missing a few opcodes such as tez by integer division or timestamp subtraction.

A more important feature is the implementation of gas monitoring, as we do not currently enforce gas limits on contracts. This is a very big, very important, critical feature, but it is relatively easy to add. Michelson is designed in a way that makes computing gas cost easy, but we haven’t gotten to it yet.

We are on the fence between two models for gas cost. In the Ethereum model, a user provides a fee greater than the gas cost and ends up being refunded the difference if she ends up spending less gas. The original Tezos model is closer to a fee market; there is a global block limit on gas (as well as a per contract limit). Users indicate the maximum amount of gas they are willing to let their transaction run for, and provide a fee, but the fee is unrelated to the gas. Bakers decide for themselves whether the fee is sufficient given the maximum gas amount, much like Bitcoin miners decide whether to include a transaction given its size in bytes and its fees. The advantage of this model is that there is no need for a global gas cost and the pricing happens dynamically. The disadvantage is that it does not incentivize users to be as parsimonious as they would be with a mandatory fee and blocks may end up always hitting their gas limits as opposed to sometimes being light. We should come to a decision about this.

The Big One is “efficient contract storage”. Currently, we deserialize and reserialize the entire contract storage every time a contract is run. This works well for simple smart-contracts like multisig, but it does not work well for contracts using a large amount of storage. Such is the case of “appcoin” like contracts which maintain a large map of addresses and balances. While we’ve said before that appcoins weren’t a primary focus for Tezos, we still want people to be able to make them! We have considered two approaches for this:

The easy one is that we make maps into a special case which is loaded and unloaded lazily. This is the only very large data structure that we expect to see in most smart contracts and would likely cover 99% of the use cases. It’s an easy hack, but it’s a hack.

The hard one is a lot more powerful, but also potentially a rabbit-hole, so we might want to wait until the next protocol upgrade. The idea would be to replicate the structure of all smart-contract storage directly into the ledger’s hash tree, using a technique known as [hash consing](https://en.wikipedia.org/wiki/Hash_consing). This has several enormous benefits:

* All data that is structurally equal is shared. It means that a contract could copy another contract’s 1GB map, modify it a bit and the size of the blockchain state wouldn’t increase by much more than the difference between the two.

* All smart contract code would be similarly shared. This means that reusing code in other smart-contract would not take up more storage on the blockchain.

* It’s really elegant.

It also has several large drawbacks:

* Implemented naively, the overhead can be very large. Imagine taking a 160 bits hash to every different integer present in the ledger state. We’d likely have to enforce a minimal leaf size, but that renders the approach more complex.

* It requires implementing a garbage collector for the ledger’s storage (reference counting will just work in this instance).

* Incentives for cleaning up storage are now blunted since you removing data does not necessarily reduces the size of the state.

We need to implement one of those two approaches. Time permitting we may implement a version of the second one but, otherwise, we’ll stick with the first one. We also need to decide on storage costs, and whether to use a rent model (pay to maintain data stored) or a buy and refund model.

*This is likely the largest missing piece in the code. We could launch without it, but we feel it would be too restrictive for those willing to create appcoins.*

## 5.3 Economic constants

There are many constants in the genesis protocol (block reward, block bond, endorsement reward, block size, gas limit, minimum fees, storage cost, cycle length) that need tuning. The original paper had reasonable values, but we should take another pass at it now that we’ve decided to shorten the length for which bonds are being held.

## 5.4 Consensus algorithm

The consensus algorithm has operated correctly on our testnet so far. We have been able to test it through a variety of network configuration and in the presence of the “attacker” client. We have also run some simulations of the consensus algorithms assuming malicious participants who try to mine selfishly. It has already convinced us to increase the number of block endorsers to 64 instead of 16 as the resilience of the algorithm increases quickly with a large number of endorsers. The malicious participants we simulated followed a near-optimal heuristic for maximizing the weight of their chain, but not theoretically optimal. We would like to analyze in more details the behavior of the chain under optimal selfish mining (designed to maximize the rewards paid to the group of selfish miners) and under optimal attempts to create the longest possible reorganizations. This is an academic project which can be conducted in parallel to the rest of the development and inform our choice of constants. We are focusing on establishing the security when a sufficiently large fraction of the miners behaves honestly. A full analysis of bribery resistance would be interesting but is out of scope for this iteration.

## 6. [Testing](https://github.com/tezos/tezos/tree/master/test)

A Tezos node was launched for the first time after six months of development, and it worked largely as intended. One of the benefits of programming in a functional programming language with a very strong emphasis on static typing is that once a program compiles it generally works. We use this to our advantage by leveraging the type system to introduce as much static verification about the properties of our code as we can. That said, correctness is particularly crucial for a public blockchain. To that end:

* We would like to conduct at least two, independent, security audits of the protocol before launching.

* Increase the test coverage of the code base. While these are imperfect metric — particularly for OCaml code, we would like code coverage to be at 100% and the ratio of lines of unit test to lines of code to be at least 1:2 (it currently stands at 1:10 but has been growing quickly)

* We plan to keep improving the alphanet (our “testnet”) and eventually do a hot-launch by upgrading it to the mainnet. We should grow the testnet as much as possible and hammer on it as hard as we can while introducing substantial bounties for vulnerabilities.

I believe our team can reasonably accomplish these goals in a three to four months period (two is the minimum, assuming everything goes well, but there’s always something). This represents the **mode** of the distribution, where the most likely values lie. **There is a tail to that distribution**. Security audits could find small issues which are easily fixable, or they could point at design issues which are harder to address. We may encounter snags as we scale up the testnet. We might catch a bad stomach virus. It’s entirely within the realm of possibility that the project takes up to 6 months to ship. Based on my assessment of the remaining development (which I’ve outlined above) that does not seem *likely*, but it’s not *impossible*.

The Tezos foundation is running a crowdfunding event for the project. Before you consider participating, you should understand the technological limitations you will encounter. Until the network launches, there will be no tokens. You will only receive a private key represented by 15 words mnemonic on a paper wallet. Some projects have launched on the Ethereum blockchain and created tokens which are tradable before the actual application is built, using the ERC20 standard. **This is not the case here**. You will be able to use the Tezos testnet of course and start deploying smart contracts and try out playing with the protocol upgrade mechanism, but this testnet will reset periodically. You will not be able to use your tokens until the genesis block is created.

We will not officially launch Tezos until we are confident, beyond a reasonable doubt, that we have sufficiently tested and analyzed it. The team has no interest in delaying the project of course, but we do not want to be pressured into shipping early if the security analysis is inconclusive, or if the network isn’t ready as this wouldn’t be in anybody’s well-understood interest
