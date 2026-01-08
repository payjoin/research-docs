# Byzantine-Tolerant Multiparty Payjoin

## High-level sketch

### Protocol phases

1. Coalition formation
   Unanimous agreement to proceed with a coalition proposal. I’m assuming the aggregator initiates this via a Byzantine broadcast.

2. Auxiliary inputs
   Exchange of any non-transaction inputs required for the session.

3. Reissuances
   Split / merge of homomorphic value commitments. This is an umbrella phase covering all reissuance-style operations.

4. Transaction outputs
   Construction of txouts.

5. *(Optional)* Silent payments
   Blind DHs plus a batched DLEQ proof per input.

6. *(Optional)* P2TR output unblinding / finalization

7. Signature aggregation
   Final signing and aggregation.

## Current mental model

* A **coalition proposal message** commits to a specific set of *listen advertisements* that opt in as replicas.
* **All nodes are validators** by default.
* Nodes establish peer communication using client authentication, which could be:

  * a ring signature,
  * authentication with a specific key, or
  * possibly just a shared secret (especially for the ring-sig case ... still TBD).

## Peer sync and gossip

When a node connects to another node:

* The accepting node authenticates the peer.
* It sends a *Merkle set root* (Curve Tree point?) committing to all protocol messages and blocks it currently knows.

If the connecting peer doesn’t already have this Merkle set:

* The two peers perform set reconciliation.

A peer may only send messages whose references (e.g. parent blocks, outputs of earlier messages) are **upper-bounded by the Merkle set commitment**. Set reconciliation enforces this naturally.

On the wire:

* Each message is **compressed** to indicate which messages it *does not* refer to, using the canonical ordering of the shared Merkle set.
  This is more efficient than listing all references explicitly.

Peers gossip in this way over a robust overlay network. Ideally, nodes connect to as many peers as possible. Some nodes will only manage a small number of connections and must rely on the assumption that at least a subset of those peers are honest.

## Blocks, clocks, and progress

Validators select sets of messages to include in "blocks", which are added to the global set union along with a list of parent blocks.

* Under **partial synchrony**, block production waits for a threshold clock to advance rounds.
* If **synchronous mode** is enabled, timeouts allow broadcasting blocks with fewer than (2f+1) parents. When this happens, the protocol effectively forks off a sub-branch that drops unresponsive (dead) validators.

**Fully asynchronous** behavior is supported by simply disabling timeouts.

## Directories and transport

Gossip can also happen via a directory, using efficient set reconciliation over encrypted blobs posted by different users.

Conceptually, this is the **same protocol** operating under different abstract clocks:

* An **async threshold clock** that always runs and progresses as fast as the network allows.
* An optional **synchronous clock** that gives up waiting and advances rounds to preserve liveness when peers are unresponsive.

The aggregator is responsible for choosing and tuning these clock parameters.

## Consensus and complexity

Within these dynamically reconfigured rounds, we can run a Byzantine set-union consensus protocol (potentially something robust even under dishonest majority) and compute the union locally.

Overall:

* Communication should be on the order of **(O(n^2))**, possibly lower on the happy path thanks to set reconciliation.

## Incentives and membership

* Replica nodes contribute more than their fair share of bandwidth.
* Other nodes act as leeches (mobile constaint devides).
* The aggregator can compensate replicas, creating an incentive to volunteer as one.

All nodes can act as validators by default, or opt out by issuing a `quit` operation that reconfigures consensus to remove them.

This feels simpler than requiring nodes to opt out a priori.
