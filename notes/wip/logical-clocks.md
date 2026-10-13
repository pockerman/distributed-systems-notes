# Lesson 55: Logical Clocks

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

When physical clocks cannot be trusted, we can still order events by causality. Logical clocks capture the happened-before relation without reading the time of day.

## Outline

- The happened-before relation and concurrent events
- Lamport clocks and total ordering with tie-breaking
- Vector clocks and detecting concurrent writes
- Version vectors and dotted version vectors in replicated stores
- Hybrid logical clocks (HLC)
- Worked example: two clients writing the same key

## Related lessons

- [Lesson 25: Common Conflict Resolution Strategies](../2026-06-29-Common-Conflict-Resolution-Strategies.md)
- [Lesson 21: Consistency Models](../2025-04-28-consistency-models.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
3. Leslie Lamport, _Time, Clocks, and the Ordering of Events in a Distributed System_, Communications of the ACM, 1978
4. Sandeep S. Kulkarni et al., _Logical Physical Clocks and Consistent Snapshots in Globally Distributed Databases_, 2014
