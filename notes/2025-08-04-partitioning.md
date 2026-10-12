# Partitioning

## Overview

We have discussed [data replication](2025-04-22-data-replication.md) as a technique to
increase the availability of a system by copying data across various locations. However, replicating the entire dataset across numerous locations can be impractical
due to storage costs and network bandwidth requirements.

Whenever we want to guarantee the scalability of a data system we need to consider partitioning.
Partitioning divides a dataset into smaller pieces, each of which can be accessed independently, enabling efficient reads and writes [4].
Partitioning as a technique applies to many situations, e.g. a [load balancer](2025-04-21-load-balancing.md) partitions TCP connections across servers [1].
In general, by partitioning a system we try to remove the scalability limits of a single component [2].

In this note I discuss the most common partitioning approaches.
On a related topic, see [Training Patterns for Distributed ML](https://github.com/pockerman/qubit-notes/blob/main/ml/2025-05-11-training-patterns-for-distributed-ML.md) and
[Data Ingestion Patterns for Distributed ML](https://github.com/pockerman/qubit-notes/blob/main/ml/2025-05-07-data-ingestion-patterns-for-dist-ml.md).

## Partitioning

For large applications, it is often infeasible to fit the complete dataset on a single server; consider, for example, training a machine learning model
on a very large dataset. To overcome this, we split the data into smaller groups and store these groups on multiple servers [3].

::: {.callout-note title="Partitioning vs sharding"}
When we discuss partitioning, the term sharding often comes up. Sharding refers to a specific
type of partitioning, namely horizontal partitioning [4].
:::

There are various ways to partition a dataset:

- Horizontal partitioning
- Range partitioning
- Hash partitioning
- Vertical partitioning
- Logical partitioning

Range and hash partitioning are two ways of deciding which rows go to which horizontal partition. Let's briefly discuss each approach.

### Horizontal partitioning

Horizontal partitioning splits a dataset by rows. It is most frequently applied to databases, but it can be used beyond database systems.
It is commonly used in online transaction processing
(OLTP) systems [4].

When partitioning a database horizontally, we typically partition the data on the values of a specific column, the partition key.
For example, if we store a `User` entity alongside the user's location, we can partition the table
by location. If, for example, we store timestamps, we could use range partitioning (see below).

When querying a horizontally partitioned database, we first need to determine which partition(s)
to query. Note also that keeping secondary indexes consistent with the data in a horizontally partitioned database can
be challenging [4].

### Range partitioning

With [range partitioning](https://www.sciencedirect.com/topics/computer-science/range-partitioning) the data is divided into non-overlapping segments based on value ranges of the partition key.
This technique is particularly useful for continuous partition keys, such as time, enabling efficient data organization and query optimization.
It is commonly used with date columns, where each partition represents a specific time period like a month or a quarter.
If the data is also stored in sorted order within each partition, range scans are fast [1].

Range partitioning has two main drawbacks [1]:

- If the keys are not more or less uniformly distributed, the partitions can become unbalanced.
- Some access patterns can lead to hotspots; e.g. when partitioning by date, all of today's writes go to the same partition.

### Hash partitioning

Hash partitioning applies a hash function to the partition key; the hash value determines the partition in which the data is stored. A good hash function spreads the keys evenly across partitions, which helps balance the load, enables parallel processing, and improves scalability. The price is that range queries over the original key are no longer efficient, since neighbouring keys end up in different partitions.

Although hash partitioning gives partitions with roughly the same number of entries, it does not eliminate hotspots [1].
Hotspots still occur when the access pattern is skewed, e.g. when one key is accessed far more often than the others.

Regardless of the approach we follow to partition a dataset, there are two challenges to address [3]:

- Distribute the data evenly across the available servers
- Minimize data movement when nodes are added or removed

[Consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing) addresses the second problem [3]: with naive `hash(key) mod N` partitioning, changing $N$ moves almost every key, whereas consistent hashing moves only about $1/N$ of them.

### Vertical partitioning

Vertical partitioning divides records by columns rather than by rows [4]. Instead of storing complete records together, related columns are grouped into separate partitions. This approach is used mainly in OLAP systems, which are read-intensive and optimized for large analytical queries such as aggregations, joins, sorting, and machine learning workloads.

Vertical partitioning improves performance through better compression, reduced read amplification, and more efficient query execution, since only the required columns are read. Storing values of the same type together lets databases compress data more effectively and use CPU optimizations like SIMD. It can also make some join operations faster.

However, vertical partitioning has drawbacks: it is harder to scale, because each partition still spans all rows, and writes are slower, since inserting or updating a record touches multiple partitions.

We can combine vertical and horizontal partitioning into hybrid partitioning to mitigate the downsides of each [4].

### Logical partitioning

The two main partitioning strategies for data are horizontal and vertical partitioning [4]. However, we can also partition data according to
application needs. Below are some alternative approaches; for more details see [4], chapter 5.

- **Functional partitioning**: partition the data based on how the application uses it, i.e. its usage patterns.
- **Geographical partitioning**: partition the data by geographical location, e.g. the physical location of the client.
- **User-based partitioning**: similar to geographical partitioning, but the data is partitioned by user.
- **Time-based partitioning**: partition the data by time.
- **Overpartitioning**: create many more partitions than nodes, so partitions can be moved between nodes as the cluster grows without re-splitting the data.

### Static vs dynamic partitioning

Another question we need to answer is how many partitions to create. This depends on the system at hand. The number of partitions can be fixed up front and
never change thereafter, i.e. _static partitioning_. This is simple, but it is hard to pick the right number: too many partitions add overhead, while too few limit scalability [1].

Dynamic partitioning creates partitions on demand. Typically, we start with one partition and split it when, for example, it exceeds a certain size or the number of requests
it serves goes beyond a threshold.

## Summary

::: {.callout-note title="Summary"}
Partitioning is a fundamental technique for scaling distributed systems, complementing data replication. It divides large datasets or workloads into smaller, independently accessible pieces, enabling efficient reads, writes, and parallel processing without fully replicating the data everywhere.

Common partitioning strategies are:

* **Horizontal partitioning (sharding):** splits data by rows based on a partition key (e.g., user location or timestamp). Widely used in OLTP systems; it improves scalability but makes indexing and consistency harder.
* **Range partitioning:** divides the data into non-overlapping key ranges (e.g., time periods). It supports efficient range queries but can suffer from unbalanced partitions and hotspots.
* **Hash partitioning:** uses a hash function to spread keys evenly across partitions, at the cost of efficient range queries. Hotspots can still arise from skewed access patterns. Consistent hashing limits data movement when nodes join or leave.
* **Vertical partitioning:** splits data by columns rather than rows, primarily in OLAP systems. It improves compression, reduces read amplification, and speeds up analytical queries, but makes writes slower and scaling harder.
* **Hybrid partitioning:** combines horizontal and vertical partitioning to balance their trade-offs.
* **Logical partitioning:** partitions data based on application needs: functional, geographical, user-based, or time-based partitioning, or overpartitioning.

Static partitioning fixes the number of partitions in advance, while dynamic partitioning creates or splits partitions on demand, trading simplicity for flexibility.

Partitioning adds a fair amount of complexity to a system. Imbalanced partitions can recreate the very problems we are trying to solve and limit the
ability of the system to scale. And because partitions are independent, updating several of them atomically requires distributed transactions [1].
:::

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Dominik Tornow, _Think Distributed Systems_, Manning Publications
3. Alex Xu, _System Design Interview: An insider's guide_
4. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
