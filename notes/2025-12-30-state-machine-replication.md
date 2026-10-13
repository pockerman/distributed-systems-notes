# Lesson 23: State Machine Replication

## Overview

In this lesson we discuss [state machine replication](https://en.wikipedia.org/wiki/State_machine_replication) (SMR).
This is a technique for implementing a fault-tolerant service by replicating servers and coordinating client interactions with the server replicas.
It also provides a framework for understanding and designing replication management protocols [1], so it is very relevant to the
discussion of data replication. In fact, the Raft replication algorithm is based on state machine replication [2].

State machine replication works under a simple principle: if two identical, deterministic processes begin in the same state and receive the same inputs in the same order, they will
produce the same output and reach the same state [4].

For our purposes, we can view state machine replication as another replication strategy (see [Lesson 21: Replication Strategies](2025-04-22-data-replication.md)).

## State machine replication

State machine replication is a technique for implementing fault-tolerant systems, and it is also used
to implement replication protocols. Many distributed SQL databases and key–value stores use state machine replication to replicate data
across nodes [3].

In state machine replication, once a request is
acknowledged, its effect on the system state is durable, even if some components fail.
Moreover, once a node has accepted a client request, any subsequent request to any other node in the cluster
is guaranteed to see the updates caused by it. This is a linearizability guarantee [3].

To be fault tolerant, the cluster has to keep working when some
nodes fail. How many failures it can tolerate depends on the size of the cluster and the
type of failures [3]. As a general rule, the larger the cluster, the more failures it can tolerate, but also the
more complex it is to operate, the higher its latency, and the higher its cost.

Assuming non-Byzantine failures, i.e. nodes crash or become unreachable but do not behave maliciously, a cluster of $2f + 1$ replicas can tolerate
$f$ failures [2,3].
For example, a five-node cluster can tolerate two failed nodes, because the remaining three nodes still form a _quorum_ (majority) of
$f + 1$ nodes, which is the minimum number of nodes required to make progress safely [3].
A cluster needs at least three nodes to tolerate even a single failure ($f = 1$), so the minimum size of a fault-tolerant cluster is three [3].

You can implement state machine replication using algorithms such as [3]:

- [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science))
- [Raft](https://en.wikipedia.org/wiki/Raft_(algorithm))
- [Viewstamped Replication](https://brooker.co.za/blog/2014/05/19/vr.html)

State machine replication guarantees that all nodes maintain the same state by executing the same commands in the same order [3].
Unlike simpler replication schemes, which copy data or services for availability or load balancing without strict ordering, state machine replication enforces a consistent global state across nodes. It does so by using a consensus algorithm to agree on the order of commands, even in the presence of failures or network partitions. This provides strong consistency and fault tolerance, at the cost of extra latency [3].

Several open-source projects implement state machine replication and expose APIs on top of it, for example [etcd](https://etcd.io/) (Raft) and [Apache ZooKeeper](https://zookeeper.apache.org/) (Zab).

## Summary

::: {.callout-note title="Summary"}
State machine replication (SMR) builds fault-tolerant systems by replicating deterministic servers and feeding them the same commands in the same order. It is widely used in distributed databases and key-value stores. Once a request is acknowledged, its effect is durable, and subsequent requests to any node see the update, providing linearizability.

Fault tolerance depends on cluster size and failure type. For non-Byzantine (crash) failures, a cluster of
$2f+1$ replicas tolerates $f$ failures, since the remaining $f+1$ nodes still form a majority quorum. The minimum size of a fault-tolerant cluster is therefore three.

SMR relies on consensus algorithms such as Paxos, Raft, and Viewstamped Replication to agree on the order of commands, even under failures or network partitions. It provides strong consistency and fault tolerance, but adds latency compared to simpler replication strategies.
:::

In the next lesson we discuss [consensus](2025-12-31-consensus.md).

## References

1. [State machine replication](https://en.wikipedia.org/wiki/State_machine_replication)
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
4. Dominik Tornow, _Think Distributed Systems_, Manning Publications
