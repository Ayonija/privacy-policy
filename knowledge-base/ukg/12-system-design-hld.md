# 12 — System Design (HLD) — HIGHEST WEIGHT AT P5

> This decides the loop. Two things to internalize: (1) a **repeatable framework** so you never freeze, and (2) the **building blocks** so every trade-off is at your fingertips. Then two worked designs that were explicitly asked: **"scale 1 → 10,000 req/sec"** and **"review/rating system."**

---

## PART A — The framework (say it, then follow it every time)

🎤 **Open every design with this:** *"Before I design, let me clarify scope and scale, because the right architecture depends entirely on the numbers. I'll start with requirements, do a quick capacity estimate, sketch the high-level components and data flow, then go deep where you want — data store, caching, partitioning, and failure modes."*

**The 6 steps:**
1. **Clarify** (2–3 min) — functional scope, scale, read/write ratio, consistency needs, latency SLA, who/where the users are.
2. **Estimate** — QPS, storage, bandwidth (back-of-envelope; show the math).
3. **API + data model** — a few key endpoints and the core entities.
4. **HLD** — components + data flow (client → LB → service → cache/DB → queue).
5. **Deep dives** — data store choice, caching, sharding, consistency, scaling path, the bottleneck.
6. **Trade-offs & failure modes** — defend each decision; what breaks and how you recover.

**Numbers to memorize (back-of-envelope):**
- 1 day ≈ **86,400 s ≈ 10^5 s**. So **1M/day ≈ 12 QPS**, **100M/day ≈ ~1,160 QPS**.
- Reads usually ≫ writes (often 10:1 to 100:1) → cache and replicate.
- Latency ladder: memory ~100 ns, SSD random read ~100 µs, network round-trip same DC ~0.5 ms, cross-continent ~100+ ms. A disk seek is ~10 ms.
- 1 commodity SQL box: ~thousands of simple QPS; one Redis node: ~100k+ ops/s.

---

## PART B — Building blocks (the vocabulary every deep-dive uses)

**Load balancer (LB)** — distributes requests across servers. L4 (TCP, fast, dumb) vs L7 (HTTP-aware, can route by path/header). Algorithms: round-robin, least-connections, consistent-hashing (sticky to a node for cache locality).

**Horizontal vs vertical scaling** — *vertical* = bigger box (simple, capped, SPOF). *Horizontal* = more boxes (scales far, needs statelessness + LB). Staff answer: **make services stateless** (push session/state to Redis/DB) so you can scale horizontally and autoscale.

**Caching** — store hot data closer/faster.
- **Where:** client → CDN → API/gateway → application (in-memory, e.g. Caffeine) → distributed (Redis/Memcached/Hazelcast) → DB buffer pool.
- **Patterns:** **cache-aside** (app checks cache, on miss loads DB and populates — most common), **read-through/write-through** (cache sits inline; write-through writes cache+DB synchronously — consistent, slower), **write-behind** (write cache, async to DB — fast, risk of loss).
- **Invalidation** ("one of the two hard problems"): **TTL** (expire after N s — simple, allows staleness), **write-through/explicit eviction** (update/delete key on write — fresh, more coupling), **versioned keys**. Pick by staleness tolerance.
- **Pitfalls:** *cache stampede* (many misses hit DB at once when a hot key expires → use locks/`SETNX`, jittered TTL, or refresh-ahead); *thundering herd*; *hot key*.

**CDN (Content Delivery Network)** — geographically distributed edge servers that cache static (and some dynamic) content near users. *Why:* cuts latency (serve from the nearest POP), offloads origin, absorbs traffic spikes, improves availability. Use for images/JS/CSS/video and cacheable API responses. (This was explicitly asked — see 🎤 below.)

**Database scaling:**
- **Replication** — copies of data. **Primary-replica**: writes to primary, reads from replicas (scales reads, adds *replication lag* → reads can be stale = eventual consistency). Failover promotes a replica.
- **Sharding / partitioning** — split data across nodes by a **shard key**. **Horizontal partitioning** = rows split by key (range / hash / directory). Scales writes + storage. Cost: cross-shard queries & joins are hard; **hotspots** if key is skewed; rebalancing is painful → **consistent hashing** softens it.
- **Index** — a sorted side-structure (usually B-tree) that turns O(n) scans into O(log n) lookups. (Deep dive in sheet 05.)

**CAP theorem** — in a network **P**artition you must choose **C**onsistency (reject/err to stay correct) or **A**vailability (answer, possibly stale). You never "give up P" — partitions happen. So it's really **CP vs AP** under failure.
- **CP** example: a bank ledger, etcd, ZooKeeper.
- **AP** example: a shopping cart, social feed, DNS.
- **PACELC** (the senior add-on): *if* Partition → C or A; *Else* (normal ops) → Latency or Consistency. Even without partitions you trade latency for consistency (sync vs async replication).

**Consistency models** — **strong** (every read sees the latest write; costs latency/availability), **eventual** (replicas converge over time; reads may be stale), **read-your-writes**, **causal**. Match to the domain: balances = strong; like-counts = eventual.

