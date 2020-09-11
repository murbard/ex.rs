---
layout: post
title: "Protocol-level transaction fees?"
author: arthurb
date: 2020-09-11 19:00:00 +0000
categories: tezos ethereum
image: assets/images/rubber_band.jpg
featured: true
hidden: true
---

The following is a collection of thoughts around fee markets in general and mechanism to fold in transaction fees in a protocol, such as the one outlined in EIP 1559. While EIP 1559 is an Ethereum-specific proposal, the fee market in Tezos, Bitcoin, and Ethereum are essentially similar, making the ideas in this proposal broadly applicable to all three.

# How a fee market works

Let's first review how a fee market work, and clear up common misconceptions about fees. Block producers control a scarce resource: block space. Whether this space is expressed in terms of bytes, in terms of gas, or a combination thereof, the general idea is that there is some limit to how many transactions can be put in a block, and the block producer has to decide which transactions to include and which not to include.

The choice to include or not include a transaction is at the discretion of the block producer. To incentivize inclusions, transactions typically authorize the payment of a fee to the block producer to incentivize inclusion. Generally speaking, the default behavior programmed in block producing software is for block producers to maximize the collected fees[^1].

This default behavior is often mistakenly assumed to be part of the protocol. It isn't. There is nothing in Bitcoin, Tezos, or Ethereum's protocol that requires block producers to select the transactions paying the highest fees. In fact, protocols typically don't even know what transactions a block producer sees as ascertaining this would require consensus in the first place.

There are many reasons why a block producer could choose to include transactions based on other criteria than the fee attached to it. In particular, they may accept off-band payments to include transactions. This is particularly helpful for users who seek to enter into long term agreements with block producers.

## Corrolaries

There are a few important corollaries that follow from properly understanding fee markets.

1. **The cost of fees is not pegged to the value of the underlying cryptocurrency.** Marginal users who choose whether to transact or not do so based on the benefit they derive from making this transaction. This might be coupled with the value of the cryptocurrency but loosely so. Just because you purchase something in some currency doesn't mean that thing becomes cheaper or more expensive as the currency depreciates or appreciates. 

2. **The “real” fees could be higher than what is observed on-chain.** If a user is paying a miner off-band to include transactions, those could appear with 0 fees but that doesn't mean they were free.

3. **The “real” fees could be lower than what is observed on-chain.** Mechanisms that attempt to measure the cost of fees by looking at averages on the chain can be fooled by block producers including transactions with very large fees in their block as it is costless for them to do so.

# Fee volatility and inelasticity

Much of the discussion around fee markets comes from the volatility of fees. When the demand for transaction surges and meets an (almost[^2]) perfectly inelastic block space, fees surge. 

Historically, “Cryptokitties” has been the canonical example of this phenomenon. A short term spike of interest for a feline phenomenon can render the chain prohibitively expensive for everyone else to use.

The solutions proposed generally involve making the block space more elastic or using a different auction mechanism which might slightly temperate surges. We argue that this approach is at best a band-aid.

## Limited elasticity

First, block space is limited for security reasons, so it cannot be arbitrarily increased when demand increases. Conversely, artificially restricting block space during times of lower demand seems like it would only smooth-out the cost of fees by making them more expensive across the board. There simply is a naturally limited amount of slack for elasticity.

Specifically, block space is limited for a few reasons: 

1. The time to download and validate a block must be smaller than the time between blocks for all validators. In fact, in the case of the Nakamoto consensus, it needs to be so small as to be negligible.

2. Validators who are intermittently offline or online need to be able to catch up with the chain, so the time to validate the average block must be smaller than the time between blocks.

3. Storing blocks takes disk space, which is why we might want to care about the average block space.

Right out of the bat, we can eliminate consideration number three. It's not that the size of disk space doesn't matter, it's just that constraint number 1 is a much stronger constraint. Assume 2MB blocks every 5 seconds, that would come to 1TB a month. A 1TB hard drive comes to less than $30. This doesn't need to be fast storage either, as the blocks are stored for archival purposes only and are only ever accessed by new peers requesting the chain's history. The storage cost for old blocks is largely dominated by the cost of the Internet connection or the cost of computing.

Consideration number one places a tight constraint on the maximum block space, so any elasticity we might find would come reducing the block space below this maximum.

