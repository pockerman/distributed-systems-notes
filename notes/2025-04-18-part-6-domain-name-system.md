# Lesson 9: Domain Name System

## Overview

We now know how to communicate reliably (see [Lesson 6: TCP](2026-03-17-part-4-TCP_101.md)) and securely (see [Lesson 7: TLS 101](2026-04-06-Part-5-TLS-101.md)) between two processes located on different machines. However, in order to do this we first need to discover the IP address of the remote process we want to connect to.
For this we use the [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System), or DNS. DNS is a distributed, hierarchical and eventually consistent key-value store [5].

## Domain name system

Computers are identified on a network using [IP addresses](https://en.wikipedia.org/wiki/IP_address).
However, we humans rarely use these addresses to reach a website. Most of the time we use the
domain name of the website, as it is much easier to remember.
The translation of a domain name into an IP address is done by the [domain name system](https://en.wikipedia.org/wiki/Domain_Name_System).

DNS is, in a sense, the naming service of the internet. Its job is to map user-friendly domain names, such as `google.com`,
to machine-friendly IP addresses. The browser that we use to access the internet queries DNS for the IP address that corresponds to
the domain name the user typed.

Given the nature of the service that DNS provides, it is implemented as a distributed system consisting of a number
of different types of servers:

- DNS resolvers
- Root name servers
- Top-level domain (TLD) name servers
- Authoritative name servers

I won't cover the details of these in this lesson. However, I want to answer the following simple question:

_How does the browser know the address of the DNS resolver?_

Typically, the operating system keeps a list of the IP addresses of DNS resolvers; e.g. on Linux these are listed
in `/etc/resolv.conf`. The system can then send any DNS query to these resolvers.
The resolvers answer queries using DNS server software such as [BIND](https://en.wikipedia.org/wiki/BIND) (Berkeley Internet Name Domain).
The addresses of these resolvers rarely change.

Given the role of DNS, it is natural that it prioritises availability and scalability over consistency.
Recall that strong consistency means that all users see the same data at the same time, no matter where they are in the world.
For DNS this would mean that once a DNS record changes, everyone immediately gets the updated IP address.
Given the scale and distributed nature of DNS, satisfying such a requirement would cause a number of problems:

- Massive network traffic
- Slower response times
- Bottlenecks at the authoritative servers

Instead, DNS relies heavily on [caching](2026-01-20-Caching.md) to scale.
When a resolver looks up a DNS record, it stores the result for a period defined by the record's [Time to Live (TTL)](https://en.wikipedia.org/wiki/Time_to_live) value.
This works well because most DNS records don't change frequently. If we do need a change to propagate faster,
we can lower the TTL ahead of making the change. The downside is that when a domain's IP address changes,
it can take minutes or hours for the change to propagate everywhere, because some resolvers keep using the cached record until its TTL expires.

## Summary

::: {.callout-note title="Summary"}
The Domain Name System (DNS) is a distributed, hierarchical service that translates human-friendly domain names (like `google.com`) into machine-friendly IP addresses, enabling browsers to locate and connect to remote servers. While protocols like TCP and TLS handle reliable and secure communication, DNS is responsible for discovering the destination address. It is made up of resolvers, root, TLD and authoritative name servers, and the operating system is typically preconfigured with the addresses of DNS resolvers. To ensure scalability and availability, DNS relies on caching: resolved records are stored for a period given by their Time to Live (TTL), reducing load and speeding up responses. The price is eventual consistency: updates to records may take time to propagate across the system.
:::

## References

1. [IP addresses](https://en.wikipedia.org/wiki/IP_address)
2. [Domain name system](https://en.wikipedia.org/wiki/Domain_Name_System)
3. [Berkeley Internet Name Domain](https://en.wikipedia.org/wiki/BIND)
4. [Time to Live (TTL)](https://en.wikipedia.org/wiki/Time_to_live)
5. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
