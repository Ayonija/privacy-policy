# Week 23 Review — HLD Foundations
**Week 23 | Phase: HLD Mastery | Month 5**

## Focus
Consolidate the seven HLD foundation topics, lock in the decision frameworks, and self-assess with a 10-question quick-fire quiz. Exit this review able to choose the right protocol, database, cache strategy, or reliability pattern in under 60 seconds and articulate the tradeoff clearly.

---

## Master Summary Table: Week 23 Topics

| Day | Topic | Core Decision / Rule | Key Numbers |
|---|---|---|---|
| 152 | HLD Fundamentals | Start monolith; split when team/scale demands; CAP: P unavoidable → choose C or A | 99.9% = 8.7h downtime/yr; 99.99% = 52min/yr |
| 153 | Databases | Default SQL; switch NoSQL at scale or schema-flex; start consistent hashing early | Consistent hashing: only K/N keys remap per node add/remove |
| 154 | Caching | Cache-aside for read-heavy; write-through for consistency-critical; Redis > Memcached | LRU default; LFU for popularity; TTL for time-sensitive |
| 155 | Messaging | Queue for task distribution (once); Kafka for event log (many consumers, replay) | At-least-once + idempotent consumer = pragmatic default |
| 156 | APIs & Protocols | REST external; gRPC internal; GraphQL flexible clients; cursor > offset for deep pages | HTTP 429 rate limit; token bucket allows burst; sliding window most accurate |
| 157 | Search + Storage + Auth | Inverted index for full-text; block for DBs; object for media; OAuth PKCE for mobile | JWT TTL 15–60min; refresh token 30 days; BM25 default in Elasticsearch |
| 158 | Observability + Reliability | RED for services; USE for infra; circuit breaker prevents cascade; saga replaces 2PC | Circuit breaker: CLOSED → OPEN → HALF-OPEN; retry: 2^n + jitter |

---

## CAP Theorem Decision Cheat Sheet

```
Network partition is unavoidable in any distributed system.
Under partition, choose:

CONSISTENCY (CP)                      AVAILABILITY (AP)
────────────────────────────────────  ────────────────────────────────────
Refuse requests rather than           Serve requests with possibly
return stale data                     stale data rather than fail

Use when:                             Use when:
- Financial data (balances, ledger)   - Social feeds, timelines
- Inventory counts                    - Product catalog reads
- Leader election, coordination       - DNS, shopping carts
- Order of events must be correct     - User presence, activity

Systems: Zookeeper, HBase, etcd       Systems: Cassandra, DynamoDB,
                                       CouchDB, Riak

Interview rule: always state which you sacrifice and why it's
acceptable for this specific use case. "For a banking ledger
I choose CP — stale balance is worse than an error. For a
social feed I choose AP — slight staleness is invisible."
```

---

## Database Selection Framework

```
Does your data require ACID transactions?
├── YES → Does it need horizontal write scale (>10K writes/sec per table)?
│          ├── YES → NewSQL: CockroachDB, Google Spanner
│          └── NO  → SQL: PostgreSQL (default), MySQL
└── NO  → What is the primary access pattern?
           ├── Key-value (session, cache, counters)
           │    └── Redis (in-memory) or DynamoDB (durable)
           ├── Document (user profiles, product catalog, varied schema)
           │    └── MongoDB or DynamoDB
           ├── Wide-column (time-series, high write throughput, IoT)
           │    └── Cassandra, HBase
           ├── Graph (social network, recommendations, fraud detection)
           │    └── Neo4j, Amazon Neptune
           └── Full-text search
                └── Elasticsearch (+ a primary DB as source of truth)

Sharding rule: default consistent hashing; range only if range
scans dominate. Always ask: what is the shard key? Does it
create hotspots?
```

---

## Caching Strategy Decision Tree

