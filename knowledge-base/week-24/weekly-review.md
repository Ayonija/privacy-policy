# Week 24 Review — HLD Advanced + Classic System Design Reference
**Week 24 | Phase: HLD Mastery | Month 5**

## Focus
Consolidate two weeks of HLD content: distributed systems coordination from Week 24 advanced topics and all eight classic system design problems. Exit this review with instant recall of the decision cheat sheet, all benchmark numbers, and the 5-step framework. Self-assess with the 10-question quiz at the bottom.

---

## What Was Covered This Week

| Day | Topic | Key Takeaway |
|-----|-------|-------------|
| 159 | Distributed Systems Advanced | Saga replaces 2PC; Redlock + fencing tokens; hot standby for RPO near zero |
| 160 | Infrastructure & Deployment | Canary limits blast radius; service mesh = mTLS + tracing for free; chaos engineering builds resilience |
| 161 | URL Shortener + Notification | Base62 + Redis cache + DynamoDB; Kafka fan-out + DLQ + Redis idempotency key |
| 162 | Twitter + Instagram | Fan-out hybrid; Redis sorted-set timeline; Cassandra for tweets; presigned S3 |
| 163 | Uber + BookMyShow | Redis Geo + WebSocket; Redis SET NX EX seat lock; State + Strategy patterns |
| 164 | WhatsApp + Google Drive | Snowflake ordering + Cassandra; 4MB chunks + delta sync + Signal Protocol |
| 165 | Full HLD Review | 5-step framework; numbers cheat sheet; decision table; all 8 problems mapped |

---

## 5-Step Framework One-Pager

```
1. REQUIREMENTS (5 min)
   Functional:     what it does | what APIs it exposes
   Non-functional: QPS, read:write ratio, latency SLO, consistency, availability

2. SCALE (5 min)
   QPS     = DAU x actions/day / 86,400  (x2-3 for peak)
   Storage = records x record size x retention
   Cache   = total records x 20% x record size (80/20 rule)
   Bandwidth = QPS x avg payload size

3. HIGH-LEVEL DESIGN (10 min)
   [Client] --> [LB] --> [API Servers] --> [Cache] --> [DB]
                                                 |
                                            [Queue] --> [Worker] --> [Storage]
   Trace write path. Trace read path. Justify DB + cache choices.

4. DEEP DIVES (20 min)
   Pick the hardest component. Have opinions.
   State trade-offs explicitly with numbers.
   Common areas: fan-out, sharding, ordering, locking, sync, real-time delivery

5. WRAP UP (5 min)
   Biggest bottleneck + how to address at 10x scale
   Monitoring: RED metrics, p99 SLO alert, distributed tracing
```

---

## Classic Problems Quick Reference

| System | DB | Cache | Queue | Real-time | Key Pattern |
|--------|----|-------|-------|-----------|-------------|
| URL Shortener | DynamoDB | Redis (10GB hot) | Kafka (analytics) | — | Cache-aside + Base62 |
| Notifications | PostgreSQL | Redis (prefs) | Kafka (per channel) | — | Observer + Strategy + DLQ |
| Twitter Feed | Cassandra | Redis sorted-set | Kafka (fan-out) | — | Fan-out hybrid |
| Instagram | Cassandra (stories) + S3 | Redis timeline | Kafka (fan-out) | — | Presigned S3 + CDN |
| Uber | PostgreSQL (trips) | Redis Geo | Kafka (trip events) | WebSocket (driver) + SSE (rider) | State + Strategy |
| BookMyShow | PostgreSQL | Redis (seat lock) | SQS FIFO (spike) | — | Distributed lock + Observer |
| WhatsApp | Cassandra (messages) | Redis (presence) | Internal routing | WebSocket | Snowflake IDs + Signal |
| Google Drive | PostgreSQL (metadata) | Redis (hot chunks) | Kafka (sync events) | WebSocket (sync) | Delta sync + ACL |

---

## HLD Decision Tree

