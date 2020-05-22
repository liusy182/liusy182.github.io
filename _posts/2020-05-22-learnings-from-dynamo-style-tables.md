---
layout: post
title:  Learnings from Dynamo-style tables
date:   2020-05-22 00:00:00 +0800
categories:
tags:
---

Some of the learnings for leaderless databases like Dynamo DB.

# Quorum

Leaderless databases usually implement a quorum. with 

```
n: the number of nodes
w: quorum writes, the number of success writes
r: quorum reads, the number of success reads
```

As long as `w + r > n`, we can get strong consistency since the latest read will
contain at least 1 node with the latest write. However, there are some limitations 
in achieving real strong consistency.

- if **sloppy quorum** is applied, `w` and `r` may not have overlapping nodes.

  Sloppy quorum: all READ and WRITE operations are performed on the first N 
  healthy nodes from the preference list, which may not be the first N nodes 
  encountered by traversing the consistent hashing ring.

  The node X that temporarily stores data for an unvailable node Y will also 
  maintain a "hint" that the data belongs to Y. X periodically checks for Y's 
  health and forwards the data to Y once it recovers.

  Sloppy quorum is an option for DDB.

- when **concurrent writes** to the same key occurs, conflict resolution needs 
to take place. Some of the techniques and associated problems:

  - **last write wins**: each write is attached with a timestamp. The write with 
  the most recent timestamp wins and the rest of the writes are silently 
  disgarded.
    
    Last write wins ensure eventual convergence of data with a cost of 
    durability. By using timestamp, it also suffers from problem with skewed 
    clock that results to an older version of the data being selected. It is a 
    poor choice of conflict resolution when data lost is unacceptable. However, 
    it can be a good choice if the data is immutable, such as choosing UUID for 
    key.

    Cassandra only supports last write wins. Riak supports it as an optional 
    feature.

  - **version vectors**: in order to support concurrent writes in a single node, 
  a version is associated with each key. Each write is preceded by a read 
  operation to get the current version. The write must include the current 
  version with all current values. The client is responsible for merging current 
  values such as taking a union of all concurrent values. A deletion marker 
  (**tombstone**) can be used for removed data.

    In a multi-node case, each replica and each key has a unique version number.

- **a write and a read happen concurrently**. When the write is only completed 
in a subset of the nodes, the read may return old value.

- **a write failed in some of the nodes and not rolled back from successful 
nodes in time**

- a node carrying new value fails, **and recovers itself by copying data from 
another node with old value**

# Note repair

There are a few ways to repair a temporarily unavailable node

- **read repair**. When a client reads from multiple nodes and detects some 
nodes has stale data with older version numbers. It sends a write request to the 
stale nodes with a newest version of the data post read. This approach is only 
workable for frequently read data.

- **anti-entropy process**. A background process consistenly look for data 
discrepancies and copy data from one node to another.