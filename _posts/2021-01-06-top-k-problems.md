---
layout: post
title:  Top K Problems
date:   2021-01-06 00:00:00 -0700
categories:
tags:
---

# Use cases

1. Most searched keywords on Google

2. Most liked photos on Instagram

3. Most retweeted tweets on Twitter

4. Most viewed videos on Youtube.

# Requirements

Functional

- topK(k, startTime, endTime)

Non-functional

- Scalable

- Highly available

- Highly performant

- Accurate (produce accurate result as much as possible)

# Design

## Iteration 1

A single host solution is easy to build, for a list of items, we store a map of 
reverse-indexed count and convert that to a a max heap based on the count in memory. 
With the max heap, we can sort out the numbers in O(Nlog(N)) time. A single host
solution will soon meet scalability problem as the amount of data ingestion grows.

## Iteration 2

For a multi-host solution, we partition ingested data into multiple processor 
hosts and later merge the results together, similar to how map-reduce works, but
the data is stored in memory for near real-time latency.

```
                                              Processor hosts
                                         --->|A = 3                |
                                         |   |B = 1 ->[A=3,C=2,B=1]|     Storage host
 (incoming data)                         |   |C = 2                |--->|            |
[A,B,C,D,E,F,A,A,C]  -> |   Data    | -> |                              |Merge n     |
                        |partitioner|    |                              |sorted lists|
                                         --->|D = 1                |    |            |
                                             |E = 1 ->[D=1,E=1,F=1]|--->|            |
                                             |F = 1                |    |            |
                                   
```

The advantage of the multi-host approach is better scalability and throughput.
However, ther are 2 drawbacks:

1. When the data is unbounded, e.g. coming from a stream, we cannot keep 
accumulating the data in memory indefinitely. Suppose we can only accumulate the 
last 1 minute data, how can we compute the top K for the last 60 minutes? One
solution is to store the partitioned data on disk and use batch processing like map
reduce to calculate the aggregated data. However, this results to a slower processing
speed that may not be able to serve near real-time traffic.

2. We need to deal with data replication, rebalancing, hot partitioning etc. 
These incurs operational cost to the team.

## Iteration 3 (Lambda Architecture)



The high level design looks like the following
```

                                  fast path
    |  API  |    | Distributed    |------>|fast processors|-->|storage|                     
--->|Gateway| -> |Messaging System|                             
                 |  e.g. Kafka    |------>|data partitioner|-->|dist. messaging queue|-->|partition processor|-->|distributed file system|-->|map reduce jobs|
                                  slow path       
                                   
```

Requests received in API gateway are aggregated to a distributed messaging system
like Kafka. Serialize data into a compact binary format helps to save network IO 
utilization if the request rate is very high. This will let CPU to pay the price. 
The data processing pipeline is then split into a fast path and a slow path.

Both of the previous attempts store a Map in memory which will grow in size as the
amount of entries grows. We can change Map to a probabilistic model like 
[count-min sketch](https://florian.github.io/count-min-sketch/). Count-min sketch 
has a nice feature of having a fixed & predefined size, but the tradeoff is that 
the result is not 100% accurate. We use the count-min sketch in the fast path.

In the slow path, we can calculate the heavy hitters precisely, but the results
will be available at a delayed interval (minutes to hours depending on data
volume). We can use a map-reduce job to do the trick.

More details can be found [here](https://serhatgiydiren.github.io/system-design-interview-top-k-problem-heavy-hitters)


# Other applications

- Popular trends

- DDOS attack

References 
[1](https://www.youtube.com/watch?v=kx-XDoPjoHw)
[2](https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch)
[3](https://serhatgiydiren.github.io/system-design-interview-top-k-problem-heavy-hitters)