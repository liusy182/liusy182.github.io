
---
layout: post
title:  Distributed Locks
date:   2021-01-02 00:00:00 -0700
categories:
tags:
---

# Why do we need distributed locks?

1. Efficiency

When there is one processing that only needs to be done once, and we have multiple
hosts to do this job, we need a lock so that only one host picks up the job. Think
about a cron job that is scheduled to run at the same time on multiple hosts.

2. Correctness

For the same reason, when we need to ensure the job is only done once, we need a
lock so that only one host does the job.

# Design

Requirements:

1. Mutual exclusion. At a given moment, only 1 client can hold the lock.

2. Deadlock free. Eventually it is always possible to acquire the lock, even if
the client that previously acquired the lock crashed.

3. Fault Tolerant. As long as the majority of the instances are up, the clients
are able to acquire the locks.

A simple design is to use a single redis instance as the lock. A client writes a 
key with TTL to acquire the lock. The lock is released either when the same client 
deletes the key, or the TTL has reached. The mutual exclusion guaranttee is only
valid for the period of the TTL. Because of this reason, "lease" is a more proper
term than "lock".

This is a good start, but there are a few problems which will
be discusssed in more details.

## Lock Ownership

If all clients write to the lock with the same key/value, the lock is no longer
effective because the lock written by client A might be accidentally unlocked by
client B. Think about the case below:

1. client A acquires the lock with a TTL of 5 seconds at time 0.

2. client A continues operations

3. the lock gets automatically released at time 0.

4. client B acquires the lock at time 6.

5. client A finishes operation at time 7, and releases the lock which is now hold 
by client B (unsafe)

To solve this problem, each client should only be allowed to release the lock owned
by itself. The lock should accept a value that is unique to each client, and it is only
released when the unique value provided by the client matches.

There are a few ways to generate this unique value on the client

1. use a combination of client ID and unix time in microseconds. This is not 100%
safe since a multi-threaded client could generate the same ID on different threads.

2. use a UUID. Note that UUID is [not 100% safe](https://github.com/ramsey/uuid/issues/80)
when the data volume is huge.

3. use another unique ID generation service. This service asynchronously generates 
a pool of unique IDs beforehand and serves clients based on request.

## Fault Tolenrance

Redis by default does [asynchronous replication](https://redis.io/topics/replication).
If the master node with lock written goes down before the it is replicated to 
the slave, the client will treat the lock as if it did not exist.

The [redlock algorithm](https://redis.io/topics/distlock) is designed to solve this
problem. The gist of this algorithm is as follows:

1. set up N redis masters.

2. a client gets the current time x

3. the client acquires the lock on N instances, using the same key and a unique value.

4. the client is considered to be successfully acquired the lock if it is able 
to lock (N/2 + 1) instances (majority lock)

5. the client computes time left after acquiring the lock, and use this remaining
time for operation. E.g. TTL = 5secs, time to acquire lock = 2secs, the actual
time that the client holds the lock is 5 - 2 = 3 secs.

6. if the client failed to acquire the lock, it will try to unlock all instances.

One point of consideration is whether the redis needs to be configured with persistence. 
Imagine the following case without persistence:

1. a client A acquires 3 out of 5 redis locks.

2. one of the redis masters with the lock restarted, losing the lock.

3. a client B is able to acquire the same lock, because it sees 3 our of 5 instances
unlocked.

There are a few ways to improve fault tolerance of such scenarios:

1. configure redis with [persistence](https://redis.io/topics/persistence). E.g.
sync to persistence on every second by default.

2. adjust the configuration such that a client needs to acquire (N/2 + 2) or even
more locks.

References:
[1](https://redis.io/topics/distlock)
[2](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
[3](http://antirez.com/news/101)
[4](https://aws.amazon.com/blogs/database/building-distributed-locks-with-the-dynamodb-lock-client/)