# Semi-Honest

iroh

use set reconciliation for txouts

optionally do blind silent payments with zk proofs

sign

no blame mechanism

iroh peers = trusted peers

LN transport also? dual funding?

# Byzantine

still use iroh peers, do private set intersection to invite to sessions or to
find out if overlapping in sessions




final proposal must include specific listen coordination opt-in advertisements



fast path optimized protocol with silent payments:

1. (optional) set reconciliation based gossip/broadcast of additional inputs

  in recovery sessions, the set of inputs allowed is bound by this set minus the non
  signing inputs, new inputs cannot be introduced but inputs not in the final
  proposal are still allowed.

2. concurrently, set reconciliation based gossip/broadcast of txouts. p2tr outputs have
   blind dh curve points instead of their spending p2tr pubkey.

   txins and txouts finalize together, inputs can be deterministically shuffled
   based on outputs & blinded values as a commitment to the join, obtained via
   byzantine set union consensus

3. if the join of the txout sets overspends or otherwise violates the
   constraints, execute txout pruning with homomorphic value commitments or MPC
  - reissuances
  - output cover proofs

4. all parties signal readyness to sign by performing silent payments
   derivation for all outputs with batch proof of DLEQ.

   if any input fails to cooperate, treat as missing witness, fail the session
   and restart without the offending input

   either scanning key must be secret or one of the input's must be controlled
   by the sender to avoid revealing that an output is a silent payments output
   to the other parties.

5. substitute p2tr output values with actual payment output values, signed by the dummy term

6. witness collection, lattice agreement and/or union consensus -> broadcast on bitcoin p2p


fast path optimized protocol without silent payments:

1. (optional) set reconciliation based gossip/broadcast of additional inputs

2. concurrently, set reconciliation based gossip/broadcast of txouts. p2tr outputs are given verbatim

3. if the join of the txout sets overspends or otherwise violates the
   constraints, execute txout pruning with homomorphic value commitments or MPC
  - reissuances
  - output cover proofs

4. ready to via lattice agreement on set of certified outputs

5. witness gathering


in recovery runs of the protocol, inputs must re-opt in

additional proposals may be added to the final quorum proposal, but only
between UTXOs in the allow list.

any party may proposal a final quorum proposal to heal a previously failed run,
not just the original aggregator



https://arxiv.org/pdf/2508.18193 - wait free replicated data types and fair reconciliation

https://theses.hal.science/tel-03584254/file/98542_FRANCA_REZENDE_2021.pdf - leaderless smr
https://malkhi.com/files/WinFS-version-vectors-DISC2005.pdf - concise version vectors

https://www.usenix.org/system/files/nsdi22-paper-shamis.pdf - individual accountability

https://arxiv.org/pdf/2505.19989
https://www.iacr.org/archive/asiacrypt2007/48330396/48330396.pdf


accountable / self-healing / reconfigurable lattice agreement -> lattice smr ?
byzantine generalized lattice agreement style? https://theses.hal.science/tel-04984550/document

     how do omission failures look? live peers on either side of a partition
     should be able to agree on sub-transactions, the parent session can be completed


healing runs:

- share any DAPS equivocation keys, zap everything associated with that
- continue lattice agreement with remaining items
  - additional proposals to repair pruned ones
  - subset of agreeing inputs
  - synchronous txout, with eager|lazy certification



dag based agreement-ish protocol

each version publishes its version:
- generation - threshold clock value
- predecessor versions
- operations/txs - capped to # of predecessor versions
- set, whether or not the uncertified output set has overflowed, predecessor 

ops/messages/txs posted anonymously

version + IBLTs polled anonymously, verified to include new ops


if honest parties can diffiuse all valid ops and all fraud proofs, even with
minority they can decide to sign by excluding non live outputs


async threshold clock to drive progress, number of messages is number of parent
blocks squared or something like that?


message types:

- initialize in degree 1 out degree k
- add input k in, k out (multiple inputs of same address allowed)
- reissuance k in, k out (add input with no input)
- add output large in degree, 0 out degree
- ready to sign
- witness


post messages on directory or via private submission
+
DAG gossip in parallel



equivocating messages abort protocol due to coinjoin liveness fragility, fast abort

DAPS everywhere

rate limiting?

two tiers of nodes? coordinators and non coordinators?

MLS transport encryption?



final proposal is parent of initial blocks



after the fact liveness proofs:

OTS of 2f+1 ACK signatures on witness message