This only makes sense if consideration number 2 places a meaningful constraint on block space. If for some reason, we want the average block space to be lower than the worst-case block space, then there is elasticity to be found. If not, the elasticity merely boils down to artificially restricting supply when demand is low.

## Does the average block validation time matter?

There are arguments on both sides of that issue.

It must indeed be much faster to synchronize with the chain than it is to merely follow the chain in real-time. Otherwise, validators who periodically go offline would need to wait a very long time before being caught up. If validation time takes, say, 95% of the block time, it would take 20 hours to synchronize with the chain after being offline for only an hour.

A counter-argument is that synchronization can be done in parallel. A node syncing with the chain can download a snapshot of the state of the ledger at present, and a snapshot at the midway point when they were offline, and validate, in parallel, that the chain correctly links the last known state to the midway point, and the midway point to the current state.

The counter-counter-argument is that doing so is somewhat impractical. It involves large fixed costs in downloading the state of the ledger, it likely requires multiple hard drives for the IO to be properly parallelized, etc.

A better argument in favor of limiting the average block validation time below the maximum block validation time is safety. Validation time is estimated based on a gas model but it is only an approximation of the real validation time on a real machine. The consequences of the average validation time exceeding the block time can be extremely serious and split the chain or prevent it from advancing. However, the consequences of some blocks occasionally exceeding the maximum validation time are quite mild. At worst some block producers might miss their chance to create a block.

The existence of a safety margin for the average block validation time which can be set lower than the maximum block validation time opens the room for at least *some* elasticity.[^3]


## Block producer extractable value

Blockchains, and particularly smart-contract platforms, sometimes leave money on the table for block producers to grab. This could be a mistake “anyone can spend” output in Bitcoin, this could be an auction about to expire on Tezos, etc. If or when that happens, fiddling with the amount of block space or the kind of auction involved in the fee market will have little impact on the block content.

# Long term service agreements: the correct solution

There is a natural solution to the problem and it doesn't involve modifying the fee market at all. As we mentioned earlier, users can enter into long term service agreements with block producers. In a proof-of-stake network, the owner of a stake controls a predictable source of block space. The output of a miner in a proof-of-work network is more difficult to predict due to difficulty adjustments but it is still somewhat predictable.

Users who need the guarantee that they can regularly include their transactions at a reasonable price and block producers both benefit from entering into long term service agreements where they are guaranteed a certain number of transactions per month with some maximum latency.

Under this model, bouts of craze for the latest fad do not affect normal users of the chain. Normal users will pay the contractually agreed upon transaction rate, and the very high transaction fees will essentially only affect those jumping on the fad bandwagon.

This is a financial solution to a fundamentally financial problem. Demand and supply of block space are commodities, they can be hedged, traded, and packaged in a way that shifts the risk.

Those who would insist on being independent on any kind of off-chain agreement still have the option of paying the fee market's spot price or, even better, to run nodes and become block producers themselves.

# Protocol-level fees?

Having given a quick lay of the land, let's look at what EIP-1559 does. When we look at full blocks, we see that, although there are variations in each block, there is an underlying, slowly moving, and persistent fee being paid per gas unit consumed. The fact that we may be able to observe that is purely a consequence of the relative lack of popularity of off-band payment for miners. By and large, transaction fees are paid using the default method, and block producers select transactions the default way. That doesn't mean we can trust those numbers for anything where security is critical, but they are good enough to give us some idea of what's going on.

The idea of EIP-1599 is to take out this baseline amount of fee and turn it into a burn. We cannot do that by looking at the fees paid, because this number is not reliable, but we can do it by searching for an equilibrium.

One way to do this is to let block producers choose the block space (within some bounds). As long as it is above average, the burn required per unit of gas consumed keeps increasing. As long as the block space is below average, the burn require keeps decreasing. This control mechanism ensures that the burn is set at the level where the amount of transactions being submitted matches the available block space. Users can still attach a transaction fee (now dubbed “tip”) to their transactions to incentivize inclusion.


## Benefits & drawbacks

There are several benefits to this approach, but also some drawbacks.

### Deflationary

