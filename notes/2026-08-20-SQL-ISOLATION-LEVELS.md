# Lesson 28: SQL Isolation Levels

## Overview

SQL isolation levels (such as Read Committed, Repeatable Read, and Serializable) determine how strictly concurrent transactions are kept apart from each other. Each level prevents certain concurrency anomalies, like dirty reads, non-repeatable reads, and phantom reads.

## Isolation levels

In database systems, the isolation level is the degree to which one transaction is isolated from the effects of other concurrent transactions.
It defines how and when the changes made by one transaction become visible to others.
Isolation is the "I" in the ACID properties of transactions (Atomicity, Consistency, Isolation, Durability), and it is crucial for correctness in concurrent environments.

There is a trade-off between isolation and performance. Lower isolation levels allow more concurrency (higher throughput) but risk more anomalies, while higher isolation levels prevent more anomalies at the cost of more blocking, waiting, or aborted transactions.
In simple terms, relaxed isolation means higher concurrency and potential inconsistencies, whereas strict isolation means fewer anomalies and potential slowdowns.
Choosing the right level is about balancing correctness and performance for your application.

The ANSI SQL standard defines four isolation levels, in increasing order of strictness: Read Uncommitted, Read Committed, Repeatable Read, and Serializable [1].
Most database systems provide these levels or similar ones. For instance, PostgreSQL, Oracle, and SQL Server default to Read Committed, while MySQL's InnoDB engine defaults to Repeatable Read.
The higher the level, the more anomalies are prevented.

### Common anomalies

When transactions run concurrently with insufficient isolation, several anomalies can occur. The main ones that isolation levels deal with are [1, 2]:

- **Dirty read.** A transaction reads data written by another transaction that has not committed yet. If that other transaction rolls back, the first transaction has read data that "never officially existed".
- **Non-repeatable read.** A transaction reads the same row twice and gets different values, because another transaction modified the row and committed in between the two reads.
- **Phantom read.** A transaction re-executes a query that returns a set of rows (e.g. `SELECT * FROM ... WHERE condition`) and finds that rows satisfying the condition have appeared or disappeared, because another transaction inserted or deleted rows and committed in the meantime.

Two further anomalies are worth knowing, because the standard levels don't name them but real systems must deal with them [2]:

- **Lost update.** Two transactions read the same value, both modify it, and both write it back; one update overwrites the other.
- **Write skew.** Two transactions read overlapping data, make disjoint updates based on what they read, and together violate a constraint that each one alone would have preserved (e.g. two doctors both going off call because each saw the other was still on call).

Each isolation level restricts concurrent reads and writes to prevent some of these anomalies.

### Read Uncommitted

Read Uncommitted is the weakest level: a transaction may see uncommitted changes of other transactions, so dirty reads are possible, as are all the other anomalies. It is rarely used. (PostgreSQL, for example, treats Read Uncommitted as Read Committed.)

### Read Committed

Read Committed is one of the most commonly used isolation levels, because it strikes a balance between consistency and performance.
A transaction only sees data that has been committed by other transactions.
In practice, dirty reads are prevented: you cannot read uncommitted data from a concurrent transaction. In MVCC-based databases, each statement sees a snapshot of the data committed as of the start of that statement.

However, Read Committed does not guarantee repeatable reads within a transaction.
If another transaction commits a change after you have read some data, reading the same data again in your transaction can return the new value. Non-repeatable reads, and therefore phantom reads, are possible.

- Prevented: dirty reads.
- Possible: non-repeatable reads, phantom reads (and lost updates and write skew).

### Repeatable Read

Repeatable Read is a stricter level which ensures that if a transaction reads the same data twice, it sees the same values each time (unless it modified the data itself).
There are two common implementations:

- **Locking:** shared locks are held on the rows a transaction reads until it completes, so no other transaction can modify them in the meantime.
- **Snapshots (MVCC):** the transaction reads from a snapshot taken at its start; other transactions can still modify the rows, but this transaction doesn't see their changes (see [Lesson 29: Multi-Version Concurrency Control](2026-09-09-Multi-Version-Concurrency-Control.md)).

Either way, dirty reads and non-repeatable reads are prevented.
In the SQL standard's definition of Repeatable Read, phantom reads are still possible: another transaction can insert or delete rows that satisfy a query's `WHERE` condition, because the locks only cover the rows that were read, not the range of possible rows.

In practice, implementations differ from the standard. PostgreSQL's Repeatable Read is snapshot isolation, which also prevents phantom reads but allows write skew. MySQL InnoDB's Repeatable Read uses a consistent snapshot for plain reads and next-key (range) locks for locking reads, which prevents most phantoms.

- Prevented: dirty reads and non-repeatable reads.
- Possible (per the standard): phantom reads. Check your database's documentation for the exact guarantees.

### Serializable

Serializable is the strictest isolation level.
A serializable execution of transactions is one whose outcome is the same as if the transactions had run one at a time, in some serial order, rather than interleaved.
Serializable therefore prevents all the anomalies described above: no dirty reads, non-repeatable reads, or phantom reads, and also no lost updates or write skew.
It provides the strongest guarantee, and it is what the "I" in ACID ideally means.

Databases implement Serializable with strict two-phase locking (including range locks), or with MVCC plus conflict detection, as in PostgreSQL's Serializable Snapshot Isolation (SSI).
If the system detects a conflict that could lead to a non-serializable outcome, it blocks one of the transactions, or aborts it with a serialization error that the application must handle by retrying.
This level therefore sacrifices concurrency: transactions may have to wait or be retried, but integrity is preserved as if the transactions had executed one at a time.

- Prevented: all of the anomalies above.
- Cost: more blocking and/or aborted transactions that must be retried.

### Choosing a level

Each level includes the guarantees of the levels below it and adds protection against more anomalies.
For example, Repeatable Read does everything Read Committed does and also guarantees repeatable reads, and Serializable does everything Repeatable Read does and also prevents phantoms and write skew.

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible (per the standard) |
| Serializable | Prevented | Prevented | Prevented |

Choose an isolation level based on the application's needs: use the lowest level that is safe for your use case, to avoid unnecessary performance costs.
For many applications, Read Committed is sufficient, offering a good balance of consistency and concurrency, provided that the code guards against lost updates (e.g. with `SELECT ... FOR UPDATE` or atomic updates). In high-integrity systems, such as financial ones, Serializable may be warranted despite the performance cost.

## Summary

::: {.callout-note title="Summary"}
Isolation levels control how much concurrent transactions can see of each other's changes, trading concurrency for correctness. The SQL standard defines four levels. Read Uncommitted allows dirty reads. Read Committed, the most common default, prevents dirty reads but allows non-repeatable and phantom reads. Repeatable Read also prevents non-repeatable reads; the standard still allows phantoms, although snapshot-based implementations such as PostgreSQL's prevent them while still allowing write skew. Serializable prevents all anomalies by making transactions behave as if they ran one at a time, at the cost of blocking or aborted transactions that must be retried. Use the lowest level that is safe for your application, and know how your database actually implements each level.
:::

## References

1. H. Berenson et al., [A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/publication/a-critique-of-ansi-sql-isolation-levels/), SIGMOD 1995
2. Martin Kleppmann, _Designing Data-Intensive Applications_, O'Reilly, chapter 7
3. [PostgreSQL documentation: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
4. [MySQL documentation: Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
