
## Random Lottery mechanism 

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

TODO: How this relates to expander graphs.
	

## Random Asyclic Graphs

Graphs with no closed cycles, directed edges. Random graphs is a model network of a given number of verticies which some features are fixed but others are placed at random. 

Degree sequence: Each vertex as K incoming and outgoing edges. 
This paper constructs such graphs within reasonable computational bounds. We pair out-stubs (pending outgoing edges) with in-stubs. The matching must respect some ordering. Out-stubs must match on a neighboors in-stub. 

i.e for all verticies match an outgoing stub with a in going stub at random. Which runs in propotial to the number of edges.

This can be analyzed as a case of preferential attachment style network (albert-barbasi) -- i.e perferencial attachment is it self a random acyclic graph when degree ordering is fixed.
 
