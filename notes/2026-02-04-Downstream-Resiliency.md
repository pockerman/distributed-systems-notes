# Lesson 33: Downstream Resiliency

## Overview

[Lesson 32: Failure Causes](2026-01-28-Failure-Causes.md) introduced
some common failures in a distributed system. In this lesson we discuss techniques to address them. Specifically,
we assume that our system depends on another service that we don't necessarily control. How should our system behave when that
service is down or slow?

## Downstream resiliency

A deployed distributed system in most cases interacts with third-party services. For various reasons, these interactions
will not always go smoothly. We want mechanisms that stop our system from degrading to a state where
it can no longer function. These mechanisms include [1]:

- Timeouts
- Retries
- Fallbacks
- [Circuit breakers](https://github.com/pockerman/qubit-notes/blob/main/software_engineering/2025-04-30-circuit-breaker-pattern.md)

Let's briefly discuss each of them.

### Timeouts

One of the simplest ways to protect a system from misbehaving third-party services is to use timeouts. A timeout specifies how long
a client waits for a response from the service. If no response arrives within
that interval, the client aborts the request. A timeout is a hard boundary, and it is very simple to implement: most libraries that
make network requests let us set one. Here is an example using Python's `requests` package:

```python
import requests
response = requests.get(url='https://get-something.com', timeout=10)
```

As a system designer, you should assume that any network call can hang, and assign a timeout to every single one.
A network request has two distinct phases:

- Establishing the connection (the TCP and TLS handshakes)
- Waiting for the server to process the request and return the data

Most HTTP libraries let you set a separate timeout for each phase.
For example, with Python's `requests` library we can pass a tuple, `timeout=(2, 8)`, meaning a 2-second connect timeout and an 8-second read timeout.

The major difficulty with timeouts is choosing the right value. Ideally, we should set the timeout based on the
desired false timeout rate [1]: for example, if we can accept 0.1% of requests timing out spuriously, we set the timeout to the 99.9th percentile of the downstream service's response time. A rule of thumb for the two phases is:

- Connection timeout (e.g. 2 seconds): a healthy server should accept a connection almost instantly. If you can't even establish a connection within this time, the server is likely down or the network is broken. Fail fast.
- Read timeout (e.g. 8 seconds): once connected, the server may need more time to query its database and produce the response.

Where do these numbers come from?
In a production environment, you don't pick them on gut feeling. You derive them from historical monitoring data (latency percentiles) and from the latency targets the downstream service commits to.
For example, if monitoring shows that 99.9% of healthy responses arrive within 6 seconds, an 8-second read timeout is a defensible, data-driven choice rather than a guess.

A Service Level Agreement (SLA) is a formal contract between a service provider (such as a payment processor or cloud platform) and its customers that defines the expected service level, often with financial penalties when it is not met.
An SLA usually covers availability (e.g., "the service will be available 99.9% of the time"), and it may also include latency targets. For example, if a provider promises that 95% of requests are processed in under 500 ms, you can use that promise to inform your timeouts. Timeouts that fire because the provider broke its promise also give you evidence when you hold the vendor accountable.

### Retries

There are many reasons why a request may fail. Fault-tolerant applications typically don't give up immediately but retry the request.
Many failures, such as most failures in the cloud, are transient. However, if the downstream service is overloaded, retrying immediately will only add to its load. Retries should therefore be spaced out with increasingly longer delays [1]. We also need to cap the number of retries. A common way to compute the
delay between retries is [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) [1].

::: {.callout-note title="Exponential backoff"}
Exponential backoff is a resilience strategy in which an application progressively increases the wait time between retry attempts. Instead of repeatedly hammering a struggling service (which can turn into a self-inflicted denial of service, a so-called retry storm), the client multiplies the delay after each failed attempt, e.g. waiting 1 second, then 2, then 4, then 8. This gives the overwhelmed downstream service room to recover. Binary exponential backoff was popularized by Bob Metcalfe and David Boggs's Ethernet work; see [Ethernet: Distributed Packet Switching for Local Computer Networks](http://www.bitsavers.org/pdf/xerox/parc/techReports/CSL-75-7_Ethernet_Distributed_Packet_Switching_for_Local_Computer_Networks.pdf), published in 1976.
:::

Here is what exponential backoff looks like:

```
Attempt 1 fails. Wait 1 second.
Attempt 2 fails. Wait 2 seconds.
Attempt 3 fails. Wait 4 seconds.
Attempt 4 fails. Wait 8 seconds.
```

In practice, we also cap the delay at a maximum value and add random _jitter_ to it [1]. Without jitter, all the clients that failed at the same moment retry at the same moments too, producing synchronized load spikes on the recovering service.

::: {.callout-warning title="Retries and idempotency"}
Retries are powerful, but they are dangerous for operations with side effects. If you retry a payment request, you must make sure you don't charge the customer twice. To do this safely, generate a unique idempotency key for the operation once, in your business layer, and send the same key with every retry attempt. The payment processor can then recognize and ignore duplicates. See [Lesson 40: Replay Attack vs Idempotency Issues](2026-07-30-Replay-Attack-Idempotency.md).
:::

### Fallbacks

What happens when the timeout is reached and all the retries have failed? This is where a fallback comes in.

A fallback is a resilience pattern that provides a pre-programmed alternative response when the primary operation fails. Instead of letting an external error crash the application or show a broken screen to the user, a fallback gracefully degrades the experience by returning default data, a previously cached response, or the result of a simplified business rule.

It is your "plan B", designed to keep the system functional, and to save the business transaction, even when a downstream dependency is broken.

- Example 1: if a video streaming app cannot reach its recommendation service to build a personalized homepage, the fallback is to return a static list of the most popular titles. The user can still watch something.
- Example 2: if an e-commerce site cannot reach the live shipping calculator, the fallback is to charge a flat shipping fee. This is a business trade-off: you may lose a few dollars on postage, but you keep a \$100 sale that would otherwise have been lost.

Timeouts, retries with exponential backoff, and fallbacks together make an application far more resilient.

But what happens when the external service isn't having a temporary hiccup, but is completely down? That is when we need a circuit breaker.

### Circuit breaker

Timeouts and retries are great for temporary network blips. But what happens if the third-party payment API you rely on is offline for an hour?
If you rely only on timeouts, every customer who tries to check out stares at a loading spinner for 10 seconds before the timeout finally fires. During those 10 seconds, a thread on your server is blocked. If thousands of users check out at the same time, your servers run out of threads, memory usage spikes, and your entire application may crash.

A failure in a downstream service has then cascaded upstream and brought your system down.

To stop such cascading failures, we use the [circuit breaker](https://github.com/pockerman/qubit-notes/blob/main/software_engineering/2025-04-30-circuit-breaker-pattern.md) pattern.
In a nutshell, after detecting repeated failures, a circuit breaker temporarily blocks calls to the faulty service and fails them immediately.
This prevents attempts that are very likely to fail, frees the caller's resources, and gives the downstream service time to recover. After a while, the breaker lets a few trial requests through to check whether the service is healthy again.
See the [circuit breaker pattern](https://github.com/pockerman/qubit-notes/blob/main/software_engineering/2025-04-30-circuit-breaker-pattern.md) note for more details.

## Summary

::: {.callout-note title="Summary"}
Downstream resiliency is about how a system should behave when it depends on external services that may be slow, unreliable, or unavailable. The goal is to prevent failures in downstream services from cascading and degrading the entire system. There are four main mechanisms:

1. Timeouts
   * Define how long the system waits for a downstream service; if there is no response in time, the request is aborted.
   * Prevent resources such as threads from being held indefinitely.
   * Should be based on monitoring data (latency percentiles) and SLA targets rather than arbitrary values.
2. Retries
   * Failed requests can be retried because many failures are transient.
   * Retries need a maximum number of attempts and increasing delays: exponential backoff (e.g. `1s → 2s → 4s → 8s`) with jitter.
   * Retrying operations with side effects, such as payments, can cause duplicates; idempotency keys make them safe to retry.
3. Fallbacks
   * When the primary call fails after timeouts and retries, the system uses an alternative response: cached data, default values, or simplified business logic.
   * The objective is graceful degradation rather than failing the user's request.
4. Circuit breakers
   * Used when a downstream service is persistently failing.
   * Temporarily block requests to that service instead of repeatedly sending requests that are likely to fail.
   * Prevent cascading failures, where one unavailable service exhausts the resources of your own application.
:::

Putting it all together:

```
Request
   │
   ▼
Circuit breaker open? ── Yes ──► Fallback
   │ No
   ▼
Call downstream service (with timeout)
   │
   ├── Success ──────────────────► Response
   │
   └── Failure / timeout
          │
          ▼
       Retry with backoff + jitter
          │
          ├── Success ───────────► Response
          │
          └── Retries exhausted
                 │
                 ▼
              Fallback
     (repeated failures trip the circuit breaker)
```

In the next lesson, [Lesson 34: Upstream Resiliency](2026-02-09-Upstream-Resiliency.md), we see how to protect our system from incoming requests that push it to its limits.

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
