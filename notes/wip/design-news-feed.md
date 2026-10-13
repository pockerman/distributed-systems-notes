# Lesson 76: Design a News Feed

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Show each user a timeline of posts from the people they follow. The main decision is when to build the feed: on write or on read.

## Outline

- Requirements and estimates
- Fan-out on write vs fan-out on read
- The celebrity problem and hybrid fan-out
- Feed storage and caching
- Ranking
- Pagination with cursors

## Related lessons

- [Lesson 62: Message Queues and Event Logs](message-queues-and-logs.md)
- [Lesson 17: Caching Methods](../2026-01-20-Caching.md)
- [Lesson 16: Sharded Counters](../2025-08-11-sharded-counters.md)

## References

1. Alex Xu, _System Design Interview – An Insider's Guide_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