```
DATA STORAGE

Is data relational with complex joins or ACID transactions?
  YES --> SQL: PostgreSQL (default), MySQL
  NO  --> What is the primary access pattern?
      Key-value (sessions, counters, cache)?
        --> Redis (in-memory) or DynamoDB (durable)
      Time-series / append-only / high write throughput?
        --> Cassandra / InfluxDB
      Document with flexible schema?
        --> MongoDB or DynamoDB
      Full-text search with relevance ranking?
        --> Elasticsearch (plus primary DB as source of truth)
      Graph traversal (social network, fraud detection)?
        --> Neo4j or Amazon Neptune

TRAFFIC PATTERN

Read-heavy (>80% reads)?
  YES --> Redis cache + read replicas + CDN for static assets
Write-heavy (>80% writes)?
  YES --> Message queue to absorb spikes + CQRS + DB sharding

REAL-TIME REQUIREMENT

Bidirectional (chat, collaborative editing, gaming)?
  --> WebSocket
Unidirectional server push (feeds, live scores, notifications)?
  --> SSE (simpler, built on HTTP, auto-reconnect)
Async, decoupled (event fan-out, task processing)?
  --> Kafka (multiple consumers, replay) or SQS (point-to-point)

CONSISTENCY REQUIREMENT

Financial data, inventory, leader election?
  --> Strong consistency (CP): PostgreSQL, ZooKeeper, etcd
Social feeds, caches, presence, product catalog?
  --> Eventual consistency (AP): Cassandra, DynamoDB, Redis
```

---

## Numbers Cheat Sheet

| Conversion | Value |
|-----------|-------|
| 1M requests/day | ~12/sec |
| 10M requests/day | ~116/sec |
| 100M requests/day | ~1,200/sec |
| 1B requests/day | ~11,600/sec |
| 99% SLA | 87.6 hours/year downtime |
| 99.9% SLA | 8.76 hours/year downtime |
| 99.99% SLA | 52.6 minutes/year downtime |
| 99.999% SLA | 5.26 minutes/year downtime |
| Redis | ~100K ops/sec per instance |
| MySQL write | ~1K/sec sustained |
| MySQL read | ~10K/sec |
| Cassandra write | ~50K/sec per node |
| Kafka | ~1M messages/sec across all partitions |
| S3 PUT | ~3,500/sec per prefix |
| S3 GET | ~5,500/sec per prefix |
| RAM access | ~100 ns |
| SSD random read | ~100 microseconds |
| HDD random read | ~10 ms |
| Same-DC network | ~1 ms RTT |
| US to EU cross-region | ~80 ms RTT |

---

## Full Knowledge Base Completion Checklist

### LLD (Weeks 20-22)
- [ ] OOP 4 pillars — can explain each with code example
- [ ] SOLID — can name all 5 and identify violations in code
- [ ] Singleton — thread-safe with enum or double-checked locking
- [ ] Factory + Abstract Factory — registration map, not if-else chain
- [ ] Builder — fluent chain, immutable result, validation in .build()
- [ ] Adapter — Object Adapter via composition, not inheritance
- [ ] Decorator — same interface, stackable wrapping, no subclass explosion
- [ ] Facade — single entry point over complex subsystem
- [ ] Proxy — lazy load, access control, caching proxy
- [ ] Strategy — external algorithm swap, composition over inheritance
- [ ] Observer — subject notifies all observers; push vs pull model
- [ ] State — state object owns its own transition logic
- [ ] Command — execute + undo, history stack for undo/redo
- [ ] Template Method — base class fixes skeleton, subclass overrides steps
- [ ] Can code all 5 core patterns from memory: Singleton, Factory, Strategy, Observer, Decorator
- [ ] Know all 5 confusion pairs and one-line distinguishing tells

### HLD Foundations (Week 23)
- [ ] CAP theorem — CP vs AP examples with real system names
- [ ] SQL vs NoSQL decision framework — can walk through decision tree
- [ ] Database sharding — shard key selection, hotspot avoidance, consistent hashing
- [ ] Caching — write strategies (cache-aside, write-through, write-back), eviction policies, stampede prevention
- [ ] Kafka vs SQS — know when to use each; at-least-once + idempotent consumer
- [ ] REST vs gRPC vs GraphQL — decision by context (external/internal/mobile)
- [ ] Rate limiting — token bucket vs sliding window; Redis implementation
- [ ] Circuit breaker — three states (CLOSED/OPEN/HALF-OPEN) and transitions

### HLD Advanced + Classic Design (Week 24)
- [ ] Saga pattern — choreography vs orchestration with compensating transactions
- [ ] Consistent hashing — virtual nodes; only K/N keys remap on node change
- [ ] Redlock + fencing tokens — algorithm steps and clock-skew mitigation
- [ ] Raft consensus — leader election + log replication step-by-step
- [ ] DR strategies — backup/restore vs pilot light vs warm vs hot standby; RPO and RTO for each
- [ ] Can design URL Shortener — Base62, Redis cache, DynamoDB, 302 vs 301
- [ ] Can design Notification System — Kafka fan-out, idempotency, DLQ, scheduling
- [ ] Can design Twitter Feed — fan-out hybrid, Redis sorted set, Cassandra tweets
- [ ] Can design Uber — Redis Geo, WebSocket, state machine, surge pricing strategy
- [ ] Can design BookMyShow — Redis SET NX EX, optimistic locking, waitlist observer
- [ ] Can design WhatsApp — Snowflake ordering, Cassandra, offline delivery, Signal
- [ ] Can design Google Drive — chunked upload, delta sync, PostgreSQL metadata, ACL
- [ ] Know all numbers in cheat sheet from memory
- [ ] Can execute 5-step framework from blank canvas in 45 minutes

