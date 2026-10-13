# Lesson 59: Distributed Locks and Leases

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

A lock across machines must cope with clients that pause, crash or lose their network while holding it. Leases and fencing tokens are the tools that make this safe.

## Outline

- Locks vs leases: why every distributed lock needs an expiry
- Process pauses (GC, VM migration) and the stale lock holder
- Fencing tokens
- Locks with etcd/ZooKeeper vs Redis
- The Redlock debate
- When to avoid locks: idempotency and compare-and-set

## Related lessons

- [Lesson 24: Consensus](../2025-12-31-consensus.md)
- [Lesson 55: Physical Clocks and Time Synchronization](physical-clocks-and-time-synchronization.md)
- [Lesson 58: Leader Election](leader-election.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Martin Kleppmann, _How to do distributed locking_, 2016 (blog post)
