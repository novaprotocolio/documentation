# Ranking Proof Of Stake Consensus

```mermaid
graph TB
    A(New epoch) -->|MOST-VOTED| B(First validator)
    B --> |RANKING-RANDOMIZATION| C(Second validator)
    B -.-> |Timeout| C1(Block reject)
    C -.-> |Timeout| C1
    C --> |Invalid block| C1
    C --> |COMMIT| D(Committed)
    D -.-> |Timeout| C1
    D --> |Insert block| I{ }
    I --> |Insertion fails| C1
    I --> |Insertion succeeds| E(Final committed)
    E --> |NEXT SLOT| R((Repeated 600 slots))
    C1 --> |CURRENT SLOT| R
    R --> |NEXT EPOCH| A

    class A,E roundedClass
```

### Consensus Protocol

Block is created by block producer, namely masternode. First block creator is most voted one and the other is randomized with ranking to help confirm a block with 2 signatures. With randomization, we can reducing risks comming from paired masternodes trying to commit malicous blocks.

1. First validator: we pick the most-voted masternode as the first validator, to produce the block.
2. Second validator: in Proof of Stake, much like Proof of Authority, one block creator done, next masternode in the ring may be dishonest and then confirm it and create a next block. That is why we will pick the second one based on factors: **random number ∗ ranking coefficient**. Ranking coefficient is based on the total amount of deposit tokens that a note is voted for.

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

   Let m be the number of masternodes, n be the number of slots in an epoch. in order to randomly generate the block verifier for the next epoch e + 1, the process is perform as the following steps:

   - **Step 1: Random number generation:** At the beginning of epcho e, each masternode $V_i$ will securely create an array of n + 1 special random numbers $Recommend_i$ = $[r_{i.1},r_{i.2},...,r_{i.n},\theta_{i}]$ where $r_{i,k} \in [1,...,m]$ indicating the recommendation of ordered list of block verifiers for the next epoch of $V_i$, and $\theta_i \in \lbrace -1,0,1 \rbrace$ is used for increasing the unpredictability of the random numbers.
     Then each masternode has to encrypt the array $Recommend_i$ using a secret key $SK_i$, say $Secret_i = Encrypt(Recommend_i, SK_i)$ as the encrypted array. Next each masternode forms a "lock" message that contains the encrypted array, signs off this message with its blockchain's private key along with the corresponding epoch index. By doing this, every masternode can check who created this lock message through ECDSA verification scheme. Finally each node sends their lock message with its signature and public key to the smart contract stored in blockchain, so that each masternode can collect and know the locks from all other masternodes.

   - **Step 2: Recovery Phase:** The recovery phase is for every node to reveal its previous lock message so that other nodes can get to know the secret array it has sent before. This happens when all masternodes have sent their lock messages or a certain timeout event occurs. Each masternode then opens its lock message by sending unlock message to the smart contract. Eventually a masternode has both locks and unlocks of others. If some electors are adversaries which might publish it's lock but intend not to send the corresponding unlock, other masternodes can ignore by simple setting all its random values to 1 as default.

   - **Step 3: Assembled Matrix and Computation Phase:** At the point of slot $n^{th}$ of the epoch e, the secret arrays $Secret_i$ will be decrypted by each masternode and return the plain version of $Recommend_i$. Each tuple of the first n numbers of each $V_i$ will be assembled as the $i^{th}$ column of an n x m matrix. All the last number $\theta_i$ forms a m x 1 matrix. Then each node will compute the block verifiers ordered list by some mathematical operations as explained below. The result is a matrix n x 1 indicating the order of block verifiers for the next epoch e + 1.

   The Second Masternode/Block Verifier: Each node computes the common array $v_2$ for the order of the block verifiers by the following steps as in Equation 1. Then, $v_2$ is obtained by modulo operation of element values of $v^{\prime}_2$ as in Equation 2.

   $$
   \begin{alignedat}{1}
   [v^{\prime}_{2}] =
   \begin{bmatrix}
   v_{2.1}^{e+1} \\
   v_{2.2}^{e+1} \\
   \vdots \\
   v_{2.n-1}^{e+1} \\
   v_{2.n}^{e+1}
   \end{bmatrix} =
   \begin{bmatrix}
   r_{1.1} & r_{2.1} & \dots & r_{m.1} \\
   r_{1.2} & r_{2.2} & \ddots & \vdots \\
   r_{1.3} & \ddots & \ddots & r_{m.3} \\
   \vdots & \ddots & r_{m-1.n-1} & r_{m.n-1} \\
   r_{1.n} & \dots & r_{m-1.n} & r_{m.n}
   \end{bmatrix}
   \begin{bmatrix}
   \theta_1 \\
   \theta_2 \\
   \theta_3 \\
   \vdots \\
   \theta_m
   \end{bmatrix} \;\;\;\;\;\;\;\;\;\;\;\;\;\;(1) \\ \; \\ \; \\
   [v_2] = [v^{\prime}_{2} \space\space mod \space m] =
   \begin{bmatrix}
   |v_{2.1}^{e+1}| & mod \space m \\
   |v_{2.2}^{e+1}| & mod \space m \\
   \vdots \\
   |v_{2.n}^{e+1}| & mod \space m
   \end{bmatrix} \;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;(2)
   \end{alignedat}
   $$

   The final randomized ranking list is as follow, with **t<sub>i</sub>** is the total deposit tokens of **masternode i**:

   $$
   r_i = \dfrac {t_i}{\textstyle\sum_{i=1}^n t_i} \\ \; \\ \; \\
   [v_2] =
   \begin{bmatrix}
   |v_{2.1}^{e+1}| \space \times \space r_1 \\
   |v_{2.2}^{e+1}| \space \times \space r_2 \\
   \vdots \\
   |v_{2.n}^{e+1}| \space \times \space r_n
   \end{bmatrix}
   $$

