1. We use Whisper for messaging protocol and Ethereum database for data storage
2. Each market consist a pair of coin symbol and contract address as related content.
3. Each orderbook represents an market and is stored decentralized across multiple node.

## Update Order flow

```mermaid
graph LR
    A[Backend] -->|User Address 1| B(Token Symbol + Contract Address)
    A[Backend] -->|User Address 2| B
    B --> C{Protocol}
    C -->|Order 1| D[Order List]
    C -->|Order 2| E[Order List]
```

**Each orderbook is a red-black tree**

```
.
├── HOT/DAI
├── WETH/DAI
│   ├── PRICE1=>[Order1, Order2, ..., OrderN]                    Order List
│   ├── PRICE2=>[...]
│   └── PRICE3=>[...]
└── NOVA/HOT
    └── PRICE1=>[...]
        ├── PRICE1_1=>[...]
        └── PRICE1_2=>[...]
            ├── PRICE1_2_3=>[...]
            ├── PRICE1_2_3=>[...]
            └── PRICE1_2_3=>[...]

```

## Overall data flow

```mermaid
sequenceDiagram
    participant Frontend
    participant Backend
    participant Protocol
    loop Web3
        Frontend->Protocol: use Web3Provider to update account info
    end
    Backend->>Protocol: Get All Blocks
     Note over Backend,Protocol: User Addresses from Transactions
    Protocol-->>Backend: User Addresses
    Note over Backend,Protocol: Orders from (Coin Topic + User Address)
    Backend->>Protocol: Get All Orders
    loop Orderbook channel
        Backend->Protocol: Subscribe to receive new order
    end
    loop Websocket
        Frontend->Backend: use websocket to enter order, update trade...
    end
```

## Architecture overview

```mermaid.architecture

graph TB
    a["<br/><span class='mediumLabel'>Decentralized Exchange A</span><br/><br/>"]
    b["<br/><span class='mediumLabel'>Decentralized Exchange B</span><br/><br/>"]
    c["<br/><span class='mediumLabel'>Decentralized Swap</span><br/><br/>"]
    p["<br/><br/><span class='bigLabel'>Nova protocol</span><br/><br/><br/>"]
    e["<br/><span class='mediumLabel'>Ethereum</span><br/><br/>"]
    e1["<br/><span class='mediumLabel'>Other Chains</span><br/><br/>"]
    style p fill:#9f5069
    style a fill:#ff6f75
    style b fill:#ff6f75
    style c fill:#ff6f75
    style e fill:#ff6f75
    style e1 fill:#ff6f75
    a---p
    b---p
    c---p
    p---e
    p---e1
```

<hr/>

```mermaid.orderbook
graph LR
    subgraph OFF-CHAIN
        traders((Traders))
    end
    subgraph ON-CHAIN
        relayers((Relayers))
        multisig(Multi-Sig Wallet)
        approval_proxy(Approval Proxy)
        hybrid_ex(Hybrid Exchange)
        deposit_proxy(Desposit Proxy)
        swap_contract(Swap Contract)
    end
    class deposit_proxy,swap_contract dashedRect

    traders-- <span style='color:black'>Place order,<br/> Cancel order </span>-->relayers
    traders-- Approve /<br/> Disapprove -->approval_proxy
    relayers-- Call 'matchOrders'<br/>Send orders to match -->hybrid_ex
    multisig-- Ownership<br/><br/> -->approval_proxy
    hybrid_ex-- Call 'transferFrom'<br/> To move funds between traders  -->approval_proxy
    multisig-. Ownership<br/><br/> .->deposit_proxy
    swap_contract-.->approval_proxy
```

## Start orderbook protocol

We have already implement orderbook as ethereum protocol and added into cmd package, so you just need to go to devnet folder.

**Node1**

```bash
go run github.com/ethereum/go-ethereum/cmd/geth/ \
  --datadir node1/ --syncmode 'full' --port 30311 \
  --rpc --rpcaddr 'localhost' --rpcport 8501 --wsport 8545 \
  --rpcapi 'personal,db,eth,net,web3,txpool,miner,orderbook' --rpc --rpccorsdomain "*" \
  --targetgaslimit 94000000 --bootnodes 'enode://6cf1b4f4391d387904c48fc18403b49d0df0588e0a20511f0ff830de30867f3e12daaf6138ccfc2583b4b76b7b5bc0b7493e1ae1cb5ae951cd1f6221b147ec50@127.0.0.1:30310' \
  --networkid 1000 --gasprice '1' --allow-insecure-unlock -unlock '0xfcc9d477af8a7fe823ecb24bbd541e779aa72f31' \
  --password node1/password.txt --mine`
```

**Node 2**

```bash
go run github.com/ethereum/go-ethereum/cmd/geth/ \
  --datadir node2/ --syncmode 'full' --port 30312 \
  --rpc --rpcaddr 'localhost' --rpcport 8502 --wsport 8546 \
  --rpcapi 'personal,db,eth,net,web3,txpool,miner,orderbook' --rpc --rpccorsdomain "*" \
  --targetgaslimit 94000000 --bootnodes 'enode://6cf1b4f4391d387904c48fc18403b49d0df0588e0a20511f0ff830de30867f3e12daaf6138ccfc2583b4b76b7b5bc0b7493e1ae1cb5ae951cd1f6221b147ec50@127.0.0.1:30310' \
  --networkid 1000 --gasprice '0' -allow-insecure-unlock --unlock '0xbd89f54c601cfee303ed61b9b5ae3be21cc09d66' \
  --password node2/password.txt --mine
```

---

![orderbook1](assets/orderbook1.png)

---

![orderbook2](assets/orderbook2.jpg)
