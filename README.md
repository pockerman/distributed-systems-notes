# Distributed Systems Notes

Notes on designing distributed systems, published as a [Quarto](https://quarto.org/) book with [Hypothesis](https://web.hypothes.is/) annotations enabled on every page.

## Building the book

You need [Quarto](https://quarto.org/docs/get-started/) installed.

```bash
quarto render          # builds the book into _book/
./preview.sh           # live preview at http://localhost:4322/
```

The chapters and their order are defined in [`_quarto.yml`](_quarto.yml). Unfinished lessons live in `notes/wip/`, which is not rendered. When a lesson is ready, move it into `notes/` and add it to the chapters. Lesson numbers follow the order of the book; see [`TODO.md`](TODO.md) for the plan.

## Contents

### Introduction

- [Lesson 1: Introduction to Distributed Systems](notes/2026-02-24-part-1-Intro.md)
- [Lesson 2: Challenges in Distributed Systems](notes/2026-03-09-part-2-Challenges.md)
- [Lesson 3: Design Concepts: Request Routing](notes/2026-05-09-Distributed-Systems-Design-Concepts-Part-1.md)
- [Lesson 4: Design Concepts: Data and Services](notes/2026-05-10-Distributed-Systems-Design-Concepts-Part-2.md)

### Networking & Communication

- [Lesson 5: Inter-Process Communication](notes/2026-03-16-part-3-OSI_MODEL.md)
- [Lesson 6: TCP](notes/2026-03-17-part-4-TCP_101.md)
- [Lesson 7: TLS 101](notes/2026-04-06-Part-5-TLS-101.md)
- [Lesson 8: UDP & QUIC](notes/2026-07-31-TCP-UDP-QUIC.md)
- [Lesson 9: Domain Name System](notes/2025-04-18-part-6-domain-name-system.md)
- [Lesson 10: APIs](notes/2026-04-09-Part-7-API.md)
- [Lesson 11: HTTP](notes/2026-04-09-Part-8-HTTP.md)
- [Lesson 12: Forward Proxy, Reverse Proxy & NAT](notes/2026-08-04-FWD-PROXY-REVERSE-PROXY-NAT.md)

### Scalability & Caching

- [Lesson 13: Little's Law](notes/2026-05-18-Scalability-Little-Law.md)
- [Lesson 14: Cold & Warm Starts](notes/2026-05-20-Scalability-Cold-Warm-Starts.md)
- [Lesson 15: Sharded Counters](notes/2025-08-11-sharded-counters.md)
- [Lesson 16: Caching Methods](notes/2026-01-20-Caching.md)
- [Lesson 17: Cache Coherence](notes/2026-01-21-cache-coherency.md)
- [Lesson 18: Cache Replacement](notes/2026-01-26-Cache-Replacement.md)
- [Lesson 19: Negative Caching](notes/2026-04-28-Negative-Caching.md)

### Replication & Partitioning

- [Lesson 20: Replication Strategies](notes/2025-04-22-data-replication.md)
- [Lesson 21: Consistency Models](notes/2025-04-28-consistency-models.md)
- [Lesson 22: State Machine Replication](notes/2025-12-30-state-machine-replication.md)
- [Lesson 23: Consensus](notes/2025-12-31-consensus.md)
- [Lesson 24: PACELC, CAP and CALM Theorems](notes/2026-01-06-pacelc-cap-calm-theorems.md)
- [Lesson 25: Common Conflict Resolution Strategies](notes/2026-06-29-Common-Conflict-Resolution-Strategies.md)
- [Lesson 26: Read Repair, Hinted Handoff & Anti-entropy in Eventually Consistent Systems](notes/2026-07-01-Read-Repair-Hinted-Handoff-Anti-entropy-Eventually-Consistent-Systems.md)
- [Lesson 27: Partitioning](notes/2025-08-04-partitioning.md)

### Databases & Storage

- [Lesson 28: SQL Isolation Levels](notes/2026-08-20-SQL-ISOLATION-LEVELS.md)
- [Lesson 29: Multi-Version Concurrency Control](notes/2026-09-09-Multi-Version-Concurrency-Control.md)
- [Lesson 30: Write-Ahead Log](notes/2026-08-21-WRITE-AHEAD-LOG.md)

### Resiliency

- [Lesson 31: Failure Causes](notes/2026-01-28-Failure-Causes.md)
- [Lesson 32: Downstream Resiliency](notes/2026-02-04-Downstream-Resiliency.md)
- [Lesson 33: Upstream Resiliency](notes/2026-02-09-Upstream-Resiliency.md)
- [Lesson 34: Load Balancing](notes/2025-04-21-load-balancing.md)
- [Lesson 35: Sticky Sessions](notes/2026-05-11-Sticky-Sessions.md)
- [Lesson 36: Liveness and Readiness Checks](notes/2026-05-13-Liveness-Readiness-Checks.md)

### Security

- [Lesson 37: Authentication & Authorization for APIs](notes/2026-10-06-AUTHENTICATION-AUTHORIZATION.md)
- [Lesson 38: Server-Stored Sessions vs JWTs for Authentication](notes/2026-07-16-Security-Server-Stored-Session-vs-JWT.md)
- [Lesson 39: Replay Attack vs Idempotency Issues](notes/2026-07-30-Replay-Attack-Idempotency.md)

### Designs

- [Lesson 40: Design a Unique ID Generation System](notes/2025-08-14-unique-id-generation.md)
- [Lesson 41: Design a Rate Limiter](notes/2026-05-21-Resiliency-Hands-On-Design-Rate-Limiter.md)

### Appendix

- [How Does gRPC Work?](notes/impl_details/how-does-grpc-work.md)
- [Short Polling, Long Polling, SSE & WebSocket](notes/impl_details/Short-long-polling-SSE-WebSocket.md)
- [Load Balancer Use Cases](notes/impl_details/Load-Balancer-Realistic-Use-Cases.md)

### Work in progress

- [Lesson 42: Container Design Patterns for Distributed Systems](notes/wip/2026-05-08-Container-Design-Patterns-Distributed-Systems.md)
- [Lesson 43: Global Server Load Balancing](notes/wip/2026-05-14-Global-Server-Load-Balancing.md)
- [Lesson 44: Hashing Strategies](notes/wip/2026-05-31-Scalability-Hashing-Strategies.md)
- [Lesson 45: Message Delivery Semantics](notes/wip/2026-06-08-Scalability-Message-Delivery-Semantics.md)
- [Lesson 46: TLS vs mTLS](notes/wip/2026-07-25-TLS-mTLS.md)
- [Lesson 47: Synchronous & Asynchronous Replication](notes/wip/2026-08-27-SYNC-ASYNC-REPLICATION.md)
- [Lesson 48: Patterns for Online Schema Changes](notes/wip/2026-09-16-PATTERNS-ONLINE-SCHEMA-CHANGES.md)
- [Lesson 49: Content-Addressable Storage & Location-Based Addressing](notes/wip/2026-09-17-CONTENT-STORAGE-LOCATION-STORAGE.md)
- [Lesson 50: Checksums & ETags](notes/wip/2026-09-18-CHECKSUMS-ETAGS.md)
- [Lesson 51: Hot, Warm, Cold, and Archive Storage Tiers](notes/wip/2026-09-21-STORAGE-TIERS.md)
- [Lesson 52: UUID, ULID, KSUID & Snowflake IDs](notes/wip/2026-09-22-UUID-ULID-KSUID.md)
- [Lesson 53: Snowflake IDs](notes/wip/2026-09-25-SNOWFLAKE-ID.md)
- [Lesson 54: Physical Clocks and Time Synchronization](notes/wip/physical-clocks-and-time-synchronization.md)
- [Lesson 55: Logical Clocks](notes/wip/logical-clocks.md)
- [Lesson 56: Failure Detection and Group Membership](notes/wip/failure-detection-and-membership.md)
- [Lesson 57: Leader Election](notes/wip/leader-election.md)
- [Lesson 58: Distributed Locks and Leases](notes/wip/distributed-locks-and-leases.md)
- [Lesson 59: Distributed Transactions: Two-Phase and Three-Phase Commit](notes/wip/two-phase-commit.md)
- [Lesson 60: Sagas, the Transactional Outbox and Change Data Capture](notes/wip/sagas-outbox-cdc.md)
- [Lesson 61: Message Queues and Event Logs](notes/wip/message-queues-and-logs.md)
- [Lesson 62: Idempotency](notes/wip/idempotency.md)
- [Lesson 63: Conflict-Free Replicated Data Types](notes/wip/crdts.md)
- [Lesson 64: Storage Engines: B-Trees and LSM-Trees](notes/wip/storage-engines.md)
- [Lesson 65: API Gateways and Service Discovery](notes/wip/api-gateway-and-service-discovery.md)
- [Lesson 66: Content Delivery Networks](notes/wip/content-delivery-networks.md)
- [Lesson 67: Autoscaling](notes/wip/autoscaling.md)
- [Lesson 68: Observability: Logs, Metrics and Traces](notes/wip/observability.md)
- [Lesson 69: SLIs, SLOs and Error Budgets](notes/wip/slos-and-error-budgets.md)
- [Lesson 70: Deployment Strategies](notes/wip/deployment-strategies.md)
- [Lesson 71: OAuth 2.0 and OpenID Connect](notes/wip/oauth2-openid-connect.md)
- [Lesson 72: Secrets Management and Encryption at Rest](notes/wip/secrets-and-encryption.md)
- [Lesson 73: Microservices Patterns](notes/wip/microservices-patterns.md)
- [Lesson 74: Design a URL Shortener](notes/wip/design-url-shortener.md)
- [Lesson 75: Design a News Feed](notes/wip/design-news-feed.md)
- [Lesson 76: Design a Chat System](notes/wip/design-chat-system.md)
- [Lesson 77: Design a Distributed Key-Value Store](notes/wip/design-key-value-store.md)
- [Lesson 78: Design a Notification System](notes/wip/design-notification-system.md)
