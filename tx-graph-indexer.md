# Tx Graph Indexer

The goal of this project is to effectively index the graph of sequentially ordered transactions and a looser representation of the transaction subgraph in order to analyze the transaction graph with a composition of hueristics and other models.

The goal of these indecies is to allow for effecient querying along the graph. i.e finding the txins of a given txout should be a O(1) lookup. and vice versa.

We will need to maintain a couple of indecies:

* Block indecies -> indexed by block height
* Cumulative txin indecies -> indexed by txid
* Cumulative txout indecies -> indexed by txid

A global txid is then `t_i = block_index + tx_id_byte_offset`.
Similarly the global txout id is `txout_i = block_index + txout_byte_offset + txout_index`.
and the global txin id is `txin_i = block_index + txin_byte_offset + txin_index`.

Globally ordered Transactions can be represented by the global id. However in a loose representation we need to represent them with a hash or a truncated hash.

We also need inverted indecies keeping track of txin index -> prevout txout short id. Indexed by txin index.
and similarly we need txout index -> nextout txin short id. Indexed by txout index.

For the heuristics and clusters we represent releated txouts as disjoin subsets i.e union find data structure -- indexed by global txout index. As we iterate over transactions we update the union find data structure according to the heuristics.

Some hueristics depends on other heuristics. And some external information will bolster the results of other heuristics. How these heuristics compose and layer is a bit of a open question. One idea is the represent the pipeline of heuristics and the transformations we make to the clustering graph as a map-reduce and conceptulize an analysis pipeline as a AST / IR.
