
# Random Lottery mechanism

At a highlevel we are interested in randomizing our choice of which UTXOs to co-spend with to achieve better sybil resistance and promote a more robust tx graph structure.

Note that there are other ways to achieve the contribute to the desired tx graph structure i.e low-hamming weight denominations. // TODO need notes on that

Expressing this lottery based system as a term in larger wallet cost function makes the most sense.
At a high level, we take the entire UTXO set that we can coinjoin with, hash each outpoint concat'd with some block hash and perfer the lower values. Clients should also take into consideration the tx structure namely the sumset density of the UTXOs selected as well. Maybe the random lottery picks some UTXOs but another set is much better respective to your own.

Takers should converge on the same set of of makers which should reduce cost.

There are a couple ways this term can be express:

1. Per-input. each input gets a tagged hash and takers perfer inputs with a lower score.
2. Pairwise term: XOR of the hashes between two inputs. All the takers won't perfer the best coin as to create some linkage.
3. Provenance term. Inhertited score from the prior history. Coins that have enjoyed good randomess in the past may be luckier.
i.e denser conectivity improves privacy.

Q: what damage can an advesary do with an approx size of the UTXO set? Isnt this already public anyways?
If you only see a single tx, you can infer alot about the surrounding tx graph. How many utxos are participating in the protocol over all. If you and advesary what you want to constuct a graph that makes your coin's lucky. Then you need enoug coins that you can
dilute the honest subset, and the top set of coins is yours.

Gaps between the hash values may allow chain observers to analyze and estimate the cadidate size of the UTXO set.

Inherited luck makes sybil attacks more unlikely as an attacker cannot bias a random beacon.

The ability to link transactions over time can still break / reduce anonymity.

**The notion of anonymity set should quantify the number of counterfactual paths from from the origin cluster to some sink cluster -- not just the number of quivalent outputs in just a single CoinJoin tx.**

## Expander Graphs

An expander graph is a sparse graph that is nevertheless extremely well connected. Formally, it has bounded degree, but any reasonably sized set of vertices has many edges leaving it. They look like complete graphs but have far few edges.

There are several equivalent ways to define expansion. One is edge expansion: for any set of vertices  S that is not too large, the number of edges from
S to its complement is proportional to  ∣𝑆∣ . Another is spectral expansion, which characterizes expansion via the eigenvalues of the graph’s adjacency or transition matrix. These definitions all capture the same intuition: the graph rapidly mixes information.

A random walk (where each step is taken uniformally at random) on an expander will spread out. In an expander graph, the distribution of a random walk’s position converges quickly to the stationary distribution (which is uniform if the graph is regular). After a few steps the graphs initial position is independent from its starting position.

Superconcentrators are directed, acyclic, and focus on worst-case routing garuntees.

## Random Asyclic Graphs

Graphs with no closed cycles, directed edges. Random graphs is a model network of a given number of verticies which some features are fixed but others are placed at random.

Degree sequence: Each vertex as K incoming and outgoing edges.
This paper constructs such graphs within reasonable computational bounds. We pair out-stubs (pending outgoing edges) with in-stubs. The matching must respect some ordering. Out-stubs must match on a neighboors in-stub.

i.e for all verticies match an outgoing stub with a in going stub at random. Which runs in propotial to the number of edges.

This can be analyzed as a case of preferential attachment style network (albert-barbasi) -- i.e perferencial attachment is it self a random acyclic graph when degree ordering is fixed.

## CoinJoin denominations

def: hamming weight -> non-zero digits in a base. ex 00110, in base2 it would be the popcount = 2.

Small hamming weight numbers (across different bases) result in denomination sets with high multiplicities. Mauers work demonstrates that denomination sets with high multiplicity results in a factorial number of input-output pairings.

Any arb satoshi amounts can be represented by using a set of these denominations.
Dense sets of denominations lead to many sets of possible subset sums (i.e sub-transaction mappings). i.e subset sum density.
**Dense ≈ less constrained ≈ more possible mappings ≈ bigger anonymity.**

Multiple bases may be preferencial (base-2 + base-3 + base-10?). Odd values in some bases can be repr in other bases.
Empirical work demonstrates that nearly all comboniations can be covered with 3-4 denominations in the set.

## Towards measuring tracebility notes

tracebility: the measure of uncertainty an advesary has on the origin of some coins. Subjective traceability allows for a measurement to be influced by external knowledge.

Whats on the chain is just objectective traceability.

This paper aims to create a formal framework to extract privacy metrics from any transaction graph.
source nodes: mint coins into existence (coinbase)
sink nodes: do not spend coins they receive.

To trace the origin of a source node we reverse the graph. The reveresed graph is then markov chain of with probabilites describing if an output is related to an input. Then perform a random walk to obtain a traceability score. The random walk is finished once a absorbed node is encountered (i.e a self-loop or a minting event).

Probability score are created proportional to the transfered amounts. however they can be influnced by any arb metrics.

An uncertainty score is then the amount of information an advesary lacks to obtain a total certaintly in the origin of the money. this can be modeled as the shannon entropy of the probability distribution / scores (denoted in the paper as B_i)

Intuitively this is the uncertaintly score about the origin of a source node given a set of sink nodes.

> Ultimately, we are interested in computing the
> distribution of absorbing probabilities over the absorbers when
> starting random walks from sinks of the original graph 𝐺

Each transition probability matrix cell denotes a probability from transitioning from state i to state j (P_ij).
Each probability is influenced by subjective information. "Custom transition probabilities".
