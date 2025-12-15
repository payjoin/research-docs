
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