**Idempotency** — an operation that, repeated, has the same effect as once. *Why it matters:* networks retry; without it a retried "charge card" double-charges. *How:* client sends an **idempotency key**; server records "key → result" and on replay returns the stored result instead of re-doing. (Pairs with at-least-once delivery — sheet 06.)

**Rate limiting** — cap requests per client to protect the system. Algorithms: **token bucket** (tokens refill at rate R, each request spends one; allows bursts — most common), **leaky bucket** (smooths to constant rate), **fixed/sliding window** (count per time window). Implement centrally in Redis at the gateway.

**Message queue / async** — decouple producers from consumers; absorb spikes (buffer), smooth load, enable retries. Sync when the caller needs the result now; async (queue) for slow/unreliable/fan-out work (emails, image processing, analytics). (Kafka deep dive in sheet 06.)

**API gateway** — single entry point: auth, rate limiting, routing, TLS termination, request aggregation. **Service discovery** (Eureka/Consul) lets services find each other's dynamic addresses. (Sheet 04.)

**Multi-tenancy** — one system serving many customers (tenants). Isolation spectrum: **shared DB shared schema** (tenant_id column — cheapest, weakest isolation), **shared DB separate schema**, **DB-per-tenant** (strongest isolation/compliance, costliest). Always scope every query + cache key + index by tenant; this is also an AI data-isolation point (sheet 14.6).

---

## PART C — WORKED DESIGN 1: "Scale from 1 → 10,000 req/sec globally" (explicitly asked)

This is really *"walk the evolution of an architecture under growing load."* Narrate it as a journey — that's what impresses.

**Stage 0 — 1 req/s (MVP):** single box: app + DB together. Fine. Don't over-engineer.

**Stage 1 — 10s req/s:** split tiers — app server ⟷ separate DB. Add a **reverse proxy/LB** in front. Make the app **stateless** (sessions → Redis or JWT) so you can add more app instances.

**Stage 2 — 100s req/s:** **horizontal scale** the app behind the LB (N stateless instances, autoscaling). Add a **read replica** + a **cache (Redis)** with cache-aside for hot reads (kills most DB load given read-heavy traffic). Add a **CDN** for static assets.

**Stage 3 — 1,000s req/s:** DB is now the bottleneck. **Primary-replica replication** for reads; move slow/non-critical work to a **message queue** (async workers). Cache aggressively (cache-aside + TTL + stampede protection). Introduce an **API gateway**.

**Stage 4 — 10,000 req/s, global:**
- **Geo-distribution:** multiple regions; **GeoDNS / Anycast** routes users to the nearest region; CDN at the edge worldwide.
- **Database:** reads scaled by replicas per region; **writes** scaled by **sharding** on a good key. If global writes must stay consistent, accept higher write latency (CP) or go **multi-region active-passive** (one write region) vs **active-active** (conflict resolution needed — CRDTs/last-write-wins).
- **Consistency:** strong where it must be (payments), eventual where it can be (counts, feeds) — explicitly call out which is which.
- **Resilience:** circuit breakers, timeouts, retries with backoff + idempotency keys, bulkheads; autoscaling (K8s HPA); graceful degradation (serve cached/partial on dependency failure).
- **Observability:** distributed tracing, RED metrics (Rate/Errors/Duration), SLOs + alerting (sheet 11).

🎤 **Walk the panel through it:** *"I'd resist designing the 10k-req/s system on day one — that's premature. I'd narrate the evolution. Start with one box. As load grows I separate app and DB, put a load balancer in front, and make the app stateless so I can scale horizontally. The next bottleneck is always reads, so I add a read replica, a Redis cache with cache-aside, and a CDN for static content — given read-heavy traffic that absorbs most of the load. At thousands of req/s the database write path is the constraint, so I offload slow work to a queue and start sharding. At 10k globally I go multi-region with GeoDNS routing to the nearest region, replicas for reads, sharding for writes, and I'm explicit about consistency: strong for money, eventual for like-counts. Throughout, resilience and observability — circuit breakers, idempotent retries, tracing, SLOs — are first-class, not bolted on. The skill here isn't knowing the final architecture; it's knowing the *order* you'd evolve into it and what bottleneck forces each step."*

🎤 **CDN, asked directly:** *"A CDN is a network of edge servers spread around the world that cache content close to users. It exists for three reasons: latency — you serve from the nearest point of presence instead of crossing oceans; origin offload — most static traffic never reaches your servers, so they handle more; and resilience — it absorbs spikes and DDoS and survives an origin blip. You cache static assets — images, JS, CSS, video — and cacheable API responses, controlled by Cache-Control headers and TTLs, with versioned URLs or purge APIs for invalidation."*

---

## PART D — WORKED DESIGN 2: "Review / Rating system" (Amazon/Flipkart style — asked)

