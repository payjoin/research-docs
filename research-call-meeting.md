* We are aiming to produce one or two research papers on privacy theory. Markdown works well for code and notes, but it gets in the way for theory-heavy writing. For the privacy theory work, LaTeX is clearly the right choice, with Overleaf or a similar workflow. GitHub can still host simulations, experiments, and supporting material.

* The objective of the call was to align on how to digest the large amount of existing information, especially around how to build transactions and what communication paradigms we assume. Before going deeper, we need to align on transaction structure.

* We are confident that we can construct a transaction that an honest subset of participants will agree on. The key question is: if you are an honest agent, what are your actual options? This can be approached in two ways. In an offline setting, we may need to rely on approximation. In an online setting, decisions are incremental and based on local information.

s* The metrics we define should be consistent across offline and online settings. They should capture a notion of anonymity in the transaction (sub)graph, likely by combining Isvans’s ideas with other graph-based approaches.

* In Istvan’s model, inputs and outputs are linked probabilistically, and linkability is largely driven by amounts. This is not granular enough. we need richer structure that accounts for how individual coins and subsets relate to the transaction as a whole -- and potentially some subgraph.

* Many rabbit holes here. To stay productive, we need a tight focus: clear questions, deadlines, and explicit goals. This work naturally splits into a theoretical component and an experimental component, where we test ideas on transaction graphs.

* This work introduces its own vocabulary, much of it coming from graph theory. Concepts from expander graph theory map cleanly onto Bitcoin privacy questions, and translating between the two gives us a useful mental model.

* At a high level, we are dealing with random graph constructions. Superconcentrators can be thought of as a DAG analogue of expander graphs. The main connection to expander graphs is about designing rules that produce high-quality graphs.

* The theory is only meaningful if we justify it experimentally. For example, Samurai CoinJoins are not expander graphs, though certain constructions might satisfy expander properties under specific conditions. We are already working on this experimentally via a simulation binary and a Chainalysis-style transaction graph framework.

* It is hard to tell by inspection whether a graph is an expander. Expander graphs are a tool for understanding different regimes in a model, not something that is visually obvious. The real question is whether we can design a protocol that produces an expander graph with high probability.

* We are implicitly considering a world where the CIH assumption holds, meaning a subset of honest participants always completes the protocol. This connects to percolation theory and to understanding robustness under partial adversarial control.

* From a random-walk perspective, expander graphs behave like complete graphs. After a small number of steps, the walk reaches a nearly uniform distribution. This is exactly the property we want for privacy.

* We can use the Bitcoin blockchain as a random beacon. For each new block, we hash UTXOs together with the block hash to assign each UTXO a fresh random score. Coins are then shuffled or ranked based on this score, potentially combined with other preference terms, such as a desire to CoinJoin with similar-looking coins.

* The cost function we are discussing is a generalization of coin selection. Some coins receive higher or lower random scores, and we may simply prefer those with lower scores. This is analogous to the German tank problem and raises the question of how biased the observed population is (?) (not sure i captured this point correctly)

* The threat model includes Eve–Alice–Eve style attacks, where the adversary has high-quality clustering information about users and tries to exploit that knowledge.

* Open questions remain around how much bias this induces over time, how much information leaks through revealed preferences, and whether there is a clean way to frame this as an edge-level differential privacy model.

* Next steps include reading the randomized transaction graph paper, reading the gist on using a random beason for coin selection and its implications for random graphs and superconcentrators, reviewing the denominations and radix CoinJoin notebook, and clarifying the implicit edge-based privacy model we are using.