The first benefit is that this codifies the use of the cryptocurrency in the fees and makes fee payment deflationary. This can matter for proof-of-work networks where the fees would otherwise be spent on hash power. This might seem equivalent to simply reducing the block subsidy, but relying more on fees to secure blocks changes the dynamic as shown in Carlsten et al (2016)[^4].

For proof-of-stake networks, this benefit is somewhat moot as transaction fees naturally accrue to stakeholders anyway, regardless of the way they are being paid. So the benefit that this brings to proof-of-work networks is already present in proof-of-stake networks. However, in some jurisdictions, it could conceivably be more tax-efficient to burn the fees.

### Elastic

A second benefit is that this brings some elasticity to the block space. Depending on how steeply the required burn increases with blocks that go beyond the target average size, this can somewhat alleviate congestion. We are however quite skeptical that moderate increases of the block space in short bursts would do much to reduce congestion.

### Better UX

A third benefit is in terms of UX. Currently, wallets must evaluate fees by looking at the fees paid over past blocks. However, we know these numbers do not necessarily reflect the reality of the fee market. If the burn cost is built in the protocol, it becomes much easier for wallets to evaluate how much should be paid for inclusion. Users only need to worry about estimating fees when the congestion is so high that the burn cost doesn't rise fast enough to balance the market.

### Meaningful, in protocol, metric

A fourth benefit is that the fees paid to include transactions now become a meaningful number that can be used within the protocol or smart-contracts. This is not the case when fees can be manipulated completely by block producers. Reliable numbers could be used to grant governance rights over smart-contracts to those who use them the most for instance, or it could be used to grant discounts to access contracts that have consumed a lot of gas over their lifetime, etc.

### Drawback: potential for oscillations

We might wonder whether we could dispense with the elastic block space altogether and still get the benefit of estimating the appropriate burn cost. This would generally suggest trying to tie very moderate changes in the block space to a very steep rate of increase for burn costs. However, doing so is almost guaranteed to produce see-saw behavior where the cost oscillates wildly. A slow rate of increase would simply lead to the "tips" added to transactions becoming the de facto fee market.

There is a trilemma between inelasticity, stability, and not having tips become a de facto fee market. You can only have two, but it's not clear just how much elasticity you need to foster stable burn costs that don't oscillate all over the map and transaction tips that remain marginal.

### Drawback: complicates long term service agreements

The main drawback is that it no longer becomes possible to hedge transaction costs by holding a stake, guaranteeing access to a given rate of block space. This means block producers can no longer easily enter into service agreements with users to guarantee them a certain amount of transactions.

There are two possible workarounds:

1. Shift the burn cost risk to some third party through insurance.
2. Allow users to prepay for transactions on-chain.

The first approach is straightforward if a bit heavy. A long term service agreement would now involve a user and an insurer who must take on the risk of paying the burn cost. This is not as ideal of a situation as a traditional fee market where the block producer's risk and the consumer's risk perfectly offset each other, there is now a cost to insure the risk.

The second approach requires quite a bit more on-chain machinery. It would work like this: for all blocks, a given capacity is reserved, tokenized, and auctioned off a few months before the blocks are created. Those who intend to include transactions in the future can buy those tokenized transaction rights now and use them instead of paying a burn cost when the time comes. However, they would still need to enter into a long term service agreement with a block producer to ensure that they can indeed exercise their right.

# Conclusion

There are interesting benefits in moving transaction fees at the protocol level, in particular in terms of UX. However, doing so can jeopardize the most natural and efficient way for regular users to access block space: through long term service agreements. Indeed, these agreements do not seem common today. The benefits of protocol-level fee assessment depend on whether there are fundamental reasons for this, or if this is because the market is still immature.

[^1]: Solving a [knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem)
[^2]: Nitpick: we say "almost" because, technically speaking, in a proof-of-work network, higher fees could lead to more hash power coming online and lowering block time in the short term, until the difficulty adjusts. This is a nitpick, it's a small effect, your takeaway should be that I want to be precise, not that proof-of-work has great built-in elasticity.
[^3]: I defended on Twitter the more radical view that only maximum validation time matters, but thoughtful comments from Dankrad Feist led me to reconsider my position.
[^4]: [On the instability of Bitcoin without block reward. (Carlsten et al. 2016)](https://www.cs.princeton.edu/~arvindn/publications/mining_CCS.pdf)