### Ranking algorithm

The algorithm is as following:

```python
LEADER = "leader"
VALIDATOR = "validator"
NUMBER_OF_DEPOSIT = "deposit"
NODE = "node"
SPEED = "speed"
RANKING = "ranking"
TRUST = "trust"
NUMBER_OF_EPOCH = "epoch"
LEADER_TEST = "leader_test"
VALIDATOR_TEST = "validator_test"

#init nodes
def init_nodes(number_of_node):
    nodes = []
    for i in range(number_of_node):
        nodes.append({})
        nodes[i][NODE] = i+1
        nodes[i][SPEED] = 0
        nodes[i][RANKING] = 0
        nodes[i][TRUST] = 0
        nodes[i][LEADER] = 0
        nodes[i][VALIDATOR] = 0
        nodes[i][NUMBER_OF_DEPOSIT] = 0
        nodes[i][NUMBER_OF_EPOCH] = 0
        nodes[i][LEADER_TEST] = 0
        nodes[i][VALIDATOR_TEST] = 0

    return nodes

#random speed of nodes
def random_nodes_speed(nodes):
    arr_speed = np.random.randint(low=5,high=10,size=len(nodes)).tolist()
    for i in range(len(arr_speed)):
        nodes[i][SPEED] = arr_speed[i]
    return arr_speed
In [4]:
#random number of epochs that masternode join in blockchain
def random_epochs_of_nodes(nodes, total_epoch):
    arr_epochs = np.random.randint(low=total_epoch/2,high=total_epoch,size=len(nodes)).tolist()
    for i in range(len(nodes)):
        nodes[i][NUMBER_OF_EPOCH] = arr_epochs[i]
    return arr_epochs

def random_leader(nodes):
    maximum = -1
    index_of_leader = -1
    arr_deposit = np.random.randint(low=50000,high=100000,size=len(nodes)).tolist()
    for i in range(len(arr_deposit)):
        if arr_deposit[i] > maximum:
            maximum = arr_deposit[i]
            index_of_leader = i
    nodes[index_of_leader][LEADER] += 1

def random_validator(nodes):
    maximum = -1
    index_of_validator = -1
    arr_validator = np.random.rand(len(nodes)).tolist()
    for i in range(len(arr_validator)):
        if arr_validator[i] > maximum:
            maximum = arr_validator[i]
            index_of_validator = i
    nodes[index_of_validator][VALIDATOR] += 1

def deposit_votes(nodes):
    deposit_vote_max = -1
    arr_deposit_vote = np.random.randint(low=50000,high=1000000,size=len(nodes)).tolist()
#     arr_deposit_vote = np.sort(arr_deposit_vote)[::-1]
    for i in range(len(arr_deposit_vote)):
        nodes[i][NUMBER_OF_DEPOSIT] = arr_deposit_vote[i]
        if arr_deposit_vote[i] > deposit_vote_max:
            deposit_vote_max = arr_deposit_vote[i]
    return arr_deposit_vote, deposit_vote_max

# chose a leader node by voting
def vote_leader_test01(nodes):

    #random array with length is number of canidate of leader
    arr_random = np.random.rand(len(nodes)).tolist()

    #select lead node
    index_of_leader = -1
    maximum = -1
    arr_leader = []
    for i in range(len(nodes)):
        multi = nodes[i][NUMBER_OF_DEPOSIT] * arr_random[i]
        arr_leader.append(multi)
        if multi > maximum:
            index_of_leader = i
            maximum = arr_leader[i]
    nodes[index_of_leader][LEADER_TEST] += 1
    return index_of_leader

# Trust for each masternode
def trust(nodes, arr_speed, total_epoch):
    trust_max = 0
    speed_arg = sum(arr_speed)/len(nodes)
    a,b,c,d = 0.25, 0.1, 0.3, 0.35
    for i in range(len(arr_deposit_vote)):
        trust = a*(nodes[i][NUMBER_OF_EPOCH]/total_epoch) + b*(
                nodes[i][SPEED]/speed_arg) + c*(
                nodes[i][LEADER]/nodes[i][NUMBER_OF_EPOCH]) + d*(
                nodes[i][VALIDATOR]/nodes[i][NUMBER_OF_EPOCH])
        nodes[i][TRUST] = trust
        if trust > trust_max:
            trust_max = trust
    return trust_max

# Ranking for each masternode
def ranking(nodes, arr_deposit_vote, trust_max, deposit_max):
    total_deposit = sum(arr_deposit_vote)

    a = 0.25
    for i in range(len(nodes)):
        vote_deposit_avg = arr_deposit_vote[i] / deposit_max
        trust_avg = nodes[i][TRUST] / trust_max
        nodes[i][RANKING] = a * vote_deposit_avg + (1-a) * math.sqrt(trust_avg)
        print(vote_deposit_avg, ":", trust_avg)

# Random validator base on Ranking of each node
def choose_validator_test(nodes, leader):
    index_of_validator = -1
    maximum = -1

    arr_random = np.random.rand(len(nodes)).tolist()
    for i in range(len(arr_random)):
        if nodes[i][RANKING] == 0:
            point = arr_random[i]
        else:
            point = math.sqrt(nodes[i][RANKING]) * math.pow(arr_random[i],4)

        if point > maximum:
            maximum = point
            if i != leader:
                index_of_validator = i

    nodes[index_of_validator][VALIDATOR_TEST] += 1
    return index_of_validator

# Init nodes and data nodes
nodes = init_nodes(1000)
total_epoch = 5000  # Total epoch in blockchain
arr_speed = random_nodes_speed(nodes)
random_epochs_of_nodes(nodes, total_epoch)
for i in range(total_epoch):
    random_leader(nodes)
    random_validator(nodes)
```

