# Lesson 33: Upstream Resiliency

## Overview

In [Lesson 32: Downstream Resiliency](2026-02-04-Downstream-Resiliency.md) we discussed
techniques for handling downstream failures, i.e. failures caused by an
interaction with another service.

In this lesson we look into how to protect our own service from the clients that call it, e.g. when the service is overloaded. Specifically,
we look into:

- [Load shedding](https://en.wikipedia.org/wiki/Load_shedding)
- Load leveling
- [Rate limiting](https://en.wikipedia.org/wiki/Rate_limiting)
- Bulkheads
- Health checks
- Watchdogs

## Upstream resiliency

Let's start with load shedding.

### Load shedding

Load shedding means rejecting requests when the system is overloaded. What it means for a system to be overloaded depends on
the system at hand, e.g. the number of concurrent requests or the CPU utilization [1]. When a server detects that it is overloaded, it can reject incoming requests by returning `503 Service Unavailable`.
Note that load shedding cannot completely remove the cost of handling a request: the server still has to accept the connection and decide to reject it [1]. How much work this takes depends
on how the rejection is implemented.

### Load leveling

When the clients of our service do not expect a response within a short time, we can use load leveling instead of load shedding.
The idea is to introduce a messaging channel, such as a queue, between the clients and the service. The clients push messages
to the channel instead of calling the service directly, and the service pulls messages at its own pace [1].

Neither load leveling nor load shedding handles an increase in load by itself. They protect the service from getting overloaded.
To handle more requests, the service has to be scaled out [1]. These mechanisms are therefore typically combined with autoscaling [1].

### Rate limiting

With rate limiting, or throttling, the server rejects a request when a specific quota is exceeded [1].
Quotas are typically applied per user, API key, or IP address [1]. When a request is rate-limited, the server returns
a response with a specific error code. For HTTP APIs, the most common choice is status code `429 Too Many Requests`.
The response can also include a `Retry-After` header indicating how long to wait before making a new request [1].

Rate limiting is also used to enforce pricing tiers and, to some extent, to help against DDoS attacks [1]. It is only a partial defence against DDoS, because the server still has to process each request to reject it.

::: {.callout-note title="Rate limiting, throttling and quotas"}
The text above uses the terms rate limiting, throttling, and quotas somewhat interchangeably.
Technically, they are different. See [Difference between Rate Limiting and Throttling and Quotas](https://github.com/pockerman/qubit-notes/blob/main/software_engineering/2026-05-06-API-Basics-Diff-Rate-Limit-Throttling-Quotas.md).
:::

We discuss rate limiting further in [Lesson 41: Design a Rate Limiter](2026-05-21-Resiliency-Hands-On-Design-Rate-Limiter.md).

### Bulkhead

The bulkhead pattern, named after the watertight partitions of a ship's hull, isolates a fault in one part of a service so that it cannot degrade the entire service [1].
Some clients put much more load on a service than others. Without a mechanism to isolate such a client, it can degrade the whole
system. We have seen that rate limiting can prevent a single client from using more resources than it should. However, rate limiting is not bulletproof.

The bulkhead pattern partitions a shared resource, e.g. the servers behind a load balancer, and assigns each user of the service to a specific partition.
The requests of a greedy user can then only consume the resources of the partition it has been assigned to. The greedy user will still
affect the other users in the same partition, but it cannot degrade the rest of the system.

### Health checks

A health check lets a load balancer find out whether a service instance is able to serve requests, and stop sending traffic to an instance that is
unhealthy or operating at its limit. The service exposes a health endpoint that the
load balancer queries periodically. If the endpoint returns an error, the load balancer considers the instance unhealthy and stops routing traffic to it [1].

Two common health checks are [liveness and readiness checks](2026-05-13-Liveness-Readiness-Checks.md).

### Watchdog

A watchdog is a background thread that monitors the running process. It wakes up periodically and checks various aspects of the
process, such as its memory consumption. When a metric exceeds a configured threshold, the watchdog restarts the
process [1]. This keeps the system working, for example in the presence of a slow memory leak, and gives the engineering team time to find the bug.
A watchdog has to be tested thoroughly; otherwise we may end up restarting the process continuously.

## Summary

::: {.callout-note title="Summary"}
Upstream resiliency is about protecting a service from the load its clients put on it. The main patterns are:

- Load shedding: reject requests with `503` when the service is overloaded.
- Load leveling: put a message queue between clients and the service so that the service processes requests at its own pace.
- Rate limiting: enforce per-client quotas and reject excess requests with `429`, optionally with a `Retry-After` header.
- Bulkheads: partition resources so that a noisy or greedy client can only degrade its own partition.
- Health checks: let load balancers stop sending traffic to unhealthy instances.
- Watchdogs: monitor a process and restart it when critical thresholds are exceeded.

Together, these techniques protect system stability by rejecting excess load, smoothing traffic, isolating failures, and enabling detection and recovery. They are usually combined with autoscaling.
:::

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
