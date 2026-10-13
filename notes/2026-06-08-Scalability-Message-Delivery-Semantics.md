# Message Delivery Semantics

::: {.callout-warning title="Work in progress"}
This note has not been written yet.
:::

## Overview

In this note we will discuss message delivery semantics: at-most-once, at-least-once, and exactly-once delivery. Asynchronous communication via message queues is one way to improve the scalability of a system (see load leveling in [Upstream Resiliency](2026-02-09-Upstream-Resiliency.md)), and the delivery guarantees that the messaging system provides determine how consumers must handle lost and duplicate messages.
