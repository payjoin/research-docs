# 1-9-26 call notes

Last time we focused on what the transaction-graph structure implies for privacy.

The random graph model we care about assumes a bounded degree sequence. The core question is how we can estimate that degree sequence in practice.

There are two relevant models in the literature (<https://arxiv.org/pdf/0907.4346>
):

One where in-degree and out-degree are explicitly fixed

One where the degree sequence is probabilistic

The degree sequence fundamentally shapes the graph. If degrees are near one, the graph collapses into something like a linked list. If nodes have higher in- and out-degrees, they can sample from a much larger population, which is what enables randomness and mixing.

The probability of connections between vertices is studied in <https://arxiv.org/pdf/0902.4013>
. If we had the full explicit graph, we could directly compute things like k-shortest paths. If we only have a degree distribution, we can still estimate these properties statistically.

We should collect empirical data from Bitcoin to understand what degree distributions real transaction structures imply. Each transaction can be viewed as a sample over the UTXO set. If selection were uniform, the inputs would look random. If we have clustering information,
