# Lesson 21: Consistency Models

## Overview

In [Lesson 20: Replication Strategies](2025-04-22-data-replication.md) we discussed data replication
as a means of increasing a system's availability and improving its performance. Data replication refers to maintaining multiple copies of the same
data in such a way that client applications perceive them as a single object. This is referred to as _replication transparency_.

However, data replication is not easy, as the copies have to be kept consistent. Consistency models therefore play a dominant role
in how data is replicated.

In this lesson I give a brief overview of the most common [consistency models](https://en.wikipedia.org/wiki/Consistency_model) used when designing distributed systems.
The choice of model depends heavily on the application requirements, so this is not a discussion
of which model is better but rather a description of the attributes of each. We will discuss the following models:

- Strong consistency
- Sequential consistency
- Eventual consistency

For more information about consistency models see [Consistency Models](https://jepsen.io/consistency/models) on the Jepsen website.

## Consistency models

In an ideal world, when a client sends a request to a replicated store, the request is executed instantly on every replica.
This is far from what happens in reality. In particular, when read requests
can be served by any follower and not only by the leader, two clients may get different views of the
state of the system [2]. This is particularly true if we favour the availability of the system
over its consistency (see the [CAP theorem](2026-01-06-pacelc-cap-calm-theorems.md)).

More formally, a consistency model defines the views that observers of a system can have of
the system's state. Let's briefly go over each of the models listed above.

### Strong consistency

Strong consistency guarantees that a client observes the state of the system as if there were a single copy of the data,
unaffected by any replica that is lagging behind. One way to achieve this is to route all
reads and writes to the leader. In other words, every operation appears to take effect atomically at some instant between its start and its completion [2].
Strong consistency is also known as [linearizability](https://jepsen.io/consistency/models/linearizable) [2].
However, if all reads and writes have to go through the leader, the leader becomes a choke point
that limits throughput.

### Sequential consistency

If we relax the real-time guarantees of strong consistency, we get sequential consistency.
In this model, every replica applies the operations in the same order as the leader, but it is not
necessarily up to date with the leader. A typical example of sequential consistency is a producer and a consumer
synchronized via a queue [2]. If we pin each client to a specific replica, clients may see the system at
different points in time, but always in the same order of states.

Sequential consistency differs from strong consistency in that it does not provide real-time guarantees [2].

### Eventual consistency

Eventual consistency allows clients to read from any replica, with no guarantee that the data is up to date with the leader.
As its name implies, eventual consistency only guarantees that all replicas will eventually
converge to the same final state, provided that writes to the system stop [2].
Pinning a client to a particular replica would give it a monotonic view, but makes that replica a single point of failure for the client.
If instead the client may read from any replica, it can observe the data going "back in time" when it moves from a fresher replica to a staler one.

Eventual consistency is the model of choice when prioritizing low latency, as it eliminates the expensive coordination between nodes [4].

### Other consistency models

There are many consistency models other than those mentioned above; causal consistency and strong eventual consistency are two examples.
I would like to briefly mention session consistency, as it is very prevalent in web and mobile applications [4].
Session consistency guarantees that, within a single user session, reads are monotonic and users always see their own writes. Updates are guaranteed to be visible immediately only to the same session, not necessarily to other users. This model suits individual user–focused applications (like e-commerce carts or web/mobile apps), where a consistent personal experience matters, but it is not suitable for collaborative applications that require shared, real-time visibility.

## Summary

::: {.callout-note title="Summary"}
A consistency model defines the rules for how operations (especially reads and writes) appear to execute across multiple nodes, threads, or users in a distributed or concurrent system.
Strong consistency (linearizability) behaves like a single copy of the data but limits throughput; sequential consistency keeps a single order of operations without real-time guarantees; eventual consistency only guarantees convergence once writes stop, but offers the lowest latency. Session consistency is a popular middle ground for user-facing applications.
It is up to the system architect to decide which consistency model to use. A system does not have to use just one consistency
model; it can mix them depending on the requirements of each subsystem.
:::

The next lesson discusses [state machine replication](2025-12-30-state-machine-replication.md).

## References

1. [Consistency models](https://en.wikipedia.org/wiki/Consistency_model)
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem)
4. Pekka Enberg, _Latency: Reduce delay in software systems_, Manning Publications
