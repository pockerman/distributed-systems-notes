# Lesson 65: Storage Engines: B-Trees and LSM-Trees

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

How a database lays data out on disk decides how fast it reads, writes and recovers. B-trees and log-structured merge trees are the two dominant designs.

## Outline

- Append-only logs and hash indexes
- SSTables and LSM-trees: memtables, flushes and compaction
- Bloom filters on the read path
- B-trees: pages, splits and the WAL
- Read, write and space amplification
- Which engines use which (PostgreSQL, InnoDB, RocksDB, Cassandra)

## Related lessons

- [Lesson 31: Write-Ahead Log](../2026-08-21-WRITE-AHEAD-LOG.md)
- [Lesson 30: Multi-Version Concurrency Control](../2026-09-09-Multi-Version-Concurrency-Control.md)

## References

1. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly Media
2. Patrick O'Neil et al., _The Log-Structured Merge-Tree (LSM-Tree)_, Acta Informatica, 1996
3. Alex Petrov, _Database Internals_, O'Reilly Media