**0. Clarify:** What's reviewed (products)? Can a user review a product more than once? Verified-purchase only? Need average rating in real time? Scale (catalog size, reviews/day, read QPS)? Moderation? Helpfulness votes?

**1. Requirements:**
- *Functional:* submit/edit/delete a review (rating 1–5 + text), one review per user per product, list reviews (paginated, sortable by recent/helpful), show aggregate (avg rating + count + histogram), mark helpful, moderation.
- *Non-functional:* read-heavy (browsing ≫ writing, ~100:1), aggregate must be fast, eventual consistency OK for averages, durable for reviews, abuse/spam resistant.

**2. Estimate:** Say 50M reviews, 5M/day new (~60 writes/s), reads 10k+/s on product pages. Each review ~1KB → ~50GB text (modest). Aggregates read constantly → must be precomputed/cached.

**3. API:**
```
POST /products/{id}/reviews        body {rating, text} + Idempotency-Key
GET  /products/{id}/reviews?sort=helpful&page=2
GET  /products/{id}/rating         -> {avg, count, histogram}
POST /reviews/{id}/helpful
```

**4. HLD / data flow:**
- Client → API gateway → **Review service** (stateless, scaled) → **Reviews DB**.
- On write: validate (enforce one-per-user via unique constraint `(user_id, product_id)`), persist, **emit a `ReviewCreated` event** to Kafka.
- An **Aggregation consumer** updates the product's running aggregate (avg, count, histogram) — **don't recompute from all reviews on every read**; maintain incrementally.
- **Cache** product rating + first page of reviews in Redis (read-through, TTL + invalidate on new review).
- Reviews list served from DB read replicas; hot products from cache/CDN.

**5. Data model:**
```
reviews(id, product_id, user_id, rating, text, helpful_count,
        created_at, status)   UNIQUE(user_id, product_id)
        INDEX(product_id, created_at), INDEX(product_id, helpful_count)
product_rating(product_id PK, sum_ratings, count, avg, hist_json)  -- precomputed
```
Aggregate update on new review: `sum += rating; count += 1; avg = sum/count` (atomic, or recomputed by the consumer). Storing `sum` + `count` lets you update average in O(1) without scanning.

**6. Deep dives & trade-offs:**
- **Why precompute the aggregate?** Reads vastly outnumber writes; computing AVG over millions of rows per page-view is the classic mistake. Maintain it incrementally on write via the event consumer → O(1) reads.
- **Consistency:** the average can lag a few seconds (eventual) — totally fine for ratings; the *review itself* should be read-your-writes for the author (so they see their post immediately) — serve their own review from primary or optimistic UI.
- **One review per user:** DB unique constraint is the source of truth (don't rely on app check — race conditions). On conflict → update existing.
- **Helpful votes / hot products:** counter on a hot product is a write hotspot → buffer/aggregate in Redis (`INCR`) and flush periodically; or shard the counter.
- **Spam/abuse:** verified-purchase flag, rate-limit per user, moderation queue (`status=pending`), ML/heuristics for fake reviews.
- **Store choice:** relational (PostgreSQL/SQL Server) fits — structured, needs the unique constraint and aggregates; could put review *text search* in Elasticsearch. NoSQL (Mongo) works if you denormalize per-product, but you lose easy cross-cutting queries.
- **Sharding path:** shard by `product_id` (reviews of a product live together → fast list + aggregate); user's-own-reviews becomes a scatter-gather (acceptable, rarer).

🎤 **Walk the panel:** *"It's a read-heavy system — browsing dwarfs writing maybe a hundred to one — so my whole design protects the read path. The single biggest decision is to never compute the average on read: I maintain a precomputed aggregate per product, updated incrementally when a review event lands, so a product page is an O(1) cached lookup. I enforce one-review-per-user with a database unique constraint, not an app-level check, because that's the only race-safe place. Writes emit a ReviewCreated event so aggregation, search indexing, and spam-checks happen asynchronously without slowing the user. Consistency-wise the average can lag a couple seconds — eventual is fine — but the author should see their own review instantly, so I give them read-your-writes. If a product is a hotspot, the helpful-vote counter is the danger, so I buffer increments in Redis and flush. I'd shard by product_id so a product's reviews and aggregate co-locate."*

---

## ⚠️ Pitfalls & seniority signals (system design)
- **Freezing / jumping to components** before clarifying scale. Always clarify + estimate first.
- **No trade-offs** — naming tech without "here's the cost." Every choice must have a "but."
- **Recomputing aggregates on read**, recomputing instead of incrementally maintaining — classic junior miss (the review system tests exactly this).
- **Ignoring failure modes** — Staff engineers are hired for the unhappy path: partitions, retries, hotspots, cache stampede, replication lag.
- **One-size consistency** — say *which* data is strong vs eventual.
- **Stateful services** — forgetting to make app tier stateless blocks horizontal scaling.
- **Seniority signal:** narrate evolution under load, quantify with rough QPS/storage math, name the bottleneck at each stage, and explicitly defend each decision against its alternative.

---

*Next topic, or drill deeper on this one?*
