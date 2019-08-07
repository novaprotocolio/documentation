## Introduction

### Design

Novalex is a public EVM-compartible blockchain that allows you to create blocks within 1 second, with some advantanges:

- allowing tokens holders vote for the block producers
- randomization with ranking for security guarantees
- built-in orderbook engine as a protocol

We propose Ranking Proof-Of-Stake consensus, a Proof-Of-Stake based blockchain with voting mechanisim and ranking algorithm. Top 68 block producers are selected by continous voting of NOVA token holders. Each Nova token holder is given the opportunity to give 80 votes, corresponding to power of staked tokens.

### Novalex incentivization

Novalex is produced and maintained by a set of masternodes in a consistent manner through the Novalex consensus. These masternodes are full nodes that hold NOVA. For a coin holder to become a masternode, it must hold a mininum amount of coin and be one of the most voted candidates in the system. The voting phase is credited through a Voting Dapp that allows coin-holders to send NOVA through the smart contract.

Double validation with ranking is added to decrease the probability of having invalid blocks in the blockchain.

After depositing 80 000 NOVA to the Voting Smart Contract, the the depositors are listed as masternode candidates in the Voting DApp, which allows coin-holders to vote for them by sending NOVA to the smart contract.
The list is sorted dynamically based on voted coins. We track CPU/Memory as workload, number of signed block as performance, last signed block as last activity, then candidate is chosen based on the performance factor. All masternodes have to race for their positions so that all weak nodes eventually will be eliminated. Masternode is
locked for 1 month after demoted.

After 600 blocks, a checkpoint block is created, the masternode who takes turn in the circular has to scan all of the created blocks in the epoch to count number of signatures.

- 60% of reward called infastructure reward goes to masternodes
- staking reward: 30% goes to the pool of all voters for that Masternode which is shared proportionally based on the token stake.
- 10% goes to development funding account.

Novalex has a built-in orderbook engine so that relayers can launch a decentralized exchange without maintaining an off-chain database.
