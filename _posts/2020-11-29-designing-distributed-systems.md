
---
layout: post
title:  Notes from Designing Distributed Systems by Brendan Burs
date:   2020-11-29 00:00:00 -0700
categories:
tags:
---

# 1. Single Node Patterns

## 1.1 Sidecar pattern

A modular container can be plugged in more than one place with minimal changes.

A sidecar is just a container that runs on the same Pod as the application 
container, because it shares the same volume and network as the main container.

```
Application Container <----> Sidecar Container
                    Shared state:
                    network
                    disk
                    etc..
```

Examples:

1. Add HTTPS to a legacy service that serves only HTTP previously. Containerize 
the legacy application to serve HTTP at localhost. Then add sidecar (e.g. Ngnix) 
to serve HTTPs, terminate TLS and forward to the legacy host.

2. Add dynamic configuration to a legacy service that only reads a static config
previously. No change on the legacy system. A new sidecar fetchs dynamic
configurations through API, then update the local config file (shared access by 
both the sidecar and the legacy app). The legacy app either watches the config
file change, or receives SIGHUP signal from sidecard to reload the configuration.

3. Log watcher, log pusher, monitoring agent.

## 1.2 Ambassadors pattern

```
Application Container <----> Ambassadors Container ----> External Services
                    Shared state:
                    network
                    disk
                    etc..
```

Also another form of Sidecar. An ambassador container brokers interactions 
between the application container and the rest of the world.

Examples:

1. A database application has different connection strings for different
environements. The application container can be configured to always point to 
localhost. The ambassador container can detect different environments and adjust 
to test, staging, production etc.

2. Request splitting. Some fraction of all requests are not serviced by the main 
production service but rather are redirected to a different implementation of the 
service, often for experimentation. Alternatively, it can be used to run shadow
mode, where all traffic goes to both the production system as well as a newer, 
undeployed version. The responses from the production system are returned to the 
user, while the responses from the shadowed service are ignored.

## 1.3 Adapter pattern

```
Application Container <--> Adaptor Container <--> External Interface <- External Consumer
```

Examples:

1. Monitoring (Prometheus)

2. Logging (Fluentd). An adapter container can help to translate different 
logging formats into a single structured representation that can be consuemd by 
the log aggregator. the adapter is taking a heterogeneous world of applications 
and creating a homogenous world of common interfaces.

# 2. Serving Patterns

## 2.1 Replicated load-balanced services

### Stateless services

Stateless services scales well with horizontal scaling. However, it is equally
important build and deploy a readiness probe to inform the load balancer. Many 
applications require some time to become initialized before they are ready to 
serve. A readiness probe determines when an application is ready to serve user 
requests. This can be in form of a special URI that implements this check.

### Session-tracked services

Generally speaking, session tracking is performed by hashing the source and 
destination IP addresses and using that key to identify the server that should 
service the requests. This is accomplished via a consistent hashing function.

### Application layer-replicated services

A caching layer in between the stateless application and the end-user request.
For example, a caching proxy for a web application. If two users request the 
same web page, only one request will go to your backend; the other will be 
serviced out of memory in the cache.

## 2.2 Shareded services

In contrast to replicated services, with sharded services, each replica, or 
shard, is only capable of serving a subset of all requests. 

### Shared cache

When designing a sharded cache, there are a number of design aspects to consider:

• Why you might need a sharded cache

The primary reason for sharding any service is to increase the size of the data
being stored in the service

• The role of the cache in your architecture

• Replicated, sharded caches

• The sharding function

Determinism - The output should always be the same for a unique input.

Uniformity - The distribution of outputs across the output space should be equal.

For our sharded service, determinism and uniformity are the most important 
characteristics.

## 2.3 Scatter / getter

Requests are simultaneously farmed out to all of the replicas in the system. 
Each replica does a small amount of processing and then returns a fraction of 
the result to the root.

Scatter/gather is quite useful when you have a large amount of mostly independent 
processing that is needed to handle a particular request. Scatter/gather can be 
seen as sharding the computation necessary to service the request.

Examples:

1. Distributed document search. Each node searches for a subset of documents.

## 2.4 Functions & event driven processing

FaaS (Function as a Service) is inherently an event-based application model.

## 2.5 Ownership election

# 3. Batch Computational Patterns

## 3.1 Work Queue System

## 3.2 Event Driven Batch Processing