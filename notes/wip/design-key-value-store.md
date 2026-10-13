# Lesson 77: Design a Distributed Key-Value Store

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

A capstone exercise that puts together partitioning, replication, quorums, conflict resolution and anti-entropy, in the style of Dynamo.

## Outline

- Requirements and the CAP/PACELC choice
- Partitioning with consistent hashing
- Replication and quorums (`R + W > N`)
- Versioning and conflict resolution
- Hinted handoff, read repair and Merkle trees
- Failure detection with gossip; the write and read paths

## Related lessons

- [Lesson 27: Partitioning](../2025-08-04-partitioning.md)
- [Lesson 20: Replication Strategies](../2025-04-22-data-replication.md)
- [Lesson 26: Read Repair, Hinted Handoff & Anti-entropy in Eventually Consistent Systems](../2026-07-01-Read-Repair-Hinted-Handoff-Anti-entropy-Eventually-Consistent-Systems.md)

## References

1. Alex Xu, _System Design Interview – An Insider's Guide_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
3. Giuseppe DeCandia et al., _Dynamo: Amazon's Highly Available Key-value Store_, SOSP 2007
