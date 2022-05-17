---
title: Proof-of-stake (PoS)
description: An explanation of the proof-of-stake consensus protocol and its role in Ethereum.
lang: en
sidebar: true
---

A [consensus mechanism](/developers/docs/consensus-mechanisms/) is a set of rules that incentives that enable nodes to come to agreement about the state of the Ethereum network. There are several different classes of consensus mechanism that have been implemented on various blockchains, including [proof-of-work (PoW)](/developers/docs/consensus-mechanisms/pow/), proof-of-stake (PoS) and proof-of-authority (PoA).  Ethereum has used PoW since its genesis, but is moving to PoS. This was always the plan because PoS is thought to be more secure than PoW, uses drastically less energy, and enables new scaling solutions to be implemented. However, proof-of-stake is also more complex than proof-of-work and refining the mechanism has taken years of research and development. The challenge now is to implement PoS on the live Ethereum network - a process known as ["The Merge"](/upgrades/merge/).

## Prerequisites {#prerequisites}

To better understand this page, we recommend you first read up on [consensus mechanisms](/developers/docs/consensus-mechanisms/).

## What is proof-of-stake (PoS)? {#what-is-pos}

Proof-of-stake is a type of [consensus mechanism](/developers/docs/consensus-mechanisms/) used by blockchains to achieve distributed consensus. In proof-of-work, miners prove they have capital at risk by expending energy. In proof-of-stake, validators explicitly stake capital in the form of ether into a smart contract on Ethereum. This staked ether then acts as collateral that can be destroyed if the validator behaves dishonestly or lazily. The validator is then responsible for checking that new blocks propagated over the network are valid and occasionally creating and propagating new blocks themselves.

Proof-of-stake comes with a number of improvements to the proof-of-work system:

- better energy efficiency – there is no need to use lots of energy on proof-of-work computations
- lower barriers to entry, reduced hardware requirements – there is no need for elite hardware to stand a chance of creating new blocks
- reduced centralization risk – proof-of-stake should lead to more nodes securing the network
- because of the low energy requirement less ETH issuance is required to incentivize participation
- economic penalties for misbehaviour make 51% style attacks exponentially more costly for an attacker compared to proof-of-work
- the community can resort to social recovery of an honest chain if a 51% attack were to overcome the crypto-economic defenses.

## Validators {#validators}

To participate as a validator, a user must deposit 32 ETH into the deposit contract and run three separate pieces of software: an execution client, a consensus client, and a validator. On depositing their ether, the user joins an activation queue that limits the rate of new validators joining the network. Once activated, validators receive new blocks from peers on the Ethereum network. The transactions delivered in the block are re-executed, and the block signature is checked to ensure the block is valid. The validator then sends a vote (called an attestation) in favor of that block across the network.

Whereas under proof-of-work, the timing of blocks is determined by the mining difficulty, in proof-of-stake, the tempo is fixed. Time in proof-of-stake Ethereum is divided into slots (12 seconds) and epochs (32 slots). One validator is randomly selected to be a block proposer in every slot. This validator is responsible for creating a new block and sending it out to other nodes on the network. Also in every slot, a committee of validators is randomly chosen, whose votes are used to determine the validity of the block being proposed.

## Finality {#finality}

