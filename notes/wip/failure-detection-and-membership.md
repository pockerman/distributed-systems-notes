# Lesson 57: Failure Detection and Group Membership

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

In an asynchronous network we cannot tell a crashed node from a slow one. Failure detectors make that guess using timeouts, and membership protocols spread the result to the rest of the cluster.

## Outline

- Why perfect failure detection is impossible
- Heartbeats and timeouts, and choosing the timeout
- The phi-accrual failure detector
- Gossip protocols and their convergence
- SWIM: indirect probing and infection-style dissemination
- Membership in practice: Cassandra, Consul/Serf, Kubernetes

## Related lessons

- [Lesson 2: Challenges in Distributed Systems](../2026-03-09-part-2-Challenges.md)
- [Lesson 37: Liveness and Readiness Checks](../2026-05-13-Liveness-Readiness-Checks.md)
- [Lesson 27: Read Repair, Hinted Handoff & Anti-entropy in Eventually Consistent Systems](../2026-07-01-Read-Repair-Hinted-Handoff-Anti-entropy-Eventually-Consistent-Systems.md)

## References

1. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
3. Naohiro Hayashibara et al., _The φ Accrual Failure Detector_, SRDS 2004
4. Abhinandan Das, Indranil Gupta and Ashish Motivala, _SWIM: Scalable Weakly-consistent Infection-style Process Group Membership Protocol_, DSN 2002
