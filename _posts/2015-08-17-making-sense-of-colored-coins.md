---
layout: post
title: "Making sense of colored coins"
author: arthurb
date: 2015-08-17 00:00:00 +0000
categories: bitcoin blockchain
image: assets/images/shipping_with_car_tech.png
---

It’s impossible to walk around the distributed ledger landscape without encountering the idea of so-called “colored coins”. The purpose of this post is to clarify their nature, the motivation behind their invention, some of the issues associated with them, and the circumstances under which their exercise may be warranted. There will be a car analogy at the end, you may skip to that section if you know all about colored coins and like car analogies.

## What are colored coins?

Due to the nature of the Bitcoin blockchain, bitcoins are not inherently fungible: every single coin mined can be uniquely identified and its entire history tracked. The smallest identifiable, indivisible, unit is known as a “satoshi.” A single “bitcoin” is a collection of 100,000,000 satoshis; thus the price of a satoshi is very low: about $0.0000026 at current market rates.

This lack of fungibility, though potentially problematic, opens the door to the implementation of ledgers on top of the Bitcoin ledger. If several parties agree to attach meaning to a particular satoshi and to recognize its control as representative of the ownership of some other asset — potentially existing outside of the blockchain — then they can use the decentralized consensus offered by the Bitcoin blockchain to track ownership of the asset and permit secure transactions. The antecedent is crucial. The asset can only be tracked on the blockchain insofar as its physical custodians, or the relevant authorities, agree to recognize the legitimacy of the colored coin. The mere technological ability to track an asset, from common stocks to parcels of land, does not magically translate into the ability to form an authoritative record of ownership. All too often is that obvious fact sacrificed upon the altar of hype.

Assuming such an agreement is in place, these tiny, identifiable, pieces of bitcoin can be used to represent and track assets. Such a coin is known as a “colored coin.” In addition to the ownership tracking abilities inherited from the Bitcoin blockchain, small amounts of data can be embedded in the blockchain, allowing for potentially more complex mechanisms.

In fact, arbitrarily complex mechanisms are possible since, in principle, any data can be encoded in the transfer patterns of a coin. Consider for instance that, although there are more practical implementations, sending a coin to an even address can encode a “0”, and sending a coin to an odd address can encode a “1.” A specialized software can parse the blockchain to decode this information and reconstruct the state of a virtual ledger maintained on top of the Bitcoin ledger. In theory, there are no limits to the type of designs that can be implemented in this format. Smart contracts, new types of cryptographic signatures — all of these *can *be implemented using a pattern of transactions. Why should they?

## Why colored coins?

### Extending Bitcoin

The ability to extend the Bitcoin protocol in this way is definitely interesting though colored coins are a very roundabout way of doing things. Why not add these capabilities directly within the Bitcoin protocol rather than building another abstraction layer on top of a layer that clearly was not designed for this purpose?

The answer is primarily political. Not political in the sense involving governments or elections, but in the sense involving a large number of people with conflicting goals and interests. Such a change requires a large amount of coordination. Of course, one can unilaterally reprogram the Bitcoin client to support a different set of functionalities, but this suffers from at least three problems:

1. Changes to the Bitcoin code base must be very conservative given the amounts at stake.

1. Few people might pay attention to this alternate blockchain.

1. This blockchain will not benefit from the large amount of hashing power which protects the Bitcoin blockchain against certain types of attacks.

Colored coins attempt to do away with problems 1 and 3. They do not involve changing the Bitcoin code base, but they manage to piggyback on the proof-of-work consensus mechanism of the Bitcoin ecosystem.

### Merged mining

