# Lesson 17: Cache Coherence

## Overview

In this lesson we discuss cache coherence, i.e. how multiple caches can maintain a uniform view of the cached data.
We also introduce the cache hit ratio and the working set.

## Cache coherence

Cache coherence means that multiple caches maintain a uniform view of the cached data [1, 2].
The classic example is CPU caches, which must stay consistent when cores access the same memory.
In multi-core systems, each core has its own cache, which can lead to stale data if one core modifies data that other cores also hold in their caches.
To prevent this, CPUs use cache-coherence protocols such as MESI.

The MESI protocol tracks the state of each cache line using four states [2]:

- Modified: this cache has changed the data; it differs from main memory and no other cache holds it.
- Exclusive: this cache holds the only copy, and it matches main memory.
- Shared: multiple caches hold an unmodified copy that matches main memory.
- Invalid: the cache line is not valid or not present.

When a core writes to a line, the copies in the other caches are invalidated. These state transitions coordinate updates so that all cores see a consistent view of memory.

### Cache hit ratio

Caches speed up access by storing frequently used data. Because caches are smaller than the full dataset, their effectiveness depends on the cache hit ratio,
the proportion of requests served from the cache [2]:

$$\text{Hit ratio} = \frac{\text{Hits}}{\text{Hits} + \text{Misses}}$$

A higher ratio means faster workloads, while a lower ratio means more accesses go to the slower primary storage.

Improving the hit ratio means focusing on the working set, i.e. the subset of data actively used during a given period.
The working set is usually much smaller than the total dataset and changes over time; think of popular products on an e-commerce site or viral posts on social media. Caching this frequently accessed data yields the biggest performance gains.

## Summary

::: {.callout-note title="Summary"}
Cache coherence is about keeping multiple caches consistent when they hold copies of the same data. In multi-core CPUs, each core has its own cache, and a write by one core can leave stale values in other caches unless a coordination mechanism exists. CPUs solve this with cache coherence protocols, most notably MESI, which tracks each cache line in one of four states: Modified, Exclusive, Shared, and Invalid.

The cache hit ratio, hits divided by total accesses, is the key metric of cache effectiveness: the higher it is, the fewer requests reach slower primary storage.

Because caches are smaller than the full dataset, performance depends on the working set, the data actively used during a given period, fitting in the cache. Popular products and viral posts are typical examples of working sets that dominate access patterns despite much larger overall datasets.
:::

## References

1. [Cache coherence](https://en.wikipedia.org/wiki/Cache_coherence)
2. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
