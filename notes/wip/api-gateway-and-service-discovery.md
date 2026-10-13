# Lesson 66: API Gateways and Service Discovery

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

As our application splits into services, clients need one entry point and services need to find each other. API gateways and service discovery solve these two problems.

## Outline

- What an API gateway does: routing, auth, rate limiting, aggregation
- API gateway vs reverse proxy vs load balancer
- Backend for frontend (BFF)
- Client-side vs server-side discovery
- Service registries (Consul, etcd) and DNS-based discovery
- Discovery in Kubernetes

## Related lessons

- [Lesson 13: Forward Proxy, Reverse Proxy & NAT](../2026-08-04-FWD-PROXY-REVERSE-PROXY-NAT.md)
- [Lesson 35: Load Balancing](../2025-04-21-load-balancing.md)
- [Lesson 3: Design Concepts Part 1](../2026-05-09-Distributed-Systems-Design-Concepts-Part-1.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Sam Newman, _Building Microservices_, O'Reilly Media
