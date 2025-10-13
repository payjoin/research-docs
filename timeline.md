# Payjoin v3 Research Roadmap

## Project Name
Multi-Party Payjoins

## Project Timeline and Milestones

### Multiparty Payjoin (V3+)

Payjoin V1 and V2 enable a sender and receiver to interact to batch transactions to potentially save money and break common patterns that hurt everyone's privacy. Because only two peers interact, they still each know each others' inputs and outputs.

In contrast, a multiparty Payjoin protocol can coordinate transactions such that each participant maintains privacy even from a counterparty they send to or receive from.

More over, we plan to design a coalition formation as a generalization of  traditional order matching by enabling multiple parties to aggregate their orders, allowing for mutually beneficial, positive-sum outcomes.

We will measure the efficacy of the said protocol by building a suite of clustering software that will allow researchers and developers to measure the privacy of their transaction structure and subgraph.

Our team has been developing the Payjoin v2 protocol with Spiral Bitcoin Wizard Yuval Kogman of Spiral to be compatible with a future multiparty Payjoin v3 protocol that re-purposes the Payjoin v2 infrastructure and potentially maintains backwards compatibility.


Our development timeline for multiparty Payjoin is as follows:

- [x] 2025 Q3: Complete a production implementation of the Payjoin v2 protocol on mobile and stabilize the spec.
- [x] [receiver coordinated multi-sender 1-receiver] 2025 Jan-Apr: Release a brittle multiparty Payjoin V3 protocol with minimal privacy guarantees
- Proof of concept already completed thanks to the modularity of Payjoin Dev Kit: https://github.com/payjoin/rust-payjoin/pull/434
  - [x] Next four weeks adapting the API to be compatible with existing use of Payjoin Dev Kit [we adapted persistence from multiparty even though brittle multiparty was pruned for now]
  - [X] Port & release wallet fingerprint that came out of papers
    - Armin is crates.io Author https://github.com/arminsabouri/rust-wallet-fingerprints/

- 2025 Q4
- [ ] Develop a library that allows wallets to create transactions with their most minimal transaction fragment components while still being able to reason about the wallet's cost function.
- [ ] Define wallet optimization framework and cost function
	- Cost functions model the decision space available to Bitcoin wallets during transaction creation. Their primary goal is to account for user-defined liabilities, such as the loss of privacy and how these liabilities interact with confirmation times.
- [ ] Define a coalition formation mechanism to allow participants to join and leave a multiparty transaction while maintaining privacy based on preferences like privacy, timeliness, and fees. The output of this is a write up intended for a  forum like Delving or the bitcoin dev mailing list.
	- Take inspiration from JoinMarket's order book approach from a single taker to multiple takers per transaction https://github.com/payjoin/research-docs/blob/main/research-milestones.md#3-coalition-formation-protocol
- [ ] Define a NIP and reference code that allows Nostr clients to use Nostr relays via an OHTTP interface.
	- [ ] Modify the BIP-77 Directory service to opt-in to using nostr relays for durable storage. As defined in https://gist.github.com/arminsabouri/1d41501b4e5caa38870097f70ec0e604/

- 2026 Q1:
- [ ] Spec and Develop mutli party multi-stage payjoin in the semi-honest setting
- [ ] Create a simulation framework for evaluating our assumption for the multiparty transaction structures 
	- https://github.com/nothingmuch/fungi/tree/btsim
  - Consider drafting Bitcoin Improvement Proposal or similar to specify open protocol within Payjoin Dev Kit
- [ ] Brute force subtransaction privacy metrics calculation for the 2-party Payjoin space with multiple inputs and outputs
        - 2-party COULD be released to existing prod. integrations
        - Combine with UIH heuristic analysis
        - Replace output(s) with one or more that improves subset sum
        density (About a quarter of work. The hard part is writing the partitioning calculations. Some of this work was already started by Armin using set-partitions https://github.com/payjoin/cja-2 
        - Other optimization will need to be employed for larger transactions. This is not a problem we need to worry about in the 2-party case.
        - Subset sum density estimation for large sets planned for later
  - [ ] Extend the Payjoin subtransaction model privacy metrics calculation functions to be compatible with a multiparty transaction setting by using dynamic programming optimization techniques (stretch goal)

Q2 2026: 
- [ ] Release transaction graph based privacy metrics in their own library integrated into the Payjoin Dev Kit
   - [ ] Implement basic clustering heuristics and transaction graph indexing
   - [ ] Incorporate wallet fingerprinting
   - [ ] Define what privacy metrics to optimize for payjoin (v2 and multiparty)
   - [ ] Evaluate their efficacy
   - [ ] Optimize the v3 transaction structure
   - [ ] Evaluate the privacy of 2-party and multiparty payjoins
   - [ ] Develop additional case studies showcasing real-world application of the tool(s)
- [ ] Nailing down details of the transport layer for v3 transactions
   - [ ] Same for state machine replication
- [ ] Design and implement a OHTTP interface for Esplora
   - [ ] Spec and implement privacy preserving reads from electrum clients.
   
Q3-4 2026, 
- [ ] Spec and implement DOS protection for the existing mutli party payjoins
- [ ] Finish the remaining design details and start implementation of the  coalition formation protocol in the adveserial setting.
