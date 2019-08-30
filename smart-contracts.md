## Nova Protocol

> Nova Protocol is an open-source framework for building decentralized exchanges on Ethereum.

Nova is designed for developers looking to build decentralized exchanges without having to deal with the complexity and expense of designing, deploying, and securing their own smart contracts.

## Features

Nova 1.0 contains a single exchange contract called `HybridExchange.sol` with the following attributes:

- No order collision
- No possibility of front-running
- Accurate market orders
- Ability to collect fees as a percentage of the traded assets
- Allows asymmetrical maker/taker fee structure, rebates, discounts
- Multiple settlement models:
  - Wallet to wallet mode
  - Contract-deposit mode (supports ETH)
- Highly optimized gas usage

## Installation

```bash
yarn
```

To build json ABI files:

```bash
yarn compile
```

## Tests & deploy

```bash
yarn coverage
yarn deploy --network development
```

## Deploy new token

```bash
NAME=TriToken SYMBOL=Tri yarn deploy-token
```

---

![smart-contracts](assets/smart-contract.jpg)

## Functions

### What exactly does a Relayer do?

Relayers provide a marketplace for traders to buy/sell tokens. They use Nova Protocol to process and complete the transactions.

Relayers perform the following actions:

- Collect trader orders (off-chain)
- Manage orders and perform Price-Time Priority matching
- When orders match, pack the details into transactions
- Send transactions to blockchain to confirm them on-chain
- When the blockchain completes the trade, notify the traders

## Concept of Smart Contracts - 01

https://ieeexplore.ieee.org/abstract/document/7467408

Nick Szabo introduced this concept in 1994 and defined a smart contract as “a computerized transaction protocol that executes the terms of a contract” [44]. Szabo suggested translating contractual clauses (collateral, bonding, etc.) into code, and embedding them into property (hardware, or software) that can self-enforce them [45], so as to minimize the need for trusted intermediaries between transacting parties, and the occurence of malicious or accidental exceptions.

Within the blockchain context, smart contracts are scripts stored on the blockchain. (They can be thought of as roughly analogous to stored procedures in relational database management systems [46].) Since they reside on the chain, they have a unique address. We trigger a smart contract by addressing a transaction to it. It then executes independently and automatically in a prescribed manner on every node in the network, according to the data that was included in the triggering transaction. (This implies that every node in a smart contract-enabled blockchain is running a virtual machine (VM), and that the blockchain network acts as a distributed VM.)

Smart contracts allow us to have general purpose computations occur on the chain. Where they excel however, is when they are tasked with managing data-driven interactions [47] between entities on the network. Let us unpack this statement with an example. Consider a blockchain network where Alice, Bob, and Carol participate, and where digital assets of type X and Y are being traded. Bob deploys a smart contract on the network that defines: (a) a “deposit” function allowing him to deposit units of X into the contract, (b) a “trade” function that sends back 1 unit of X (from the contract’s own deposits) for every 5 units of Y it receives, and (c) a “withdraw” function that allows Bob to withdraw all the assets that the contract holds.

Note that the “deposit” and “withdraw” functions are written so that only Bob (via his key) can call them, because this is what Bob decided, and also what makes sense for our example; they could have been written so that they can be called successfully by any user on the network.

Bob sends a transaction to that smart contract’s address, calling its “deposit” function and moving 3 units of X to the contract. This transaction is recorded on the blockchain. Alice, who owns 12 units of Y, then sends a transaction that moves 10 units of Y to the contract’s “trade” function, and gets back 2 units of X. This transaction is also recorded on the blockchain. Bob then sends a signed transaction to the contract’s “withdraw” function. The contract checks the signature to make sure the withdrawal is initiated by the contract’s owner, and transfers all of its deposits (1 unit of X, and 10 units of Y) back to Bob.

Let us observe the following:

- The contract has its own state and can take custody over assets on the blockchain [48]. We say that a contact has its own account on the blockchain, and the blockchain supports an account-based model [49]. In the example above, it can hold assets X and Y. (If we go back to the shared database model, a contract is a separate “user”/entity that can own, delete, and create rows.)

- The contract allows us to express business logic in code; “will trade 1 unit of X for every 5 units of Y received”.

- A properly written smart contract should describe all possible outcomes of the contract. For instance, the “trade” function above may be written so as to reject offers that bring in quantities of Y that are not multiples of 5; i.e. an offer of 12 units of Y will be rejected. Or the function may be written so as to parse the incoming offer as the biggest multiple of 5 (that will be traded without issues) and a remainder (that will be returned). An offer of 12 units of Y then, would return 2 units of X and 2 units Y to the sender.

