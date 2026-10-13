# Lesson 34: Load Balancing

## Overview

In this lesson I go over [load balancing](https://en.wikipedia.org/wiki/Load_balancing_(computing)) approaches. Modern systems have to
deal with heavy load and traffic, which means the work has to be distributed over a number
of workers. A load balancer partitions the incoming traffic in one way or another so that no single machine gets overloaded.

We can place load balancers at various points in a system. Furthermore, load balancing can be static or dynamic,
and stateful or stateless.

## Load balancing

Modern datacenters typically serve millions of requests per day. These requests are processed
by the machines that make up the datacenter. Given the volume of requests, and
the amount of computation and data transfer needed to serve them, sending everything to one machine, or
just a few machines, will not work. We need ways to spread the load across the available machines.
This is where load balancers step in. In general, load balancers allow us to improve or maintain the following
aspects of our application:

- Scalability
- Availability
- Performance

<!-- TODO: add a figure showing a load balancer in front of a pool of servers (imgs/load_balancer_schematics.png is missing). -->

It is fairly obvious why a load balancer can improve the overall performance of a system.
In addition, a load balancer can provide several key services, such as:

- Security: since the load balancer sits in front of our servers, it gives us a place to mitigate attacks such as [denial of service](https://en.wikipedia.org/wiki/Denial-of-service_attack).
- Health checking: a load balancer needs to know which servers are available before it sends them a request. It therefore monitors the health of the servers behind it.

Typically, we place a load balancer between the clients and the available servers.
However, this need not be the only place; for example, we can place a load balancer between the application servers and the
database servers, and/or between the web servers and the application servers. Load balancing can also be implemented at different layers of the stack [5]:

- DNS load balancing
- Transport layer (L4) load balancing
- Application layer (L7) load balancing
- Geo load balancing

Now that we have a conceptual understanding of what a load balancer does, let's briefly discuss how it distributes
the load. This is done using various [load balancing algorithms](https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques):

- Round-robin
- Weighted round-robin
- Least connections
- URL hash

I won't go over the details of these algorithms here; see, for example, reference [4].
I do want to mention that, in general, load balancing algorithms are either static or dynamic.
A static algorithm does not take the changing state of the servers into account. Its major advantage is simplicity.
In contrast, a dynamic algorithm considers the current state of the servers, which it learns by communicating with them.
Dynamic algorithms therefore add communication overhead, but they make better forwarding decisions.

## Summary

::: {.callout-note title="Summary"}
Load balancing is a core technique in modern distributed systems for handling high traffic and computational load. Load balancers distribute incoming requests across multiple workers to prevent overloading individual machines, thereby improving scalability, availability, and performance.

Load balancing can be implemented at different points in a system (e.g., between clients and servers, or between the application and database tiers), at different layers of the network stack, and can be static or dynamic, as well as stateful or stateless.

Beyond traffic distribution, load balancers also provide important services:

- Security, such as mitigating denial-of-service attacks
- Health checking, by monitoring server availability and routing traffic only to healthy nodes

To distribute traffic, load balancers rely on algorithms such as round-robin, weighted round-robin, least connections, and URL hashing.
Static algorithms are simpler and ignore server state, while dynamic algorithms adapt to server load by communicating with the backend servers, trading simplicity for better routing decisions.
:::

## References

1. [Load balancing](https://en.wikipedia.org/wiki/Load_balancing_(computing))
2. [What is load balancing?](https://aws.amazon.com/what-is/load-balancing/)
3. [Cloud load balancing](https://cloud.google.com/load-balancing)
4. [Load balancing algorithms](https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques)
5. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
