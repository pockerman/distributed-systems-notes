# Design Concepts Part 2

## Overview

In this note we look into the following concepts:

- Caching: temporarily store frequently used data to speed up future access.
- Data partitioning: split large datasets into smaller chunks (shards) to spread the load across servers.
- Data replication: create and maintain copies of data on different servers for better availability and fault tolerance.
- Distributed messaging systems: enable services to communicate asynchronously by sending messages via a broker (e.g. Kafka, RabbitMQ).
- Microservices: an architecture in which features are split into small, independent services that work together.

The material in this note is primarily taken from [1].

## Design concepts part 2

### Caching

A cache is a fast storage layer that reduces latency by keeping frequently accessed data close to the application. When data is requested, the system first checks the cache; if the data is there, it is returned immediately, avoiding slower access to the original source, such as a database or a remote service. If it is not, the data is fetched from the original source, stored in the cache for future requests, and returned to the application. In distributed systems, caching can be implemented at multiple layers, including the client, DNS, CDN, API gateway, load balancer, server, and database, improving scalability, responsiveness, and efficiency. See the [caching series](2026-01-20-Caching.md).

Note that we can also use [negative caching](2026-04-28-Negative-Caching.md), i.e. cache the outcome of requests that failed, e.g. with a `404`.

### Data partitioning

Database partitioning improves scalability and performance by splitting data into smaller parts. Horizontal partitioning, also known as sharding, divides the rows of a table across multiple servers or database instances, distributing traffic and storage load. Vertical partitioning instead splits a table by columns, placing different groups of columns into separate tables to reduce table width and speed up queries that only need a subset of the data. Horizontal partitioning scales data volume and throughput, while vertical partitioning improves query efficiency and reduces unnecessary data access. See [Partitioning](2025-08-04-partitioning.md).

![Horizontal and vertical data partitioning. Image from [1].](imgs/data_partitioning.webp)

### Data replication

Database replication maintains multiple synchronized copies of the same database on different servers or in different locations to improve availability, fault tolerance, and performance. In a typical setup, one database acts as the primary (leader) and handles writes, while replicas (followers) receive the updates and mainly serve read requests. Replication spreads read traffic across replicas, reducing the load on the primary and improving response times. It also increases reliability, since a replica can take over if the primary fails, and protects data by keeping redundant copies on different machines or in different regions. See [Data Replication](2025-04-22-data-replication.md).

### Distributed messaging systems

Distributed messaging systems are infrastructure components that enable reliable, scalable, and fault-tolerant communication between distributed applications and services. They decouple message producers from consumers, letting different parts of a system operate independently and communicate asynchronously. This improves scalability, resilience, and flexibility, especially in large-scale systems and microservices architectures where services run on different machines or in different regions. Messaging systems also help absorb traffic spikes (see load leveling in [Upstream Resiliency](2026-02-09-Upstream-Resiliency.md)), retry failed work, and build event-driven workflows. Popular examples include Apache Kafka and RabbitMQ.

### Microservices

Microservices are an architectural approach in which an application is built as a collection of small, loosely coupled, independently deployable services, each responsible for a specific business capability. Unlike monolithic systems, microservices let teams develop, deploy, and scale services independently, improving flexibility, maintainability, and development speed. Each service typically owns its own data and business logic, while services communicate through lightweight protocols such as HTTP/REST, gRPC, or messaging systems. Because services run independently, a failure in one service is less likely to bring down the entire application, provided that the callers use resiliency techniques such as timeouts and circuit breakers (see [Downstream Resiliency](2026-02-04-Downstream-Resiliency.md)). The price is the complexity of operating a distributed system.

## Summary

::: {.callout-note title="Summary"}
This note covered several core concepts used to build scalable and reliable applications. Caching improves performance by keeping frequently accessed data close to the application. Data partitioning splits large datasets horizontally (sharding rows across servers) or vertically (splitting columns into separate tables) to improve scalability and query efficiency. Data replication keeps synchronized copies of databases on multiple servers to increase availability, fault tolerance, and read performance. Distributed messaging systems such as Apache Kafka and RabbitMQ enable asynchronous communication between independent services. Finally, microservices organize applications into small, independently deployable services, letting teams develop and scale them independently and improving fault isolation, at the cost of operational complexity.
:::

## References

1. [25 Fundamental System Design Concepts You Must Know Before Your Interview](https://www.designgurus.io/blog/system-design-interview-fundamentals)
