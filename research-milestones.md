# Research Plan Meeting Notes

The following are notes taking during the research plan meeting. This is a living document and will be updated as we progress. The goal is to convert this document into a list of a achievable milestones with a rough timeline.

## 1. Foundation for Version 3 / multiparty payjoins

### 1.1 PSBT Join Semi-Lattice Type

We plan to define, spec and implement a monotonic set type for an underspecified PSBT. We will model these proto-transactions as a join semi-lattices(lattices for short). Lattices are a good fit for collaborative transaction building as we often know some of the transaction components but others are contingent on the other participants. But we can still reason about a wallet's cost function and optimization framework (discussed below). The goal is to reason about unordered internal transaction states and later apply sorting rules before finalization and signing. This design avoids assumptions about ordering and is intended purely for internal representation. Meaning we don't need to worry about a serliazable format.

We will also implement traits over rust-bitcoin types. E.g: `HasAmount`, `HasSpk`, etc. These traits provide common interfaces across partially specified transaction objects.

### 1.2 Cost Function and Optimization Framework

We plan to design a cost modeling and optimization framework that quantifies time-dependent liabilities and/or deadlines, input-specific privacy-related costs but more generally any arbitrary cost metrics in satoshi terms. This is foundational work that allows us to reason about how and if a wallet will collaborate with others in a coalition to form a collaborative transaction.

#### Objective

Model the value of a proto-transaction based on:

`Cost(proto_tx, per_input_cost, per_output_liability, proposed_fee_rate, other_global_network_params) = satoshis`

In other words, given the current state of the wallet in terms of per input cost and per output liability, what positive satoshi value does the wallet get from contributing to thisproto-transaction?

#### Open Questions

* Is a piecewise-linear model sufficient for representing trade-offs?
* How should we model uncertainty in fee estimates or privacy valuations?

## 2. Optimize Transaction Creation for Specific Privacy Metrics

We will evaluate our cost function over the entire wallet state but optimize for specific privacy metrics.

### 2.1 Privacy Metrics

Couple metrics to consider but non-exhaustive:

#### a. Liquidation Metric

Simulate a wallet-wide liquidation transaction and compute privacy loss metrics.

* Determine how many transactions are needed to fully liquidate without severe linkability.
* Measure degradation in anonymity across these transactions.

#### b. Sybil Resistance / Graph Topology

Consider the transaction graph of your counter parties. Evaluate the cost function on inputs that are not in the control of the user wallet.
If there are multiple inputs that are "worth" the same then decide which one based on a random lottery using the latest block hash as a random beacon.

#### c. Subset Sum Density

Quantify the number of possible interpretations for a given input output decomposition.

### 2.2 Evaluation Framework

We plan to develop a clustering toolkit to evalute the efficacy of the privacy metrics (related and maybe dependent on 4. below).

* Simulate created transaction based on various different parameters to the cost function.
* Use real Bitcoin transaction graphs to validate privacy and cost metrics.
* Benchmark heuristic outcomes across different wallet strategies.

## 3. Coalition Formation Protocol

TODO: What are the different steps here?

We will design and implement a collaborative transaction protocol. Briefly, the following protocol generalizes traditional order matching by enabling multiple parties to aggregate their orders, allowing for mutually beneficial, positive-sum outcomes. Participants will join a proposed coalition depending on the cost-benefit evaluation outcomes.

## 4. Quantitative Graph Analysis

Develop an analysis framework for the Bitcoin transaction graph to evaluate the privacy properties of both simulated and existing privacy protocols -- but more generally analyze any sub-graph of the transaction graph. This framework will likely serve as a dependency for the projects above, providing a suite of reusable privacy heuristics.

TODO: Add a Gantt chart here.
