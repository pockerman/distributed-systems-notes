# Lesson 26: Read Repair, Hinted Handoff & Anti-entropy in Eventually Consistent Systems

## Overview

In an [eventually consistent](2025-04-28-consistency-models.md) system, data updates are not synchronized across all replicas immediately.
Instead, the system guarantees that if no new updates occur, all replicas eventually converge to the latest value.
This weak consistency model (often associated with the acronym BASE: Basically Available, Soft state, Eventual consistency) favours availability and partition tolerance over strict synchronization.

Temporary discrepancies can therefore arise: one node may hold an older value than another, but given time and corrective protocols, the differences are resolved.
The benefit is high availability and performance, since replicas don't need to coordinate on every write; the challenge is making sure that all copies of the data reconcile their differences over time.

To achieve convergence, distributed databases in the Dynamo family, such as Apache Cassandra and Riak, employ special repair mechanisms [1, 2].
Read repair, hinted handoff, and anti-entropy are three complementary techniques that address inconsistencies at different times and in different failure scenarios:

- Read repair kicks in during read operations and updates stale replicas.
- Hinted handoff handles writes during temporary outages by saving hints and delivering the missed updates later.
- Anti-entropy (via Merkle trees) runs in the background, comparing whole datasets and syncing replicas, especially after prolonged failures or node replacements.

Together, these techniques let an eventually consistent database remain highly available while still converging to correct data.
Let's look at each mechanism in turn.

## Read repair: repairing data on reads

Read repair corrects inconsistencies during read operations.
When a read contacts multiple replicas and finds that some of them hold stale values, the system updates the out-of-date replicas with the latest value as part of the read.
In other words, the act of reading triggers a repair of any discrepancy it detects, so subsequent reads are more consistent.

### How read repair works

1. **Read query.** The client requests some data (e.g., a key). In an eventually consistent database, the coordinator node often sends the read to multiple replicas; for example, with a quorum read (read consistency level $R > 1$) it queries several replica nodes.
2. **Detecting inconsistency.** The coordinator compares the responses. If all replicas return the same value, the data is consistent. If a replica returns an older value than the others, an inconsistency has been detected. The coordinator determines which version is the most recent using timestamps or vector clocks (see [Lesson 25: Common Conflict Resolution Strategies](2026-06-29-Common-Conflict-Resolution-Strategies.md)).
3. **Updating stale replicas.** The coordinator sends the latest data to the stale replicas. Depending on the system, this happens before returning the result to the client (blocking read repair) or in the background.
4. **Returning the latest value.** The client receives the latest value, possibly after concurrent updates have been resolved. The read may be slightly slower because of the repair step, but future reads are more consistent.

This process is called "read repair" because it repairs replicas that have missed an update in the course of a read.
It reduces the burden on background synchronization: the system doesn't have to wait for a periodic job to fix the inconsistency, because it is fixed as a side effect of the read.

### Why it matters

Read repair propagates recent writes to all replicas as soon as an inconsistency is noticed.
This is especially useful when a previous write reached some but not all replicas.
Over time, with enough reads across the dataset, the frequently read data converges on every replica.
Read repair is used in NoSQL stores like Apache Cassandra when reading at a consistency level that contacts more than one replica; a read from a single replica cannot detect inconsistencies.
It provides self-healing on the read path, at the cost of slightly higher read latency when a repair is needed. Note that data which is rarely read is rarely repaired this way.

## Hinted handoff: dealing with temporary failures

Hinted handoff improves write availability and durability when some nodes are temporarily down.
In an eventually consistent database, we don't want a temporary outage of one replica to make writes fail.
Hinted handoff "hands off" the write to another node, which holds it until the target replica comes back online, so the write is not lost and consistency can be restored later.

### How hinted handoff works

1. **Write with an unavailable replica.** During a write, if one of the target replicas (say node B) is unreachable, the system doesn't fail the write outright. Instead, another healthy node (say node X) accepts the write meant for B and stores it together with a "hint" saying that the update is intended for B.
2. **Storing the hint.** The hint (the data plus the identity of the missed node) is stored durably on disk, e.g. in a separate hints file or table on node X, so that it survives a restart of X. Node X is essentially keeping a sticky note with B's missed write.
3. **Node recovery.** The system detects B's recovery, for example via gossip or heartbeat messages. Once B is back online, X replays the saved write to B, bringing B up to date, and then discards the hint.
4. **Consistency restored.** B eventually receives the write it missed, fulfilling the promise of eventual consistency without having rejected the write. From the client's perspective, the write succeeded because it was stored somewhere in the cluster, and the system later healed the missing replica.

In simpler terms, hinted handoff acts like the post office holding your mail while you're on vacation and delivering it when you return.

### Why it matters

Hinted handoff greatly improves write availability and the resilience of the system.
Even when a replica is temporarily unavailable (because of a crash, a network glitch, or maintenance), the database keeps accepting writes by storing hints on other nodes.
This means higher uptime and a better user experience, since writes don't fail just because one replica is down.
It also helps maintain the replication factor: once the node recovers, the data exists on the intended number of replicas.
Amazon's Dynamo and Apache Cassandra both implement hinted handoff [1, 2].

