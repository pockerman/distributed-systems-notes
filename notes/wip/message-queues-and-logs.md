# Lesson 62: Message Queues and Event Logs

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Message brokers decouple producers from consumers and absorb load spikes. This lesson compares the two main models, queues and append-only logs, and the problems each brings.

## Outline

- Point-to-point queues vs publish/subscribe
- Queues (RabbitMQ, SQS) vs logs (Kafka)
- Partitions, consumer groups and ordering
- Acknowledgements, redelivery and dead-letter queues
- Backpressure and consumer lag
- Retention, compaction and replaying events

## Related lessons

- [Lesson 46: Message Delivery Semantics](2026-06-08-Scalability-Message-Delivery-Semantics.md)
- [Lesson 34: Upstream Resiliency](../2026-02-09-Upstream-Resiliency.md)
- [Lesson 31: Write-Ahead Log](../2026-08-21-WRITE-AHEAD-LOG.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Jay Kreps, _The Log: What every software engineer should know about real-time data's unifying abstraction_, 2013 (blog post)
