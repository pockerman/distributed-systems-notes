# Little's Law

## Overview

In this note we discuss Little's law,
a simple result from queueing theory which states that the average number of items in a system equals the average arrival rate multiplied by the average time each item spends in the system [1].

## Little's law

Little's law was proved by John Little in 1961, and it provides a powerful way to analyze queues and performance, both in everyday scenarios and in computer systems [1].

Little's law links three key metrics:

- Throughput
- Time (latency)
- Concurrency

Mathematically, Little's law states [1]:

$$ L = \lambda \times W$$

where:

- $L$ is the average number of items in the system at the same time, i.e. the load or concurrency;
- $\lambda$ is the average rate at which items arrive at (and, in a stable system, leave) the system, e.g. the requests per second the server receives;
- $W$ is the average time each item spends in the system, e.g. the response time per request.

::: {.callout-note title="Work in progress"}
The average number of items in a system is sometimes called work in progress, or WIP.
:::

There are two things to be aware of. First, the law holds for a stable system, i.e. one where, on average, items leave as fast as they arrive. Second, the law does not depend on the distribution of the arrival or service times, or on the order in which items are served [1].

How can we use Little's law when designing distributed systems? Little's law connects throughput, latency, and concurrency, which are exactly the metrics we care about when building scalable systems. It provides a quick reality check and estimation tool for
system capacity, and helps identify bottlenecks.

Below is an example from [1]. Consider a web API that receives 150 requests per second, where each request takes 0.35 seconds to complete on average.
How many requests is the API processing concurrently? This is a direct application of Little's law:

$$L = 150 \times 0.35 = 52.5$$

Now assume that a database call slows down and the average response time increases to 1.5 seconds.
With the same arrival rate, the concurrency becomes:

$$L = 150 \times 1.5 = 225$$

The number of requests in flight has gone up more than four times. If the server can only handle, say, 100 concurrent requests (e.g. because of its thread pool size), it will now start queueing or rejecting requests, even though the arrival rate hasn't changed.

## Summary

::: {.callout-note title="Summary"}
Little's law is a fundamental result from queueing theory that relates concurrency, throughput, and latency in a system through the equation $L = \lambda \times W$, where $L$ is the average number of items in the system (work in progress), $\lambda$ is the average arrival rate, and $W$ is the average time an item spends in the system. The law applies to any stable system, regardless of the arrival or service-time distributions, which makes it very useful for scalability analysis. By connecting throughput and response time to system load, Little's law gives a practical way to estimate capacity and identify bottlenecks. The web API example shows how an increase in latency, at the same arrival rate, sharply raises concurrency and therefore load.
:::

## References

1. [What is Little's Law and How to Use It for Quick Capacity Estimates in System Design](https://www.designgurus.io/course-play/grokking-scalable-systems-for-interviews/doc/what-is-littles-law-and-how-to-use-it-for-quick-capacity-estimates-in-system-design)
2. [Little's law](https://en.wikipedia.org/wiki/Little%27s_law)
