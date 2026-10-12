# Data Replication Part 4 | Consensus

## Overview

In this note we discuss [consensus](https://en.wikipedia.org/wiki/Consensus_(computer_science)) in distributed systems. Consensus has many applications, e.g.
a set of processes deciding which one should hold a lock or whether to commit a transaction [2]. The [state machine replication](2025-12-30-state-machine-replication.md)
approach we discussed previously is closely tied to the consensus problem.

## Consensus

The consensus problem is the challenge of getting a group of independent processes (nodes) to agree on a single value or decision, even when some of them may fail or messages may be delayed or lost. More precisely, we require a set of processes to agree on a value such that [2]:

- the algorithm tolerates the failure of some of the processes
- every non-faulty process eventually decides on a value (termination)
- the final decision of every non-faulty process is the same (agreement)
- the value decided has been proposed by one of the processes (validity)

State machine replication can be used to solve consensus: propose values to the leader node and choose the first one
to be durably replicated to the followers [2]. Several open-source projects implement this and expose APIs on top of it, for example [etcd](https://etcd.io/) and [Apache ZooKeeper](https://zookeeper.apache.org/).

When components never fail, achieving consensus is trivial. In practice, however, components
do fail, and the network may reorder, delay, lose, or duplicate messages. Under these conditions consensus
is hard [3]. In fact, the FLP impossibility result shows that no deterministic algorithm can guarantee consensus in a fully asynchronous system if even one process may crash; practical algorithms work around this with timeouts and leader election.
Consensus is nevertheless essential in distributed systems, as it lets processes advance in lockstep and makes the
group fault tolerant by allowing it to compensate for failed members [3].

So how do we implement consensus? One approach is leader-based consensus, used by algorithms such as Multi-Paxos, Raft, and Viewstamped Replication [3].
Another approach is quorum-based consensus. The two can also be combined [3].

## Summary

::: {.callout-note title="Summary"}
The consensus problem is the challenge of getting multiple independent nodes to agree on a single decision despite failures, message delays, or losses. Consensus underpins tasks such as lock acquisition, transaction commits, and data replication.

A consensus algorithm must tolerate failures and guarantee termination (every non-faulty process eventually decides), agreement (all non-faulty processes decide the same value), and validity (the decided value was proposed by some process). State machine replication is a practical way to solve consensus: proposed values are ordered and durably replicated, an approach used by systems like etcd and Apache ZooKeeper.

While consensus is trivial when nothing fails, real systems must handle crashes and unreliable networks, which makes the problem hard. Common implementation strategies are leader-based algorithms (Multi-Paxos, Raft, Viewstamped Replication), quorum-based approaches, and hybrids of the two.
:::

## References

1. [Consensus](https://en.wikipedia.org/wiki/Consensus_(computer_science))
2. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
3. Dominik Tornow, _Think Distributed Systems_, Manning Publications
