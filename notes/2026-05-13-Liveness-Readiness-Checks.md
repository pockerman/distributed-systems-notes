# Liveness and Readiness Checks

## Overview

In this note we look into another topic related to the resiliency of a distributed system,
namely:

- Liveness checks
- Readiness checks

Both are types of [health checks](2026-02-09-Upstream-Resiliency.md). Readiness checks are typically consumed by load balancers, while liveness checks are typically consumed by the platform that runs the process, such as Kubernetes.

## Liveness and readiness checks

In plain words, a liveness check verifies that an application instance is still running properly, and triggers a restart if it is not. A readiness check determines whether the application is prepared to handle requests, and therefore tells the load balancer whether it should send traffic to it.

Let's see how they are implemented and used in practice.

### Liveness checks

A liveness check should be a lightweight test that confirms the process is alive and functioning.
It runs periodically, so we don't want it to be expensive.
If the liveness check fails, the operating environment restarts the process in order to recover from the failure [1].
For example, Kubernetes uses liveness probes to detect situations like deadlocks, and restarts the container to restore service. A simple liveness probe is an HTTP `GET` to an endpoint such as `/healthz` that returns success if the application process is responsive. Alternatively, it could be a lower-level check, like opening a TCP connection or executing a trivial command inside the container. If the check fails (e.g., the app doesn't respond or returns an error), the platform knows the app is unhealthy and restarts it [1].

Key points about liveness checks [1]:

- **Purpose:** ensure the application keeps running. A liveness check is essentially a heartbeat: if the heart stops (the check fails), the platform assumes the app is dead and restarts it.
- **On failure:** a failing liveness check triggers a restart.
- **Implementation:** liveness checks should be minimal and fast. In particular, they should not check external dependencies such as the database: if the database is down, restarting every instance of the application will not fix it and may make things worse.

### Readiness checks

Even when an application is up and running, it might not be ready to handle traffic; e.g. it may still be loading data or warming its caches, or a dependency it needs may be unavailable.
A readiness check prevents traffic from reaching instances that can't fully serve requests [1].
When a readiness check fails, the instance is marked as not ready and removed from the load balancer's pool of active servers.

Unlike liveness, a failing readiness check does not restart the app. It simply tells the load balancer (or orchestrator) to stop sending new traffic to the instance until it becomes ready again [1].

### Why they matter

In load-balanced architectures, both checks are vital for high availability and smooth deployments. Specifically, they enable [1]:

- High availability: traffic only goes to instances that can serve it.
- Zero-downtime deployments: new instances receive traffic only once they are ready, and old ones are drained.
- Graceful degradation: an instance that is temporarily overloaded or missing a dependency is taken out of rotation instead of failing requests.
- Auto-healing: hung or deadlocked processes are restarted automatically.
- Protection against false alarms: separating "not ready" from "dead" avoids restarting instances that only need a moment to recover.

## Summary

::: {.callout-note title="Summary"}
Liveness and readiness checks are two types of health checks used in modern cloud environments, especially with microservices and container orchestration platforms like Kubernetes. A liveness check is essentially a heartbeat: when it fails, the platform restarts the process. It should be cheap and should not depend on external services. A readiness check tells whether an instance can serve traffic: when it fails, the instance is removed from the load balancer's pool, but it is not restarted. Together they enable high availability, zero-downtime deployments, and automatic recovery.
:::

## References

1. [What Is the Difference between Liveness Checks and Readiness Checks in Load Balancers?](https://www.designgurus.io/course-play/grokking-scalable-systems-for-interviews/doc/what-is-the-difference-between-liveness-checks-and-readiness-checks-in-load-balancers)
2. [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/), Kubernetes documentation
