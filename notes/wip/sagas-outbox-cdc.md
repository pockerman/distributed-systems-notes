# Lesson 61: Sagas, the Transactional Outbox and Change Data Capture

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Between microservices, two-phase commit is usually too slow or not available. Sagas split a transaction into local steps with compensations, and the outbox pattern and CDC publish those steps reliably.

## Outline

- Sagas: local transactions and compensating actions
- Choreography vs orchestration
- The dual-write problem
- The transactional outbox pattern
- Change data capture from the write-ahead log (Debezium)
- Isolation anomalies in sagas and how to limit them

## Related lessons

- [Lesson 60: Distributed Transactions: Two-Phase and Three-Phase Commit](two-phase-commit.md)
- [Lesson 31: Write-Ahead Log](../2026-08-21-WRITE-AHEAD-LOG.md)
- [Lesson 46: Message Delivery Semantics](2026-06-08-Scalability-Message-Delivery-Semantics.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Sam Newman, _Building Microservices_, O'Reilly Media
3. Hector Garcia-Molina and Kenneth Salem, _Sagas_, SIGMOD 1987
4. Chris Richardson, _Microservices Patterns_, Manning Publications
