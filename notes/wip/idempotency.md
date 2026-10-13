# Lesson 62: Idempotency

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Retries, at-least-once delivery and impatient users all lead to the same request being processed twice. Idempotency makes the second attempt harmless.

## Outline

- Naturally idempotent operations vs operations that need help
- Idempotency keys: generation, storage and expiry
- Deduplication stores and their race conditions
- Idempotent consumers for message queues
- Returning the original response on a duplicate
- Examples: payment APIs (e.g. Stripe), HTTP PUT vs POST

## Related lessons

- [Lesson 39: Replay Attack vs Idempotency Issues](../2026-07-30-Replay-Attack-Idempotency.md)
- [Lesson 32: Downstream Resiliency](../2026-02-04-Downstream-Resiliency.md)
- [Lesson 45: Message Delivery Semantics](2026-06-08-Scalability-Message-Delivery-Semantics.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
