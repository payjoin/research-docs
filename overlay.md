# Notes on overlay networks:

Overlay networks assume that nodes can draw "random peers".
In a dynamic byzantine setting the peer sampling itself can be become a target.
In a dynamic setting there is churn.
Churn = set of active nodes over time. 

Local views over the set of peers must evolve as churn happens.
Small local views can be maintainable in the absence of malicious peers. And global membership view in some setting is intractable.

> Faulty nodes may attempt to skew the system-wide distribution, as well as the individual local view of a given node.

i.e poisoning the node is entries of an honest peer (eclipse attack).

The question is how can peers uniformally choose peer samples and can this be process be efficient.

[Brahms](https://dl.acm.org/doi/pdf/10.1145/1400751.1400772) provides sub linear solution that converges to uniform random over time. 

Computing a uniform random sample of all nodes is essential overlay networks -- this is done by the union of their local views. Otherwise
two partitions will remain disconnected perpetually. 

Overlays need to be resilient to adversaries that chose to over represent the faulty node ids of honest nodes.
They do this by either flooding correct nodes with faulty ids or skew the pull responses of correct nodes to have fault ids. 

Brahms can has two key components: a local uniform sampling  of the set of ids that traversed that node. And a gossip component that spreads known ids to peers.

The sample layer essentially treats the ids observed via gossip with unknown vias and then uses a extracts uniform random samples from the biased sample via min-wise independent permutations -- aka hash.

The membership layer is then a union of the evolving view and the sample list. This prevents partitions to a high probability even if a linear fraction is Byzantine.
More concretely the view is a randomly chosen from the sample layer coming from pulls (from peers) and pushed ids (new peers).

If an advesary is successful with superseding a view with more fault than correct nodes then future pulls will lead to more faulty nodes.
Historical samples are used in Brahm to overcome these attacks. Whereby a portion of the view reflects previously learned ids. 

## Overlay BB

Typical byzantine consensus cannot tolerate f > n / 2. However, a byzantine majority can be achieved if we use byzantine broadcast instead of traditional consensus algos.

## How this relates to BFT MP PJ

A permissionless multiparty protocol requires a peer discovery and communication layer where any participant can join without prior coordination. However, such networks are inherently vulnerable to Sybil attacks, and we cannot assume classical BFT conditions such as $n≥3f+1$.
Peers join the network by presenting a BIP-322 proof and enrolling a online key. A BIP-322 proof bulletin board is public infrastructure (directory) and used for bootstraping.

Peers cannot maintain a full membership set (fully connected graphg). And Not all peers will be reachable.
Some peers will advertise as listening peers. These peers maintain open connections. Non-listening peers will make short lived connections to listening peers. Listening peers will desiminate information. 

When a tx construction protocol bootstraps, should aim to connect to their immediate listening counterparties. In essence, a new overlay network or subnetwork forms.

Critical question: Should a node desiminate information for a tx construction session they are not a part of?

Additionally privacy preserving transport is difficult on mobile clients.
The peer list can be limited to anyone that proves ownership over a UTXO; however, byzantine node can just have many utxos and flood the network with bad node ids.

UTXOs provide a scarce on-chain resource, but they do not map 1:1 to peers, since an adversary can split funds across many outputs. Therefore, UTXOs are used as a cost signal and rate-limiting mechanism rather than as identities.

To ensure efficient dissemination of peer information, nodes use anti-entropy and set reconciliation protocols to synchronize ownership proofs and advertisements. This allows the network to converge on a shared view of available peers without requiring global broadcast or trusted infrastructure.

The worst outcome for a correct node is to have a view of mainly byzantine nodes and be percived as a ommiting peer (ommiting witness) and get their txins whitelisted.

Pairwise channels form the basis of an overlay network. We specifically target robust overlays that tolerate high churn and dishonest majorities. Random-walk-based peer sampling and rotating views help prevent long-term eclipse and bias (as described in Brahm).

This architecture does not attempt to guarantee consensus on membership. Instead, it provides a probabilistically robust discovery and coordination layer, while transaction safety remains enforced by cryptographic validity.
