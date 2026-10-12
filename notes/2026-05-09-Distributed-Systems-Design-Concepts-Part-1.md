# Design Concepts Part 1

## Overview

Throughout this series we have discussed a number of concepts related to distributed systems.
In this short series we briefly go over some of the most common concepts one should be aware of
when designing a distributed system. In this note we discuss the following:

- Domain Name System (DNS): translates human-friendly website names (like `google.com`) into the IP addresses computers use.
- Load balancer: distributes incoming traffic across multiple servers to keep the system fast and reliable.
- API gateway: a single entry point that routes client requests to backend services in a microservices system.
- Content Delivery Network (CDN): a network of servers that delivers static content (like images or videos) from locations closer to the user.
- Forward proxy vs reverse proxy: a forward proxy acts on behalf of the client; a reverse proxy acts on behalf of the server.

The material in this note is primarily taken from [1].

## Design concepts part 1

### Domain Name System (DNS)

The Domain Name System (DNS) is a core part of the internet that translates human-readable domain names, such as _www.cnn.com_, into the IP addresses computers use to communicate. When a user enters a website address into a browser, the computer sends a DNS query to a recursive resolver, which works through the hierarchy of DNS servers, from the root servers to the Top-Level Domain (TLD) servers to the authoritative name servers, to find the IP address. The resolved address is returned to the browser, which can then connect to the server and retrieve the requested website. See [Part 6 | Domain Name System](2025-04-18-part-6-domain-name-system.md).

### Load balancer

A load balancer is a software component or networking device that distributes incoming traffic across multiple servers to improve scalability, reliability, and performance. By spreading requests, load balancers prevent individual servers from becoming overloaded, reduce response latency, and maintain high availability during traffic spikes or server failures. They use different algorithms to route traffic, including round robin, which cycles requests evenly across servers; least connections, which sends traffic to the server with the fewest active connections; and IP hash, which routes requests based on the client's IP address so that the same client consistently reaches the same server (session persistence). See [Load Balancing](2025-04-21-load-balancing.md).

### API gateway

An API gateway is a server or service that acts as a single entry point between external clients and backend services, especially in microservices architectures. It routes incoming requests to the appropriate services and handles cross-cutting concerns such as authentication, authorization, rate limiting, caching, and request transformation. Centralizing these responsibilities improves security, reduces backend complexity, and helps manage traffic. A gateway can cache common responses to reduce latency and backend load, enforce throttling policies to protect services from excessive requests, and transform requests or responses to keep clients and internal services compatible.

### Content Delivery Network (CDN)

A Content Delivery Network (CDN) is a network of geographically dispersed servers that delivers web content, such as images, videos, scripts, and stylesheets, efficiently to users. Instead of serving content from a central origin server, a CDN routes each request to a nearby edge server, reducing latency. If the requested content is already cached on the edge server, it is served immediately; otherwise, the CDN fetches it from the origin (or another CDN node), caches it, and serves it to the user. Cached content expires or is refreshed periodically to stay in sync with the origin, improving scalability, reliability, and user experience.

### Forward proxy vs reverse proxy

A forward proxy and a reverse proxy are both intermediary servers, but they serve different purposes and sit on opposite sides of a connection. A forward proxy sits in front of client machines and forwards their requests to the internet on their behalf, often providing anonymity, access control, filtering, or caching. A reverse proxy sits in front of one or more backend servers and forwards incoming client requests to them. Reverse proxies are commonly used for load balancing, security, TLS termination, and caching, and they hide the internal server infrastructure from clients. In simple terms, a forward proxy represents the client, whereas a reverse proxy represents the server. See [Forward Proxy, Reverse Proxy & NAT](2026-08-04-FWD-PROXY-REVERSE-PROXY-NAT.md).

## Summary

::: {.callout-note title="Summary"}
This note introduced five foundational concepts in distributed systems design. The Domain Name System (DNS) translates human-readable domain names into IP addresses, and load balancers distribute traffic across multiple servers to improve performance and availability. API gateways act as centralized entry points that handle routing, authentication, caching, and traffic management in microservices architectures, while Content Delivery Networks (CDNs) improve user experience by serving cached content from servers close to the user. Finally, forward proxies act on behalf of clients, whereas reverse proxies front and protect backend servers.
:::

The next note is [Design Concepts Part 2](2026-05-10-Distributed-Systems-Design-Concepts-Part-2.md).

## References

1. [25 Fundamental System Design Concepts You Must Know Before Your Interview](https://www.designgurus.io/blog/system-design-interview-fundamentals)
