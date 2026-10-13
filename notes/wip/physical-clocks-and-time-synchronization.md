# Lesson 54: Physical Clocks and Time Synchronization

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Every server has its own clock, and no two clocks agree exactly. Last-write-wins conflict resolution, Snowflake IDs, lease expiry and TTLs all rely on wall-clock time, so we need to know how far we can trust it.

## Outline

- Quartz clocks, drift and skew
- Time-of-day clocks vs monotonic clocks, and which to use for timeouts
- NTP: how it works and what accuracy to expect
- Leap seconds and clock smearing
- Clocks jumping backwards, and the bugs this causes (LWW, Snowflake IDs, leases)
- Bounded clock uncertainty: Google's TrueTime and commit wait

## Related lessons

- [Lesson 25: Common Conflict Resolution Strategies](../2026-06-29-Common-Conflict-Resolution-Strategies.md)
- [Lesson 40: Design a Unique ID Generation System](../2025-08-14-unique-id-generation.md)
- [Lesson 53: Snowflake IDs](2026-09-25-SNOWFLAKE-ID.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
3. David L. Mills et al., _Network Time Protocol Version 4_, RFC 5905
4. James C. Corbett et al., _Spanner: Google's Globally-Distributed Database_, OSDI 2012
