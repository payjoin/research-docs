Graphs with no closed cycles, directed edges. Random graphs is a model network of a given number of verticies which some features are fixed but others are placed at random. 

Degree sequence: Each vertex as K incoming and outgoing edges. 
This paper constructs such graphs within reasonable computational bounds. We pair out-stubs (pending outgoing edges) with in-stubs. The matching must respect some ordering. Out-stubs must match on a neighboors in-stub. 

i.e for all verticies match an outgoing stub with a in going stub at random. Which runs in propotial to the number of edges.

This can be analyzed as a case of preferential attachment style network (albert-barbasi) -- i.e perferencial attachment is it self a random acyclic graph when degree ordering is fixed.
 
