p2p client
    - broadcast logger
      - txns
        concatenate to logfile
        auxiliary log of timing info, provenance?
      - blocks
        concatenate to logfile
    - header client
      concatenate to logfile
    - CBF
    - assume utxo
    - historical blocks
electrum client
electrs client


---
indexer:

foreach file in directory (e.g. blockdata), use `bitcoin_slices` to parse

name index files as "${prefix}${basename}.${offset}-${len}-${hash}" where $hash is blake3

name WAL streams by UUID

as file is appended or rewritten, old indices can be ammended with new indexes,
extended by appending, pruned if invalidated, etc.

loose txns
arbitrary order concatenated txns
loose blocks
arbitrary order concatenated txns
blockdata files
blockchain order concatenated headers

block sizes (tx count and byte size)

undo files

assumeutxo

cbf

arbitrary orders can be marked contiguous by augmenting with merkle proof


virtual files/arrays: canonical txn order




sparse/lazy client:

- headers first, maybe only
- descriptor
- obtain txs from 





capabilities/traits:

  txid -> Option<Tx> - disambiguate sparse vs. doesn't exist in the chain
 
  outpoint -> Option<spending tx>
  outpoint -> IntoIterator<spending txs>

  blockhash -> block -> txs

---


indexer:
   transactions are stored together in files, like block data, or loose txs, or txs concatenated together.
   they could be according to confirmation order, or txid order, or some incidental order.

   in any sparse set, every txn gets a unique ID, the negated 63 bit truncation
   of some hash of its txid, like rapidhash or xx3, or even just the txid
   itself. Special case coinbase collisions?

   every input and every output has a unique ID, its offset relative to a txid.

   blocks = rrd vector of immutable arrays of txs. for a given chain tip, every
   txn gets a unique ID, its order in the blockchain. here collisions can be
   tolerated, so no special case required for using blockchain component.

   in blockchain order these are also totally ordered, (txid, offset) or as an
   absolute offset.

   ID spaces can overlap using types, they are all newtypes around a i64 or
   i32. The space of confirmed and unconfirmed txs can be merged, with 63 or 31
   bits reserved for each. 31 bits suffices for indexing confirmed transactions
   in the near term.

   every input and every output has a unqiue ID. 64 bits are needed.

   the blockchain gives a total ordering, positive integers can represent that
   given a block tip and this numbering is monotone in new blocks
   
   we can assign negative 63 bit values to unconfirmed tx in/out by offsetting
   them from the random txid, with negative offsets for inputs and positive for
   outputs. This effectively requires no collisions on the first 48-50 bits,
   which may require re-hashing.

   sled's get_id() for unconfirmed?

implement graph traits for these sets of integers, compute and cache them.
tolerate loss or mutation of tx files by reindexing. maybe make append
detection more efficient? later.

alga style traits for typed graphs.

this is a specialized tri-partite graph:

tx <- out <- in <- tx

through confirmed tx ids we can know confirming block

however, 31 bit suffice for all txs in chain order, early blocks don't need
most of the tx count bits. array map of cumulative tx counts by block maps
between these.

any coin indexed with 53 bits: = 21 + 15 + 17 bits for (blockheight,
tx offset, in/out offset). even for inputs, odd for outputs leaves 15 bits to
count, should suffice? 53 bits fits in a double nicely.

or 56 = 24 + 15 + 17

20 bit blockheight is good for another 150k blocks, 21 for another decade and a
half barring time warps.

15 maybe 14 is enough for tx count per block, 1e6/2^15 ~= 30.52 virtual
bytes, which is insufficient to the smallest tx i can think of, a single input
spending an anyone-can-spend output with 0 outputs needs at least 36 bytes for
the outpoint.

15 bits suffice for inputs in pathological block with 1MvB dedicated to
smallest txins.

17 needed for pathological block of smallest txouts.

= 49 bits

so top 2 bits of in/out and tx count 16 bit ranges are reserved, as
are top 4 bits of block height?



union find for equivalence graph over any set of objects.
