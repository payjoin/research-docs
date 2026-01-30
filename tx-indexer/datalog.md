# Datalog inspired data pipeline

We draw inspiration from how datalog ingests facts to derive facts using rules.
We have base facts (blocks, raw txs from the mempool, tagged info about coins, etc..) and rules that ingest those txs to create derived facts.
These derived facts in turn can emit new facts that are used by other rules. 

For example: consider a coinjoin classififcation rule that emits if a tx is a coinjoin and MIH rule that needs that information to prevent cluster collapse.

## A more complicated example

Facts are not limited to a transaction or its parents / children. A single fact may affect the transaction graph of an entire cluster of txout

Consider a transaction graph where two subgraphs are clustered seperately (cluster A and B). Cluster A has a tx with known change output with spk X. Eventually in cluster B a transaction shows up creating
and output to the same spk X. These disjoint sets now need to union.

## Data dependency

Rules should express what their input shape is. The most intuitive way to express this is to just use the native rust type system.
In essence this is a an embedded DSL in rust s.t expressesion are evaluated lazily. Each nodes in the AST represents some stronly typed phase of a computation.

Example:
```rust
let all_known_txs = ...; // monotonic growing 

// Symbols of computation that will occur
let is_coinjoin_mask = IsCoinJoin::new(all_known_txs)
let non_coinjoin = all_known_txs.filter_with_mask(is_coinjoin_mask.negate())
let MIH_clustering = MultiInputHeuristic::new(non_coinjoin)

let change_mask = ChangeIdentification::new(all_known_txs) // outputs = labling over outputs
let txs_with_change = non_coinjoin.outputs().filter_with_mask(change_mask).txs();  

// Join of change_clustering and MIH
let mut global_clustering = Placeholder::new() // Anon var that will be unified  

let unilateral_txs_mask = IsUnilateral::new(global_clustering)
let change_clustering = ChangeClustering::new(non_coinjoin.filter_with_mask(unilateral_txs_mask & txs_with_change))

let combined_clustering = change_clustering.join(MIH_clustering)

global_clustering.unify(combined_clustering)

```

A typed AST of expressions is then created. But we also have cyclical dependencies as mentioned above. This needs careful consideration. The placeholder concept can express circular dependencies.
The engine takes care of evaluating expressions. Each output of a expression is cached / memiozed seperately. How we persist certain information is still an open question.

i.e Global clustering facts after a fixpoint has been reached should get persisted to durrable storage. But what about coinjoin annotation. We need a way to express this. 


