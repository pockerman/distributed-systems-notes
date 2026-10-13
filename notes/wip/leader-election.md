# Lesson 58: Leader Election

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Many systems need exactly one node to do a job: accept writes, run a scheduled task or own a partition. Leader election picks that node and replaces it when it fails.

## Outline

- Why we need a single leader
- The bully and ring algorithms (for intuition)
- Election in Raft: terms, votes and randomized timeouts
- Delegating election to a coordination service (ZooKeeper, etcd)
- Split brain and how to detect it
- Cost of a leader: bottlenecks and failover time

## Related lessons

- [Lesson 24: Consensus](../2025-12-31-consensus.md)
- [Lesson 23: State Machine Replication](../2025-12-30-state-machine-replication.md)
- [Lesson 21: Replication Strategies](../2025-04-22-data-replication.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
3. Diego Ongaro and John Ousterhout, _In Search of an Understandable Consensus Algorithm_, USENIX ATC 2014
