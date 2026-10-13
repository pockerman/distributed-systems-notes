# Caching Part 3 | Cache Replacement

## Overview

[Caching Part 2 | Cache Coherence](2026-01-21-cache-coherency.md) discussed
cache coherence, i.e. how multiple caches can maintain a uniform view of the cached data. A cache, however, has a limited capacity, and when this
capacity is reached we must decide what to remove. In this note we review some [cache replacement policies](https://en.wikipedia.org/wiki/Cache_replacement_policies):

- Least recently used (LRU)
- Least frequently used (LFU)
- First-in, first-out (FIFO)
- SIEVE

We also discuss time-to-live (TTL) expiry.

## Cache replacement

The purpose of caching is to reduce access latency by keeping a temporary copy of the data that is faster to access than the
primary storage [2]. The caching layer is therefore typically located close to where the data is accessed.
Cache memory is, however, more expensive than disk storage; for
example, the price per gigabyte of DRAM is about ten times higher than that of SSDs [2].
As a result, we can only cache a limited portion of the data.

When the cache is full, some elements have to be evicted to make room for new ones.
The cache replacement policy determines which elements to evict.
Choosing the right policy is critical for maintaining a high cache hit ratio and therefore low-latency
access, and the best choice is workload-specific [2]. Typically, replacement policies are based on how long an element
has been in the cache, how recently it was used, or how frequently it is accessed. Below are some commonly used policies.

### Least recently used (LRU)

The least recently used (LRU) policy evicts the elements that have not been used for the longest time.
When a cache element is accessed, it is eagerly promoted by moving it to the head of the cache, so that it stays in the cache.
LRU maintains a high hit ratio for workloads with high
temporal locality: if recently accessed data is likely to be accessed again
shortly, LRU keeps that data in the cache. On the other
hand, if the workload has little locality, LRU can perform worse than not caching at all, because the
workload brings data into the cache that is never reused, resulting in high overhead from constant caching and evicting [2].

### Least frequently used (LFU)

The least frequently used (LFU) policy evicts the elements that are accessed least often.
Whereas LRU looks at the recency of access, LFU looks at the frequency of access.
LFU can maintain a high cache hit ratio, and therefore low-latency access, for workloads where some elements are consistently more popular than others, even if their accesses are irregular in time [2].

### First-in, first-out (FIFO)

The first-in, first-out (FIFO) policy evicts elements based on when they were added to the cache [2].
In contrast to LRU, basic FIFO has no promotion: elements are evicted purely based on insertion order, regardless of how often they are accessed.
Perhaps surprisingly, FIFO combined with _lazy promotion_ (only marking an element on access, and acting on the mark at eviction time) and _quick demotion_ (evicting new elements quickly unless they prove useful) can outperform LRU [2].

### SIEVE

SIEVE is a recent cache replacement policy that is a variant of FIFO with re-insertion (also known as second chance or CLOCK) [2].
Each element has a "visited" bit that is set on access. An eviction "hand" moves through the FIFO queue from the oldest element towards the newest: if the element under the hand has been visited, its bit is cleared and it is kept; otherwise it is evicted.
Whereas FIFO with re-insertion moves a visited element back to the head of the queue, SIEVE keeps the element in its current position. This lets new elements that are never reused be evicted quickly.

### Time-to-live (TTL)

The replacement policies we looked at above determine which elements to remove when the cache is full.
Sometimes, however, we want to control the freshness of the data even when there is no
need to evict anything. Time-to-live (TTL) limits how
long an element can stay in the cache before it is considered stale and purged.

TTL is a very simple approach to cache invalidation.
In particular, if you are not using [write-through or write-behind caching](2026-01-20-Caching.md),
where updates to the data go through the cache, making sure that updates to the backing store are reflected in the cache can be complex.
TTL side-steps this problem by letting entries live in the cache only for a limited time, after which the application must retrieve the data from the database
or backing store again [2].

Although TTL is simple, it has downsides [2].
Picking the right TTL value is tricky and can seriously affect the performance of the application if we get it wrong: too short and the hit ratio drops, too long and clients see stale data.
As a rule of thumb, TTL is great for caching data that changes only occasionally, but it is tricky for highly dynamic data.

## Summary

::: {.callout-note title="Summary"}
Cache replacement policies decide which data to evict when a cache reaches capacity. Because caches are expensive and limited in size, choosing the right policy is critical for a high cache hit ratio and low latency, and the best choice depends on the workload.

- Least Recently Used (LRU): evicts the items that haven't been accessed for the longest time and promotes items on access. It works well for workloads with strong temporal locality but can perform poorly when data is rarely reused.
- Least Frequently Used (LFU): evicts the items that are accessed least often, focusing on frequency rather than recency.
- First-In, First-Out (FIFO): evicts items in insertion order, with no promotion on access. With lazy promotion and quick demotion, FIFO-based policies can outperform LRU.
- SIEVE: a FIFO variant with a "visited" bit, where retained elements keep their position instead of moving to the head of the queue.

Time-to-Live (TTL) limits how long entries remain valid regardless of cache pressure. It simplifies cache invalidation, especially when writes do not go through the cache, but choosing a good TTL value is difficult. TTL works best for data that changes infrequently.
:::

## References

1. [Cache replacement policies](https://en.wikipedia.org/wiki/Cache_replacement_policies)
2. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
