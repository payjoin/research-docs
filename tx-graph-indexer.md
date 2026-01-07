# Tx Graph Indexer

The goal of this project is to efficiently index the graph of sequentially ordered transactions and a looser representation of the transaction subgraph in order to analyze the transaction graph with a composition of hueristics and other analytical models.

The goal of these indecies is to allow for effecient querying along the graph. i.e finding the txins of a given txout should be a O(1) lookup. and vice versa.

## Indecies

We will need to maintain a couple of indecies:

* Block indecies -> indexed by block height
* Cumulative txin indecies -> indexed by txid
* Cumulative txout indecies -> indexed by txid

These indecies will be saved on disk in some form TBD.
And they will be loaded into process memory incrementally or using mmap or async IO.

A global txid is then `t_i = block_index + tx_id_byte_offset`.
Similarly the global txout id is `txout_i = t_i + txout_byte_offset + txout_index`.
and the global txin id is `txin_i = t_i + txin_byte_offset + txin_index`.

Globally ordered Transactions can be represented by the global id. However in a loose representation we need to represent them with a hash or a truncated hash as we dont know their place in the ordered set (blockchain).

We also need inverted indecies keeping track of **global txin index -> prevout txout short id**. Indexed by the global txin index.
and similarly we need **global txout index -> nextout txin short id**. Indexed by the global txout index.

How can we accomodate for custom indecies? Perhaps some researcher wants to index addresses and txs. Some kind of plugin system or hook system for indecies?

## Heuristics

For the heuristics and clusters we represent releated txouts as disjoin subsets i.e union find data structure -- indexed by global txout index. As we iterate over transactions we update the union find data structure according to the heuristics.

Some hueristics depends on other heuristics. And some external information will bolster the results of other heuristics. How these heuristics compose and layer is a bit of a open question. One idea is the represent the pipeline of heuristics and the transformations we make to the clustering graph as a map-reduce and conceptulize an analysis pipeline as a AST / IR.

## Interface

After creating the indecies and running the heuristics we need to provide a interface to query the data. The output of the running the heuristics should be a statically collection type.

Example:

```rs
let clusters = chain_manager.calculate_clusters([
    Heuristic::ChangeIdentification, 
    Heuristic::CommonInput,
]);

let txouts = clusters.get_txouts();
let txs = clusters.get_txs().iter().filter(|tx| tx.block_height > 10_000 && !tx.is_coinbase()).collect::<Vec<_>>();

clusters.save("/path/to/clusters.bin").await?;
```

Creating heuristics should be pluggable (similar to creating new indecies). Currently in blocksci there are limited options. You can add some custom filters to exsiting heuristics. Otherwise you have to create a new heuristic from scratch in CPP, re-compile, expose in FFI and then use it.

How can we create a more flexible system where you can create new heuristics and they compose? This is an open question.
