# Distributed Systems Notes

Notes on designing distributed systems, published as a [Quarto](https://quarto.org/) book with [Hypothesis](https://web.hypothes.is/) annotations enabled on every page.

## Building the book

You need [Quarto](https://quarto.org/docs/get-started/) installed.

```bash
quarto render          # builds the book into _book/
./preview.sh           # live preview at http://localhost:4322/
```

The chapters and their order are defined in [`_quarto.yml`](_quarto.yml). Unfinished notes are listed under `project.render` with a `!` prefix so they are not rendered; when a note is ready, remove that line and add it to the chapters.

## Contents

### Introduction

- [Part 1 | Introduction to Distributed Systems](notes/2026-02-24-part-1-Intro.md)
- [Part 2 | Challenges in Distributed Systems](notes/2026-03-09-part-2-Challenges.md)
- [Design Concepts Part 1](notes/2026-05-09-Distributed-Systems-Design-Concepts-Part-1.md)
- [Design Concepts Part 2](notes/2026-05-10-Distributed-Systems-Design-Concepts-Part-2.md)

### Networking & Communication

- [Part 3 | Inter-Process Communication](notes/2026-03-16-part-3-OSI_MODEL.md)
- [Part 4 | TCP 101](notes/2026-03-17-part-4-TCP_101.md)
- [Part 5 | TLS 101](notes/2026-04-06-Part-5-TLS-101.md)
- [Part 6 | Domain Name System](notes/2025-04-18-part-6-domain-name-system.md)
- [Part 7 | APIs](notes/2026-04-09-Part-7-API.md)
- [Part 8 | HTTP](notes/2026-04-09-Part-8-HTTP.md)
- [HTTP/1.1, HTTP/2 & HTTP/3](notes/2026-08-05-HTTP-1-2-3.md)
- [TCP, UDP & QUIC](notes/2026-07-31-TCP-UDP-QUIC.md)
- [Forward Proxy, Reverse Proxy & NAT](notes/2026-08-04-FWD-PROXY-REVERSE-PROXY-NAT.md)

### Scalability & Caching

- [Little's Law](notes/2026-05-18-Scalability-Little-Law.md)
- [Cold & Warm Starts](notes/2026-05-20-Scalability-Cold-Warm-Starts.md)
- [Sharded Counters](notes/2025-08-11-sharded-counters.md)
- [Caching Part 1 | Caching Methods](notes/2026-01-20-Caching.md)
- [Caching Part 2 | Cache Coherence](notes/2026-01-21-cache-coherency.md)
- [Caching Part 3 | Cache Replacement](notes/2026-01-26-Cache-Replacement.md)
- [Caching Part 4 | Negative Caching](notes/2026-04-28-Negative-Caching.md)

### Replication & Partitioning

- [Data Replication Part 1 | Replication Strategies](notes/2025-04-22-data-replication.md)
- [Data Replication Part 2 | Consistency Models](notes/2025-04-28-consistency-models.md)
- [Data Replication Part 3 | State Machine Replication](notes/2025-12-30-state-machine-replication.md)
- [Data Replication Part 4 | Consensus](notes/2025-12-31-consensus.md)
- [Data Replication Part 6 | PACELC, CAP and CALM Theorems](notes/2026-01-06-pacelc-cap-calm-theorems.md)
- [Common Conflict Resolution Strategies](notes/2026-06-29-Common-Conflict-Resolution-Strategies.md)
- [Read Repair, Hinted Handoff & Anti-entropy](notes/2026-07-01-Read-Repair-Hinted-Handoff-Anti-entropy-Eventually-Consistent-Systems.md)
- [Partitioning](notes/2025-08-04-partitioning.md)

### Databases & Storage

- [SQL Isolation Levels](notes/2026-08-20-SQL-ISOLATION-LEVELS.md)
- [Multi-Version Concurrency Control](notes/2026-09-09-Multi-Version-Concurrency-Control.md)
- [Write-Ahead Log](notes/2026-08-21-WRITE-AHEAD-LOG.md)

### Resiliency

- [Resiliency Part 1 | Failure Causes](notes/2026-01-28-Failure-Causes.md)
- [Resiliency Part 2 | Downstream Resiliency](notes/2026-02-04-Downstream-Resiliency.md)
- [Resiliency Part 3 | Upstream Resiliency](notes/2026-02-09-Upstream-Resiliency.md)
- [Load Balancing](notes/2025-04-21-load-balancing.md)
- [Sticky Sessions](notes/2026-05-11-Sticky-Sessions.md)
- [Liveness and Readiness Checks](notes/2026-05-13-Liveness-Readiness-Checks.md)

### Security

- [Authentication & Authorization for APIs](notes/2026-10-06-AUTHENTICATION-AUTHORIZATION.md)
- [Server-Stored Sessions vs JWTs](notes/2026-07-16-Security-Server-Stored-Session-vs-JWT.md)
- [Replay Attack vs Idempotency Issues](notes/2026-07-30-Replay-Attack-Idempotency.md)

### Designs

- [Design a Unique ID Generation System](notes/2025-08-14-unique-id-generation.md)
- [Design a Rate Limiter](notes/2026-05-21-Resiliency-Hands-On-Design-Rate-Limiter.md)

### Appendix

- [How Does gRPC Work?](notes/impl_details/how-does-grpc-work.md)
- [Short Polling, Long Polling, SSE & WebSocket](notes/impl_details/Short-long-polling-SSE-WebSocket.md)
- [Load Balancer Use Cases](notes/impl_details/Load-Balancer-Realistic-Use-Cases.md)

### Work in progress

- [Container Design Patterns for Distributed Systems](notes/2026-05-08-Container-Design-Patterns-Distributed-Systems.md)
- [Global Server Load Balancing](notes/2026-05-14-Global-Server-Load-Balancing.md)
- [Hashing Strategies](notes/2026-05-31-Scalability-Hashing-Strategies.md)
- [Message Delivery Semantics](notes/2026-06-08-Scalability-Message-Delivery-Semantics.md)
- [TLS vs mTLS](notes/2026-07-25-TLS-mTLS.md)
- [Synchronous & Asynchronous Replication](notes/2026-08-27-SYNC-ASYNC-REPLICATION.md)
- [Patterns for Online Schema Changes](notes/2026-09-16-PATTERNS-ONLINE-SCHEMA-CHANGES.md)
- [Content-Addressable Storage & Location-Based Addressing](notes/2026-09-17-CONTENT-STORAGE-LOCATION-STORAGE.md)
- [Checksums & ETags](notes/2026-09-18-CHECKSUMS-ETAGS.md)
- [Hot, Warm, Cold, and Archive Storage Tiers](notes/2026-09-21-STORAGE-TIERS.md)
- [UUID, ULID, KSUID & Snowflake IDs](notes/2026-09-22-UUID-ULID-KSUID.md)
- [Snowflake IDs](notes/2026-09-25-SNOWFLAKE-ID.md)
