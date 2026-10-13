# Lesson 24: PACELC, CAP and CALM Theorems

## Overview

In this lesson we look into three results that apply to distributed systems:

- The [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)
- The [PACELC design principle](https://en.wikipedia.org/wiki/PACELC_design_principle)
- The CALM theorem, see [Keeping CALM: When Distributed Consistency Is Easy](https://cacm.acm.org/research/keeping-calm/)

## PACELC, CAP and CALM theorems

Let's start with the CAP theorem.

### CAP theorem

According to [1]:

> In database theory, the CAP theorem, also named Brewer's theorem after computer scientist Eric Brewer, states that any distributed data store can provide at most two of the following three guarantees:

- Consistency
- Availability
- Partition tolerance

Let's see what these terms mean.

**Consistency**

Every read receives the most recent write or an error. In other words, all clients see the same data at the same time, no matter which node they connect to, as if there were a single copy of the data. This is the linearizability guarantee we saw in [consistency models](2025-04-28-consistency-models.md). Consistency as defined in the CAP theorem is quite different from the consistency guaranteed by ACID database transactions.

**Availability**

Every request received by a non-failing node in the system must result in a (non-error) response, without the guarantee that it contains the most recent write.
Note that availability as defined in the CAP theorem is different from high availability in software architecture [1].

**Partition tolerance**

The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.

Network partitions are unavoidable in distributed systems, so in practice the theorem says that when a partition occurs we must choose between availability and consistency.

::: {.callout-note title="Remark"}
We will see below, with the CALM theorem, that monotonic programs can produce consistent results without coordination, and can therefore stay available during a partition.
:::

### PACELC theorem

The CAP theorem is useful for understanding the requirements on a distributed system. The PACELC theorem offers a more comprehensive framework and can be seen as an extension of CAP.
It states that in the event of a network partition (P), a system must choose between availability (A) and consistency (C), as in the CAP theorem; else (E), in normal operation without partitions, the system must choose between latency (L) and consistency (C).
In other words, PACELC acknowledges that the latency/consistency trade-off is present even when the system is functioning normally, not just during network failures [2].

The PACELC theorem was first described by Daniel J. Abadi in 2010 and formally proved in 2018.
It gives four possible configurations [2]:

- PA/EL: availability during partitions, low latency otherwise
- PA/EC: availability during partitions, consistency otherwise
- PC/EL: consistency during partitions, low latency otherwise
- PC/EC: consistency at all times

The default configurations of Amazon's original Dynamo, Cassandra, and Riak are classified as PA/EL: they give up consistency to remain available during partitions, and give up consistency for lower latency in normal operation.
In contrast, fully ACID-compliant distributed systems such as VoltDB/H-Store, MySQL Cluster, BigTable, and HBase are PC/EC: they ensure consistency at all times, at the cost of availability and latency [2].
The theorem is particularly relevant for designing cloud applications and systems operating in intermittently connected environments, such as IoT and mobile applications, where both the partition and the non-partition trade-offs must be considered.

### CALM theorem

The CALM theorem, standing for Consistency As Logical Monotonicity, establishes that a program has a consistent, coordination-free distributed implementation if and only if it is expressible in monotonic logic [3].

A program is monotonic when its outputs only grow (and are never retracted) as new information arrives. Such programs can proceed without coordination [3]; set union and counting "likes" are typical examples.
In contrast, in a non-monotonic program a conclusion can change when new information arrives (e.g. "this item is _not_ in the set"), so it requires coordination to ensure consistent outcomes.
The theorem shifts the focus from storage consistency models to properties of the program itself, enabling static analysis and reducing reliance on expensive runtime mechanisms like two-phase commit or Paxos. CALM has inspired new programming language designs, such as Bloom, and tools that help developers build systems that are both highly available and consistent by identifying and isolating the non-monotonic components.

## Summary

::: {.callout-note title="Summary"}
CAP, PACELC, and CALM explain fundamental trade-offs in distributed system design. The CAP theorem states that in the presence of a network partition a system must choose between consistency (linearizability) and availability. PACELC extends CAP: systems face trade-offs not only during partitions (availability vs. consistency) but also in normal operation (latency vs. consistency), and real-world systems can be classified by these choices (e.g. Dynamo-style stores are PA/EL, BigTable and HBase are PC/EC). Finally, the CALM theorem offers a complementary perspective: programs expressible in monotonic logic can be implemented consistently without coordination, shifting the focus from storage-level guarantees to program-level properties and influencing modern distributed languages such as Bloom.
:::

## References

1. [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)
2. [PACELC design principle](https://en.wikipedia.org/wiki/PACELC_design_principle)
3. [Keeping CALM: When Distributed Consistency Is Easy](https://cacm.acm.org/research/keeping-calm/)