```
Is the workload read-heavy (>80% reads)?
└── YES → Cache-aside (lazy loading)
           ├── Can you tolerate stale data for seconds/minutes? YES → TTL-based expiry
           └── NO → Event-driven invalidation on write

Is consistency between cache and DB critical?
└── YES → Write-through (simultaneous cache + DB write)
           └── Trade: higher write latency for zero staleness

Are writes extremely high-frequency and some data loss acceptable?
└── YES → Write-back (write to cache, async flush to DB)
           └── Only for non-critical data (metrics, counters, analytics)

Is the data write-once, read-rarely?
└── YES → Write-around (write to DB only; cache on first read)

Stampede protection: always add for popular keys
├── Use mutex (Redis SET NX EX) for cache miss serialization
├── Or probabilistic early expiration (XFetch) for no coordination
└── Or background refresh for predictable high-demand keys
```

---

## API Protocol Decision Framework

```
External public API facing third-party developers?
└── REST — wide compatibility, cacheable, familiar tooling

Internal microservice-to-service communication?
└── gRPC — binary, fast, strongly typed, streaming support

Mobile client with varying data requirements?
└── GraphQL — client specifies exact shape; minimize payload

Browser-to-server with no streaming requirement?
└── REST or GraphQL — both work; gRPC requires gRPC-Web proxy

Real-time: server pushing to client only (feeds, notifications)?
└── SSE (Server-Sent Events) — unidirectional, simple, HTTP

Real-time: bidirectional (chat, gaming, collaborative tools)?
└── WebSocket — full-duplex, persistent, low per-message overhead

Rate limiting: highest accuracy with burst support?
└── Token bucket (burst allowed) — most common production choice
   OR Sliding window (most accurate) — Redis sorted set
   AVOID Fixed window — boundary spike problem
```

---

## Reliability Pattern Reference

| Pattern | Problem It Solves | Key Detail |
|---|---|---|
| Circuit Breaker | Prevent cascade failure | CLOSED → OPEN (fast-fail) → HALF-OPEN (probe) |
| Retry + Jitter | Recover from transient failures | Exponential backoff + random jitter; only on idempotent ops |
| Idempotency Key | Safe retries without duplicates | UUID per request; server checks + stores atomically |
| Saga (Orchestration) | Distributed transaction without 2PC | Each step has compensating transaction for rollback |
| Bulkhead | Isolate downstream failure blast radius | Separate thread pools per downstream dependency |
| Graceful Degradation | Avoid full failure on partial outage | Return cached/default data; disable non-critical features |
| Dead Letter Queue | Handle unprocessable messages | After N retries → DLQ; alert on DLQ depth > 0 |

---

## Quick-Fire Quiz: 10 Questions

Answer these cold before checking answers below.

1. A social media feed system must stay available during a network partition even if some reads return slightly stale data. Is it CP or AP? Name a database that fits.
2. You have a table with 500M rows. You need to paginate a user's post feed. Why is cursor pagination better than offset?
3. You need an API for internal microservice calls with strict schema and low latency. Which protocol?
4. A popular cache key expires and 50K concurrent requests all miss and hit the DB. What is this problem called and name two solutions.
5. An order service, payment service, and inventory service must all succeed for a checkout to complete. They have separate databases. What pattern replaces 2PC?
6. A payment API must handle retries safely without charging a customer twice. What mechanism do you add?
7. What is the difference between SLI, SLO, and SLA?
8. Circuit breaker is in OPEN state. What happens to incoming requests and why?
9. You need full-text search over 10M product descriptions with relevance ranking. What data structure does Elasticsearch use internally?
10. A mobile app uploads a 2GB video. The connection drops halfway. What upload strategy allows resuming without re-sending the full file?

### Answers