A transaction has "finality" in distributed networks when it's part of a block that can't change without a significant amount of ether getting burned. On proof-of-stake Ethereum, this is managed using "checkpoint" blocks. The first block in each epoch is a checkpoint. Validators vote for pairs of checkpoints that it considers to be valid. If a pair of checkpoints attracts votes representing at least two-thirds of the total staked ether, the checkpoints are upgraded. The more recent of the two (target) becomes "justified". The earlier of the two is already justified because it was the "target" in the previous epoch. Now it is upgraded to "finalized". To revert a finalized block, an attacker would commit to losing at least one-third of the total supply of staked ether (currently around $10,000,000,000). The exact reason for this is explained [in this Ethereum Foundation blog post](https://blog.ethereum.org/2016/05/09/on-settlement-finality/). Since finality requires a two-thirds majority, an attacker could prevent the network from reaching finality by voting with one-third of the total stake. There is a mechanism to defend against this: the [inactivity leak](https://arxiv.org/pdf/2003.03052.pdf). This activates whenever the chain fails to finalize for more than four epochs. The inactivity leak bleeds away the staked ether from validators voting against the majority, allowing the majority to regain a two-thirds majority and finalize the chain.

## Crypto-economic security {#crypto-economic-security}

Running a validator is a commitment. The validator is expected to maintain sufficient hardware and connectivity to participate in block validation and proposal. In return, the validator is paid in ether (their staked balance increases). On the other hand, participating as a validator also opens new avenues for users to attack the network for personal gain or sabotage. To prevent this, validators miss out on ether rewards if they fail to participate when called upon, and their existing stake can be destroyed if they behave dishonestly. There are two primary behaviors that can be considered dishonest: proposing multiple blocks in a single slot (equivocating) and submitting contradictory attestations. The amount of ether slashed depends on how many validators are also being slashed at around the same time. This is known as the ["correlation penalty"](https://arxiv.org/pdf/2003.03052.pdf), and it can be minor (~1% stake for a single validator slashed on their own) or can result in 100% of the validator's stake getting destroyed (mass slashing event). It is imposed halfway through a forced exit period that begins with an immediate penalty (up to 0.5 ETH) on Day 1, the correlation penalty on Day 18, and finally, ejection from the network on Day 36. They receive minor attestation penalties every day because they are present on the network but not submitting votes. This all means a coordinated attack would be very costly for the attacker.

## Fork choice {#fork-choice}

When the network performs optimally and honestly, there is only ever one new block at the head of the chain, and all validators attest to it. However, it is possible for validators to have different views of the head of the chain due to network latency or because a block proposer has equivocated. Therefore, consensus clients require an algorithm to decide which one to favor. The algorithm used in proof-of-stake Ethereum is called [LMD-GHOST](https://arxiv.org/pdf/2003.03052.pdf), and it works by identifying the fork that has the greatest weight of attestations in its history.

## Proof-of-stake and security {#pos-and-security}

The threat of a [51% attack](https://www.investopedia.com/terms/1/51-attack.asp) still exists on proof-of-stake as it does on proof-of-work, but it's even riskier for the attackers. A attacker would need 51% of the staked ETH (about $15,000,000,000 USD). They could then use their own attestations to ensure their preferred fork was the one with the most accumulated attestations. The 'weight' of accumulated attestations is what consensus clients use to determine the correct chain, so this attacker would be able to make their fork the canonical one. However, a strength of proof-of-stake over proof-of-work is that the community has flexibility in mounting a counter-attack. For example, the honest validators could decide to keep building on the minority chain and ignore the attacker's fork while encouraging apps, exchanges, and pools to do the same. They could also decide to forcibly remove the attacker from the network and destroy their staked ether. These are strong economic defenses against a 51% attack.

51% attacks are just one flavor of malicious activity. Bad actors could attempt long-range attacks (although the finality gadget neutralizes this attack vector), short range 'reorgs' (although proposer boosting and attestation deadlines mitigate this), bouncing and balancing attacks (also mitigated by proposer boosting, and these attacks have anyway only been demonstrated under idealized network conditions) or avalanche attacks (neutralized by the fork choice algorithms rule of only considering the latest message).

Overall, proof-of-stake, as it is implemented on Ethereum, has been demonstrated to be more economically secure than proof-of-work.

## Pros and cons {#pros-and-cons}

| Pros                                                                                                                                                                                                                | Cons                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Staking makes it easier for individuals to participate in securing the network, promoting decentralization. validator node can be run on a normal laptop. Staking pools allow users to stake without having 32 ETH. | Proof-of-stake is younger and less battle-tested compared to proof-of-work                                               |
| Staking is more decentralized. Economies of scale do not apply in the same way that they do for PoW mining.                               | Proof-of-stake is more complex to implement than proof-of-work                                                            |
| Proof-of-stake offers greater crypto-economic security than proof-of-work                                                                                                                                                                | Users need to run three pieces of software to participate in Ethereum's proof-of-stake compared to one for proof-of-work. |
| Less issuance of new ether is required to incentivize network participants                                                                                                                                          |                                                                                                      |


## Further reading {#further-reading}

- [Proof of Stake FAQ](https://vitalik.ca/general/2017/12/31/pos_faq.html) _Vitalik Buterin_
- [What is Proof of Stake](https://consensys.net/blog/blockchain-explained/what-is-proof-of-stake/) _ConsenSys_
- [What Proof of Stake Is And Why It Matters](https://bitcoinmagazine.com/culture/what-proof-of-stake-is-and-why-it-matters-1377531463) _Vitalik Buterin_
- [The Beacon Chain Ethereum 2.0 explainer you need to read first](https://ethos.dev/beacon-chain/) _Ethos.dev_
- [Why Proof of Stake (Nov 2020)](https://vitalik.ca/general/2020/11/06/pos2020.html) _Vitalik Buterin_
- [Proof of Stake: How I Learned to Love Weak Subjectivity](https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity/) _Vitalik Buterin_
- [A Proof of Stake Design Philosophy](https://medium.com/@VitalikButerin/a-proof-of-stake-design-philosophy-506585978d51) _Vitalik Buterin_

## Related topics {#related-topics}

- [Proof-of-work](/developers/docs/consensus-mechanisms/pow/)
