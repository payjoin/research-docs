socratic, build step by step like privacy pass

payjoin to mp payjoin
honest -> semi honest -> bft

payjoin (bip 79-77) is designed for the honest setting. bip 77 has additional
considerations for metadata privacy but like its predecessors trusts the
counterparty with the on chain data.

payjoin is designed to combat clustering. but recent results in clustering
research call this in to question.

3 parties qualitatively different from 2, makes semi-honest threat model
possible, decreased burden on honesty assumption wrt privacy but increased wrt
liveness. n-1 deanonymization attacks always a contingency.

2 party = necessarily honest wrt privacy


bft means:

- no trust wrt privacy
- sybil resistant against n-1 deanonymization
- liveness in presence of dishonest majority


communication models (sync, async, partial/hybrid)


---

ns1r needs to be mentioned

multiparty payjoin, honest setting, with directory

- establish group secret a priori (sharing links)
- directory as broadcast channel
- join semi lattice (set union) of txins and txouts
- agreement on termination, signature aggregation doesn't require consensus
- txin and txout additions don't need to be accountable but aren't technically
  posted anonymously either

- silent payments?
  - TODO BIP for self spend address derivation (change, mixing, etc) using
    silent payments, for stateless address reuse protection
- multi txs?


any byzantine party can fail the protocol

---

multiparty payjoin, honest, p2p

- same as above with iroh as transport
  - set reconcilation
  - no metadata privacy, making txin/txout attribution easier

any byzantine party can fail the protocol

---

multiparty payjoin, semi-honest

- txins and txouts need to be added anonymously
  - anonymous broadcast with iroh? probably too expensive
  - easy with directory, but requires directory as trusted party

---

multiparty payjoin, bft

- transport layer must protect privacy (no iroh)
  - tor
  - i2p
  - nym
  - directory
- protocol messages need to be authorized
  - proofs that txout cost is covered by txin funds with unlinkability
- bft agreement on txins, txouts
  - against dishonest majority?

---

multiparty coinjoin/arbitrary txn construction


## Background: wallet clustering & 2-party PayJoin

PayJoin was created as a response to the wallet clustering concern. Bitcoin
transaction transaction outputs can be clustered together, labeling a group of
TXOs as belonging to the same wallet. Clustering allows any blockchain external
information linked to one coin in a cluster, such as personally identifying
information from KYC requirements, to be associated with any other coin in the
cluster.

Clustering is a problem for Bitcoin. For businesses this can reveal information
to competitors. For individuals, especially with self-custody, it is about
personal safety and freedom from surveillance. And for the system as a whole it
degrades fungibility and censorship resistance.

Clustering techniques have advanced significantly over the years and continue
to improve. The oldest of these is the common input ownership heuristic, also
known as the multi-input heuristic, which assumes that coins that were spent as
inputs to the same transaction are owned by the same entity. CoinJoin
transactions are created in order to contradict this assumption, but this
heuristic can be refined to filter obviously multiparty transactions.

Existing PayJoin protocols (BIPs 79-77) allow two parties to collaboratively
construct a transaction. Such transactions are not as easily distinguished from
"regular" payment transactions (where the CIOH is be accurate), but since they
are multi- party transactions they cast doubt on the heuristic applied to
similar transactions.

Since PayJoin is a two party protocol, the counterparty is necessarily trusted
with regards to privacy. Each party knows which inputs and outputs belong to
it, and after eliminating those only the counterparty's inputs and outputs
remain.

## Motivation

### Privacy

A PayJoin transaction with 3 or more parties reduces the counterparty trust
with regards to privacy. Suppose Alice is paying Bob, Bob is paying Carol, and
Carol is paying Alice. Alice doesn't need to know which of the inputs to such a
transaction belong to Bob and which Carol. By the same logic, Bob wouldn't know
which input belongs to Carol, nor if Carol is paying Alice, possibly utilizing
the funds from his payment to do so.