---

## 10-Question Quick-Fire Quiz

Answer these without looking at notes. Check answers below.

1. Fan-out on write vs read — which approach does Twitter use for celebrity accounts?
2. What Redis command atomically sets a seat lock only if it does not already exist?
3. Which SLA gives approximately 52 minutes per year of downtime?
4. Kafka vs SQS — which retains messages for replay by multiple consumer groups?
5. What is RPO and how does it differ from RTO?
6. When does a circuit breaker transition from CLOSED to OPEN?
7. What design pattern does BookMyShow's waitlist notification use?
8. Why is Cassandra chosen for tweet storage over PostgreSQL?
9. How does Google Drive avoid uploading unchanged parts of a large edited file?
10. What delivery semantic requires idempotent consumers to be safe?

---

### Answers

1. **Fan-out on read.** Celebrity accounts (> 1M followers) do not have tweets pre-pushed to follower timelines. At feed read time, celebrity tweets are lazily queried and merged with the precomputed Redis sorted-set cache.

2. **`SET seatId lockToken NX EX 600`** — NX means "only set if Not eXists"; EX 600 sets a 10-minute TTL. Single round-trip, atomic; eliminates the check-then-set race condition.

3. **99.99% SLA** — 52.6 minutes per year, 4.4 minutes per month. Know all four: 99% = 87.6 hrs, 99.9% = 8.76 hrs, 99.99% = 52.6 min, 99.999% = 5.26 min.

4. **Kafka.** Kafka is an append-only log; messages are retained for a configurable period (default 7 days) and any number of consumer groups can read independently. SQS deletes the message once it is consumed — point-to-point only.

5. **RPO = Recovery Point Objective** — the maximum acceptable data loss measured in time (e.g., RPO = 1 hour means at most 1 hour of data can be lost). **RTO = Recovery Time Objective** — the maximum acceptable downtime (e.g., RTO = 4 hours means the system must be back up within 4 hours). Lower = more expensive.

6. The circuit breaker transitions CLOSED → OPEN when **the error rate or failure count exceeds a configured threshold** within a time window (e.g., 5 failures in 10 seconds, or error rate > 50%). Once OPEN, all requests fail fast without calling the downstream service.

7. **Observer pattern.** Booking Service (subject) publishes a SEAT_RELEASED event to Kafka on cancellation. Notification Service (observer) consumes the event, queries the waitlist for the first user, and sends notification. The first user receives a 5-minute exclusive Redis lock to complete the booking.

8. Cassandra suits tweet storage because: (1) tweet workload is append-only (no updates), matching Cassandra's LSM-tree write path; (2) partitioning by userId enables fast user timeline queries without scans; (3) Snowflake clustering key gives time-ordered rows within a partition; (4) horizontal scaling handles 50K writes/sec per node; (5) no joins are needed — tweets are self-contained.

9. **Delta sync with SHA256 chunk hashing.** The Google Drive client splits every file into 4MB chunks and computes the SHA256 hash of each. It maintains a local index of chunk hashes. On file change, only chunks whose hash changed are uploaded. A 1GB presentation with one edited slide = upload 1 chunk (4MB) instead of 1GB.

10. **At-least-once delivery.** At-least-once guarantees a message is delivered at least one time but may deliver it multiple times (on retry after consumer crash before ack). Consumers must be idempotent — processing the same message twice produces the same result as processing it once. Implement via idempotency key checked in Redis before processing.

**Score guide: 9-10 = interview-ready | 6-8 = review the weak areas' day files | < 6 = re-read weeks 23-24 before the next interview**

---

## Week 24 Checklist
- [ ] Quiz completed and scored — reviewed any missed topics against the corresponding day file
- [ ] 5-step framework recited cold without notes
- [ ] Availability nines stated from memory (99% through 99.999%)
- [ ] All benchmark numbers stated: Redis 100K, MySQL 1K write / 10K read, Cassandra 50K, Kafka 1M
- [ ] Can draw architecture for at least 3 classic problems from memory without notes
- [ ] HLD decision cheat sheet reviewed — can identify correct solution for each problem type
- [ ] Weak areas marked in `knowledge-base/revision-log.md`
- [ ] STAR stories reviewed for at least 3 problems (Twitter, Uber, WhatsApp recommended)
