# Lesson 15: Cold & Warm Starts

## Overview

In this lesson we review two ideas associated with scalability and system behaviour:
cold starts and warm starts.

## Cold and warm starts

Conceptually, cold and warm starts are easy to understand. A cold start refers to starting a system, application, or function from scratch, with no pre-existing state, whereas a warm start means reusing a system that is already initialized or cached, resulting in much faster startup and response times [1].

When it comes to performance, cold starts are best avoided, as they introduce extra latency.
During a cold start, a system typically has to load code, establish database connections, read configuration files, and fill empty caches. Warm starts don't have this overhead, so the system can respond quickly.
Cold starts typically exhibit the following characteristics [1]:

- They require full initialization.
- They have high latency on the first request.

::: {.callout-note title="Remark"}
A cold start occurs when something runs for the first time, or when it hasn't been used for a while [1].
An example of the latter is serverless platforms like AWS Lambda: Lambda performs a cold start for a function invoked after a period of inactivity, or when it scales up to handle more traffic. Similarly, a database cache goes cold after a restart or failover, meaning the cache has to fill up again from empty [1].
:::

Warm starts are preferable for latency and throughput. However, keeping things warm consumes memory or compute resources continuously. For example, keeping a cache warm means using extra memory to store data, and keeping a server always on (to avoid a cold boot) incurs costs. If the performance gains outweigh the costs, keeping things warm is worth it.

Some places where cold and warm starts show up, some of which we have already mentioned:

- System boots
- Launching an application
- Serverless functions
- Web and database caches

## Summary

::: {.callout-note title="Summary"}
Cold and warm starts are about initialization state and performance. A cold start initializes everything from scratch and pays a latency penalty on the first requests, while a warm start reuses already-initialized state and responds quickly, at the cost of keeping resources allocated.
Understanding cold versus warm starts matters for scalability, because it affects how an architecture handles growth and load.
A scalable system should minimize the impact of cold starts, so that adding capacity or handling sporadic traffic doesn't degrade the user experience.
For example, load balancers can route traffic so that some servers stay warm, and cloud services offer auto-scaling with warm pools (pre-initialized instances ready to take traffic).
In addition, [cache warming](https://www.geeksforgeeks.org/system-design/what-is-cache-warming/) can be used after a deployment so that users don't hit empty caches.
By anticipating cold start costs, architects can keep the system responsive during scale-ups and deployments.
:::

## References

1. [What are Cold Starts and Warm Starts, and Why Do They Matter for Performance?](https://www.designgurus.io/course-play/grokking-scalable-systems-for-interviews/doc/what-are-cold-starts-and-warm-starts-and-why-do-they-matter-for-performance)
