# Semi-honest Multi-Party Payjoins -- dev notes

## Threat model (what we assume)

In the semi-honest setting, safety is basically free: no one will sign a final transaction that violates their payment intent. If the final unsigned tx doesn't pay me correctly, I just don't sign.

What we don't get in this model:

* No way to blame or exclude peers who stall, omit messages, or quietly drop out
* No protection against equivocation

By equivocation here, we mean stuff like:

* Re-using a reissuance token
* Re-committing to a different set of outputs after previously committing

We assume peers are:

* Economically motivated to finish the transaction
* Known entities (not fully anonymous adversaries)
  So we lean on incentives rather than cryptographic enforcement.

## Directory changes

The protocol progresses via best-effort broadcast. There's no guarantee that other peers have seen your message within any bounded time window.

Because of that, the BIP77 directory needs to change:

* It should be append-only, not write-and-replace
* Messages become an event log rather than mutable state

All peers in a multi-party payjoin share the same mailbox.

## Authenticated communication

(Still under-specified.)

Once peers are in the same mailbox, all messages need:

* Sender authentication
* Confidentiality against outsiders
* Replay protection

This can be done with a shared secret or something MLS-like.

## Phases / protocol flow

Once peers have:

* A mailbox
* A shared secret

They can start exchanging messages.

### Group formation

One option:

* The first peer creates a group secret and shares it with invitees

Another option:

* Use something like MLS

Open question:

* How do we know the group size is fixed?
* Do we require explicit membership closure before progressing?

### Input / output contribution

Peers register:

1. Inputs
2. Outputs

Attack surface:

* If an adversary learns the shared secret, they could inject outputs
* If fees line up, they might get away with it

Mitigations:

* Use BIP-322 proofs to bind inputs to identities
* Use homomorphic value commitments so peers can't spend more than they contribute

Going further:

* If we want KVAC-style guarantees in a distributed setting, things get hairy
* Threshold issuance would likely require something like Coconut credentials

### Safety checks

Safety still comes from local verification:

* Each peer checks the final unsigned tx includes all outputs they registered
* Peers agree on global tx parameters ahead of time:

  * feerate
  * nSequence
  * locktime, etc.

If anything is off, peers just refuse to sign.
Who ever is creating the sk also proposes the global tx params.

one approach here is do this with the directory
* Include dir uri
* Sender can do OHTTP GET to get payment instructions
* Post quantum hpke with no minimal overhead 


In the honest/semi-honest setting if the protocol fails you just need to restart.

### Unanimity on the final unsigned tx

Open question:

* How do we know everyone agrees on the same unsigned tx before signing?

One idea:

* We already know all identity public keys (the ring)
* Peers use a linkable ring sign and ACK the final tx hash
  - This would be overkill for honest/semi-honest setting. Ack / ready to sign message can be broadcasted confedentially and once you have N of those you know when to sign.

Optimization:

* The ACK could be the signature itself over the sigash
* That avoids an extra round of communication

## CRDT / state model

Transaction fragments can arrive in any order:

* Inputs
* Outputs
* Reissuances
* Param declarations

This naturally forms a monotonic, mergeable state:

* Essentially a CRDT

Final ordering:

* Hash the directory event log
* Use that hash as a deterministic seed
* Shuffle inputs/outputs deterministically before signing

This gives everyone the same canonical transaction without extra coordination.

### Notes / Brain dump from convos with nothingmuch

Notes here describe a system which allows peers to use discrepate directories but allows them to effeciently sync.

We can either appoint a single leader or just rely on best-effort broadcast to signal progress.

I think it's cleaner to structure everything around best-effort broadcast and treat the directory as providing an efficient, encrypted broadcast-channel abstraction. The directory doesn't need to impose strong semantics; it just needs to move sets of messages around efficiently.

Longer term, we can start from a simple shared secret for the semi-honest case and later upgrade this to something like MLS once we want stronger group membership and key rotation guarantees.

On the messaging side, if the directory natively supports rateless set reconciliation, then clients can use multiple directories simultaneously to broadcast and sync messages for a single session — even if those directories only partially overlap in content.

The directories could either:

* be completely oblivious to this multiplexing, or
* actively sync sets between each other.

Either way works.

The main reason this design feels right is mobile friendliness.

You wake up, immediately connect to *all* the directories you know about, and start downloading data in parallel for the topics you care about. Bandwidth naturally aggregates across directories, letting you recover the full message set quickly.
IBLTs are efficient enough that decoding isn't a big problem, and something like minisketch with a shared key would also work well.

As quickly as possible, you reconstruct the complete message set, which includes blocks. From there, you compute DAG-based "leader" blocks locally, and then derive a height-indexed set that represents the union of all messages that leader block has seen.

Toward the end of a phase, the set of known messages should converge with the set of acked / voted messages referenced by the leader block.
