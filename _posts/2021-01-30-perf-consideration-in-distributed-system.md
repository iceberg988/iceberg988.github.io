---
layout:     post
title:      "blktrace"
subtitle:   "Performance consideration in distributed system"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/coke1.jpg"
tags:
  - System Design
  - Distribute System
  - Performance
---

This article is mostly a note of learning for distributed system. The motivation is to learn how to design large-scale system from performance point of view.

# A possible high level design

The following might be an ugly high level design. We start from it to see how to scale each component. To make a component scalable means when we increase the resources in the system running the component, performance should also increase proportionally. Linear performance increase would be a ideal case. 

Instead of vertical scaling, horizontal scaling would be preferred in many cases.

```shell
         Client Requests
              |
              V
         Load Balancer
              |
              V
         Web Server
              |
              V
         Application Server       
         |                |
         V                V
    Read API            Write API --> Message Queue 
     |    |               |             |
     V    |(cache miss)   |             |
   Cache  |               V             V
          V           Primary DB   Workers(Consumers)
      DB Replicas <-------|             |
                   (async)             ...      
```
# scale up or out

eventually hit an endpoint

# 

```shell
 build ----> measure
       <----
```

SQL or NoSQL?

* Start with SQL
  * eastablished and well-known technology
  * tons of resources avaiable
  * Should not be a problem to handle the first millions for users
  * Identify the patterns for futher scalability
* Need NoSQL?
  * Very low-latency applications
  * highly non-relational data
  * Metadata-driven data sets
  * Schema-less data constructs
  * Rapid ingestion of data(thoursands of records per second)
  * Massive amounts of data(in terabyte range)  

# Scale the design

Typically, the following techniques can be used to address the scalability issues after the bottlenecks are identified. Remember everything is a trade-off. We should take the business need priority into account while we apply the design to the system. 

* Load balancer
* Horizontal scaling vs. Vertical scaling
* Caching
* Database sharding

## Load balancer

Load balancers determine which worker instance should handle the user requrests.  It can help prevent requests overloading the server resources.

![Image](/assets/images/posts/load-balancer.png){:.shadow}
[source](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)

## Caching

### Where can the cache be located?

From the front-end client to the backend servers and storage, the cache can be located at different layer to optimize the read/write performance.

* Client caching
* CDN caching
* Web server caching
* Application server caching
* Database caching

### When to update the cache?

* Cache asise
* Write through
* Write back(write behind)
* Refresh ahead

### Disadvantage


# Communication

## HTTP

## TCP/UDP

## RPC/REST API


## References

* <https://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html>