- The relationship that Bob wishes to establish with his counterparties is one driven by data [47]. A transaction after all is a signed data structure indicating a transfer of value [12]. Bob deploys a smart contract that effectively says, “if you send this contract this data (5 units of Y), here’s how it will respond (1 unit of X)”.

- A smart contract is triggered by messages/transactions sent to its address.

- A smart contract is deterministic; the same input will always produce the same output. If one writes a non-deterministic contract, when it is triggered it will execute on every node on the network and may return different random results, thus preventing the network from reaching consensus on its execution result. In a properly built blockchain platform, writing non-deterministic smart contracts is either impossible (by forcing the contract developers to use a programming language that does not have any non-deterministic constructs [50]), or it is possible but an attempt to deploy such a contract on the network will be rejected [38].

- A smart contract resides on the blockchain, and as such its code can be inspected by every network participant.

- Since all the interactions with a contract occur via signed messages on the blockchain, all the network participants get a cryptographically verifiable trace of the contract’s operations.

A blockchain that supports Bitcoin-style transactions enables asset transfers between counterparties that do not trust each other. A blockchain that supports smart contracts however, takes this further and allows for multi-step processes (or more generally: interactions) to occur between mutually distrustful counterparties. The transacting entities get to (a) inspect the code and identify its outcomes before deciding to engage with the contract, (b) have certainty of execution since the code is already deployed on a network that neither of them controls fully, and (c) have verifiability over the process since all the interactions are digitally signed. The possibility of a dispute is eliminated (when all possible outcomes are accounted for) since the participants cannot disagree over the final outcome of this verifiable process they engaged in.

Smart contracts operate as autonomous actors, whose behavior is completely predictable. As such they can be trusted to drive forward any on-chain logic that can be expressed as a function of on-chain data inputs, provided that the data they need to manage is within their own reach (in the example above, the contract wouldn’t be able to trade assets that it did not own).

We close this section by noting that smart contracts also give rise to the concept of “decentralized autonomous organizations” (DAOs), entities on the blockchain whose behavior may be modified, if a certain process that is encoded in the contract is followed. As described in [51], the simplest example is that of a smart contract that calls another contract by address to perform its main function. This address resides on the mutable part of the contract’s internal database. The contract also carries a list of members, addresses (public keys) that get to vote on its behavior. A rule can be included in the contract so that if a majority of those voters vote in a certain way, the contract will modify its behavior by calling the address that received the majority of the votes to execute its main function.

## Concept of Smart Contracts - 02

https://github.com/ethereumbook/ethereumbook/blob/develop/07smart-contracts-solidity.asciidoc#contract_reg

The term smart contract has been used over the years to describe a wide variety of different things. In the 1990s, cryptographer Nick Szabo coined the term and defined it as “a set of promises, specified in digital form, including protocols within which the parties perform on the other promises.” Since then, the concept of smart contracts has evolved, especially after the introduction of decentralized blockchain platforms with the invention of Bitcoin in 2009. In the context of Ethereum, the term is actually a bit of a misnomer, given that Ethereum smart contracts are neither smart nor legal contracts, but the term has stuck. In this book, we use the term “smart contracts” to refer to immutable computer programs that run deterministically in the context of an Ethereum Virtual Machine as part of the Ethereum network protocol—i.e., on the decentralized Ethereum world computer.

Let’s unpack that definition:

### Computer programs

Smart contracts are simply computer programs. The word “contract” has no legal meaning in this context.

### Immutable

Once deployed, the code of a smart contract cannot change. Unlike with traditional software, the only way to modify a smart contract is to deploy a new instance.

### Deterministic

The outcome of the execution of a smart contract is the same for everyone who runs it, given the context of the transaction that initiated its execution and the state of the Ethereum blockchain at the moment of execution.

### EVM context

Smart contracts operate with a very limited execution context. They can access their own state, the context of the transaction that called them, and some information about the most recent blocks.

### Decentralized world computer

The EVM runs as a local instance on every Ethereum node, but because all instances of the EVM operate on the same initial state and produce the same final state, the system as a whole operates as a single "world computer."

## Concept of Smart Contracts - 03

https://blockgeeks.com/guides/smart-contracts/#Smart_Contracts_are_Awesome

### Introduction

One of the best things about the blockchain is that, because it is a decentralized system that exists between all permitted parties, there’s no need to pay intermediaries (Middlemen) and it saves you time and conflict. Blockchains have their problems, but they are rated, undeniably, faster, cheaper, and more secure than traditional systems, which is why banks and governments are turning to them.

In 1994, Nick Szabo, a legal scholar, and cryptographer, realized that the decentralized ledger could be used for smart contracts, otherwise called self-executing contracts, blockchain contracts, or digital contracts. In this format, contracts could be converted to computer code, stored and replicated on the system and supervised by the network of computers that run the blockchain. This would also result in ledger feedback such as transferring money and receiving the product or service.

