# Write-Ahead Log

## Overview

A write-ahead log (WAL) is a technique in which every change is first recorded in a durable, append-only log on disk before it is applied to the main database or storage structures. If the system crashes, the log is used to recover the data, so no committed change is lost.

Durability is the "D" in the ACID properties of transactions: once a transaction is committed, its changes are permanent and survive any system failure.
WAL is the fundamental mechanism databases use to guarantee durability, and it also helps with atomicity, the all-or-nothing property of transactions, by maintaining a sequential log of changes that acts as a safety net for recovery.

## How write-ahead logging ensures durability

WAL follows a strict order of operations to protect data:

1. **Write to the log first.** When a transaction modifies data, a record describing the change is appended to the WAL. The WAL is an append-only journal: new records are always added sequentially at its end. Each record contains enough information to redo the change (and, in some designs, to undo it).
2. **Flush the log on commit.** When the transaction commits, the log, up to and including its commit record, is flushed to stable storage (e.g. with `fsync`). The transaction is not considered committed, and the client is not told that it succeeded, until this flush completes. This is what makes the commit durable: even if the system crashes right afterwards, the record of the change is on disk.
3. **Apply changes to the data pages lazily.** The actual data pages are modified in memory (in the buffer pool) and written to the data files later, in the background. The one strict rule, the write-ahead rule, is that a modified data page may only be written to disk after the log records describing its changes have been flushed. Because the change was logged first, the system can always catch up later. This also lets databases update data files in place safely.
4. **Crash recovery (redo and undo).** If the system crashes before the changes reach the data files, the WAL is used for recovery. On restart, the database reads the WAL and redoes the logged operations that were not yet applied to the data files, bringing the database back to its last committed state. Since every committed change is in the log, no committed transaction is lost. Changes of transactions that were in progress or aborted at the time of the crash are rolled back, using undo information from the log (as in ARIES-style recovery) or, in MVCC databases like PostgreSQL, simply by treating them as never committed. The database thus recovers to a consistent state containing all committed transactions.

### Checkpointing

During normal operation, the system periodically takes a checkpoint: it writes all modified in-memory pages to the data files and records the position in the WAL up to which all changes have been applied.
After a checkpoint, the older part of the log is no longer needed for crash recovery and can be archived or recycled.
Checkpointing prevents the log from growing indefinitely and bounds recovery time, since only the changes after the last checkpoint need to be replayed after a crash.

## Benefits of WAL

- **Reliable crash recovery and durability.** If a server crashes or loses power, the WAL provides a reliable record to bring the database back to a consistent state without losing any committed transaction. Once a transaction is reported as committed, it "sticks".
- **Atomicity.** WAL also contributes to atomicity. During recovery, the changes of committed transactions are redone and those of incomplete transactions are discarded or undone, so no transaction is ever left half-applied. This prevents corruption and inconsistency.
- **Performance.** Perhaps surprisingly, WAL often improves performance. Appending to a sequential log is fast compared to random writes scattered across the data files. WAL lets the database defer the expensive writes to the data files and batch them, e.g. at checkpoints, and several transactions can share a single log flush (group commit). WAL thus provides durability without flushing every data page to disk on every commit.
- **Replication and backup.** The WAL is also the foundation of replication and backup. The log can be streamed to a standby server, which replays it to maintain an identical copy of the database, or archived to enable point-in-time recovery. Many databases ship WAL records to replicas in real time, keeping them in sync and providing high availability. See [Data Replication](2025-04-22-data-replication.md).
- **A proven design.** Write-ahead logging is conceptually simple yet powerful, and it underpins almost all modern relational databases and many non-relational storage systems. Alternatives, such as shadow paging, exist, but WAL is by far the most common approach.

## WAL in practice

- **Relational databases.** Nearly all major relational databases use a form of WAL. PostgreSQL's WAL is central to its design, guaranteeing that no committed transaction is lost in a crash or power failure, and it also drives PostgreSQL's streaming replication. SQL Server and Oracle follow the same principle, calling it the transaction log and the redo log respectively. MySQL's InnoDB uses a redo log for crash-safe commits, and SQLite can operate in WAL mode for better concurrency.
- **NoSQL and key-value stores.** Many non-relational stores also use a WAL. Storage engines like RocksDB and LevelDB write each new write to a WAL file before inserting it into the in-memory table, so the data can be recovered if the process crashes before the memtable is flushed. MongoDB's WiredTiger engine uses a journal, its WAL, for durability, while its oplog, a separate log of operations, is used for replication to secondary nodes.
- **File systems (journaling).** Modern file systems such as ext4 and NTFS use the same idea, called journaling. They write metadata changes (and optionally file data) to a journal before applying them to the main file system structures, so that a crash in the middle of an update leaves the file system in a recoverable state.
- **Distributed systems and messaging.** In distributed data systems, logs are often the core mechanism for durability and ordering. Apache Kafka is built around append-only logs: each partition is a log to which messages are appended. Kafka, however, does not fsync each message by default; it relies on replication across brokers (e.g. with `acks=all`) for durability. Consensus algorithms such as [Raft](2025-12-30-state-machine-replication.md) also persist a replicated log of state changes, so that nodes can recover and replicate state reliably.

## Summary

::: {.callout-note title="Summary"}
A write-ahead log records every change in a sequential, append-only log before the change is applied to the data files. A transaction is committed only once its log records are flushed to stable storage, and data pages may only be written after the log records describing them. After a crash, the database replays the log to redo committed changes and discards or undoes incomplete ones, guaranteeing durability and atomicity. Checkpoints bound the size of the log and the recovery time. WAL also improves performance, by turning random writes into sequential ones and batching data-file writes, and it is the basis of replication and point-in-time recovery. The same idea appears in relational databases (PostgreSQL, InnoDB, SQL Server, Oracle), storage engines (RocksDB), journaling file systems, and the replicated logs of Kafka and Raft.
:::

## References

1. [PostgreSQL documentation: Write-Ahead Logging (WAL)](https://www.postgresql.org/docs/current/wal-intro.html)
2. C. Mohan et al., [ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging](https://dl.acm.org/doi/10.1145/128765.128770), ACM TODS, 1992
3. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly, chapter 3
