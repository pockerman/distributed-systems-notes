# Lesson 36: Sticky Sessions

## Overview

The note on [load balancing](2025-04-21-load-balancing.md) discussed how we can balance the load across the servers of a system. Often, however, we want all
requests from a specific user to be routed to the same server. Session affinity, or sticky sessions, is a load balancing technique
that does exactly this.

## Sticky sessions

As the traffic of a web application grows, we have to deploy it on multiple servers. A load balancer is placed in front of these
servers and distributes the requests across them.
Web applications are usually built on [HTTP](2026-04-09-Part-8-HTTP.md), which is a stateless protocol:
each request is independent, and previous interactions are not remembered. This leads to problems if user-specific data is stored in the memory of a single server.
A very common example of user interaction data we need to remember is a shopping cart.
If a user adds items to their cart and a later request is routed to a different server that doesn't have the user's session, the user may suddenly be logged out or find their cart empty.

Sticky sessions solve this problem by binding a user's session to one server, ensuring continuity of data and experience [1].
How is this done? The load balancer must identify the user and remember which server the user is tied to.
There are two common ways to do this:

- Assign an identifier to the user, typically in a cookie
- Track the client's IP address

With the cookie approach, on the user's first request the load balancer sets a special cookie that encodes which server handled the request.
On every subsequent request the load balancer reads this cookie and routes the user to the same server.

::: {.callout-note title="Remark"}
Sticky sessions based on IP address tracking are less reliable, because multiple users may share an IP address (e.g. behind a NAT) and a single user's IP address may change (e.g. on a mobile network) [1].
:::

Sticky sessions therefore keep a user's session data, e.g. their login state, on the same server.
They exist to address real needs in session management. Their benefits include [1]:

- Consistent session data: since a user's session data lives on a single server, session management is greatly simplified.
- Improved performance: session data can be kept in the server's memory.
- Easy implementation: enabling sticky sessions is usually a configuration change on the load balancer and requires no changes to the application code.

::: {.callout-note title="Remark"}
Most modern load balancers (NGINX, HAProxy, cloud load balancers, etc.) support session affinity out of the box.
This makes it an attractive quick solution for session management, especially in legacy applications:
you don't have to redesign how sessions are stored, since the load balancer keeps each user's traffic on the same server [1].
:::

However, sticky sessions have serious drawbacks that affect scalability and reliability:

- Uneven load: session affinity can lead to an uneven distribution of load across servers, since the load balancer can no longer freely pick the least busy server.
- Session loss on crash: with sticky sessions, session data lives only in one server's memory. If that server goes down or restarts, all users tied to it lose their sessions (e.g. they are logged out or lose unsaved data).
- Harder maintenance and scaling: because users are tied to specific servers, taking a server down for maintenance, or scaling in, disrupts its users, and new servers only receive new users.

Sticky sessions should therefore be avoided by default, and especially when the system has:

- High availability requirements
- A need to scale horizontally
- A microservices-based architecture

::: {.callout-tip title="Alternatives"}
In cloud-native applications, it is generally better to store session state externally, e.g. in a database or a cache such as Redis, or not to keep server-side session state at all.
Another option is token-based authentication such as JWT; see [Server-Stored Sessions vs JWTs](2026-07-16-Security-Server-Stored-Session-vs-JWT.md).
All in all, avoid session affinity unless you have a specific requirement for it, such as a stateful legacy application.
:::

## Summary

::: {.callout-note title="Summary"}
Sticky sessions, also known as session affinity, are a load balancing technique that routes all requests from the same user to the same server. Since HTTP is stateless, user-specific session data such as login state or shopping cart contents would be lost if requests went to different servers. Sticky sessions solve this by letting the load balancer identify users, typically through cookies or IP tracking, and route them back to the server that holds their session. This simplifies session management, allows in-memory session storage, and is easy to configure on load balancers like NGINX or HAProxy. However, sticky sessions can create uneven server load, lose sessions when a server crashes, and complicate maintenance and horizontal scaling. Modern cloud-native and microservices-based systems therefore prefer storing session state externally, such as in a database or Redis, or using stateless authentication like JWTs.
:::

## References

1. [What Are Sticky Sessions (Session Affinity) and When Should I Avoid Them?](https://www.designgurus.io/course-play/grokking-scalable-systems-for-interviews/doc/what-are-sticky-sessions-session-affinity-and-when-should-i-avoid-them)
