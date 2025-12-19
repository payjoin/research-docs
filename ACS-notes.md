
## On Async Common Subset -- Dev Notes

### Network models

We usually define network models in terms of what the adversary can do.

* **Asynchronous network**
  There is no bound on message delivery time. The adversary can’t destroy messages, but they can delay them for an arbitrary amount of time.

* **Synchronous network**
  There is a known global upper bound on message delays.

* **Partially synchronous network**
  The network starts out asynchronous. At some unknown point during execution, message delays become bounded. Messages are eventually delivered, but the bound is unknown and may vary over time.

* **Weak synchrony**
  The message delay bound can vary over time, but eventually it does not grow faster than a polynomial function of time.

### Byzantine agreement (BA)

Byzantine agreement asks a set of parties to agree on a value even when an adversary controls some of them and actively tries to break agreement. BA typically shows up in two flavors: consensus and broadcast. 

* Consensus
  Every party starts with an input value. The goal is for all honest parties to agree on an output value. If all honest parties begin with the same input, they must decide on that value (validity).

* Broadcast
  Only one designated party (the sender) has an input. All honest parties must agree on the same value (agreement), and if the sender is honest, that value must equal the sender’s input (validity).


If you abstract away application details, the goal is to build a replicated state machine. Clients submit messages to a network of peers, and the peers construct a globally consistent, totally ordered, append-only log. This is also called atomic broadcast.

These systems assume adversaries and rough network conditions -- high latency, partitions, nodes going offline. Despite that, we want strong safety and liveness guarantees.

### Why async is hard

Most BFT systems avoid fully asynchronous networks because of the FLP impossibility result. Instead, they rely on timing assumptions. These assumptions often hurt in practice:

* Too strict: lower throughput
* Too loose: underutilized bandwidth

PBFT-style systems are a common example. Partially synchronous or synchronous systems also tend to recover slowly from network partitions.

FLP says that no deterministic protocol can guarantee termination in an asynchronous system, even with just one faulty node.

To get around this, async BFT protocols use randomness, which gives probabilistic (asymptotic) liveness or termination instead of deterministic guarantees.

### Async Common Subset (ACS)

We use the Async Common Subset as a building block to make progress in a fully asynchronous model.

The standard approach reduces ACS to:

* Multiple parallel instances of **Byzantine Reliable Broadcast (BRB)**, and
* Multiple instances of **Asynchronous Binary Agreement (ABA)**.

Each node proposes a value using BRB. Then, for each proposal, the network runs an N concurrent ABA instances  to decide whether that value should be included in the final output set.

ABA itself is often reduced to binary agreement plus a common coin (or coin-tossing primitive). I need more notes here.

ABA often includes a shared random coin ( in HBBFT case implemented via threshold signatures). Assuming the advesary has some influnce on the message scheduling, this is to prevent an adversary from forcing a disagreement forever. Ultimately messages cannot be delay forever in this network model, once N-f honest nodes exchange messages we can perform the coin toss which allows them to obtain the same unpredicatable random bit which should settle disputes that could occur during the ABA. This is what allows ABA and consequently ACS to conclude with probability 1.


### Example: HoneyBadgerBFT ACS

In HoneyBadgerBFT:

1. Phase 1: Reliable Broadcast
   Each node uses reliable broadcast (RBR) to disseminate its proposed value to the rest of the network.

2. Phase 2: Binary Agreement
   The protocol runs **N parallel ABA instances** to agree on a bit vector indicating which values to include in the final set.

A naïve approach would be:

* Set a bit to `1` for every BRB you’ve seen complete
* Set `0` for broadcasts that haven’t completed

But this doesn’t work. Correct nodes may see completed broadcasts at different times and in different orders. If nodes propose `0` too early, they can prevent agreement.

Instead, nodes must delay proposing `0` until they are sure the final vector will contain at least `N − f` ones. 


## On dynamic participation

In the permissionless model there may be no exact upper bound on the number of participants in a Byzantine agreement protocol. Unknown participants (UP) broadcast model provides a tight worst-case round complexity. One key assumption here is that parties have point-to-point channels with a shared common clock -- i.e synchornous rounds.

Consensus cannot be achieved if the number of non-honest nodes is > n/2.  So, the question is what is the best form of agreement on honest parties inputs that can be achieved even if t > n/2. Specifically, honest parties can output the same n-tuple values from every party. This is called interactive consistency(IC). IC is valid if every honest party has the same i-th value of the output tuple of all other honest party. And it terminates in some a-priori round R.

tldr; IC allows for all honest nodes to agree on per-node inputs despite of byzantine behavior. IC can be reduced to multiple instances of BA (one per sender).

### Tangent: Original Dolev-Strong Protocol
Unlike Lamport's original IC algorithm (which signs recursively), DS collects honest signatures on protocol values. Note that DS is solving IC for a single sender instance. In that sense its closer to a signed RBC or PBFT.
In round 1 a designated sender signs and multicasts their input. In subsequent rounds parties that collect messages with valid signatures, add their own and re-broadcast. If after f+1 rounds you have values with f+1 valid signatures then decide v. This works because honest nodes only sign one value and do not equivocate. i.e two conflicting values cannot have f+1 honest signatures.

This is a form of quorum certificate (r sigs on some value are sufficient to convince any party to accept b in round r) 

### UP-setting

In this setting the number and public keys of the protocol participants is unknown. In this setting we assume parties can communicate via a diffusion network ( sync network where messages send at some round r will be delivered r+1). Note that dolev-strong agreement variants require a known membership list in order to have a valid "quorum". In UP parties have to dynamically agree on the set of active parties, s.t ceritificates are signed by the parties in that active set. 


## On Dag-rider Consensus

Operates in the async. network model. Using a threshold-based coin implementation. Reminder that async algos typically use randomness to achieve consistency. Where byzantine nodes have a harder time distrupting protocol flow because they cannot predict with full certaintity how nodes will behave.

Consists of two phases:
1. Communication layer where peers reliably broadcast that help them form a DAG of the messages they delivered.
2. Ordering layer. No extra communication. Peers will locally order dfelivered messages in their local DAG.

Reliable broadcast: Ensures that if a honest node broadcasts a message m, all other honest nodes will eventually see m -- given that an adaptive advesary can indefintely delay message arrival. 

Global perfect coin: an unpredictable yet fair element which chooses the leader of the protocol. One way to implement this is by using a PKI and threshold-signatures.

Each DAG is an abstraction for reliable broadcast message from other peers process. Each vertex is a message and it refrences to previously broadcasted verticies. Those refrence are the edges. Reliable broadcase ensure that eventually all peers end up with consistent view of a DAG. Each protocol round has a set of vertices associated with it (?). Each round has at most n verticies with a different source (peer). There are strong edges with (2f + 1) and weak edges (f) (?). 

Intuitively each vertext from some round is strongly accepted if the next round has 3f + 1 outgoing edges that point to it. Implicitly encoding voting in the DAG structre. 
