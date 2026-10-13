# Lesson 63: Conflict-Free Replicated Data Types

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

CRDTs are data types whose replicas can be updated independently and always merge to the same state. They give strong eventual consistency without coordination.

## Outline

- Strong eventual consistency
- State-based vs operation-based CRDTs
- Counters: G-Counter and PN-Counter
- Registers: LWW-Register and MV-Register
- Sets: G-Set, 2P-Set and OR-Set
- CRDTs in practice: Riak, Redis Enterprise, collaborative editing (Automerge, Yjs)

## Related lessons

- [Lesson 25: Common Conflict Resolution Strategies](../2026-06-29-Common-Conflict-Resolution-Strategies.md)
- [Lesson 24: PACELC, CAP and CALM Theorems](../2026-01-06-pacelc-cap-calm-theorems.md)
- [Lesson 55: Logical Clocks](logical-clocks.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Marc Shapiro et al., _Conflict-free Replicated Data Types_, SSS 2011
