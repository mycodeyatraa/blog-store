---
title: "AWS Storage & Databases: S3, RDS, DynamoDB, and ElastiCache"
excerpt: "A practical guide to AWS storage and database services — S3 deep dive with lifecycle policies, RDS Multi-AZ and Aurora, DynamoDB table design, and ElastiCache caching strategies."
date: 31-May-2026
author: Abhishek-sinha
tags: ["AWS", "S3", "RDS", "DynamoDB", "ElastiCache", "databases", "storage", "cloud-computing", "devops"]
---

# AWS Storage & Databases: S3, RDS, DynamoDB, and ElastiCache

> **Key insight:** Every application is a state machine. Choosing the right storage technology &mdash; object, relational, NoSQL, or cache &mdash; determines your app&rsquo;s consistency, scalability, and cost characteristics more than any other architectural decision.

We&rsquo;ve covered AWS fundamentals (IAM, VPC, EC2, S3) and compute services (EC2, Lambda, ECS, EKS). Now let&rsquo;s dive into the storage layer &mdash; the part of your architecture that holds your data. AWS offers four major storage paradigms, each optimized for different access patterns, consistency requirements, and cost profiles.

## S3: Object Storage Deep Dive

In Article 1, we introduced S3 basics. Now let&rsquo;s dive deeper into the features that make S3 the backbone of data lakes, backups, and content distribution.

### Storage Classes and Lifecycle

S3&rsquo;s six storage classes let you optimize cost as data ages. The key is to design your lifecycle policy early:

| Class | Durability | Availability | Min object size | Retrieval time | Cost/GB/month |
|-------|-----------|-------------|-----------------|----------------|---------------|
| Standard | 99.999999999% | 99.99% | None | Milliseconds | ~$0.023 |
| Intelligent-Tiering | 99.999999999% | 99.99% | None | Milliseconds | ~$0.023 + monitoring |
| Standard-IA | 99.999999999% | 99.9% | 128 KB | Milliseconds | ~$0.0125 |
| One Zone-IA | 99.999999999% | 99.5% | 128 KB | Milliseconds | ~$0.01 |
| Glacier Instant | 99.999999999% | 99.9% | 128 KB | Milliseconds | ~$0.004 |
| Glacier Deep Archive | 99.999999999% | 99.9% | 128 KB | 12 hours | ~$0.001 |


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-storage-and-databases-s3-rds-dynamodb-and-elasticache/images/diagram_1.png)


> **Key insight:** Set up lifecycle policies on day one, not after your bill spikes. A common pattern: 30 days Standard &rarr; 60 days Standard-IA &rarr; 90 days Glacier &rarr; 365 days Deep Archive, then delete old versions after 2 years.

### S3 Performance Best Practices

S3 automatically scales to high request rates, but performance depends on your key naming scheme. Each partition in S3 handles ~5,500 GET/3,500 PUT requests per second. To maximize throughput:

- **Use random key prefixes** &mdash; instead of `users/1/photo.jpg`, use `users/a1b/photo.jpg`. The hash prefix spreads load across partitions.
- **Use S3 Transfer Acceleration** for large uploads over long distances (uses AWS edge locations)
- **Multipart uploads** for objects >100 MB &mdash; parallel upload parts for faster transfer
- **Byte-range fetches** to download specific ranges of large objects

### S3 Event Notifications

S3 can trigger events on object creation, deletion, or restoration. Common targets: Lambda, SQS, SNS. This is the foundation for event-driven architectures on AWS. Example use cases: auto-resize uploaded images via Lambda, trigger ETL jobs on data arrival, send notifications on file uploads.

## RDS: Relational Databases, Managed

Amazon RDS handles database administration &mdash; provisioning, patches, backups, failover &mdash; for six database engines: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

### Deployment Architectures

**Single-AZ** &mdash; one database instance in one AZ. Cheapest option but no failover. Suitable for dev, test, and non-critical workloads.

**Multi-AZ** &mdash; synchronous standby replica in a second AZ. Automatic failover with no data loss.

**Read replicas** &mdash; up to 15 asynchronous read replicas (cross-region supported). Reduce load on primary for read-heavy workloads.


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-storage-and-databases-s3-rds-dynamodb-and-elasticache/images/diagram_2.png)


### Aurora: AWS&rsquo;s Custom Database Engine

