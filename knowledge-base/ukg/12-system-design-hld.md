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

## PART E — UKG-SPECIFIC DESIGNS (🔥 most likely tomorrow — Pune 2026 loop)

> **Why this section:** UKG (Ultimate Kronos Group) builds **workforce-management / HCM SaaS** — timekeeping, scheduling, payroll, leave, HR. Pune system-design rounds this year have been **domain-flavored**: they hand you a UKG-shaped problem and watch how you handle **multi-tenancy, eventual consistency, audit/compliance, and spiky load (everyone clocks in at 9 AM)**. Lead with the domain framing below and you signal "I understand the business," which is exactly what gets the select.
>
> 🎤 **Opening line for any UKG design:** *"Since UKG is multi-tenant SaaS for workforce management, three constraints shape everything I design: every record is **tenant-scoped** and isolated, the data is **compliance-grade** (payroll/labor law → immutable audit trail, correct timezones), and load is **bursty** — clock-ins spike at shift boundaries. I'll design for those explicitly."*

### Cross-cutting UKG concerns (name these in *every* answer)
- **Multi-tenancy:** `tenant_id` on every row, every cache key, every index. Shared-DB-shared-schema for SMB tenants, DB-per-tenant for enterprise/compliance. (See Part B.)
- **Timezone & DST correctness:** store all timestamps in **UTC**; convert at the edge using the **location's** timezone, not the server's. A punch at "9 AM" means 9 AM *local*. This is a classic UKG gotcha — mention it unprompted.
- **Audit & immutability:** payroll/time data is legally sensitive → **append-only event log**, never hard-delete; corrections are new versioned records. Who-changed-what-when.
- **Bursty load:** shift boundaries (7/9 AM, lunch) create thundering-herd writes → **queue + async**, idempotent punches.

---

### UKG DESIGN 1 — Employee Time & Attendance (Punch-Clock) system 🔥 #1 likely

This is UKG's flagship product. If they ask one design, it's probably this.

**0. Clarify:** Punch sources (web, mobile, physical clock/kiosk, biometric)? Offline support on kiosks? Real-time attendance dashboard for managers? Feeds payroll? Geofencing? Scale (employees, tenants, punches/day)?

**1. Requirements**
- *Functional:* punch in/out (+ breaks/transfers), view timecard, manager approves/edits, compute worked hours + overtime, export to payroll, real-time "who's in" dashboard.
- *Non-functional:* **bursty writes** (millions punch within minutes at 9 AM), **durable & auditable** (never lose a punch — it's someone's pay), tenant-isolated, eventual consistency OK for dashboards, **strong/exactly-once for the payroll feed**, timezone-correct.

**2. Estimate:** 10M employees, ~4 punches/day = 40M punches/day ≈ **460/s average but ~50k/s peak** at shift change (the real number). Each punch ~200 B. Spiky → **must buffer**.

**3. API**
```
POST /v1/punches      body {employeeId, type:IN|OUT|BREAK, deviceTs, geo}
                      + Idempotency-Key   (offline retries must not double-punch)
GET  /v1/timecards/{employeeId}?period=2026-06
POST /v1/timecards/{id}/approve
GET  /v1/sites/{siteId}/attendance/live      (manager dashboard)
```

**4. HLD / data flow**
```
                                        ┌─────────────────────┐
  [Web]  [Mobile]  [Kiosk/Clock]        │  Payroll Service     │
     │       │          │               │  (exactly-once feed) │
     └───────┴────┬─────┘               └──────────▲──────────┘
                  │ HTTPS                           │
            ┌─────▼──────┐    enqueue    ┌──────────┴──────────┐
            │ API Gateway│──────────────▶│  Kafka: punch-events│ (durable buffer,
            │ authN/Z,   │   (absorbs    │  partitioned by     │  absorbs the 9AM
            │ ratelimit, │    the burst) │  tenant_id)         │  spike)
            │ tenant ctx │               └──────────┬──────────┘
            └─────┬──────┘                          │
                  │ ack fast (202)        ┌─────────┴─────────┐
                  │                       ▼                   ▼
                  │              ┌─────────────────┐  ┌──────────────────┐
                  │              │ Timecard        │  │ Aggregation/      │
                  │              │ Consumer        │  │ Dashboard Consumer│
                  │              │ → Timecard DB   │  │ → Redis "who's in"│
                  │              │  (append-only,  │  │  (live counts)    │
                  │              │   per tenant)   │  └────────┬─────────┘
                  │              └────────┬────────┘           │
                  └──────read timecards──▶│                    ▼
                                          ▼            [Manager dashboard]
                                  [Timecard DB +
                                   read replicas]
```

- **Punch write path:** Gateway validates + stamps tenant context → **publishes to Kafka and returns 202 immediately**. The kiosk never waits on the DB. Kafka is the durable shock-absorber for the burst — punches are *never* lost even if the DB is briefly slow.
- **Idempotency key** = `(employeeId, deviceTs, type)` so offline kiosks replaying queued punches don't double-count → **at-least-once delivery + idempotent consumer = effectively exactly-once**.
- **Timecard consumer** writes an **append-only** punch log per tenant; a timecard is the *derived* view (hours = sum of IN/OUT pairs, with overtime rules).
- **Dashboard consumer** maintains live "who's clocked in" counts in Redis — eventual consistency is fine for a dashboard.
- **Payroll feed** reads the immutable log at period close → exactly-once (dedupe on punch id).

