# Semi-honest Multi party payjoins -- devnotes

In the semi honest case safetey is always garunteed by virtue of peers not signing a final tx which does not satisfy their payment intents.
If peers decide to ommit txs or deviate from the protocol we have not way to blaming them or exluding them -- hence semihonest. We also cannot protect against equivocations. An equivocation here means a peer re-uses a reissuance token or oppurtunity to re-commit to a new set of outputs. In the semi honest setting we assume peers are economically motivated to finish the tx and known they are entities. 

## Changes to the directory

Peers will rely on best-effort broadcast to progress the protocol. Meaning we are not sure if other peers have read our messages withing some time window.
The bip77 directory therfore needs to be upgraded to support append-only semantics as oppose to write and replace.

Peers in the multi-party payjoin will all use the mailbox. 

## Authenticated communication

## Phases

Once peers have a established a mailbox and shared secret we can start exchanging messages.
Peers will register their outputs, then their outputs. If a advesary has learned about the shared secret they can contribute outputs and if the fee contributions workout they may be able to get away with it. This can be mitigated by using BIP322 proofs. And homomorphic values to ensure output integrity (not spending more than what is being contributed).
However, if we want to use KVACs in a distributed settings that would complicate things and we would need to use something like coconut credentials for threshold issuance. 

Safety, again, is enforced by each peer checking that the final unsigned tx contains all outputs they registed.

How do we know if we have uninamity on the unsigned final tx before contributing ?
Assuming we know all the identity public keys in the peer list (the ring) peers can contribute and ACK on the final tx.

## CRDT

Transaction fragments in this protocol can be contributed and collected in any order. Order can be established by hashing event log on the directory and using that as a deterministic seed for shuffling. 


### Notes / Brain dump from convos with nothingmuch

Notes here describe a system which allows peers to use discrepate directories but allows them to effeciently sync.

We can either appoint a single leader or just rely on **best-effort broadcast** to signal progress.

I think it’s cleaner to structure everything around **best-effort broadcast** and treat the directory as providing an **efficient, encrypted broadcast-channel abstraction**. The directory doesn’t need to impose strong semantics; it just needs to move sets of messages around efficiently.

Longer term, we can start from a simple shared secret for the semi-honest case and later upgrade this to something like MLS once we want stronger group membership and key rotation guarantees.

On the messaging side, if the directory natively supports **rateless set reconciliation**, then clients can use **multiple directories simultaneously** to broadcast and sync messages for a single session — even if those directories only partially overlap in content.

The directories could either:

* be completely **oblivious** to this multiplexing, or
* actively **sync sets between each other**.

Either way works.

The main reason this design feels right is **mobile friendliness**.

You wake up, immediately connect to *all* the directories you know about, and start downloading data in parallel for the topics you care about. Bandwidth naturally aggregates across directories, letting you recover the full message set quickly.
IBLTs are efficient enough that decoding isn’t a big problem, and something like **minisketch with a shared key** would also work well.

As quickly as possible, you reconstruct the **complete message set**, which includes blocks. From there, you compute **DAG-based “leader” blocks locally**, and then derive a height-indexed set that represents the **union of all messages that leader block has seen**.

Toward the end of a phase, the set of known messages should converge with the set of **acked / voted messages** referenced by the leader block.
