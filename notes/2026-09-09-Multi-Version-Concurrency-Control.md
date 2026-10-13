# Lesson 30: Multi-Version Concurrency Control

## Overview

Multi-Version Concurrency Control (MVCC) is a concurrency control technique in which the database keeps multiple versions of each record, so that reads and writes can proceed simultaneously, with each transaction seeing a consistent snapshot of the data. In simple terms, under MVCC readers don't block writers, and writers don't block readers.

This approach is used by most modern databases, including PostgreSQL, MySQL's InnoDB, and Oracle, to improve performance and avoid locking conflicts, enabling fast concurrent transactions even under heavy multi-user workloads.

## The problem MVCC solves

When many users access a database at the same time, some form of concurrency control is needed to keep the data consistent.
Without it, a reader could see half-written data while another user is in the middle of an update.
Imagine a bank transfer: one step debits an account and another credits a different account. A concurrent reader might catch the system between those steps and see money "disappear".
The isolation property, the "I" in ACID, is meant to prevent such anomalies by controlling concurrent access (see [Lesson 29: SQL Isolation Levels](2026-08-20-SQL-ISOLATION-LEVELS.md)).

A traditional solution is locking, for instance making readers wait until a writer finishes (a read-write lock).
Locking ensures consistency, but it introduces contention: long-running reads block writes, and writes block reads, so transactions wait and throughput drops.
In a high-traffic application this becomes a bottleneck, and can even cause deadlocks (two transactions waiting on each other indefinitely).

MVCC was created to solve this bottleneck.
Instead of making transactions take turns, the database keeps multiple versions of the data. Each transaction sees a snapshot of the database at a particular point in time, and writers don't overwrite the data that readers are using.
Updates create a new version, so other transactions can keep reading the old one without waiting.
Once the writer commits, the new version becomes the one that subsequent transactions see, and older versions are eventually cleaned up.
MVCC thereby provides isolation without coarse locking.

## How MVCC works

MVCC is based on versioned data and snapshots.
Instead of a single current value for each item, there can be several versions of it, each tagged with the transaction ID (or timestamp) of the transaction that created it, and, once superseded, of the one that replaced it.
Here is a simplified view:

1. **Versioned records.** Each record version carries version information, e.g. the ID of the transaction that created it.
2. **Readers use snapshots.** A read sees the latest version that was committed as of its snapshot, regardless of writes that happen afterwards. Depending on the isolation level, the snapshot is taken at the start of each statement (Read Committed) or at the start of the transaction (Repeatable Read / snapshot isolation).
3. **Writers create new versions.** A transaction that updates a record doesn't modify it in place; it creates a new version. The original version is left untouched, so active readers can still access it. (Some engines, such as InnoDB and Oracle, update the row in place but keep the old version in an undo log, which achieves the same effect.)
4. **Non-blocking reads.** While the writer is working on its new version, other transactions keep reading the older one. The writer is isolated with its uncommitted version, and readers are unaffected by the in-progress change.
5. **Commit makes the version visible.** When the writing transaction commits, its version becomes visible to transactions whose snapshots are taken from then on.
6. **The cycle continues.** A later update creates yet another version, and the previous one becomes historical. Old versions are kept as long as some running transaction may still need them.

Under this mechanism, readers and writers don't block each other.
A reader simply picks, for each item, the version visible to its snapshot, and a writer creates a new version for its changes.
Many transactions can therefore query and update the same data simultaneously with minimal contention.

Transactions only need to synchronize when two writers update the same item.
How this is handled depends on the database and the isolation level. In PostgreSQL, for example, the second writer waits for the first to finish (row-level write locks still exist); under Read Committed it then proceeds against the newly committed version, while under Repeatable Read it is aborted with a serialization error and must be retried ("first updater wins").
Either way, reads never conflict with writes under MVCC, which is the key to its high concurrency.

## Benefits

- **Non-blocking reads.** Readers don't take locks, so reads never wait for writes. Queries don't get stuck behind updates under heavy load, and long-running analytical queries can run without freezing out updates.
- **High concurrency and throughput.** Because reads and writes proceed in parallel, the database can handle more transactions per second, which is ideal for read-heavy or mixed workloads, such as web applications and real-time dashboards.
- **Less contention and fewer deadlocks.** Since reads don't lock, there are fewer locks to fight over and fewer opportunities for deadlocks.
- **Consistent snapshots.** Every transaction sees a point-in-time consistent view of the data, so it never sees uncommitted data (dirty reads) or half-done updates. MVCC naturally provides snapshot isolation.
- **Predictable read performance.** For workloads that mix heavy reading (reporting, analytics) with writes, read performance stays stable even as updates happen in the background.

## Trade-offs

MVCC also has costs:

- **Storage overhead.** Keeping several versions of the data uses more storage. Every update creates a new row version (or an undo log entry), so tables can bloat with old versions, especially in write-heavy systems or when long-running transactions keep old versions alive.
- **Garbage collection.** The system must eventually remove versions that no running transaction can see anymore. PostgreSQL, for example, uses `VACUUM` to purge dead row versions, and InnoDB purges its undo logs. This background work consumes resources, and if it falls behind, performance suffers.
- **Implementation complexity.** MVCC is more complex to implement than simple locking: the database must track transaction IDs, decide which versions are visible to which transaction, and garbage-collect old ones. This is hidden from users, and the performance gain usually justifies it.
- **Write-write conflicts remain.** MVCC doesn't eliminate conflicts between concurrent writers; it handles them by blocking or aborting one of them, so applications must be prepared to retry transactions in high-contention scenarios.
- **Snapshot isolation is not serializability.** A snapshot alone allows anomalies such as write skew. Databases that offer serializability on top of MVCC, such as PostgreSQL's Serializable Snapshot Isolation, need extra conflict detection.

Databases mitigate these issues with tuning and design, e.g. frequent (auto)vacuuming and avoiding long-running transactions.
In practice, the benefits outweigh the drawbacks for the vast majority of applications that need high concurrency.

## Summary

::: {.callout-note title="Summary"}
Multi-Version Concurrency Control keeps multiple versions of each record so that every transaction reads from a consistent snapshot, while writers create new versions instead of overwriting data in use. As a result, readers never block writers and writers never block readers, which gives high concurrency, fewer deadlocks, and stable read performance. Concurrent writers to the same row still conflict and are resolved by blocking or aborting one of them. The costs are extra storage, background garbage collection of old versions (e.g. PostgreSQL's `VACUUM`), implementation complexity, and the fact that snapshot isolation alone still permits anomalies like write skew. MVCC is the default concurrency control mechanism in PostgreSQL, InnoDB, Oracle, and many other databases.
:::

## References

1. [PostgreSQL documentation: Concurrency Control](https://www.postgresql.org/docs/current/mvcc.html)
2. [MySQL documentation: InnoDB Multi-Versioning](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)
3. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly, chapter 7
