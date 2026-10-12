# Part 2 | Challenges in Distributed Systems

## Overview

[Part 1 | Introduction to Distributed Systems](2026-02-24-part-1-Intro.md) introduced the idea of a distributed system and gave some examples.
It also introduced the running example that we will be expanding on in these notes.

In this note we briefly discuss the core challenges we need to address when designing a distributed system, and
relate them to our running example. Designing distributed systems revolves around
the following challenges [1]:

- Communication
- Coordination
- Scalability
- Resiliency
- Operations

## Challenges of distributed system design

### Communication

In [Part 1](2026-02-24-part-1-Intro.md) we split our imaginary system into two tiers: the web server tier and the database tier.
Now that the two services live on different machines, they need to communicate over the network. This adds uncertainty, as the network is unreliable,
and it adds latency to our application. Network libraries abstract away the communication details, but we still need to understand how the network stack works,
because these abstractions leak.

### Coordination

As our database grows, we will have to add more nodes to support it. This is easier now that the web application and the database are separated.
However, the database nodes have to coordinate in order to present a single, coherent view of the data to the clients.
Coordination in distributed systems is difficult, and we will discuss a number of techniques for it.

### Scalability

The performance of a distributed system describes how efficiently it handles load [1]. It is usually measured with _throughput_ and _response time_.
Throughput is the number of operations per second the system can process [1]. Response time is the time between a client sending a request and
receiving the response. How load is measured is specific to the system's use cases; it could be requests per second, the number of concurrent users, or the volume of stored data.
A system is scalable if it can handle more load, without degrading its performance, by adding resources.

### Resiliency

A system is resilient when it can continue to do its job even when failures happen [1]. Given the scale of distributed systems,
any failure that can happen will eventually happen [1]. No matter how small the probability of a failure, as the number of operations grows,
the absolute number of failures grows with it.

Failures unavoidably affect the availability of a system. Availability is defined as the amount of time the system can serve requests divided by the duration of the
period measured, i.e. the percentage of time the system is capable of serving requests [1]. It is often expressed in "nines": 99.9% availability allows about 8.8 hours of downtime per year, while 99.99% allows about 53 minutes.

### Operations

Like any other system, a distributed system is built to be deployed and to do useful work.
This implies a number of activities, such as testing, maintenance, rolling back changes, and monitoring the system.
All of these operations should happen in a way that does not affect the availability of the system [1].

## Summary

::: {.callout-note title="Summary"}
Designing distributed systems involves several key challenges that arise when components run on different machines. Communication becomes complex because services (such as a web server and a database) interact over unreliable networks that add latency and uncertainty. Coordination is required when multiple nodes, like the instances of a distributed database, must present a consistent view of the data to clients. Scalability is about handling increasing load, measured through throughput (operations per second) and response time. Resiliency ensures the system keeps functioning despite inevitable failures, keeping availability high. Finally, operations covers running the system in production: testing, monitoring, maintenance, and safely deploying or rolling back changes without disrupting service.
:::

The next part discusses [inter-process communication](2026-03-16-part-3-OSI_MODEL.md).

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