Another approach to reap similar benefits is a technique known as a “[merged mining](https://en.bitcoin.it/wiki/Merged_mining_specification).” The idea is to convince Bitcoin miners to include in the blocks they are trying to solve a special transaction representing the hash of a block belonging to an alternate blockchain. This allows the new chain to benefit from the hashing power available to the Bitcoin network and only requires miners to exert only a nominal amount of computation beyond what they would normally perform. This is the approach taken by the [namecoin](https://namecoin.info/) blockchain which focuses on recording titles to domain names. A drawback of merged mining is that it requires convincing miners to bother doing it, which probably entails using a native token as a reward, something colored coin can dispense with. Colored coins, however, do not require convincing miners. They simply plug in on top of the Bitcoin ledger, albeit in an awkward way.

## Issues with colored coins

Despite their potential attractiveness, colored coins potentially suffer from several issues.

### Issues with the use itself

Vitalik Buterin describes his choice to develop [Ethereum](https://etherchain.org/) as an independent chain rather than a colored coin due to the latter’s inability to handle SPV: [Simple Payment Verification](https://en.bitcoin.it/wiki/Thin_Client_Security).

Simple Payment Verification is a mechanism in Bitcoin that allows lightweight clients to verify the state of the chain without having to inspect the entire history of the blockchain. These clients are typically useful for cell phones and other devices where space and computing power may be at a premium. Unfortunately, it is necessary to track the history of “colored” coins since their inception to determine their “color” or lack thereof. Therefore, clients typically need to work with the complete chain, which can be unwieldy.

I, for one, do not see this as a major issue. A thin-client can connect to several servers it trusts not to collude and query them for the state of the ledger. If the stakes are so high than extending even such a small amount of trust becomes a concern, parsing the entire blockchain might be warranted anyway.

Another issue is that colored coins are potentially nefarious to the Bitcoin ecosystem. The security of Bitcoin rests on the assumption that miners stand to lose more by departing from consensus than they stand to gain. This assumption requires a balance between the reward received by miners, and the amounts they might stand to gain by reversing transactions. If colored coins represent valuable assets, this balance might be upset, endangering the status of all transactions.

### Semantic issues

Beyond these issues, there remains a somewhat heuristical but important issue. Implementing a new ledger on top of transactions is particularly gauche. Such transactions were never designed to encode information. Their use is inherently inefficient; it introduces friction in their ability to scale and an ugly bias in the types of design that they encourage.

Rather than being designed in the way that best addresses the problem they seek to address, colored coin solutions tend to be designed in a way that minimizes the pain of expressing the design in terms of micro-transactions. Designing software involves defining concepts and their relationships. The semantic available to the designer always shapes the resulting design. This is why most programmers care so deeply about programming languages. Even though all languages are capable of expressing the same computations, they do not encourage the same designs.

In his [Turing award lecture](http://www.eecg.toronto.edu/~jzhu/csc326/readings/iverson.pdf), Kenneth E. Iverson argues the case for the importance of notation in thought. He quotes in particular George Boole
> That language is an instrument of human reason, and not merely a medium for the expression of thought, is a truth generally admitted. — George Boole

and the prescient Charles Babbage
> The quantity of meaning compressed into small space by algebraic signs, is another circumstance that facilitates the reasonings we are accustomed to carry on by their aid. — Charles Babbage

In a 2001 essay, “[Beating the Averages](http://www.paulgraham.com/avg.html)”, Paul Graham argues
> What’s less often understood is that there is a more general principle here: that if you have a choice of several languages, it is, all other things being equal, a mistake to program in anything but the most powerful one. — Paul Graham

Another one from Edsger Dijkstra…
> The use of COBOL cripples the mind; its teaching should, therefore, be regarded as a criminal offense. — Edsger Dijkstra

What do all these quotes have in common? They are all arguing that the language in which ideas are expressed affects those ideas. In essence, they are claiming the Sapir-Whorf hypothesis is true, at least for programming languages.

The same is true of cryptographic ledgers. When expressing ledgers in the form of micro-transactions on the Bitcoin blockchain, the designs will always end up gravitating towards specific patterns which accidentally happen to be more easily expressible.

## The worst of both worlds

*In which a car analogy is introduced*

As we’ve seen, colored coins have several drawbacks, making their use rather maladroit. However, they do allow a type of permissionless innovation that can piggyback off the mining power of the Bitcoin blockchain. Yet…

Imagine that one day, someone invented a car.

![In this story, there were no Model T, the modern sedan is invented as is](/assets/images/grey_sedan.png)*In this story, there were no Model T, the modern sedan is invented as is*

Many people saw the car and said: “This is great! Not only can we now go from place to place, this car technology might have a lot more applications, like shipping.”

After a while, many (O, so, so, many) design groups came up with the following technology.

![Shipping, using car technology — not to scale](/assets/images/shipping_with_car_tech.png)*Shipping, using car technology — not to scale*

The solution had been staring at us the whole time! One could easily bolt the four corners of a large wooden platform on the roofs of four cars and place a shipping container on top. While there had been talks of a truck technology, it was summarily dismissed.

To be sure, they were several problems with the design. The aerodynamics were atrocious, but that could be somewhat alleviated by placing a tent over the contraption. Turning was initially difficult, but some clever engineers introduced swivels on top of the car, making the process easier. The cars would not always stay at the same speed, but using radio communication between the drivers more or less remedied the issue.

But, truck technology? Well that was unproven, and also trucks looked a lot like train wagons, and the real innovation was the car, so cars had to be used!

Where am I going with this? A large number of projects in the space of distributed ledgers have been peddling solutions involving the use of colored coins ***within permissioned ledgers***. As we’ve explained earlier, colored coins were born out of the near impossibility of amending the code base of Bitcoin. They are first and foremost a child of necessity in the Bitcoin world… a necessary evil, a fiendish yet heroic hack unlocking new functionality at a dire cost.

One could argue that reusing the core bitcoind code offers the benefit of receiving downstream bug fixes from the community. This argument falls flat as the gist of such fixes can be incorporated into any implementation. Issues encountered by Bitcoin have ranged from a lack of proper integer overflow checking to vulnerabilities with signature malleability. Such issues can potentially affect any blockchain implementation; the difficulty lies in identifying them, not in producing a patch to fix them, a comparatively straightforward process. Of course, other bugs might be introduced when developing new functionalities, but the same is true regardless of the approach undertaken.

Basing a fresh ledger, independent from the Bitcoin blockchain, on a colored coin implementation is nothing short of perversion. It is akin to designing a truck using a wooden board bolted on the top of four cars. If, for some reason, the only type of vehicle that could use a highway were sedans, that solution *might* make sense. But if you have the chance to build a truck and instead *chose* to rig a container on top of a few cars then perhaps you should first learn how to engineer trucks.
