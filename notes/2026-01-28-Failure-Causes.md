# Lesson 32: Failure Causes

## Overview

In this lesson we discuss how to make a distributed system more resilient to failure.
To do so we first need to know the most common causes of failure in a distributed system. In this lesson we discuss:

- Single points of failure
- Unreliable networks
- Slow processes
- Load spikes

## Failure causes

Informally, a failure is an unwanted state that the system can transition to.
On a failure, the system transitions from a good state to a bad state.
Fault tolerance is the ability of a system to behave in a well-defined manner even when it is in a bad state [2].

### Single point of failure

A distributed system consists of many nodes, and many things can go wrong within each node, e.g. software bugs or disk crashes.
A single point of failure (SPF) is a component whose failure, or degraded performance, brings the whole system down with it [1].
The most glaring example is perhaps an application that cannot do anything useful because the single backend service it depends on is down.

SPFs should be identified when designing the system. Sometimes this is easy. In general, the best way to find out
whether a component is an SPF is to ask what would happen to the system if that component failed [1].

### Unreliable networks

The network that a distributed system runs on is unreliable in many ways. For example, packets may be dropped, or the network may simply be slow.
When a client sends a request to a server, it expects a response. However, the client has no way of
knowing whether a response is on its way or not, so it has two options: wait, or fail the request [1].
There are several reasons why the client hasn't received a response [1]:

- The request has been dropped by a proxy, router, or network switch
- The server is slow or has crashed
- The server's response has been dropped by a proxy, router, or network switch

When a network partition splits the nodes into groups that cannot talk to each other, every node in one group sees the nodes in the other as failed.

### Slow processes

A process running on a server can slow down. From a client's perspective, a slow process is often indistinguishable from one that isn't running at all.
A common reason for a process slowing down is a resource leak, with memory leaks probably being the best known. Another example is
a thread that makes HTTP requests without setting a timeout: if a call never returns, the thread is never returned to the thread pool, and the pool is eventually exhausted [1].

### Load spikes

A system is typically designed with a certain load in mind. In practice, the actual load often deviates from this assumption for a variety of reasons [1]:

- The requests may exhibit seasonality
- Some requests may be much more expensive than others
- Some requests may be malicious and intended to abuse the system, e.g. DDoS attacks

## Summary

::: {.callout-note title="Summary"}
Building resilient distributed systems starts with understanding how they fail. This lesson covered four major failure causes:

- Single points of failure (SPFs): components whose failure brings down the whole system
- Unreliable networks: requests and responses can be dropped or delayed, and a client can't tell why no response arrived
- Slow processes: often caused by resource leaks or missing timeouts, and from the outside indistinguishable from crashed ones
- Load spikes: caused by seasonality, expensive requests, or malicious traffic
:::

In the next parts we look into specific mechanisms that make a system fault tolerant.
Following [1], we split these mechanisms into [downstream](2026-02-04-Downstream-Resiliency.md) and [upstream](2026-02-09-Upstream-Resiliency.md) resiliency.

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Dominik Tornow, _Think Distributed Systems_, Manning Publications