1. **AP**; Cassandra or DynamoDB — they serve reads during partition with eventual consistency; sacrifice C for A.
2. Offset pagination must scan and discard OFFSET rows (O(N)); cursor pagination uses an index seek on the last-seen ID (O(log N)). Also stable: new posts don't shift existing pages.
3. **gRPC** — binary Protobuf encoding, HTTP/2, strongly typed schema, bidirectional streaming support, designed for internal service-to-service.
4. **Cache stampede (thundering herd)**. Solutions: (a) Mutex/lock — first miss acquires lock, others wait; (b) Probabilistic early expiration (XFetch) — re-fetch before TTL expires; (c) Background refresh — pre-emptively refresh before expiry.
5. **Saga pattern** — sequence of local transactions, each with a compensating transaction; Orchestration saga uses a central orchestrator to direct steps and trigger rollbacks.
6. **Idempotency key** — client generates UUID, includes as header; server checks if key was processed before processing; stores (key, response) atomically; replays stored response on duplicate.
7. **SLI**: the actual measured metric (e.g., 99.95% requests succeed). **SLO**: internal target (e.g., 99.9% success). **SLA**: external contract with penalties (e.g., 99.5% or credit). SLO is stricter than SLA to provide an error budget buffer.
8. All requests **fast-fail immediately** with an error — no calls made to the downstream service. This gives the failing downstream time to recover and prevents the caller's thread pool from exhausting.
9. **Inverted index** — maps each term to the list of documents containing it; enables O(1) per-term lookup. Elasticsearch uses BM25 for relevance scoring (improvement over TF-IDF, normalizes for document length).
10. **Multipart upload** — split into chunks, upload in parallel, track which chunk ETags were confirmed; on resume call `ListParts` to find confirmed chunks, re-upload only failed chunks, call `CompleteMultipartUpload` with all ETags.

**Score guide:** 10/10 = ready. 8–9/10 = review the missed topic's day file. ≤7 = re-read the failing days before the next interview.

---

## HLD Foundation Mental Model — Exit Map

```
ARCHITECTURE
  Single deployable, simple ops ────────────────► Monolith (default)
  Scale/team demands independence ──────────────► Microservices (Strangler Fig)
  Sync response needed immediately ─────────────► REST / gRPC
  Decouple producer/consumer ───────────────────► Queue / Kafka

STORAGE
  ACID transactions, relational ────────────────► PostgreSQL/MySQL
  Horizontal write scale + ACID ────────────────► NewSQL (CockroachDB)
  High-write, flexible schema ──────────────────► Cassandra / DynamoDB
  Session/counter/leaderboard ──────────────────► Redis
  Media/files/backups ──────────────────────────► S3 (Object Storage)
  Full-text search ─────────────────────────────► Elasticsearch

PERFORMANCE
  Read-heavy workload ──────────────────────────► Cache-aside + Redis
  Static assets + global reach ─────────────────► CDN
  High-throughput pagination ───────────────────► Cursor pagination

RELIABILITY
  Downstream service failing ───────────────────► Circuit Breaker
  Distributed transaction ──────────────────────► Saga Pattern
  Safe retries ─────────────────────────────────► Idempotency Key
  One service starving threads ─────────────────► Bulkhead

OBSERVABILITY
  What happened in one request ─────────────────► Distributed Trace
  Service health aggregate ─────────────────────► Metrics (RED method)
  Step-by-step debug ───────────────────────────► Structured Logs + Correlation ID
```

---

## Next Week Preview — Week 24: HLD Advanced + Classic System Design Problems

| Day | Topic |
|---|---|
| 159 | Classic Design: URL Shortener (Bit.ly) |
| 160 | Classic Design: Rate Limiter (full system) |
| 161 | Classic Design: News Feed / Timeline (Facebook/Twitter) |
| 162 | Classic Design: Notification System |
| 163 | Classic Design: Distributed File Storage (Dropbox/Google Drive) |
| 164 | Classic Design: Search Autocomplete |
| 165 | Classic Design: Web Crawler + Weekly Review |

---

## Week 23 Checklist
- [ ] Quiz scored — reviewed any topic with wrong answers against the day file
- [ ] CAP Theorem cheat sheet stated cold: named one CP system and one AP system with justification
- [ ] Database selection framework walked through for 3 hypothetical use cases
- [ ] Caching decision tree applied to a read-heavy system (described aloud)
- [ ] Named all 4 API protocols and their primary use case without notes
- [ ] Named all 7 reliability patterns and the problem each solves
- [ ] Marked any weak areas in `knowledge-base/revision-log.md`
- [ ] Previewed Week 24 topics — confirmed classic system design problems are next
