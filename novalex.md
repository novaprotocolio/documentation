## Introduction

### Design

Novalex is a public EVM-compartible blockchain with the following advantages:

- low transaction fee
- fast confirmation time
- double validation
- randomization for security guarantees

We propose Delegated Proof-Of-Stake consensus, a Proof-Of-Stake based blockchain with fair voting mechanisim.
It allows attempting consensus in a permissionless manner: anyone can join the protocol execution. EVM proposed serveral significant enhancements compared to Bitcoin, including Smart Contracts.
Some bottlenecks of classic blockchains:

1. Efficiency: Existing blockchains do not scale well to handle a large transaction volume. (10 TPS)
2. Confirmation times: a Bitcoin block requires 5 subsequent blocks following it so that it can be confirmed.
3. Fork generation: double validation, randomization, fast confirmation time for finallity or rebase.

### Novalex architecture

Novalex is produced and maintained by a set of masternodes in a consistent manner through the Novalex consensus. These masternodes are full nodes that hold NOVA. For a coin holder to become a masternode, 2 requirements must be satisfied:

1. The coin-holder must hold at least an minimum required amount of coin.
2. The coin-holder must be one of the most voted masternode candidates in the system. The voting phase is credited through a Voting Dapp that allows coin-holders to send NOVA through the smart contract.

Double validation to decrease the probability of having invalid blocks in the blockchain.

After depositing 50 000 NOVA to the Voting Smart Contract, the the depositors are listed as masternode candidates in the Voting DApp, which allows coin-holders to vote for them by sending NOVA to the smart contract.
The list is sorted dynamically based on voted coins. We track CPU/Memory as workload, number of signed block as performance, last signed block as last activity. All masternodes have to race for their positions so that all weak nodes eventually will be eliminated.
Locked for 1 month after demoted.

After 900 blocks, a checkpoint block is created, the masternode who takes turn in the circular has to scan all of the created blocks in the epoch to count number of signatures.

- 40% of reward called infastructure reward goes to masternodes
- staking reward: 50% goes to the pool of all voters for that Masternode which is shared proportionally based on the token stake.
- 10% goes to a special account controlled by the Masternode Foundation, which is run by Novalex company initially.

### Consensus Protocol

1. Double validation: block is created by block producer, namely masternode, but with DV, we simple require. One block creator and the other is block verifier to confirm with 2 signatures. With randomization, we can reducing risks comming from paired masternodes trying to commit malicous blocks. Only require 2 signatures, the confirmation time is short
2. Single validation: in Proof of Stake, much like Proof of Authority, one block creator done, next masternode in the ring may be dishonest and then confirm it and create a next block.

   $$
   [v_1]=
   \begin{bmatrix}
   V_{1.1}^e \\
   V_{1.2}^e \\
   \vdots \\
   V_{1.n-1}^e \\
   V_{1.n}^e
   \end{bmatrix}
   $$

   Special random number $Recommend_i$ = $[r_{i.1},r_{i.2},...,r_{i.n},\theta_{i}]$ where $r_{i,k} \in [1,...,m]$

3. Randomization: each masternode will create an array of n+1 special random number Recommendi =
