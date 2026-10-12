# Common Conflict Resolution Strategies

## Overview

When multiple nodes or processes update the same data at the same time, such as in a distributed database with multi-leader replication, or in collaborative editing, conflicts can occur.
Conflict resolution strategies are vital to ensure that all copies of the data end up consistent.
In this note we review the following strategies [1]:

- Last-write-wins (LWW)
- Vector clocks
- Logical (Lamport) clocks

See also [Data Replication Part 1](2025-04-22-data-replication.md).

## Common conflict resolution strategies

### Last-write-wins (LWW)

Last-write-wins (LWW) is a straightforward conflict resolution policy where the latest update wins.
Each write is tagged with a timestamp (or a monotonically increasing logical counter), and if two updates conflict, the system keeps the one with the most recent timestamp.
For example, Apache Cassandra uses LWW: every write is timestamped, and when replicas disagree the database selects the value with the newest timestamp.
This picks a single consistent value quickly, without complex merging.

**Advantages.** LWW is simple to implement and fast.
There is no need to keep multiple versions or perform a merge; you just keep the newest update.
It works well in systems where occasional lost updates are acceptable and clock synchronization is reasonably reliable.

**Drawbacks.** The simplicity of LWW comes at the cost of potential data loss. If two updates occur at nearly the same time on different nodes, one of them is silently dropped in favour of the other.
In other words, resolving a conflict with LWW means discarding all but one of the concurrent updates, so one user's change can be lost.
Additionally, LWW usually relies on synchronized physical clocks.
If clocks drift or a timestamp is wrong, a newer update can be mistaken for an older one and be overwritten. This makes pure LWW risky for globally distributed systems where clock skew is common.
Some systems mitigate this with hybrid logical clocks or similar schemes, but ordering by time remains imperfect.

### Vector clocks

A vector clock is a mechanism for tracking causality between events in a distributed system.
Instead of a single timestamp, each node keeps a vector of counters, one per node in the system.
This allows the system to record a partial order of events without any global physical clock.
By comparing vector clocks, we can tell whether an event happened before, happened after, or was concurrent with another event on a different node.

If every element of event A's vector clock is less than or equal to the corresponding element of event B's vector clock (and at least one is strictly less), then A happened before B, and may have influenced it.
If neither clock is less than or equal to the other (each has at least one element larger than the other's), the two events are concurrent: they happened independently, with no causal relationship.

Vector clocks are essential for handling conflicts in systems with concurrent updates, because they capture causality explicitly.
Unlike LWW, which blindly picks one update based on time, vector clocks allow a system to _detect_ a conflict, i.e. recognize that two updates happened concurrently, and then decide how to resolve it.
For example, Amazon's Dynamo used vector clocks to tag each version of a data item [2]: if two clients update the same item on different servers concurrently, the two vector clocks are incomparable, signalling a conflict that needs resolution, such as merging the versions or asking the client to reconcile them.
This way no update is lost silently: the system knows that two versions exist and can merge them or keep both for later resolution, instead of arbitrarily discarding one.

**Overhead and drawbacks.** The main trade-off of vector clocks is the cost of storing and transmitting the vector.
The vector has one entry per node or replica, so it grows with the system.
Every message or data item must carry this vector, which can become large in a big cluster and consume extra bandwidth and storage.
Maintaining vector clocks also adds complexity to the code.
Moreover, vector clocks only _detect_ conflicts; how to merge the concurrent updates is still up to the application or a higher-level mechanism (e.g. CRDTs or custom merge logic).

### Logical clocks (Lamport timestamps)

Logical clocks are a broader family of schemes that assign artificial timestamps to events.
The most common form is the Lamport timestamp, introduced by Leslie Lamport, which orders events in a distributed system without relying on physical time.
A Lamport clock is a counter that each process maintains and increments on each event. When a process sends a message, it attaches its current counter value, and the receiver sets its clock to the maximum of its own value and the received value, then increments it by one.
This guarantees that if event A happened before event B (A may have causally influenced B), then A's timestamp is smaller than B's.
Logical clocks thereby establish an ordering of events that is consistent with causality.

**Guarantees and limitations.** Lamport timestamps can be turned into a total order of all events by breaking ties between equal timestamps with the process ID.
This is useful for many distributed algorithms that need a consistent global sequence of events, such as ordering operations or coordinating actions.
However, Lamport timestamps cannot identify concurrent events.
If two events are independent (neither caused the other), they still get different timestamps, and the order between them is arbitrary; it does not reflect any causal relationship.
In other words, if A happened before B then $L(A) < L(B)$, but $L(A) < L(B)$ does not imply that A happened before B, so from the numbers alone you can't tell that two events were concurrent.
This is why vector clocks were developed: to detect when events are concurrent.

### Comparison

Each strategy has different strengths and trade-offs in handling concurrent updates:

| Strategy | Strengths | Weaknesses | Typical use |
|---|---|---|---|
| Last-write-wins | Simple and fast; one timestamp per update | Concurrent updates are not reconciled; one is lost. Relies on (physical or logical) clock ordering | Caches, user settings, cases where occasional lost updates are acceptable |
| Vector clocks | Detect concurrency and preserve all causally distinct versions | Metadata grows with the number of nodes; merging is still left to the application | Multi-leader / leaderless databases and collaborative applications where no update should be lost |
| Lamport clocks | Very lightweight (one integer per process); order consistent with causality | Cannot detect concurrency; used for conflict resolution, they behave like LWW | Ordering events: distributed logging, global snapshots, mutual exclusion |

## Summary

::: {.callout-note title="Summary"}
Choosing a conflict resolution strategy in a distributed system depends on your consistency needs and your tolerance for lost updates.
LWW offers simplicity but can drop data and depends on clock quality; vector clocks detect conflicts precisely but carry more overhead; and Lamport clocks give a cheap ordering of events that respects causality, without being able to detect concurrency.
Many modern systems combine these techniques, or use structures like CRDTs (Conflict-free Replicated Data Types) that merge concurrent changes automatically without losing updates.
:::

## References

1. [What Are Common Conflict Resolution Strategies (Last-write-wins, Vector Clocks, Logical Clocks)?](https://www.designgurus.io/course-play/grokking-scalable-systems-for-interviews/doc/what-are-common-conflict-resolution-strategies-lastwritewins-vector-clocks-logical-clocks)
2. G. DeCandia et al., [Dynamo: Amazon's Highly Available Key-value Store](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf), SOSP 2007
3. L. Lamport, [Time, Clocks, and the Ordering of Events in a Distributed System](https://lamport.azurewebsites.net/pubs/time-clocks.pdf), Communications of the ACM, 1978