### What are Smart Contracts?

Smart contracts help you exchange money, property, shares, or anything of value in a transparent, conflict-free way while avoiding the services of a middleman.

The best way to describe smart contracts is to compare the technology to a vending machine. Ordinarily, you would go to a lawyer or a notary, pay them, and wait while you get the document. With smart contracts, you simply drop a bitcoin into the vending machine (i.e. ledger), and your escrow, driver’s license, or whatever drops into your account. More so, smart contracts not only define the rules and penalties around an agreement in the same way that a traditional contract does, but also automatically enforce those obligations. If you are looking for a more detailed walkthrough of smart contracts please check out our blockchain courses on smart contracts.

### What is our Smart Contract give you?

**Autonomy** – You’re the one making the agreement; there’s no need to rely on a broker, lawyer or other intermediaries to confirm. Incidentally, this also knocks out the danger of manipulation by a third party, since execution is managed automatically by the network, rather than by one or more, possibly biased, individuals who may err.

**Trust** – Your documents are encrypted on a shared ledger. There’s no way that someone can say they lost it.

**Backup** – Imagine if your bank lost your savings account. On the blockchain, each and every one of your friends has your back. Your documents are duplicated many times over.

**Safety** – Cryptography, the encryption of websites, keeps your documents safe. There is no hacking. In fact, it would take an abnormally smart hacker to crack the code and infiltrate.

**Speed** – You’d ordinarily have to spend chunks of time and paperwork to manually process documents. Smart contracts use software code to automate tasks, thereby shaving hours off a range of business processes.

**Savings** – Smart contracts save you money since they knock out the presence of an intermediary. You would, for instance, have to pay a notary to witness your transaction.

**Accuracy** – Automated contracts are not only faster and cheaper but also avoid the errors that come from manually filling out heaps of forms.

## Life Cycle of a Smart Contract

Smart contracts are typically written in a high-level language, such as Solidity. But in order to run, they must be compiled to the low-level bytecode that runs in the EVM. Once compiled, they are deployed on the Ethereum platform using a special contract creation transaction, which is identified as such by being sent to the special contract creation address, namely 0x0 (see [contract_reg]). Each contract is identified by an Ethereum address, which is derived from the contract creation transaction as a function of the originating account and nonce. The Ethereum address of a contract can be used in a transaction as the recipient, sending funds to the contract or calling one of the contract’s functions. Note that, unlike with EOAs, there are no keys associated with an account created for a new smart contract. As the contract creator, you don’t get any special privileges at the protocol level (although you can explicitly code them into the smart contract). You certainly don’t receive the private key for the contract account, which in fact does not exist—we can say that smart contract accounts own themselves.

Importantly, contracts only run if they are called by a transaction. All smart contracts in Ethereum are executed, ultimately, because of a transaction initiated from an EOA. A contract can call another contract that can call another contract, and so on, but the first contract in such a chain of execution will always have been called by a transaction from an EOA. Contracts never run “on their own” or “in the background.” Contracts effectively lie dormant until a transaction triggers execution, either directly or indirectly as part of a chain of contract calls. It is also worth noting that smart contracts are not executed "in parallel" in any sense—the Ethereum world computer can be considered to be a single-threaded machine.

Transactions are atomic, regardless of how many contracts they call or what those contracts do when called. Transactions execute in their entirety, with any changes in the global state (contracts, accounts, etc.) recorded only if all execution terminates successfully. Successful termination means that the program executed without an error and reached the end of execution. If execution fails due to an error, all of its effects (changes in state) are “rolled back” as if the transaction never ran. A failed transaction is still recorded as having been attempted, and the ether spent on gas for the execution is deducted from the originating account, but it otherwise has no other effects on contract or account state.

As mentioned previously, it is important to remember that a contract’s code cannot be changed. However, a contract can be “deleted,” removing the code and its internal state (storage) from its address, leaving a blank account. Any transactions sent to that account address after the contract has been deleted do not result in any code execution, because there is no longer any code there to execute. To delete a contract, you execute an EVM opcode called SELFDESTRUCT (previously called SUICIDE). That operation costs “negative gas,” a gas refund, thereby incentivizing the release of network client resources from the deletion of stored state. Deleting a contract in this way does not remove the transaction history (past) of the contract, since the blockchain itself is immutable. It is also important to note that the SELFDESTRUCT capability will only be available if the contract author programmed the smart contract to have that functionality. If the contract’s code does not have a SELFDESTRUCT opcode, or it is inaccessible, the smart contract cannot be deleted.
