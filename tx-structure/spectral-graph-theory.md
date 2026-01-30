# WIP - Spectral Graph Theory

The study of eigenvalue and eigenvectors of a graph matrix. Be it laplacian or adjecency matrix.

For an adjecency matrix the eigenvalues tell us how connected the graph is. 

So instead of analyzing the graph directly, we use a matrix repr, compute eigenvalues and infer structural properties. 

Eigenvectors place nodes in a continuous space where distances reflect connectivity.


## Laplacian Matrix

Intuitively the laplacian is a matrix that measures how each node / vertex differes from its neighbors

$L = D - A$
Where D_ij = deg(i)
and D_ij = 1 if there is an edge

A is the adjecency matrix and D is the degree matrix. 

A couple observations:
1. The smaller the eigen value The "smoother" the eigen vector
2. The smaller the eigen value, the less connected.

## How this relates to random walks

Start at a node:

* At each step, move to a random neighbor
* Probability is uniform over neighbors

This defines a Markov chain on the graph. The transition matrix is persisely: P = D^-1 A
