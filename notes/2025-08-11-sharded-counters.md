# Lesson 15: Sharded Counters

## Overview

In this lesson we discuss the sharded counters pattern. We can use this pattern whenever
we need to scale counting in our system. It is also a building block for problems such as top-k (e.g. most liked posts).

## Sharded counters

Let's start by describing the problem we want to address.

### The problem

In a distributed system we often need to keep track of counts. Some examples where
such a need arises:

- The number of likes on a social media post
- The number of views of a video
- The number of active sessions of a user

We can certainly store the counter as a single row, document or key in our database. However, this approach
breaks down under high traffic. It has two main problems:

1. Write contention
2. A throughput limit

When many clients try to update the same row at the same time, the database has to serialize these updates, typically
with locks, which degrades performance. In addition, the machine that holds the counter eventually becomes a bottleneck.

### The solution

With sharded counters, we don't store one counter but $N$ separate sub-counters (shards). Each of these
is stored independently. A write increments just one shard. When the total is needed, we sum all the shards.

There are two questions we need to answer: how to pick $N$, and how to pick which shard to increment.
The number of shards is important for performance. If the shard count is too small for the write workload,
we get high write contention and slow writes. If the shard count is too high, reads become more expensive.

There are various ways to choose which shard to increment. Two popular approaches are:

- Random choice
- Using application-specific information (e.g. hashing the user or the server ID)

Note that sharded counters make reads more expensive. Shards may reside on different nodes, and we need to read all of them
to compute the sum. This is why a very high shard count is problematic. In practice the total is often cached or
computed periodically, so readers see a slightly stale value.

## Summary

::: {.callout-note title="Summary"}
The sharded counter pattern trades read simplicity for write scalability. It distributes increments across $N$ independent sub-counters and sums them to get the total.
Choosing $N$ balances write contention (too few shards) against read cost (too many shards).
It is an approach we can follow whenever a single hot counter (likes, views, sessions) would otherwise become a write bottleneck.
:::

## References

1. [Sharding counters in Firestore: conception of a real-time polls app, part 1](https://medium.com/%40awesomefirebase/sharding-counters-in-firestore-conception-of-a-real-time-polls-app-a06f896e6483)