Amazon Aurora is MySQL and PostgreSQL-compatible with custom storage architecture. It decouples compute from storage &mdash; the storage layer auto-scales up to 128 TB and is replicated across 6 copies in 3 AZs.

**Key features:**
- **Auto-scaling storage** &mdash; grows in 10 GB increments up to 128 TB
- **Aurora Serverless v2** &mdash; scales from 0.5 to 256 ACUs in milliseconds
- **Global Database** &mdash; one primary region, up to 5 secondary regions with sub-second replication
- **Backtrack** &mdash; rewind your database to a specific point in time without restoring a backup

> **Key insight:** Aurora costs more than standard RDS, but the storage auto-scaling, faster failover, and better read replica performance often make it cheaper overall when you factor in reduced operational overhead.

## DynamoDB: NoSQL at Scale

Amazon DynamoDB is a fully managed key-value and document database with single-digit millisecond latency at any scale.

### Table Design: The Single Most Important Thing

DynamoDB&rsquo;s performance depends entirely on your data model. Unlike RDS where you optimize queries after designing the schema, DynamoDB requires you to design for your access patterns first.

**Primary key options:**
- **Partition key only** &mdash; simple primary key (e.g., `user_id`). All items with the same PK are stored together.
- **Partition key + sort key** &mdash; composite key (e.g., `user_id` as PK, `timestamp` as sort key). Enables querying by PK with range conditions on sort key.

**Secondary indexes:**
- **Local Secondary Index (LSI)** &mdash; same partition key, different sort key. Created at table creation only.
- **Global Secondary Index (GSI)** &mdash; different partition and sort key. Can be added later.


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-storage-and-databases-s3-rds-dynamodb-and-elasticache/images/diagram_3.png)


### Read/Write Capacity Modes

| Mode | Description | Best for |
|------|-------------|----------|
| Provisioned | Set WCU/RCU, enable auto-scaling | Predictable workloads |
| On-Demand | Pay per request, no capacity planning | Unpredictable traffic, new applications |

> **Key insight:** Start with on-demand for new applications. After a few weeks, examine CloudWatch metrics and switch to provisioned if your traffic pattern is predictable. On-demand costs up to 5x more for steady-state workloads.

### DynamoDB Streams and Change Data Capture

DynamoDB Streams captures a time-ordered sequence of item-level changes. Combined with Lambda, this powers event-driven architectures: sync to Elasticsearch for search, replicate to another region, trigger downstream processing.

## ElastiCache: In-Memory Caching

Amazon ElastiCache provides managed Redis and Memcached. Caching is the most cost-effective performance optimization &mdash; a well-cached application reduces database load by 10-100x.

### Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Strings, hashes, lists, sets, sorted sets, streams | Simple key-value |
| Persistence | RDB snapshots, AOF logs | None &mdash; ephemeral |
| Replication | Multi-AZ, read replicas, cluster mode | No replication |
| Atomic operations | INCR, DECR, transactions | Limited |
| Use case | Caching, session store, rate limiting, pub/sub, leaderboards | Simple caching, CPU-bound workloads |

### Caching Strategies

**Lazy caching (cache-aside)** &mdash; application checks cache first. On miss, reads from database and writes to cache with TTL. Simplest pattern, handles cache failures gracefully (falls back to DB).

**Write-through** &mdash; application writes to cache first, then database. Data is always fresh in cache, but writes are slower.


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-storage-and-databases-s3-rds-dynamodb-and-elasticache/images/diagram_4.png)


> **Key insight:** Always set a TTL on cached items. Without a TTL, stale data lives forever and you&rsquo;ll need manual cache invalidation logic &mdash; the hardest problem in computer science after naming things and off-by-one errors.

## Putting It Together: Storage Decision Framework


![diagram_5](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/aws-storage-and-databases-s3-rds-dynamodb-and-elasticache/images/diagram_5.png)


> **Final Takeaway:** Choose storage based on data shape and access patterns, not familiarity. Relational databases are not obsolete &mdash; they&rsquo;re optimal for structured data with complex joins. DynamoDB excels at key-value lookups at scale. S3 is the right place for files, backups, and data lakes. ElastiCache is the cheapest way to make everything faster. The best architectures combine multiple storage services, each serving its optimal purpose, orchestrated through event-driven patterns.