Note, however, that while hints are pending, the down node's data is stale, so there is a temporary inconsistency until the hint is applied. Moreover, a hint stored on a single node is not as durable as a replicated write, so hinted handoff complements, rather than replaces, the other repair mechanisms.
If a node stays down for too long, or permanently, hints are no longer kept (e.g. Cassandra only stores hints for a configurable window, three hours by default).
In such cases, a full anti-entropy repair is needed, which leads us to the next mechanism.

## Anti-entropy and Merkle trees: background synchronization

Even with read repair and hinted handoff, replicas can drift out of sync for longer periods;
for example, when a node was down for an extended time, or when some writes were missed and the data was never read.
Anti-entropy is the general term for mechanisms that compare replicas, continuously or periodically, and reconcile their differences in the background.
An efficient anti-entropy technique, used by systems like Dynamo and Cassandra, relies on Merkle trees (hash trees) to find mismatched data without transferring the entire dataset [1, 2].

### What is a Merkle tree?

A Merkle tree is a hash tree that provides a compact fingerprint of a large dataset:

- Each leaf is the hash of an individual data item, or of a small range of keys.
- Each parent node is the hash of the concatenation of its children's hashes.
- A single root hash therefore represents the entire dataset. If any data item changes, its leaf hash changes, and so do the hashes of all its ancestors up to the root.

By comparing the root hashes of two trees, we can quickly tell whether the two datasets are identical.
The key advantage for anti-entropy is that each branch of the tree can be checked independently.
Two nodes don't need to compare every data item; they compare their Merkle tree roots.
If the roots are equal, the datasets are identical and nothing needs to be done.
If not, they compare the hashes at the next level down to pinpoint which part of the key space differs, and recurse until they find the specific keys that mismatch.
Only those keys (or ranges) are then transferred and synchronized.
Merkle trees thereby drastically reduce the amount of data exchanged when comparing replicas, saving bandwidth and I/O.

In practice, each node maintains a Merkle tree for the data it holds, often one per partition or key range.
Periodically, or upon certain events like a node joining or recovering from a long outage, nodes exchange the relevant parts of their Merkle trees to find inconsistencies.
This process is called an anti-entropy repair.

### How anti-entropy repair works

1. **Initiation.** Anti-entropy repair can be scheduled (e.g., an operator runs a repair job, or the system triggers it periodically during off-peak hours). Suppose nodes X and Y are two replicas of the same key range.
2. **Exchanging hashes.** X and Y each build a Merkle tree over their data for that range and compare the root hashes. If they are identical, the range is consistent and no repair is needed.
3. **Narrowing down differences.** If the roots differ, the nodes compare the hashes of the next level, which correspond, for example, to halves of the key range, and identify which subtrees mismatch. This zooming in continues level by level, and different branches can be compared in parallel.
4. **Identifying mismatched keys.** Eventually, the process narrows down to the leaves (keys) that differ. For those keys, the nodes determine which version is newer (using timestamps or vector clocks) and the out-of-date node updates its data.
5. **Completion.** After syncing the differing keys, both replicas hold identical data for the range. The repair then moves on to other ranges or other pairs of nodes until the cluster is consistent.

### Why it matters

Anti-entropy is the final safety net for eventual consistency.
Whereas read repair and hinted handoff handle recent or transient issues, anti-entropy corrects lingering inconsistencies that escape those mechanisms.
It is especially important after long outages, or when a node is replaced and has no knowledge of recent writes.
By running anti-entropy regularly, the system ensures that even rarely read data, which doesn't benefit from read repair, is eventually checked and fixed.
Merkle trees make this feasible at scale: without them, anti-entropy would require comparing entire datasets between replicas, which is too slow and bandwidth-heavy.
With them, the system only transfers the differences.

## Summary

::: {.callout-note title="Summary"}
Read repair, hinted handoff, and anti-entropy are three mechanisms that keep eventually consistent systems converging, each operating at a different time:

- **Read repair (read path):** stale replicas discovered during a read are updated, so reads not only fetch data but also fix it.
- **Hinted handoff (write path):** writes intended for temporarily unavailable nodes are stored as hints on other nodes and replayed when those nodes recover, giving high write availability.
- **Anti-entropy with Merkle trees (background):** replicas periodically compare Merkle trees to find and repair differences efficiently, handling long-term divergence, e.g. after a long outage or a node replacement.

Read repair and hinted handoff tackle short-term inconsistencies, while Merkle-tree-based anti-entropy repairs long-term or hidden ones.
Together, they let systems like Dynamo, Cassandra, and Riak relax immediate consistency to gain availability and performance, while still making all copies of the data consistent eventually, without manual intervention.
:::

## References

1. G. DeCandia et al., [Dynamo: Amazon's Highly Available Key-value Store](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf), SOSP 2007
2. [Apache Cassandra documentation: Repair](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html)
3. [Apache Cassandra documentation: Hints](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/hints.html)
4. [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree)
