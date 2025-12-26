# Research Goals

Our research efforts can be catagorized into two broad categories:

1. Multi-party payjoins
2. Qualitative analysis of privacy metrics

## Multiparty Payjoin (V3+)

Payjoin V1 and V2 enable a sender and receiver to interact to batch transactions to potentially save money and break common patterns that hurt everyone's privacy. Because only two peers interact, they still each know each others' inputs and outputs.

In contrast, a multiparty Payjoin protocol can coordinate transactions such that each participant maintains privacy even from a counterparty they send to or receive from.

More over, we plan to design a coalition formation as a generalization of  traditional order matching by enabling multiple parties to aggregate their orders, allowing for mutually beneficial, positive-sum outcomes.

We will measure the efficacy of the said protocol by building a suite of clustering software that will allow researchers and developers to measure the privacy of their transaction structure and subgraph. **Payjoin is designed to combat clustering, but recent results in clustering research call this in to question**.

The protocol can be conceptualized as a two seperated phases:

1. Find your peers (Coalition Formation)
2. Coordinate your transaction (Transaction Coordination)

### Transaction Construction

**What are some of the constraints**
With `SIGHASH_ALL` all parties must sign the same transaction and then combine their signatures, or the transaction can't be broadcast on the bitcoin network. This means that all parties must come to agreement about what transaction to sign. If the total txout amount exceeds the total txin amount, the transaction will
not be valid. therefore addition of txouts must be restricted. Adding txout must be authorized, fair and accountable (wasabi solves this with keyed anonomous credentials and homomorphic value commitments) -- in the sense that txin funds must verifiably cover txout funds per user.

Lastly, txouts must not be linkable by other parties in the transaction, so txout restriction can't rely on knowing which inputs are related to the input.

In its first iteration we will be focusing on multiparty payjoins in the **semi-honest** setting. Where economically aproximate nodes will coordinate transactions construction over a confidential broadcast channel (directory). Peers trust the directory for data consistency and will use OHTTP to preserve their metadata privacy. Peers do not trust their peers with counter-party privacy. This protocol has safety guaranteed by virtue of peers not signing a final transaction that does not satisfy their payment intents. However, it does not protect against DOS attacks and as a result has poor liveness properties. i.e any byzantine participant can fail the protocol.

In the **byzantine setting** or "coinjoining-with-strangers", peers will establish agreement on the event log which allows us to detect and excluding byzantine participants. An intresting research questions here is: can we reach agreement on the event log in the presence of a dishonest majority given that saftey is free?

### Coalition Formation

The following protocol aims to generalize the type of order matching that JoinMarket supports, allowing multiple users' intents to be aggregated together. More generally this protocol allows incentive compatible peers to find each other to coordinate an instance of the transaction construction protocol in a privacy preserving manner.

## Qualitative analysis of privacy metrics

An analysis framework for the Bitcoin transaction graph to evaluate the privacy properties of both simulated and existing privacy protocols -- but more generally analyze any sub-graph of the transaction graph. This framework will likely serve as a dependency for the projects above.

We want to measure the quality of transaction construction protocols (coinjoins / payjoins) using the latest research in wallet clustering and subtransaction metrics. The goal is to provide a suite of reusable privacy heuristics that can be used to evaluate the privacy of a transaction construction protocol. These tools are most likely already in use by the chainanalysis firms behind closed source products. We want to make these tools open source and accessible to the research and development community. Furthermore, no framework exists (blockci is the closest) that allows for infromation to be composed to develop more refined heuristics. Latest work in cluster demonstrates how much worse privacy gets when we apply external information to the transaction graph.

Lastly, latest work in Coinjoins only quantify privacy for a single transaction and does not consider the transaction (sub)-graph however latest work intersection attacks demonstrates how pre and post mix activity can degrade privacy (generally intersection attacks Goldfeather et al).

## Notes on Tx Graph structure

The research direction is to treat on-chain privacy as an emergent property of **transaction-graph structure**, rather than as a per-transaction heuristic, and to analyze that structure using tools from random graphs, expanders, superconcentrators, and random walks. Dense transaction construction (e.g. radix CoinJoins) provides strong *local* ambiguity, while verifiable randomness in peer and coin selection induces *global* graph properties that drive rapid mixing and many plausible paths. Entropy captures the size of anonymity sets, but edge-differential-privacy–style parameters capture their **robustness** to information revelation; the open problem is interpreting these parameters meaningfully when the algorithm and data are fixed and ( \varepsilon ) must be estimated rather than chosen. Key open questions include how fragile large anonymity sets are under realistic edge deletions, how degree sequences evolve over time in the randomized subgraph, and how much information leaks through revealed preferences. Simulations can cover most of the empirical work: generating transaction graphs under different randomized selection rules, estimating degree distributions and path counts, measuring mixing times and stationary distributions, stress-testing edge removals and deanonymization events, and validating whether the resulting graphs behave like expanders or superconcentrators with high probability.



## Tangential Research

Light clients and privacy preserving reads from electrum clients.
Mobile and/or resource constraint devices often use light clients to scan the chain and sync their wallets. A privacy concious user of this catagory could put into alot of resources and energy to preserving their privacy however reveal information about them selves during the last mile. i.e reading from the chain.

This problem breaks up into two subproblems:

1. Privacy preserving reads from electrum clients (PIR or Oblivious message retrieval (OMR))
2. Protecting network privacy with OHTTP
