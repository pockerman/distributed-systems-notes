# Lesson 41: Design a Unique ID Generation System

## Overview

In this lesson I discuss three ways to create unique IDs in a distributed system:

- Using UUIDs
- Using a database
- Using a range handler

Each approach has its advantages and disadvantages, and which one to use depends on the system's requirements.

## Unique ID generation

We frequently need to create unique IDs so that we can label entities. Sometimes this is
straightforward, e.g. use a user's email address to identify them. Sometimes it is not, e.g.
how do we label the tasks executed in the background of our system? We need a way to uniquely identify these tasks so that
we can poll their status.

### UUID

The first approach that comes to mind is to use a UUID. A UUID is a 128-bit number used to identify information in
computer systems. The probability of generating a duplicate UUID is very low. According to Wikipedia, one would have to generate
1 billion version 4 UUIDs per second for about 86 years before the probability of a single duplicate reaches 50%.

Generating a UUID in Python is trivial:

```python
import uuid

# Create a random UUID
my_uuid = uuid.uuid4()
print(my_uuid)  # e.g., 550e8400-e29b-41d4-a716-446655440000
```

There are several UUID versions. Version 4, used above, is generated from 122 random bits.
This approach has the following advantages [1]:

- It is simple.
- It needs no coordination between servers.

However, 128 bits may be more than the application needs, e.g. it may only need 64-bit IDs. In addition, version 4 UUIDs do not increase over time
and are not numeric [1]. Newer versions such as UUIDv7 (RFC 9562) are time-ordered, which addresses the ordering problem.

### Using a database

Relational databases offer `auto_increment` functionality to assign unique IDs to rows. However, this approach does not
work out of the box in a distributed system. A single database server may not be able to handle the load, and once we add more database servers,
generating unique IDs across them is not easy [1].

Flickr uses what they call _ticket servers_ [2]: dedicated database servers whose only job is to hand out IDs using `auto_increment`.
To avoid a single point of failure, Flickr runs two ticket servers, one generating odd and the other even IDs (`auto_increment_increment = 2` with different offsets).

### Using a range handler

One way to avoid constant communication with a central ID generator is to allocate a block, or range, of IDs to each node or service.
A database or ID service is responsible for handing out these ranges.
Each node generates IDs locally within its assigned block until the block is exhausted.

The central allocator keeps track of which ID ranges have already been assigned.
When a node requests a range, the allocator hands out the next unused block (e.g., 1–1000, 1001–2000, etc.).
The node generates IDs sequentially within its range, without any network calls.
When the range is used up, the node requests a new one.
Since ranges never overlap, IDs are unique system-wide.

The advantages of a range handler include:

- Low latency: IDs are generated locally, without a remote call per ID.
- Scalability: it works well with many nodes.
- Simple logic: no complex coordination once ranges are assigned.

On the flip side, the range handler approach has some disadvantages:

- Possible waste: if a node fails before using its whole range, the unused IDs are lost.
- Central dependency: it still requires a reliable central service for range allocation.
- Range exhaustion bursts: if many nodes run out of ranges at the same time, the central allocator may be stressed.
- Ordering: IDs from different nodes are not ordered by creation time.

## Summary

::: {.callout-note title="Summary"}
This lesson explores three common strategies for generating unique IDs in distributed systems: UUIDs, database auto-increment, and the range handler. UUIDs are simple and need no coordination between servers, but they may be larger than necessary and (in version 4) are not ordered. Databases can generate sequential IDs via `auto_increment`, but scaling this across servers takes extra work, as in Flickr's odd/even ticket servers. The range handler assigns each node a unique block of IDs from a central allocator, enabling low-latency local ID generation without constant coordination, though it wastes unused IDs when nodes fail, relies on a central service, and can stress that service when many nodes request ranges at once.
:::

## References

1. Alex Xu, _System Design Interview: An insider's guide_
2. [Ticket Servers: Distributed Unique Primary Keys on the Cheap](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)
3. [Universally unique identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier)
