# Multi-Party Dependency Graph

Leaf nodes: multi-party pj, coinjoins (meaning with strangers)

Non-leaf node: potential tracking issue that can be closed
ex: Broadcast channel abstraction: one for BFT and semi-honest

TODO: are the non-leaf nodes too abstract or concrete?

Deliverable of this conversation is to facilate a group disucssion on the deliverable.

-------- 
## *B*IP-322 Proofs
Depends on: [O]

Proof of input ownership.  Should commit to ephemeral session key.

## *O*nline key
Depends on: []

Ephemeral key pair used for a single mp pj session.
Each key pair commits to the single input.

## *A*ppend semantics on directory
Depends on: []

If all parties are using the same shared secret and using the same sub directory then we need to 
support append operation as not to overwrite messages that peers have not downloaded yet.

## *W*ire Protocol Messages
Depends on: [T, CP, ..., Pretty much everything]

Define wire representation for protocol messages.

## *I*nvitation
Depends on: []
In the semi-honest setting one party can generate a secret key and share it with the others.
But more generally how can peers "join" existing sessions?

## *T*ransport
Depends on: [?A]

Linking individual transaction fragments to the same entity should not be feasible. 
We need a mechanism for anon broadcast.
From overview: 
* tor
* i2p
* nym
* katzenpost?
* payjoin directory service w/ OHTTP relay


## *C*RDT *P*SBT
Depends on: []

We need a more refined definition of PSBT. Transactions should be representable in their more minimal form.
We can then define union and sorting operations on this type. 

https://github.com/arminsabouri/lattice-psbt

## *Ag*reement
Depends on: [CP]

In the presence of byznatine peers, honest parties will converge on the same transaction.
We can operate 

## *S*et *R*econciliation
Depends on: [Ag]

Peers/replicas may need to sync their state against the lastest. SR defines the protocol for how they do that.

## *D*APS
Depends on: [Ag]

Get byzantine party key  by means of them equivocating. i.e "spending" the same commitment in two places.

## *H*omomorphic Value Commitments
Depends on: [Ag]

Unfair txout additions can be prevented by requiring a proof that its effective cost is covered by some txin contribution.
We can commit to txins and their value and "spend" the commitment when registering outputs. A nullifier set must be 
agreed on to prevent equivocatoins. 

## *R*ange Proof & Balance Proof
Depends on: [H]
TBD

## *C*redential System
Depends on: [H, Ag, D]

We need to break the link between messages -- input and output registrations
Generally this is a 1-out-of-n statement with respect to the set of commitments. 
Candidates: 
* Linkable Ring sigs
* Coconut credentials
* ... 
 
