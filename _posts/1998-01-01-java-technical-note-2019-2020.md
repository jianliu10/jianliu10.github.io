---
layout: post
title:  "Java-Spring Technical notes - 2019"
date:   2019-10-01 00:00:00 -0500
categories: tech-java-spring
---

# 2019 technical notes #

## 2020 IT job market trend

Enterprise Java applications:
- Kafka (streaming data app, streaming data pipeline, streaming data store)
- Spring Core, Spring Boot, Spring MVC, Spring Integration and Spring Security
- Database development and tools with SQL, O/RM, Flyway, OLTP Database Tuning, etc.
- UNIX (Linux) environment and scripting (bash, shell and python)
- implementing integration solutions with RESTful Web Services
- AWS development using EC2, EB, IAM, S3, SDK, CLI, Code Deploy, Code Commit, Lambda/Step Functions, Cloudfront, Cloudwatch, etc.
- testing automation using Selenium, Cucumber, and Junit is preferred
- NoSQL Database experience
- Spring Batch experience

big data applications:
- Spark


## the art of scalability

- computing/service layer: functional decomposition into micro-services, controller-worker computing cluster.
- storage layer: partitions
- replica: storage partition replication (leader, follower), micro-service replication (load balancing)

## four types of distributed system architecture

- modern three tiered architecture (SPA, api-gateway, micro-services)
- sharding (Storage - data storage physical nodes, partitions, replica, key range/list based partitioning, columnar partitioning)
- lambda serverless architecture (Computation - small annonymous function that is short-lived run. )
- reactive programming  (event messaging, data streaming)


## mode based hashing vs consistent hashing  

https://www.toptal.com/big-data/consistent-hashing

### mode based hashing  

It is used in single local process.  
hash table entry -> bucket, where a bucket is a linked list of (key, value) pairs.  
hash table entry = HashFunc(key) mod N. where N is the hash table size

Problem of mode based hashing: rehashing every keys when the hash table resized. 
solution: consistent hashing. 

### consistent hashing algorithm  

Hash ring.  
In general, only k/N keys need to be remapped when k is the number of keys and N is the number of servers

It is used in distributed system, including sharding, load balancing, distributed caches.  
consistent hash ring is used in Cassandra, Memcached and Redis.


## event sourcing

for example, we add a record in a business table, then publish a event msg for the record to a topic. The db table write and topic msg publish need to be in a single distrubuted transaction. But we cannot use 2PC in modern distributed system. 2PC is not reliable.

Instead, we still need to use local transaction. so the solution is event sourcing.

**event sourcing solution:**    
use db Event Table as message queue, this make a local transaction across multiple tables (business table and Event table).  use db triggers to append events to db Event table.  
another process read events from the event table and publish to Topics.  
Here the Event table is the source of events.

**Use case: **  
Order created with pending state, Order approved, Order cancelled, Order shipped.   
A customer can cancel a order either from a pending state, or from a approved state. But you can not cancel a order from a shipped state. A customer has to return a shipped item to get refund.


## CQRS - Command Query Responsibility Segregation  

Command - upsert, delete operations;  
Query - read operations

use cases:  
aws DocumentDB cluster contains a write node and a read node. upsert and delete requests are sent to the write node. the data change is replicated to the read node. Query requests are sent to the read node.

## Testing
- unit test is less relevant in distributed system
- Integration test is important in distributed system
- set up at lease one basic performance metrics
- debugging - log tracing with request ID throughout the plumbs.
- performance debug - load testing - send test requests in production peak load traffic
- failure/negative testing before release



 