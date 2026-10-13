# Lesson 59: Distributed Transactions: Two-Phase and Three-Phase Commit

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Once data is partitioned, one business operation may need to update several partitions or services. Atomic commit protocols make all of them commit or none.

## Outline

- Atomic commitment and why it is hard
- Two-phase commit: prepare and commit
- Coordinator failure and blocking
- Three-phase commit and its assumptions
- XA transactions and their cost
- Consensus-based commit (Spanner, CockroachDB)

## Related lessons

- [Lesson 27: Partitioning](../2025-08-04-partitioning.md)
- [Lesson 28: SQL Isolation Levels](../2026-08-20-SQL-ISOLATION-LEVELS.md)
- [Lesson 23: Consensus](../2025-12-31-consensus.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Philip A. Bernstein, Vassos Hadzilacos and Nathan Goodman, _Concurrency Control and Recovery in Database Systems_, Addison-Wesley
