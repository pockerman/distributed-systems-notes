# Lesson 20: Replication Strategies

## Overview

Data replication is one of the techniques we can employ to improve the latency and availability of a distributed system [2].
In this series of lessons we go over the essential elements of data replication. Specifically, we discuss the following ideas:

- Replication strategies (this lesson)
- [Consistency models](2025-04-28-consistency-models.md)
- [State machine replication](2025-12-30-state-machine-replication.md)
- [Lesson 23: Consensus](2025-12-31-consensus.md)
- [Broadcast protocols](https://github.com/pockerman/qubit-notes/blob/main/software_engineering/2026-01-05-broadcast-protocols.md)
- [PACELC, CAP and CALM theorems](2026-01-06-pacelc-cap-calm-theorems.md)

In this lesson we discuss what data replication is
and go over some common replication strategies.

## Data replication

As the name implies, data replication is the process of creating and maintaining multiple copies of the same data across different locations, systems, or servers.

Why do we need data replication? Data replication improves the availability, reliability, performance, and fault tolerance of a distributed system. It's easy to see why: storing data on a single machine introduces a single point of failure (SPF). Keeping copies on several machines allows the system to survive the loss of one of them. Placing copies closer to the users also reduces access latency.

However, data replication is not easy, primarily because we need to keep the copies consistent. Consistency models, such as eventual consistency, and replication algorithms, like Raft,
play a key role here [3]. Also central to the discussion of replication is _replication transparency_, i.e. the ability of the system to hide the existence of multiple copies and provide the
illusion of a single object [1].

### Replication strategies

There are two main replication approaches: synchronous and asynchronous.
In synchronous replication, the data is copied to the replicas as part of the operation itself. An operation is not complete until the data has been copied and acknowledgments have been received from all the participating nodes [1]. This ensures consistency but adds latency, and a single slow or unavailable replica can block writes.
In asynchronous replication, the data is copied in the background after the operation has completed. This allows for faster responses, but replicas may temporarily hold outdated data, and acknowledged writes can be lost if the leader fails before they are replicated.
In practice, we can use a hybrid approach where a majority of the nodes has to acknowledge the operation. This majority is often called a quorum [1].
The remaining replicas receive the update asynchronously.

On top of the synchronous/asynchronous distinction, we can classify strategies by which nodes accept writes [1]:

- Single-leader systems
- Multi-leader systems
- Leaderless systems

In systems with leaders, the leader nodes handle writes (and possibly reads), while the
follower nodes typically serve only reads.
In a single-leader system there is just one leader that coordinates the operation.
The leader accepts writes and propagates them to the followers. This is a simple configuration,
but the leader is a single point of failure for writes until a new leader is elected (failover), which partly undermines the point of replication.
Multi-leader systems have more than one leader. Although this avoids the single point of failure, conflicting concurrent writes may arise and need resolution [1].
Finally, in leaderless systems there is no leader that coordinates operations, and any node can accept reads and writes.
Just like multi-leader systems, this creates conflicts that we need to detect and resolve. Common approaches are [4]:

- Last-Write-Wins (LWW)
- Version vectors (or vector clocks), which detect concurrent writes so they can be merged
- Operational Transformation (OT)
- Conflict-free Replicated Data Types (CRDTs)

See [Lesson 25: Common Conflict Resolution Strategies](2026-06-29-Common-Conflict-Resolution-Strategies.md) for more details. The next lesson discusses [consistency models](2025-04-28-consistency-models.md).

## Summary

::: {.callout-note title="Summary"}
Data replication is the process of maintaining multiple copies of the same data across different locations or servers to improve availability, reliability, performance, and fault tolerance. It can be performed synchronously, ensuring consistency at the cost of latency, or asynchronously, which offers faster responses but may lead to temporary inconsistencies; quorum-based replication sits in between. Replication strategies also differ in which nodes accept writes: single-leader systems (simple, but the leader is a single point of failure for writes until failover), multi-leader systems (more resilient but prone to conflicts), and leaderless systems (fully distributed but requiring robust conflict resolution). Common conflict handling methods include last-write-wins, version vectors, operational transformation, and conflict-free replicated data types (CRDTs).
:::

## References

1. Dominik Tornow, _Think Distributed Systems_, Manning Publications
2. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
3. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
4. [What are some methods for conflict resolution in distributed databases?](https://milvus.io/ai-quick-reference/what-are-some-methods-for-conflict-resolution-in-distributed-databases)
