---
title: Redis in System Design: From Caching to Distributed Locking
date: 2026-05-30
author: Abhishek-sinha
authorName: Abhishek Sinha
authorRole: Full Stack And Mobile Developer
authorAvatar: https://avatars.githubusercontent.com/abhisheksinha20p
authorBio: Full Stack And Mobile Developer
authorGithub: https://github.com/abhisheksinha20p
authorLinkedin: https://www.linkedin.com/in/abhishek-sinha-0897aa23b/
tags: [redis, system-design, caching, database, backend]
category: Redis
excerpt: >-
  Redis is an in-memory data structure store that powers some of the highest-throughput systems in production.. From caching to real-time leaderboards, its versatility makes it a cornerstone of modern s
readTime: 4 min read
---

Redis is an in-memory data structure store that powers some of the highest-throughput systems in production. From caching to real-time leaderboards, its versatility makes it a cornerstone of modern system design.

## What Makes Redis Different

Unlike traditional databases that store data on disk, Redis keeps everything in memory. This fundamental choice gives it sub-millisecond latency, but it also means you need to think differently about data persistence and memory management.

Redis offers more than just key-value storage. It provides rich data structures — strings, hashes, lists, sets, sorted sets, bitmaps, and hyperloglogs — each optimized for specific use cases. This is what makes Redis a swiss army knife for backend engineers.


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/redis-in-system-design-from-caching-to-distributed-locking/images/diagram_1.png)


## Caching — The Primary Use Case

Caching is where Redis shines brightest. By storing frequently accessed data in memory, you can reduce database load by orders of magnitude.

### Cache-Aside Pattern

The most common pattern is cache-aside. The application checks Redis first. On a miss, it loads from the database and populates the cache.


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/redis-in-system-design-from-caching-to-distributed-locking/images/diagram_2.png)


### Eviction Policies

Redis gives you six eviction policies when memory runs full:

- **noeviction** — return errors on write
- **allkeys-lru** — evict least recently used keys
- **allkeys-lfu** — evict least frequently used keys
- **volatile-lru** — evict LRU keys with TTL set
- **volatile-ttl** — evict keys with shortest TTL
- **volatile-random** — evict random keys with TTL set

For most production systems, `allkeys-lru` is the right default. It requires no TTL management and naturally keeps hot data in memory.

## Rate Limiting with Redis

Redis is exceptionally good at rate limiting because of its atomic operations and the ability to set TTL on keys.

### Sliding Window Algorithm

Using Redis sorted sets, you can implement a sliding window rate limiter in just a few commands:

```
ZREMRANGEBYSCORE user:api:rate:42 0 {now - window}
ZCARD user:api:rate:42
ZADD user:api:rate:42 {now} {request_id}
EXPIRE user:api:rate:42 {window}
```

This pattern tracks timestamps in a sorted set, removes expired entries, and checks the count — all atomically. Twitter and GitHub use variations of this approach.

## Distributed Locking with Redlock

When multiple services need to coordinate access to a shared resource, Redis provides a distributed locking mechanism called Redlock.


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/redis-in-system-design-from-caching-to-distributed-locking/images/diagram_3.png)


Redlock works by acquiring the lock on a majority of Redis nodes. If the majority responds with OK, the lock is held. This prevents issues when a single Redis node goes down.

## Real-Time Leaderboards

Sorted sets make leaderboards trivial. Each player's score is stored as a member-score pair, and Redis keeps them sorted efficiently.

```
ZADD leaderboard:game:daily 1500 player42
ZADD leaderboard:game:daily 2200 player17
ZADD leaderboard:game:daily 1800 player99
ZREVRANGE leaderboard:game:daily 0 9 WITHSCORES
```

This pattern scales to millions of players. Discord uses Redis sorted sets for their community leaderboards, handling thousands of score updates per second.

## Session Stores

Redis is the de facto standard for session storage in web applications. With automatic TTL expiration, you don't need cleanup jobs. A session object stored as a Redis hash lets you read or update individual fields without fetching the entire object.

```
HSET session:abc123 user_id 42 role admin expires 3600
EXPIRE session:abc123 3600
```

## Pub/Sub for Real-Time Messaging

Redis Pub/Sub enables lightweight real-time messaging between services. While not as feature-rich as Kafka, it's perfect for use cases like cache invalidation notifications or broadcasting events to multiple subscribers.

## Production Considerations

Running Redis in production requires careful planning:

- **Persistence** — Use both RDB (point-in-time snapshots) and AOF (append-only log) for durability
- **High availability** — Redis Sentinel provides automatic failover
- **Sharding** — Redis Cluster distributes data across nodes automatically
- **Memory monitoring** — Track used_memory and evicted_keys metrics
- **Connection pooling** — Use connection pools to avoid TCP overhead


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/redis-in-system-design-from-caching-to-distributed-locking/images/diagram_4.png)


## When Not to Use Redis

Redis is not a silver bullet. Avoid it when:

- Your dataset is larger than available memory (unless you use Redis on Flash)
- You need complex SQL queries with joins and aggregations
- You require ACID transactions across multiple keys
- Durability is critical (AOF fsync=always is slow; RDB has data loss window)

For those cases, reach for PostgreSQL, MongoDB, or your database of choice.

---

Redis mastery comes from understanding both its strengths and its limits. Use it for the right reasons — speed, data structure versatility, and operational simplicity — and your system designs will be better for it.