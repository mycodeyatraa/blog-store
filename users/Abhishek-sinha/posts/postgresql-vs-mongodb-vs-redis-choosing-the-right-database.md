# PostgreSQL vs MongoDB vs Redis: Choosing the Right Database for Your System Design

Choosing the right database is one of the most critical decisions in system design. PostgreSQL, MongoDB, and Redis each excel in different scenarios — and understanding their strengths is the difference between a system that scales gracefully and one that crumbles under load.

## The Three Database Paradigms

Modern applications rarely rely on a single database. Most production systems use a polyglot persistence approach, combining multiple databases for different workloads. Let's break down when to use each one.

---

## PostgreSQL — The Battle-Tested Relational Giant

PostgreSQL is an advanced object-relational database management system with over 30 years of development. It's the gold standard for data integrity and complex queries.

### When PostgreSQL Shines

**Financial Systems & Transactions**
PostgreSQL's full ACID compliance and serializable isolation make it the default choice for banking, payments, and accounting systems. If your transaction either completes fully or rolls back cleanly, PostgreSQL guarantees it.

**Complex Joins and Reporting**
Need to join 8 tables with aggregations, window functions, and CTEs? PostgreSQL's query planner handles it efficiently, making it ideal for analytics and reporting.

**Geospatial Data**
PostGIS extends PostgreSQL into a full geospatial database. If you're building location-based features, you get spatial indexing, distance queries, and GIS functions built in.

### Example Schema

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  total DECIMAL(10,2) NOT NULL,
  status VARCHAR(50) DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### Trade-offs

- Scaling writes horizontally requires manual sharding or extensions like Citus
- Memory footprint per connection is higher than simpler databases
- Schema changes (ALTER TABLE) can lock large tables

---

## MongoDB — Schema-Flexible Document Store

MongoDB is a NoSQL document database that stores data as JSON-like BSON documents. Its schema flexibility and horizontal scaling make it the go-to for rapid iteration and large-scale applications.

### When MongoDB Shines

**Rapid Prototyping & MVPs**
No migrations, no ALTER TABLE. Add fields to your documents on the fly as your product evolves. Your dev velocity is directly tied to how fast you can change your data model.

**Content Management Systems**
Blogs, product catalogs, configuration stores — any data with varying structure benefits from MongoDB's document model. Each product can have different attributes without schema changes.

**Real-Time Analytics & Event Sourcing**
MongoDB's aggregation pipeline processes streaming data efficiently. Its change streams let you react to data changes in real time.

### Example Document

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "email": "user@example.com",
  "profile": {
    "name": "Jane Doe",
    "avatar": "https://...",
    "preferences": {
      "theme": "dark",
      "notifications": true
    }
  },
  "orders": [
    { "productId": "abc123", "total": 29.99, "status": "shipped" }
  ],
  "created_at": ISODate("2026-05-30T10:00:00Z")
}
```

### Trade-offs

- No built-in joins (application-level joins via $lookup are expensive)
- Multi-document transactions exist (v4.0+) but with performance cost
- Data duplication leads to larger storage — normalize or denormalize carefully

---

## Redis — The Speed Layer

Redis is an in-memory data structure store that operates at sub-millisecond latency. It's not a primary database for most use cases — it's the accelerator that sits in front of everything else.

### When Redis Shines

**Caching**
The most common use case. Cache database query results, API responses, rendered HTML fragments. With TTL-based eviction, stale data cleans itself up automatically.

```python
# Cache pattern
def get_user(user_id):
    cache_key = f"user:{user_id}"
    user = redis.get(cache_key)
    if user:
        return json.loads(user)
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    redis.setex(cache_key, 3600, json.dumps(user))  # Expires in 1 hour
    return user
```

**Rate Limiting**
Redis's atomic INCR with EXPIRE provides production-ready rate limiting in 3 lines of code:

```python
def check_rate_limit(user_id, max_requests=100, window_secs=60):
    key = f"ratelimit:{user_id}:{int(time.time() / window_secs)}"
    count = redis.incr(key)
    if count == 1:
        redis.expire(key, window_secs)
    return count <= max_requests
```

**Real-Time Leaderboards & Sorted Sets**
Redis sorted sets power gaming leaderboards, trending topics, and any ranked list at lightning speed:

```python
# Add scores
redis.zincrby("leaderboard:weekly", 10, "user:42")
# Get top 10
top = redis.zrevrange("leaderboard:weekly", 0, 9, withscores=True)
```

**Message Queues & Pub/Sub**
Redis lists provide reliable queues; Pub/Sub provides real-time messaging for chat, notifications, and event streaming.

### Trade-offs

- Data must fit in RAM (expensive at scale)
- No complex querying — key-value access patterns only
- Persistence options (RDB/AOF) trade durability for performance

---

## Direct Comparison

| Feature | PostgreSQL | MongoDB | Redis |
|---------|-----------|---------|-------|
| **Data Model** | Relational (tables) | Document (JSON) | Key-Value / Structures |
| **Query Language** | SQL | MQL (JSON-like) | Commands |
| **ACID** | Full ACID | Multi-doc (v4.0+) | Transactional (limited) |
| **Schema** | Strict / Migrations | Flexible / Schema-less | No schema |
| **Scaling** | Vertical + Read Replicas | Horizontal (Sharding) | Cluster Mode |
| **Persistence** | Disk (full) | Disk (WiredTiger) | RAM + optional disk |
| **Latency** | 1-10ms | 1-10ms | <1ms |
| **Use Case** | Primary data store | Flexible data store | Speed layer |
| **Indexing** | B-tree, GiST, GIN, BRIN | B-tree, Text, Geospatial | Primary key only |

---

## Decision Flowchart

When designing a new system, ask yourself these questions:

**Need strict data integrity and complex relationships?**
→ PostgreSQL. It's your source of truth for anything involving money, inventory, or user accounts.

**Need to iterate fast with flexible data shapes?**
→ MongoDB. Content, catalogs, event data — anywhere your schema evolves rapidly.

**Need sub-millisecond response times?**
→ Redis. Cache, sessions, rate limiting, leaderboards — anything that needs to be fast.

---

## Polyglot Persistence: Using All Three Together

The most robust architectures use all three databases together:

```
┌─────────────────────────────────────────────────────┐
│                   Application                        │
├─────────────────────────────────────────────────────┤
│        Redis (Cache + Sessions + Rate Limit)        │
│              ┌──────────────────┐                    │
│              │   Write-Through  │                    │
│              └────────┬─────────┘                    │
├───────────────────────┼─────────────────────────────┤
│        MongoDB (Operational Data)                    │
│     - User profiles, content, events                │
│     - Flexible schema, fast iteration               │
├───────────────────────┼─────────────────────────────┤
│        PostgreSQL (Source of Truth)                  │
│     - Payments, orders, relationships               │
│     - ACID guarantees, complex queries              │
└─────────────────────────────────────────────────────┘
```

### Real-World Example: E-Commerce Platform

- **PostgreSQL**: Orders, payments, inventory (needs transactions and integrity)
- **MongoDB**: Product catalog, reviews, user preferences (schema varies by product type)
- **Redis**: Cart cache, session store, real-time analytics, rate limiting

This pattern gives you the strengths of each database while mitigating their weaknesses.

## Key Takeaways

1. **PostgreSQL** for data integrity, complex queries, and anything involving money
2. **MongoDB** for flexible schemas, rapid iteration, and horizontal scale
3. **Redis** for speed: caching, rate limiting, real-time features
4. **Use all three together** — no single database solves every problem
5. **Match the database to the workload**, not the workload to the database

The best architects don't pick one database — they pick the right database for each job within the system.
