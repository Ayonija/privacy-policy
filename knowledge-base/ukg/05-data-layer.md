# 05 — Data Layer (SQL, NoSQL, Indexing, JPA, Caching)

> Includes explicitly-asked **clustered vs non-clustered index**, **what is an index**, **SQL joins/grouping/filters**, plus JPA N+1, SQL-vs-NoSQL, Mongo, and caching/invalidation.

---

## 5.1 What is an index? (asked) + clustered vs non-clustered (asked)
**Plain English:** An index is a **sorted side-structure** (usually a **B-tree**) that lets the DB find rows by a column without scanning the whole table — like a book's index instead of reading every page.

**How:** a B-tree keeps keys sorted with O(log n) lookup; the leaf either *is* the row (clustered) or points to it (non-clustered). Turns a `WHERE email=?` from an O(n) full scan into O(log n).

**Clustered vs non-clustered (know cold):**
- **Clustered index** — defines the **physical order** of the table's rows; the table *is* the B-tree, leaf nodes hold the full rows. **One per table** (data can only be sorted one way). Usually the primary key. Range scans on the clustered key are very fast (rows are contiguous).
- **Non-clustered index** — a **separate** structure: sorted keys + a pointer (the clustered key / row locator) back to the row. **Many per table.** A lookup hits the index then does a second step to fetch the row (**bookmark/key lookup**) — unless the index is **covering** (includes all needed columns, so no trip to the table).

🎤 **Say it like this:** *"An index is a sorted B-tree the database keeps alongside the table so it can find rows in log time instead of scanning every row. The key distinction is clustered versus non-clustered. A clustered index actually orders the table physically — the leaf nodes are the rows — so there's only one per table, and range queries on it are fast because rows are contiguous. A non-clustered index is a separate structure that stores the key plus a pointer back to the row, so you can have many, but a lookup costs an extra hop to fetch the row — unless it's a covering index that already includes every column the query needs, which avoids that hop entirely. The trade-off across the board: indexes speed reads but slow writes and cost storage, because every insert/update has to maintain them."*

**Follow-ups:**
- *Q: When NOT to index?* Low-cardinality columns (e.g. boolean), tiny tables, write-heavy columns — the maintenance cost outweighs read gains.
- *Q: Composite index column order?* Leftmost-prefix rule: an index on `(a,b)` helps `WHERE a` and `WHERE a AND b`, but not `WHERE b` alone. Put the most selective / most-filtered column first.
- *Q: How to know if an index is used?* `EXPLAIN`/execution plan — look for index seek vs scan.

---

## 5.2 SQL joins, grouping, filtering (asked across companies)
- **Joins:** **INNER** (rows matching in both), **LEFT** (all left + matched right, NULLs otherwise), **RIGHT**, **FULL OUTER** (all, matched where possible), **CROSS** (cartesian). **Self-join** = table joined to itself.
- **Filtering:** `WHERE` filters **rows before grouping**; `HAVING` filters **groups after aggregation**. (Common trap — know the difference.)
- **Grouping/aggregation:** `GROUP BY` + `COUNT/SUM/AVG/MIN/MAX`. **Logical order of evaluation:** `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`.
- **Window functions** (senior flourish): `ROW_NUMBER()/RANK()/SUM() OVER (PARTITION BY ... ORDER BY ...)` — aggregate *without* collapsing rows. Great for "top N per group," running totals.

```sql
-- Avg rating + count per product, only products with >10 reviews, top 5
SELECT product_id, AVG(rating) avg_r, COUNT(*) n
FROM reviews
WHERE status = 'APPROVED'          -- row filter BEFORE grouping
GROUP BY product_id
HAVING COUNT(*) > 10               -- group filter AFTER aggregation
ORDER BY avg_r DESC
LIMIT 5;
```

🎤 **WHERE vs HAVING:** *"WHERE filters individual rows before any grouping happens; HAVING filters the groups after aggregation. So 'only approved reviews' is a WHERE, but 'only products with more than ten reviews' is a HAVING, because that condition is on the aggregate. Knowing the logical order — from, where, group by, having, select, order by — is what makes the difference obvious."*

---

