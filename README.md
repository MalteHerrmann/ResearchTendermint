# Tendermint Core

As part of the interview process for Evmos, 
the following notes on the functionality and concepts of Tendermint Core
were written up. <br>
If you are wondering 
about the structure of the .md file itself, 
I am adhering to [semantic linefeeds](https://rhodesmill.org/brandon/2012/one-sentence-per-line/) 
as a means to improve tracking text changes with Git.


## Reference topics

 - What is Byzantine Fault Tolerance (BFT)? Why is BFT relevant in the context of blockchains
 - Understand what finality means in blockchains.
 - Explain and understand the tradeoffs of safety over liveness (i.e BFT vs Nakamoto Consensus chains). Why does this matter and what are the restrictions for design?
 - What is Synchrony, Asynchrony, Partial Synchrony? Why is it relevant to Tendermint?
 - Explain how an application communicates with the Tendermint Consensus Engine (ABCI). How does it relate to Cosmos and the Cosmos SDK?
 - What is ABCI++? Explain the differences with ABCI its problems/limitations that motivated the new ABCI++ specification. What are the new phases/methods introduced in the Consensus/block execution phase?

## General Info about Tendermint

"Tendermint in a nutshell": https://docs.tendermint.com/master/assets/img/tm-transaction-flow.258ca020.png

https://docs.tendermint.com/master/ <br>
https://docs.tendermint.com/master/introduction/what-is-tendermint.html

- blockchain application platform
  - comparable to a web-server for websites, 
  as Tendermint is "serving" blockchain applications

- Consists of two major components
  - Blockchain consensus engine (Tendermint Core)
  - Application Blockchain Interface ([ABCI](#application-blockchain-interface-abci))

- Software for state machine replication (SMR) for any 
deterministic, finite state machine, 
with the characteristic of being byzantine fault tolerant (BFT).

  - Finite State machine: (https://blog.markshead.com/869/state-machines-computer-science/)

    - A state machine is basically any black box, 
    which takes an input, 
    contains state variables, 
    which define the system state, 
    and changes these in response to the input.
    
    - Finite means, that the machine/algorithm cannot describe continuous state changes, 
    but rather describes a finite amount of possible states (unlike water levels).

    - Deterministic means, that the same result is always achieved 
    when given the same initial conditions 
    and the same requests
    between states.

  - State machine replication (https://link.springer.com/chapter/10.1007/BFb0042323)

    - State machine replication is a process, 
    especially useful for implementing distributed systems.

    - Distributed systems are defined as being processors, 
    which are physically and electrically isolated.

    - The process of voting 
    on the coordinated output of the distributed machines 
    is suited to real-time applications, 
    as opposed to "failure detection and retry"-systems, 
    which would take additional computing time if a failure or wrong information occurs on one of the machines.

- Consensus is the central principle 
through which it is guaranteed,
that all participating subsystems in a distributed system
receive the exact same information/transactions 
in the same order [[p.1, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- Classic SMR algorithms concerned themselves with
a small amount of connected machines,
that were often in the same network,
and all participants could be trusted 
[[p.1, BucKwoMil18]](https://arxiv.org/abs/1807.04938). 

- Blockchains are networks 
of hundreds or thounsands of nodes,
which can be in arbitrary locations,
and the participants do not know each other.
Additionally, nodes are usually only
connected to their peers, 
which is a subset of the total system. 
These peers communicate through a gossip protocol
[[p.1, BucKwoMil18]](https://arxiv.org/abs/1807.04938). 


### Motivation behind Tendermint

- Proof-of-Work algorithms such as the one used in Bitcoin 
have a large environmental impact,
as the security of the chain 
lies in solving a complex mathematical problem,
the complexity of which can be adjusted,
so that solutions to the problem are found, 
and therefore new blocks mined,
at a desired throughput rate. <br>
As more and better processors tackle this challenge,
and PoW blockchains in general become more widely used,
the environmental footprint of this operation severely increases. <br>
Proof-of-Stake algorithms, which Tendermint and others use,
require much less computational effort 
[[p.1, Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

- Before Tendermint, all blockchains had to include 
custom software to establish the peer-to-peer connections, 
broadcast transactions, 
handle the mempool, 
reach consensus, 
keep track of accounts, balances and contracts, 
etc. 

- large, interconnected and monolithic codebases 
are a bad practice in software design, 
because too many modules were interconnected
 and could not be used and developed separately.

- Modular approach to reduce strong coupling between subsystems

- Leave application details to blockchain developers and
only offer the base layers, 
which are concerned with networking and consensus.

### Possible exploits

- *Nothing at stake problem*: <br>
Network participants do not face a high enough incentive to not act maliciously
[[p.1, Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

- *Long range double-spending attacks*: <br>
A 2/3 majority of validators unbond their assets 
and sell their coins to others, 
who are not aware of any malicious activity. 
Then, they publish a fork of the blockchain, 
but have already made profit from the selling of the now double-spent coins.
[[p.4, Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).


- *Short range double-spending attacks*: <br>
This describes a blockchain fork,
which has happened at such a recent time,
that the malicious validators could not unbond their assets yet. 
In this case, the bonded assets of the validators can be slashed,
who participate in the fork. <br>
However, this means, that >1/3 of bonded assets are destroyed at once,
which is still damaging the network as a whole. <br>
By adjusting the bonding requirements to become a validator,
the probability for these kinds of attacks can be adjusted
[[p.4, Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).


## Predecessors of Tendermint

- Direct predecessors of Tendermint are the PBFT and DLS algorithms.
  
  - Tendermint has three communication steps per round like PBFT
  (proposal + two voting steps)
  
  - Tendermint uses different block proposers each round as in DLS

[[p.2, BucKwoMil18]](https://arxiv.org/abs/1807.04938)

- PBFT (Practical Byzantine Fault Tolerance) SMR [[p.X, ]](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/01/p398-castro-bft-tocs.pdf)

- DLS algorithm [[]](https://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf)


## Byzantine Fault Tolerance

- Concept to describe a distributed system, 
which continues to produce correct results, 
even if 1/3 of all participating subsystems 
are fraudulent or misbehaving in any other way 
(e.g. being broken, failing to connect, ...).

- BFT solves the Byzantine Generals Problem,
which describes a group of generals, who have a common mission (=well-being of the blockchain). 
They can only communicate with messages (=P2P communiation)
and some of the generals might be traitors (=malicious nodes)
or messengers might be captured (=timeout)
[[p.1, LamShoPea16]](https://www.microsoft.com/en-us/research/uploads/prod/2016/12/The-Byzantine-Generals-Problem.pdf)

## Application Blockchain Interface (ABCI)

https://docs.tendermint.com/master/introduction/what-is-tendermint.html <br>
https://github.com/tendermint/tendermint/tree/master/spec/abci/


- Interface to feed input from software 
written in any language 
to the Tendermint Core consensus engine.

- The implementation in Tendermint (Tendermint Socket Protocol (TSP))
satisfies this interface.

- Tendermint Core implements three socket connections 
between ABCI and applications
for the following purposes:
  
  - Validation of transactions, that are added to the mempool

  - Running block proposals in the consensus engine

  - Querying the application state

- Contains methods, 
which offer corresponding `Request` and `Response` message types 
in order to interact with the interface. 

- All messages and methods are defined in protocol buffers, 
which makes the interface agnostic to the programming language of the application, 
with which it communicates.


## Tendermint Consensus Process

https://docs.tendermint.com/master/assets/img/consensus_logic.e9f4ca6f.png

Tendermint achieves BFT SMR
by sequentially reaching consensus on new blocks,
that shall be appended to the canonical representation of the blockchain.
These blocks include transactions, 
which update the state variables of the system. <br>
The consensus is organized into voting rounds,
where for each round a new validator is chosen from the validator set
to propose a new block.
This is handled by a deterministic function (not at random),
which selects validators in a pattern relating to their bonded assets.
[[p.6 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

A single round consists of at least three and at most 5 steps: <br>
*Propose*, *prevote* and *precommit* happen in every round, 
whereas *commit* and *new height* only occur if consensus on a block was reached.
[[p.5 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

In the first round at a new block height 
(=after the last block was committed),
the proposer can freely choose 
which transactions from the mempool to include in the proposed block. 
The proposed block is forwarded to the validator's peer nodes.<br>
These check the proposed block (=value $v$) for validity
and sign a *prevote* message containing $id(v)$. 
If the prevote timeout expires before sending the message,
the node could not identify the block as valid,
or no proposal has reached the node,
`nil` is sent as the prevote value. <br>
When a node receives a block proposal and $2f+1$ prevote messages containing $id(v)$,
it sends a *precommit* message with $id(v)$. 
Again, in any other case (the node deems the block invalid, 
$2f+1$ prevotes not received,
or precommit timeout is expired), 
a precommit message containing `nil` is sent. 
If a node receives a block proposal as well as $2f+1$ precommit messages,
it decides, that this block will be committed
and signs a *commit* message. 
If the node has not received the actual block during the previous steps,
it is supplied with it during the *commit* step. 
All commit messages are broadcasted to the network.
When a node receives a 2/3 majority of commits for a specific block,
the CommitTime is written to the block header
and the *NewHeight* step is invoked. <br>
After commiting, a certain period of time is waited,
to include commit signings from slower validators. 
Then, the next block is proposed.<br>
[[p.1, 4, 7f. BucKwoMil18]](https://arxiv.org/abs/1807.04938)
[[p.4, 6-8 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

When a round terminates without consensus on a new block,
i.e. less than 2/3 majority has casted a commit vote,
a new round starts.
In order to achieve consensus when timeouts are the cause of non-consensus,
the timeout values are increased with each starting round.

*Commit* messages are special, 
because they are always broadcasted in any step or step.
If a node receives a 2/3 majority vote for commiting a block,
even though the node itself was only in the prevote round,
the commit stage is entered
[[p.8 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

![Consensus Logic [[Ten22]](https://docs.tendermint.com/master/assets/img/consensus_logic.e9f4ca6f.png)](./images/consensus_logic.png)

- TODO: Locking erläutern

### Notes on consensus mechanism

- Tendermint introduced a new termination mechanism,
to prevent endless rounds and locked nodes from blocking
[[p.5, BucKwoMil18]](https://arxiv.org/abs/1807.04938):

  - For each step in a round (proposal, pre-vote, pre-commit)
  there are defined timeouts, 
  after which `nil` is returned,
  if there was no decision.

  - Before the global stabililization time (GST),
  the timeouts are increased after each round, 
  if a sufficient value was not found before.

- Also, unlike in predecessor algorithms (PBFT, DLS),
does not include actual messages, 
when voting,
but rather cryptographic proof of the message contents
[[p.2-3, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- There is only one "mode", 
meaning that the same rounds are completed,
whether or not a valid block is proposed
and sufficient votes are collected
[[p.3, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- All communication is done within a specified time $\Delta t$
after a message is sent. Final time is $t_{end} = \max\{t_{message}, GST\} + \Delta t$,
where $GST$ is the Global Stabilization Time.
[[p.3, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- Through use of cryptographic signatures,
no "fake" messages from different senders can be generated 
[[p.4, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- Based on Validity Predicate-based Byzantine consensus,
which has three main properties [[p.4-5, BucKwoMil18]](https://arxiv.org/abs/1807.04938):

  - Agreement: Non-malicious nodes will always decide
  on the same values.

  - Termination: 
  All participating nodes will produce a decision 
  (either nil or pre-vote/pre-commit)

  - Validity:
  Application-specific --> e.g. in terms of blockchain 
  a block is not valid if it doesn't contain the hash of the last block.

- Tendermint (and other BFT algorithms) works, 
if not more than 1/3 of nodes are misbehaving 
(with or without malicious intent)
[[p.5, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- A fork (=two blocks at the same height get a 2/3 majority vote) can only happen,
if >1/3 of validators try to double sign.
[[p.4 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

- TODO: Mempool noch erläutern

- Messages with invalid signatures
are removed before entering the mempool 
[[p.4, BucKwoMil18]](https://arxiv.org/abs/1807.04938).

- If a block is not committed, 
the next round is started at the same block height

- In order to commit a block, all checks must be passed, 
as well as two stages of voting (pre-vote and pre-commit)

- All voting rounds need a 2/3 majority vote, 
in order to pass the proposed block on to the next stage

- "Polka" = more than 2/3 of the validator set pre-vote for the same block

- Locking rules, so that 
validators do not commit different blocks at the same block height.
The validator is locked to the block 
when pre-commiting a block.
It must pre-vote the block
and can only unlock
if there is a polka for that block.

- TODO: hier noch mehr Infos sammeln -> in
[[p.7 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

- In PoS applications (like all Cosmos based blockchains), 
2/3 of voting power 
and not 2/3 of actual validators 
have to reach consensus

- When using PoS applications, 
the validators can be forced
to bond the chain currency in a security deposit
which will be slashed when misbehaving
with regards to the consensus protocol.

- TODO: Explain Proof of Safety and Proof of Liveness from 
[[p.8 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

- The Tendermint PoS algorithm does not benefit those,
who try to claim a bigger share of the fees
by removing a certain amount of signatures:
[[p.9 Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

Expected benefit:
$\sum Fees * v_a * v_p * v_p$

Expected detriment:
$\sum Fees * v_a * v_p$

Because $v_i < 1$, the benefit will always be smaller than the detriment.


## Synchrony, Asynchrony, Partial Synchrony

Distributed systems can be classified into 
synchronous, asynchronous and partially synchronous systems.

Fully asynchoronous systems do not hold any assumptions about the time 
it takes to communicate between nodes
or the speed at which instructions can be processed at the receiving end (=computing power).<br>
Fully synchronous systems can guarantee, 
that messages are correctly communicated
and processed within a known timeframe.<br>
Partially synchronous systems can present hybrid forms of these two extremes,
where certain constraints are defined 
and freedom is given on the other side.<br>

The consensus algorithm in Tendermint 
can be considered **weakly** or **partially synchronous**,
because it assumes, that there is an upper bound $\Delta$,
after which all messages are delivered.
This describes the real-world phenomenon, 
that previously unknown latency will be present in the system,
but that this latency will not be endless,
if the network participants are not offline. <br>
This is implemented by means of several timeout values,
after which a `nil` message is signed,
if the node has not responded yet
[[p.5, Kwo14]](https://www.semanticscholar.org/paper/Tendermint-%3A-Consensus-without-Mining-Kwon/df62a45f50aac8890453b6991ea115e996c1646e).

FLP impossiblility describes the characteristic, that
out of the three properties safety, liveness and fault tolerance,
only two can be achieved. 
In globally distributed systems, like blockchains,
where communication can fail
and network participants do not necessarily trust each other,
fault tolerance **must** be given. <br>
This means, that blockchain algorithms can choose between implementing
safety or liveness.

- If validators are offline, lagging, etc. 
they can be skipped in Tendermint.
Any validator waits a certain, small amount of time
to receive a proposal block from the proposing validator
before jumping to the next voting round
and abandoning the currently proposed block.

## Implementation details

https://docs.tendermint.com/master/tutorials/go-built-in.html <br>
https://github.com/tendermint/tendermint/blob/95cf253b6df623066ff7cd4074a94e7a3f147c7a/spec/abci/abci.md <br>
https://docs.tendermint.com/master/assets/img/abci.3542de28.png


- Several methods are defined for a key-value store application

  - `Info`
  - `DeliverTx`
  - `CheckTx`
  - `Commit`
  - `Query`
  - `InitChain`
  - `BeginBlock`
  - `EndBlock`
  - `ListSnapshots`
  - `OfferSnapshot`
  - `LoadSnapshotChunk`
  - `ApplySnapshotChunk`

The ABCI comprises three primary types of socket connections:

1. Mempool connection: `CheckTx` <br>
Responsible for validating new transactions 
before being shared 
or included in a block. <br>
Called whenever a transaction is added to Tendermint Core <br>
Checks if the transaction is valid 
(wrong nonce, request format, signatures, ...).

2. Consensus connections: `InitChain`, `BeginBlock` -> [`DeliverTx`] -> `EndBlock` -> `Commit` <br>
Responsible for block execution. <br>
Upon starting a new blockchain, `InitChain` is called.
When a new block is decided on,
by reaching consensus with Tendermint Core,
this flow of function calls is executed. <br>
The actual transaction 
is sent to the application
using `DeliverTx`.
This method is called 
once for every transaction
included in the proposed block.
`Commit` is called 
to permanently store the transaction
on chain. <br>
The header of the next committed block
contains cryptographic traces of the execution
of `DeliverTx`, `EndBlock`, and `Commit`.

3. Info connections: `Info`, `SetOption`, `Query` <br>
Responsible for user queries and initialization <br>
Called in order to know
if a given key exists
and if it does,
what its stored value is.

Additionally, `Flush` is a method, 
which is called on every connection, 
and `Echo` serves debugging purposes. 
The different `Snapshot` methods
help new nodes, that enter the distributed system,
to get up to date with the blockchain state.
This has the advantage,
that not every node has to recompute all previous state changes.

## Application Architecture

https://docs.tendermint.com/master/app-dev/app-architecture.html

The basic signal flow 
when using a blockchain application, 
that is built with Tendermint,
is described here:

- End-user communicates with application
to create a transaction
or query some information from the blockchain.

- The application logic checks
whether the user request is valid
and reroutes the request
to the Tendermint Core RPC

- Tendermint then delivers
the transaction or query
to the ABCI

- The ABCI is responsible for 
the actual update of the chain state
or querying the desired information.


## Meaning of Tendermint for Cosmos SDK?

From https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#Determinism:

"[...] it is recommended that applications not be exposed to any external user or process except via the ABCI connections to a consensus engine like Tendermint Core. The application must only change its state based on input from block execution (BeginBlock, DeliverTx, EndBlock, Commit), and not through any other kind of request. This is the only way to ensure all nodes see the same transactions and compute the same results."

--> this states, that in order to guarantee 
the expected/desired deterministic behaviour 
on all participating machines,
it is mandatory
to only update the chain state 
using the ABCI, 
which is encapsuled by Tendermint Core.
No direct requests to the application layer
should be able to change state variables.

This is supported by [[p.4, BucKwoMil18]](https://arxiv.org/abs/1807.04938),
where it is stated,
that transaction verification should be handled by the application
that uses Tendermint.


## Random things

A message is not sent to the whole network,
but rather to the group of the sender's peers.
These forward these messages to their respective peer groups,
until it is fully spread across the network.
Hence the term gossiping [[ByiKwoBuc16]](https://open.spotify.com/episode/6eSJAtWT9btfca4mwYO7KT).

The incentive for validators 
to work towards the "good" for the blockchain
is not the economic sacrifice 
in terms of computing power
but rather the staked assets, 
which can be slashed for undesired behaviour 
[[ByiKwoBuc16]](https://open.spotify.com/episode/6eSJAtWT9btfca4mwYO7KT).

In Nakamoto consensus anybody can put up the work to be the next block proposer,
which makes this algorithm *censorship-resistant*. <br>
In opposition to that, 
PoS only allows a selection of validators to propose a block. 
This can therefore be classified as *non-censorship-resistant*
[[ByiKwoBuc16]](https://open.spotify.com/episode/6eSJAtWT9btfca4mwYO7KT).


Technically, one does not need to have a large amount of Bitcoin
in order to attack the blockchain. 
One just needs sufficient resources to build the longest chain (51% attack). <br>
To propose malicious or invalid blocks to a PoS network, 
the attacker has to be in control of at least 33% of the staked assets 
of the current validator set. <br>
New validator sets are only selected after a specified amount of time (*epoch*),
which means, that an attacker cannot immediately gain control of the necessary assets, 
without some time passing, where the suspicious behaviour could be noticed
[[ByiKwoBuc16]](https://open.spotify.com/episode/6eSJAtWT9btfca4mwYO7KT).

!(../images/block_structure.png)

### Block structure
A *block* contains a number of transactions,
which users want to execute on chain.
Additionally, there is block information contained, 
like block height, fees used, or commit time.
In order to easily verify the correctness of the block,
a number of cryptographic information is stored as well. <br>
This contains the signatures from all validators,
who have signed to commit this block, 
the block hash of the previous block,
as well as the *state hash*,
which is a representation of the account state. <br>
More hashes (*header hash*, *validation hash* & *transactions hash*) are generated 
from the header, signatures and transaction data included in the block.
In turn, these are hashed together into the *block hash* of the new block.<br>
These are [*Merkle tree root hashes*](https://en.wikipedia.org/wiki/Merkle_tree), 
which contain cryptographic trails of all subitems, 
that were hashed together.




