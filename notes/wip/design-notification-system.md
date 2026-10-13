# Lesson 79: Design a Notification System

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Send push, SMS and email notifications reliably, without sending the same notification twice or flooding users.

## Outline

- Requirements and notification channels
- Third-party providers (APNs, FCM, SMS and email gateways)
- Queues per channel and workers
- Retries, deduplication and idempotency
- User preferences, rate limits and templates
- Tracking delivery

## Related lessons

- [Lesson 62: Message Queues and Event Logs](message-queues-and-logs.md)
- [Lesson 63: Idempotency](idempotency.md)
- [Lesson 42: Design a Rate Limiter](../2026-05-21-Resiliency-Hands-On-Design-Rate-Limiter.md)

## References

1. Alex Xu, _System Design Interview – An Insider's Guide_