**5. Data model**
```
punch_events(id, tenant_id, employee_id, type, event_ts_utc, tz,
             source, geo, idempotency_key, created_at)      -- APPEND-ONLY, immutable
   UNIQUE(tenant_id, idempotency_key)        -- race-safe dedupe
   INDEX(tenant_id, employee_id, event_ts_utc)
timecard(tenant_id, employee_id, period, worked_minutes, ot_minutes,
         status:DRAFT|APPROVED, version)      -- DERIVED, versioned (corrections = new version)
```

**6. Deep dives & trade-offs**
- **Why a queue, not direct DB write?** The 50k/s shift-boundary spike would topple a relational DB on direct writes. Kafka absorbs it, smooths to the consumers' pace, and gives durability + replay. *This is the single biggest decision — lead with it.*
- **Offline kiosks:** buffer locally, sync on reconnect; idempotency key makes replay safe. Use the **device** timestamp (when the punch happened), not server receive time.
- **Timezone:** store UTC + the site's IANA tz; "did they clock in late?" is computed in *local* time. DST transitions are where naive designs break.
- **Audit/compliance:** punches are immutable; a manager edit is a *new* event referencing the original ("adjustment"), so payroll disputes have a full trail.
- **Consistency split:** dashboard = eventual; **payroll feed = exactly-once/strong** (money). Say which, and why.
- **Sharding:** partition Kafka and shard the DB by **tenant_id** (natural isolation boundary; a tenant's data co-locates).

🎤 **Walk the panel:** *"The defining property here is bursty, mission-critical writes — everyone punches at 9 AM and a lost punch is someone's missing pay. So I don't write punches straight to the DB; the gateway validates, stamps the tenant context, publishes to Kafka partitioned by tenant, and acks immediately. Kafka is my durable shock-absorber for the spike and gives me replay. Consumers then write an append-only, immutable punch log — the timecard is a derived, versioned view, so corrections never overwrite history, which is what payroll compliance and audits demand. Punches carry an idempotency key of employee-plus-device-timestamp, so an offline kiosk replaying its queue can't double-punch — at-least-once plus an idempotent consumer gives me effectively exactly-once. Everything is UTC stored and converted in the site's local timezone, because 'late at 9 AM' is a local-time question and DST is where naive systems break. I split consistency deliberately: the manager's live dashboard is eventual via a Redis projection, but the payroll feed is exactly-once. And I shard by tenant_id throughout, which is both my scaling axis and my isolation boundary."*

---

### UKG DESIGN 2 — Employee Scheduling / Shift Management (🔥 #2 likely)

**0. Clarify:** Who builds schedules (manager) vs views (employee)? Constraints (max hours, skills/certifications, labor laws, availability, min rest between shifts)? Shift swapping/open-shift bidding? Notifications? Scale?

**1. Requirements**
- *Functional:* manager creates/publishes shifts, assign employees respecting constraints, employees view + request swaps + bid on open shifts, conflict detection, notify on changes.
- *Non-functional:* constraint validation is the hard part; read-heavy (employees check schedules constantly); changes must notify reliably; tenant-isolated.

**2. HLD**
```
[Manager] ─create/publish─▶┌──────────────────┐     ┌─────────────────────┐
                           │ Scheduling Service│────▶│ Constraint Engine   │
[Employee] ─view/swap/bid─▶│ (stateless)       │◀────│ (rules: max-hours,  │
                           └────────┬──────────┘     │ skills, rest, law)  │
                                    │                 └─────────────────────┘
                    ┌───────────────┼────────────────┐
                    ▼               ▼                 ▼
            ┌──────────────┐ ┌────────────┐  ┌──────────────────┐
            │ Schedule DB  │ │ Redis cache│  │ Kafka: schedule- │
            │ (per tenant) │ │ (hot reads:│  │ changed events   │
            └──────────────┘ │ my-week)   │  └────────┬─────────┘
                             └────────────┘           ▼
                                              ┌──────────────────┐
                                              │ Notification Svc │
                                              │ (push/email/SMS) │
                                              └──────────────────┘
```

- **Constraint engine** is the senior signal: model labor rules as **pluggable rule objects** (Strategy/Chain) — "no more than 40h/week," "8h rest between shifts," "must hold certification X." Adding a rule = new class.
- **Publishing a schedule** emits `ScheduleChanged` → Notification service fans out → employees get push/email. Async so the manager's "publish" is instant.
- **Open-shift bidding / swaps:** treat as a request with **optimistic concurrency** (version on the shift) so two employees can't grab the same open shift — last writer fails and retries.
- **Read path:** "my week" is cached per employee (read-through, invalidate on ScheduleChanged).

🎤 **One-liner:** *"The interesting part isn't CRUD on shifts — it's the constraint engine. I model each labor rule as a pluggable validator in a chain, so legal/skill/rest rules compose and adding one is a new class, not an edit. Publishing fans out asynchronously through Kafka to a notification service, and open-shift grabs use optimistic version checks so two people can't claim the same slot."*

---

### UKG quick-hit: other designs they may throw (1-line attack each)
- **Notification system** (shift change, approval, reminder): producer → Kafka → notification service → per-channel adapters (push/email/SMS) with **user preferences**, **dedup**, **retry with backoff**, template service. (Adapter + Strategy patterns.)
- **Leave / Time-off management:** request → **approval workflow** (state machine) → balance ledger. *Strong* consistency on balance (can't over-spend leave), audit trail. (Full LLD in sheet 13.)
- **Payroll calculation engine:** batch over the immutable timecard log at period close; **idempotent + exactly-once**; pluggable pay rules (overtime, shift differentials) as strategies; reproducible (re-run same period → same result).
- **Multi-tenant rate limiting / noisy-neighbor:** token bucket **per tenant** in Redis so one big tenant can't starve others.
- **Reporting / analytics over workforce data:** CQRS — writes to OLTP, async to a columnar store (Redshift/BigQuery) for manager reports; don't run heavy analytics on the transactional DB.

---

## PART F — TIMED MOCK RUN (rehearse out loud against a clock) ⏱️

> Do this **once tonight, out loud, with a timer.** Prompt: *"Design UKG's employee time & attendance system."* The goal is muscle memory for **pacing** — the #1 reason strong candidates fail is running out of time before the deep dive. Target **~45 min**. Say the 🎤 lines verbatim; fill the rest in your words.

**⏱️ 0:00–0:02 — Frame (don't skip, it's the whole differentiator)**
> *"Since UKG is multi-tenant workforce SaaS, three constraints shape this: tenant isolation, compliance-grade auditable data, and bursty load — everyone clocks in at 9 AM. Let me clarify, estimate, sketch the HLD, then go deep on the write path and consistency."*

**⏱️ 0:02–0:06 — Clarify (ask 4–5, then state your assumptions)**
Punch sources (web/mobile/kiosk)? Offline kiosk support? Real-time manager dashboard? Feeds payroll? Scale? → *"I'll assume web + mobile + offline-capable kiosks, a live dashboard, a payroll feed, ~10M employees across thousands of tenants."*

**⏱️ 0:06–0:10 — Estimate (show the math, land the peak number)**
*"10M employees × ~4 punches/day = 40M/day ≈ 460/s average — but that's misleading. The real number is the **shift-boundary peak: tens of thousands/sec in a few minutes at 9 AM**. That single fact drives the architecture: I cannot write punches straight to a DB."*

**⏱️ 0:10–0:14 — API + data model**
The 4 endpoints (POST /punches with Idempotency-Key, GET timecard, approve, live attendance) + the two tables: append-only `punch_events` and derived `timecard`. Call out `UNIQUE(tenant_id, idempotency_key)`.

**⏱️ 0:14–0:22 — HLD (draw the diagram from Design 1)**
Walk the data flow: gateway validates + stamps tenant → publishes to Kafka (partitioned by tenant) → acks 202 → consumers fan out to Timecard DB (append-only) and Redis dashboard projection → payroll reads the log at close. **Narrate *why* the queue, not just that it's there.**

**⏱️ 0:22–0:38 — Deep dives (this is where you win; spend the most here)**
Hit these four in order, ~4 min each:
1. **Burst handling** — Kafka as durable shock-absorber; ack fast, process at consumer pace; replay safety.
2. **Idempotency / offline** — key = employee + device-ts + type; at-least-once + idempotent consumer = effectively exactly-once; use device time not receive time.
3. **Consistency split** — dashboard eventual, **payroll exactly-once**; say which and why.
4. **Timezone + audit** — UTC storage, local-tz computation, DST gotcha; immutable log, edits = adjustment events.

**⏱️ 0:38–0:43 — Failure modes (the P5 unhappy-path signal)**
Consumer lag/backpressure, Kafka partition outage, duplicate punches, a hot tenant (noisy neighbor → per-tenant rate limit), DB failover + replication lag on reads.

**⏱️ 0:43–0:45 — Close with the trade-off summary**
> *"To summarize the key decisions: a queue absorbs the shift-boundary burst and makes punches durable; idempotency keys make offline replay safe; the punch log is immutable for audit while the timecard is a derived, versioned view; I split consistency — eventual for dashboards, exactly-once for payroll; and I shard by tenant, which is both my scaling axis and isolation boundary. The harder I'd push next is exactly-once into payroll and multi-region failover."*

**Self-scoring checklist (tick after your run):**
- [ ] Framed the domain constraints in the first 2 min
- [ ] Landed the *peak* QPS number, not just the average
- [ ] Reached deep dives by minute ~22 (pacing!)
- [ ] Named the queue-not-direct-write decision *with its reason*
- [ ] Said which data is eventual vs exactly-once
- [ ] Raised concurrency/idempotency + a failure mode *unprompted*
- [ ] Closed with an explicit trade-off recap

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