With regards to 3rd party observers, with improved clustering techniques the
privacy of PayJoin transactions degrades. For example, by utilizing wallet
fingerprint based techniques to cluster coins, a PayJoin transaction that would
otherwise lead two clusters to be incorrectly collapsed into one could be
filtered out if the clusters appear too distinct based on their associated
fingerprints. This flags PayJoin transactions, with context clues singling them
out from the background of "regular" on-chain payment transactions.
Furthermore, if the outputs of such a transaction can be linked to the inputs
then the payment amount can be inferred. With additional parties involved, the
task of linking inputs a PayJoin transaction to each other, or the outputs to
the clusters of the inputs, both become more difficult.

### Blockspace savings

Multiparty PayJoin provides the potential for more additional blockspace
savings over PayJoin. If $n$ parties all transact with each other on chain that
would require $O(n^2)$ block space, but they can instead coordinate and create
a single net-settlement transaction with size $O(n)$ with the same outcome.

RBF cut-through. custer mempool.

If cross input signature aggregation is enabled on Bitcoin, full aggregation
would require more or less the same interaction as multiparty transactions, and
incentivizes collaboration because it requires only $O(\frac{1}{n})$ witness
data per participant.

Similarly with any kind of UTXO sharing, such as payment channels or offchain
vUTXOs, on chain payments can still be supported as a kind of splicing or
cooperative exit operation again through interactivity.

---


The receiver initiates the protocol, providing the sender with a payjoin
enabled payment URI (BIPs 21, 321). The sender replies with a fully signed
payment transaction, delivered over a peer to peer communication channel
instead of by broadcasting to the network. The receiver at that point can
opt-in to replacing the transaction, with their inputs as well, and replies to
the sender with all of the signatures for the receiver's  adding their inputs
and signing, and replying to the sender 

replies with a fallback transaction. This is a unilateral, fully signedThis 
transaction as would be created by a sender without payjoin support.

- clustering is the problem
- multi user transactions are the solution
  - coinjoin w/ robust theory of anonymity sets is maxxing version of that
  - payjoin not it:
    - requires couinterparty trust
    - anonymity set size is small
    - ...

# CoinJoin constraints

with `SIGHASH_ALL` (and without `SIGHASH_ANYONECANPAY`, which is a malleability
issue, or other hypothetical sighash flags), all parties must sign the same
transaction and then combine their signatures, or the transaction can't be
broadcast on the bitcoin network

this means that all parties must come to agreement about what transaction to sign

if the total txout amount exceeds the total txin amount, the transaction will
not be valid. therefore addition of txouts must be restricted

if only the total txout amount is restricted, some users may include txouts
exceeding the inputs they are spending, while other users would not be able to
get theirs in, so restriction of txout addition must be fair and accountable,
ensuring that txin funds cover txout funds per user

for privacy (aspect of safety), txouts must not be linkable by other parties in
the transaction, even in the semi-honest setting (fine in the honest setting),
so txout restriction can't rely on knowing which inputs are related to the input

if all of this is satisfied, i.e. the honest parties are able to come to
agreement about an unsigned transaction without compromising their privacy,
then no honest user should have a reason not to sign the resulting
transaction.

for liveness txouts must only be included if covered by txin funds, this
ensures the unsigned txn could be valid if it were signed, and that every 


---

permissionless network -> new gossip network where peers can start communicating TODO rephrase

how do we get there? the best we have is UTXOs but we can't assume 1 utxo = 1
peer, an adversary can make many small UTXOs, so we can't assume anything about n >= 3f + 1

- circular dependency between these elements needs to be broken by bootstrapping
- ability to verify UTXOs and BIP 322 ownership proofs
- ownership proofs certify listen advertisements, which include metadata
  privacy preserving authenticated endpoint, suitable for establishing
  pairwise channels: i2p destination, tor hidden service, directory mailbox, etc...
- peer to peer channels allow construction of an overlay network
- specifically we are interested in robust overlay networks, resistant to
  dishonest majority and tolerating high churn, for example random walk
  based peer sampling
- anti-entropy or set reconcilation over peer channels makes efficient gossip
  possible, allowing all parties to share the set of ownership proofs and
  listen advertisements