## 5.3 SQL vs NoSQL + MongoDB
- **SQL (relational)** — structured schema, **ACID** transactions, joins, strong consistency. Best for structured, related data + integrity (finance, orders). **ACID:** Atomicity (all-or-nothing), Consistency (valid state), Isolation (concurrent txns don't interfere), Durability (committed = persisted).
- **NoSQL** — flexible schema, horizontal scale, types: **document** (Mongo), **key-value** (Redis), **wide-column** (Cassandra), **graph** (Neo4j). Often **BASE** (Basically Available, Soft state, Eventual consistency). Best for huge scale, flexible/evolving schema, denormalized read patterns.
- **Choose by:** access pattern + consistency need + scale. *"Model NoSQL around your queries; model SQL around your data."*

**MongoDB modeling:** **embed** (nest related data in one document — fast reads, atomic per-doc, but bounded size & duplication) vs **reference** (store IDs, join in app — normalized, but multiple reads). **Embed for "contains/read-together," reference for "many/large/shared."** Aggregation pipeline: `$match → $group → $sort → $project` (Mongo's GROUP BY analog).

🎤 *"My default is relational, because most business data is relational and I want ACID and joins — especially in a financial context. I reach for NoSQL when the access pattern is well-known and scale or schema-flexibility dominates: a document store when I read an aggregate together, key-value like Redis for caching, wide-column for massive write throughput. The rule I use: in SQL you model the data and query flexibly; in NoSQL you model around the queries, because you're trading join flexibility for scale."*

---

## 5.4 JPA / Hibernate + the N+1 problem
- **ORM** maps objects↔tables; **JPA** is the spec, **Hibernate** the common implementation. `@Entity`, `@OneToMany`, etc.
- **N+1 problem** — you fetch N parents, then lazily fetch each parent's children one query at a time → **1 + N queries**. The classic ORM performance killer.
- **Fixes:** `JOIN FETCH` / `@EntityGraph` (fetch in one query), batch fetching (`@BatchSize`), or a projection/DTO query. **Lazy vs eager:** default to **LAZY** associations and fetch explicitly when needed (eager-everything causes accidental N+1 and over-fetching).

🎤 **N+1:** *"The N+1 problem is when I load a list of N parents and then the ORM fires one extra query per parent to load its children lazily — so a screen of 100 orders becomes 101 queries and the page crawls. I keep associations lazy by default and fix it where it bites with a JOIN FETCH or an entity graph to pull parents and children in a single query, or a DTO projection when I only need a few fields. The deeper lesson is to make fetching an explicit, query-by-query decision rather than a mapping default."*

---

## 5.5 Caching strategies & invalidation (Redis/Hazelcast)
- **Why:** offload the DB, cut latency for hot/repeated reads.
- **Patterns:** **cache-aside** (app reads cache, on miss loads DB + populates — most common), **read-through/write-through** (cache inline; write-through writes cache+DB together — consistent, slower), **write-behind** (write cache, async DB — fast, risk of loss).
- **Invalidation:** **TTL** (expire, tolerate staleness), **explicit eviction on write** (fresh, more coupling), **versioned keys**. Match to staleness tolerance.
- **Pitfalls:** **stampede** (hot key expires, many misses hit DB → lock/`SETNX`, jittered TTL, refresh-ahead), **hot key**, stale data after writes.
- **Redis vs Hazelcast/Memcached:** Redis = rich data structures, persistence, single-threaded core, ubiquitous; Memcached = simple LRU string cache; **Hazelcast** = in-memory data grid embeddable in the JVM (distributed maps/locks) — good for Java-native distributed caching/compute.

🎤 *"Default pattern is cache-aside: check Redis, on a miss read the database and populate the cache, with a TTL sized to how stale I can tolerate. The hard part is invalidation — on writes I evict or update the key so I don't serve stale data, and I add jitter to TTLs plus a lock on cache-miss for hot keys to prevent a stampede taking out the database when a popular key expires. I pick Redis for its data structures and ubiquity; Hazelcast when I want an in-JVM distributed data grid in a Java stack."*

---

## ⚠️ Pitfalls & seniority signals
- Confusing `WHERE` vs `HAVING`; not knowing SQL logical evaluation order.
- "Just add an index" without the write/storage cost, or indexing low-cardinality columns.
- Not recognizing N+1 / eager-fetch-everything.
- Caching without an invalidation story or stampede protection.
- **Seniority signal:** clustered-vs-non-clustered *with the extra-hop/covering-index nuance*, composite-index leftmost-prefix, ACID vs BASE, "model NoSQL around queries," and cache-stampede mitigation.

---

*Next topic, or drill deeper on this one?*