**Start running**

```python
arr_deposit_vote, deposit_max = deposit_votes(nodes)
trust_max = trust(nodes, arr_speed, total_epoch)
ranking(nodes, arr_deposit_vote, trust_max, deposit_max)

for i in range(len(nodes)):
    nodes[i][LEADER_TEST] = 0
    nodes[i][VALIDATOR_TEST] = 0
for i in range(100000):
    leader_test = vote_leader_test01(nodes)
    validator_test = choose_validator_test(nodes, leader_test)
nodes.sort(key=lambda x: x[RANKING])
```

![selection](assets/node_selection.png)

**With 1 000 000 nodes**  
![1 million node](assets/consensus_1m.png)

```mermaid
sequenceDiagram
    participant E1 as Epoch (K - 1)
    participant E2 as Epoch (K)
    E1->>E2: Create, broadcast, validate, sign the blocks for Epoch K
    E1->>E2: Randomly generate seeds, send message to Smart Contract
    E1->>E2: Checkpoint

```

Because the order of block creation masternodes is pre-determined for each epoch, random/arbitrary forks are hardly happened. As long as the number of attackers is less than $\frac 1 4$ the number of masternodes, block is finallized and no chance to create longer valid chain.

The random order of masternode array is then multiply with a rank array to get the final ordered list.
By applying ranking algorithm ( AI enhanced ) we can solve the most vexing limitation and increase TPS close to Visa.
