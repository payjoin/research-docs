Notes on the following works:
* [Random graph models for directed acyclic networks](https://arxiv.org/pdf/0907.4346)
* [Random acyclic networks](https://arxiv.org/pdf/0902.4013)


A random graph model is intuitively: what graph structures appears by chance given some constraints
i.e what is allowed: number of nodes, degree sequence, ordering, 

Essentially highlightning which properties of the graph are suprising or explain the other strucutres of the graph.
Useful resource: https://network-science-notes.github.io/chapters/09-random-graphs.html

Why are graphs acyclic? The paper says its because the vertexes are ordered.
> The acyclic structure is merely a corollary of the ordering.

A better name for direct acyclic graphs would be directed ordered graphs.


The first model fixes the ordered degree sequence of the vertecies. k in and out degrees. 
The total numver of in and out degrees = m = # of total edges.
The graph is constructed simply by iterating over verticies, matching in edge stubs with prev outgoing stubs until the max degree sequence is met.

Flux = is a property of a gap between verticies . If we were to partition the graph at a vertex i and i-1 how many edges would be crossing that cut. 

>  Low values of flux indicate “bottlenecks” in a network—lines across which few edges flow—
and high values indicate regions in which there are many
edges

Random graph construction: 

> In the language of “stubs” introduced
above, a graph on a graphical degree sequence is created
by matching in- and out-going stubs in pairs to create m
complete edges while respecting the ordering of the vertices 

Of course ordering must be respected. Earlier vertex will be prefered. Late nodes will be constrained in open in stubs


Other result: 
If I sample uniformly from all valid graphs with this ordered degree sequence, what is the probability that node j connects to i; 
it is: weighted by how many other nodes in between could "steal" that connection.

suppressed when there is heavy competition in intermediate layers.

This is expressed in f_ij. 
the intuition is that if j is close to i the probability is high. If many vertecies are betweem them then its lower chance they are connected. 
So distance in time matters even after conditioning on degree.




