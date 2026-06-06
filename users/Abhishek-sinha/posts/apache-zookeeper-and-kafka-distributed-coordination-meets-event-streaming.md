---
title: Apache Zookeeper and Kafka: Distributed Coordination Meets Event Streaming
date: 30-May-2026
author: Abhishek-sinha
authorName: Abhishek Sinha
authorRole: Full Stack And Mobile Developer
authorAvatar: 
authorBio: Full Stack And Mobile Developer
authorGithub: https://github.com/abhisheksinha20p
authorLinkedin: https://www.linkedin.com/in/abhishek-sinha-0897aa23b/
tags: [zookeeper, kafka, system-design, distributed-systems, microservices, event-streaming]
category: Zookeeper
categories: [Zookeeper, Kafka, System Design]
excerpt: >-
  Apache Zookeeper and Kafka: Distributed Coordination Meets Event Streaming

When building distributed systems, two names come up constantly — Apache Zookeeper and Apache Kafka.. They solve very differ
readTime: 5 min read
---

# Apache Zookeeper and Kafka: Distributed Coordination Meets Event Streaming

When building distributed systems, two names come up constantly — Apache Zookeeper and Apache Kafka. They solve very different problems but share a deep historical bond. Zookeeper provides the coordination backbone that distributed systems need to stay consistent, while Kafka is the high-throughput event streaming platform that modern data pipelines rely on.

This article breaks down both systems, how they work individually, how they integrate, and where the ecosystem is heading with KRaft.

---

## What is Apache Zookeeper?

Apache Zookeeper is a centralized coordination service for distributed systems. It maintains configuration information, naming, synchronization, and group services — the kind of plumbing every distributed application needs but no one wants to rebuild from scratch.

**Core primitives:**
- **Znodes** — hierarchical, file-system-like data nodes
- **Ephemeral znodes** — auto-delete when the creating session disconnects
- **Sequential znodes** — append a monotonic counter for ordering
- **Watchers** — one-shot notifications when znodes change


![diagram_1](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/apache-zookeeper-and-kafka-distributed-coordination-meets-event-streaming/images/diagram_1.png)


The key insight is **ZAB (Zookeeper Atomic Broadcast)** — a crash-recovery consensus protocol. Writes go through the leader, which proposes them to a majority of followers. Once a majority acknowledges, the write is committed. Reads can be served by any follower from local memory, making reads extremely fast.

## What is Apache Kafka?

Apache Kafka is a distributed event streaming platform. It publishes, stores, and processes streams of records in a fault-tolerant, scalable way. Think of it as a distributed commit log — durable, ordered, and replayable.

**Core concepts:**
- **Topics** — logical channels for related records
- **Partitions** — shards within a topic for parallelism and scaling
- **Brokers** — servers that store partition data
- **Producers** — applications that write records
- **Consumers** — applications that read records


![diagram_2](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/apache-zookeeper-and-kafka-distributed-coordination-meets-event-streaming/images/diagram_2.png)


Each partition is an ordered, immutable sequence of records. Producers append to the tail, consumers advance their offset. Partitions are replicated across brokers for fault tolerance — one leader handles all reads/writes, followers replicate passively.

---

## How Kafka Used Zookeeper

For over a decade, Kafka used Zookeeper as its metadata store and coordination layer. Zookeeper managed:

| Zookeeper Path | What it stores |
|---|---|
| `/brokers/ids/{id}` | Ephemeral znodes for each live broker |
| `/brokers/topics/{topic}` | Partition assignment and replica sets |
| `/controller` | Ephemeral znode for the active controller broker |
| `/consumers/{group}/offsets` | Consumer group offset commits (legacy, now uses internal topic) |
| `/admin/delete_topics` | Pending topic deletion requests |


![diagram_3](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/apache-zookeeper-and-kafka-distributed-coordination-meets-event-streaming/images/diagram_3.png)


The **controller broker** is a singleton among brokers that handles partition leadership assignment and administrative operations. If it dies, another broker detects the missing ephemeral znode via a watcher and takes over.

---

## The KRaft Revolution

In Kafka 2.8+, the KRaft (Kafka Raft Metadata) mode was introduced as an alternative to Zookeeper-based operation. By Kafka 3.5+, KRaft is production-ready for new deployments.

**Why KRaft?**
- **One fewer system** — no Zookeeper ensemble to manage
- **Single consensus protocol** — Raft instead of ZAB + custom Kafka logic
- **Scalable metadata** — millions of partitions without hitting ZK limits
- **Simpler operations** — fewer moving parts, one config instead of two
- **Faster controller failover** — metadata quorum reacts faster than ZK watchers

**KRaft architecture in a nutshell:**
- A **metadata quorum** (odd number of controller nodes) runs Raft consensus
- The quorum stores all cluster metadata in a compacted internal topic `__cluster_metadata`
- Controllers are part of the quorum; brokers are metadata observers
- No ephemeral znodes, no watchers — metadata is streamed via Raft logs


![diagram_4](https://raw.githubusercontent.com/mycodeyatraa/blog-store/main/users/Abhishek-sinha/apache-zookeeper-and-kafka-distributed-coordination-meets-event-streaming/images/diagram_4.png)


---

## When to Use What

| Aspect | Zookeeper 3.8+ | Kafka + KRaft (3.5+) |
|---|---|---|
| **Purpose** | General distributed coordination | Event streaming + metadata |
| **Consensus** | ZAB (crash-recovery Paxos variant) | Raft |
| **Data model** | Hierarchical znodes | Append-only log / compacted topic |
| **Read performance** | Very fast (in-memory) | Moderate (disk + replication) |
| **Write performance** | Moderate (majority ack) | High (sequential I/O) |
| **Scale limit** | ~100k znodes | Millions of partitions |
| **Typical latency** | ~1-5ms | ~2-10ms |
| **Operational overhead** | Moderate (3-node ensemble) | Lower (no external dependency) |

**Real-world advice:**
- **New Kafka deployments on 3.5+** → Use KRaft mode. Skip Zookeeper entirely.
- **Existing Kafka 2.x clusters** → Plan the upgrade path. Migration tools exist from Kafka 3.2+.
- **Other distributed systems** → Zookeeper still shines for HBase, Solr, and custom coordination needs. But consider etcd or Consul for newer projects.
- **Service discovery / config management** → Don't use Zookeeper for this unless you already run it. etcd or Consul are better fits.

---

## Key Takeaways

1. Zookeeper solves **distributed coordination** — consensus, leader election, naming, and configuration for any distributed system
2. Kafka solves **event streaming** — durable, scalable, replayable record storage and processing
3. For over a decade, Kafka used Zookeeper as its metadata backend — broker registration, controller election, topic metadata
4. **KRaft** eliminates Zookeeper from Kafka, using a Raft-based metadata quorum instead — fewer systems, simpler operations, better scalability
5. The ecosystem is converging: KRaft is the future for Kafka, while Zookeeper remains relevant for its original coordination use cases

Understanding both systems — and their relationship — is essential for anyone designing modern distributed architectures. Whether you're running a streaming pipeline or building a consensus-critical control plane, knowing where each tool fits saves you months of operational headaches.
