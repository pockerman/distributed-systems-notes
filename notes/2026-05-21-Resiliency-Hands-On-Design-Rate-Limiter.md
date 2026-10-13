# Design a Rate Limiter

## Overview

We looked into various techniques for improving the upstream resiliency of a system in [Upstream Resiliency](2026-02-09-Upstream-Resiliency.md).
One of them was rate limiting. Since rate limiting is ubiquitous in modern systems, we discuss it a bit further here,
and look at how we could design a rate limiter.

## Rate limiting

Recall from [Upstream Resiliency](2026-02-09-Upstream-Resiliency.md) that with rate limiting, or throttling, the server rejects a request when a specific quota is exceeded [1].
Whether the threshold is enforced strictly, or requests are allowed to exceed it for a short period of time, distinguishes hard from soft rate limiting [2].
We can apply rate limiting at various levels of the stack, for example at the HTTP layer, or by IP address at the network layer using iptables, see e.g. [3].
There are a number of algorithms for implementing rate limiting [2]:

- Token bucket
- Leaking bucket
- Fixed window counter
- Sliding window log
- Sliding window counter

As a client, you should try not to get rate limited, as this affects the user experience. In particular:

- Know the limits of the external APIs you are using.
- Use caches to avoid frequent API calls.
- Add sufficient backoff time to your retry logic (see [Downstream Resiliency](2026-02-04-Downstream-Resiliency.md)).
- Always catch and handle the errors returned when you are rate limited.

::: {.callout-note title="Rate limiter headers"}
When rate limiting at the HTTP level, you typically return status code `429 Too Many Requests`.
The server can also tell the client more about its limits via HTTP headers. [2] uses the following:

- `X-Ratelimit-Remaining`: the remaining number of allowed requests within the window.
- `X-Ratelimit-Limit`: how many calls the client can make per time window.
- `X-Ratelimit-Retry-After`: the number of seconds to wait before making a request again without being rate limited.

When a user has sent too many requests, the server returns `429` together with the retry-after information.

The `X-` headers are a convention rather than a standard. The standard way to tell a client when to retry is the `Retry-After` header ([RFC 9110](https://www.rfc-editor.org/rfc/rfc9110#field.retry-after)), and the IETF is standardizing `RateLimit` and `RateLimit-Policy` headers for the rest [4].
:::

## Designing a rate limiter

Let's now see how we could design a rate limiter.

<!-- TODO: write the design (requirements, algorithm choice, where to place the limiter, storage of counters e.g. Redis, distributed concerns such as race conditions and synchronization). -->

## Summary

::: {.callout-note title="Summary"}
Rate limiting rejects requests once a client exceeds its quota, protecting a service from overload and abuse. Limits can be hard or soft, and can be enforced at different layers, from the network (e.g. iptables) to the HTTP layer. Common algorithms are the token bucket, leaking bucket, fixed window counter, sliding window log, and sliding window counter. HTTP services signal rate limiting with status `429` and headers that tell the client its limits and when to retry, ideally the standard `Retry-After`. Clients, in turn, should know the limits, cache responses, and back off when they are limited.
:::

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Alex Xu, _System Design Interview: An insider's guide_, second edition
3. [Rate Limit Requests with Iptables](https://blog.programster.org/rate-limit-requests-with-iptables)
4. [RateLimit header fields for HTTP](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/), IETF draft
