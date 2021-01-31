---
layout: post
title:  Distributed Cache
date:   2021-01-24 00:00:00 -0700
categories:
tags:
---

# Functional requirements

- put(key, value)

- get(key) -> value


# Non-functional requirements

- scalable

- highly available

- highly performant

# Design

First, we can think of a locale cache first, and later extend that into a distributed
cache.

### Local v.s. remote cache

A local cache can be implemented by designing a LRU with a hashmap and a 
double-linked list. A LRU cache has constant memory footprint.

A naiive distributed solution is to have a simple sharding stretegy to map a 
key range to a host with a MOD function. Each host runs a LRU cache 
implmentation. The problem of such approach is that if we need to later 
add / remove a cache host, we need to remap the entire key range to all the hosts.

### consistent hashing

A better solution is to use consistent hashing. With consistent hashing, each
host is mapped to a location in a consistant hash ring by certain hash funcction, such as
hash of the host IP address. Each host is responsible for a range of keys on the 
hash ring, starting from its hash until the next host's hash. During the look up,
we first calculate the hash of the key and identify its location on the hash ring,
then look anti-clockwise to identify the host. When a new host is added, it is
responsible for a subset of the key ranges from its predecessor. This means we only
need to rehash a subset of key ranges when a new host is added.

Consistent hashing has 2 major drawbacks:

1. Domino effect. When a cache host dies, its load gets transferred to a different
host. This transfer may overload that host, causing it to fail. One by one, this 
causes a chain of failures.

2. Since hosts pick key ranges based on certain hash function, cache hosts are 
not split evenly on the hash ring, causing uneven distribution of key ranges.

One simple solution is to add one server to the hash ring multiple times. There are
also solutions like [Jump Hash](https://arxiv.org/pdf/1406.2294.pdf).

### picking the shard

The next question is, who is responsible for computing the hash and routing
to the right host?

One design is to have the cache client knowing a list of server IPs in sorted order. 
When it needs to query for a cache, it first uses binary search to locate the 
right host, then talks to the host with TCP or UDP protocol.

There are several ways to maintain the list of server IPs

- Store the host IPs in a configuration file and deploy it with a CD pipeline.
This approach is simple but less flexible in terms of update.

- Store the host IPs in a file in a S3 storage. The client service has a daemon
process that periodically pulls the updated version of the file. This approach
is more flexible, but the drawback is that we need to keep the file in S3 storage
up-to-date whenever there is host changes, either manually or through a CI/CD
pipeline.

- Use a configuration service (e.g. zookeeper). Each cache host registers itself
to the configuration service and sends periodic heartbeat to the configuration 
service. Whenever a heartbeat stops, the configuration service unregisters
the cache host that is no longer accessible. The cache client fetches the updated
host list from the configuration service. This approach has highest operational
cost and is most complicated, but it is the most flexible.

A common problem for sharding is hot patition, when a particular shards serve
more traffic than its peers. Adding more hosts to the hash
ring may not help the problem, because the new host could split any shard but
what we are looking for is to split a particular shard.

Another problem with the current design is the availability. When a shard
dies, the cache it holds will become unavailable, resulting to cache misses until
keys are rehashed.

Both the hot patition and availability problem can be solved with data replication.
The data replication can be broaded categorized into 2 categories:

1. probabilistic protocols like gossip protocol that tend to favor eventual consistency.

2. consensus protocols like 2- 3-phase commit, raft, paxos etc. They tend to favor
strong consistency.

One solution is to use a leader-follower approach where we have 1 leader partition
with 2 follow patitions. A write operation goes to the leader partition, but a read
operation goes to all patitions. With this design, we can easily scale out read
replicas to deal with hot partitions.

Another point of discussion is the leader election. One way to do leader election
is to use a configuration service like zookeeper. Configuration service acts like
a source of authority for all clients. It is usually consists of an odd number of
nodes so that to achieve quorum.

One caveat is that if we do asynchronous replication to replicas (for performance
reasonse), we may not achieve true high availability as there are possibilities 
of data loss if the leader host goes down before it successfully replicates the 
data. For a cache service, this is probably an acceptable tradeoff since we favor
performance over consistency. However, if consistency is a concern, we could choose
synchronous replication or quorum write instead.

Using a cache client to pick shards has a drawback of complicating the client
logic. Another approach is to have a simple cache client send request to a random
host. The host is responsible for computing the hash and redirects the request to
the shard that stores the data. This idea is used by redis cluster.

### other notes

Another performance improvement we can do is to introduce local cache on the cache
client. The cache client wraps a local cache in memory, and only looks up from
the distributed cache in the event of local cache miss.

Potential metrics we want to emit for a distributed cache:

- number of faults

- latency

- number of hits and misses

- CPU and memory utilization

- network IO

References 
[1](https://www.youtube.com/watch?v=iuqZvajTOyA&t=589s)
[2](https://www.youtube.com/watch?v=tzsOc-hBPfw)
[3](https://www.youtube.com/watch?v=DUbEgNw-F9c&t=24s)
[4](https://www.youtube.com/watch?v=U3RkDLtS7uY)

