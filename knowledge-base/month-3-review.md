# Month 3 Review — Days 61–90
**Phase 1: DSA Mastery | Slots 7, 8, 9**

## Overview
Month 3 covers the three hardest DSA pattern groups and three advanced system design architectures:
- **Slot 7 (Days 61–70):** Graphs Advanced — Dijkstra, Bellman-Ford, Floyd-Warshall, 0-1 BFS, Topological Sort variants | URL Shortener + Key-Value Store
- **Slot 8 (Days 71–80):** Heaps & Tries — Top-K, K-way merge, two-heap median, IPO, Trie variants | Newsfeed / Social Network
- **Slot 9 (Days 81–90):** Greedy & Backtracking — Intervals, Jump Game, subsets, permutations, combinations, N-Queens | Message Queues / Pub/Sub

---

## All Algorithms and Patterns — Master Reference

### Slot 7 — Graphs Advanced

#### Shortest Path Algorithms
| Algorithm | Time | Space | Use when |
|-----------|------|-------|---------|
| Dijkstra (standard) | O((V+E) log V) | O(V) | Non-negative weights; single source |
| Dijkstra (max-prob) | O((V+E) log V) | O(V) | Maximise product/probability; negate log |
| Bellman-Ford K stops | O(K × E) | O(V) | K-constrained shortest path; copy prev each round |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs; dense graph; V ≤ 500 |
| 0-1 BFS | O(V+E) | O(V) | Edge weights ∈ {0, 1}; deque appendleft/append |
| Multi-source Dijkstra | O(n log n) | O(n) | Minimum path through Y-subgraph (3 Dijkstra runs) |

**Bellman-Ford K-stops copy pattern:**
```python
for _ in range(k + 1):
    prev = dist[:]   # snapshot before this round
    for u, v, w in edges:
        if prev[u] + w < dist[v]:
            dist[v] = prev[u] + w
```

**0-1 BFS deque:**
```python
dq = deque([(0, start)])
while dq:
    cost, node = dq.popleft()
    for neighbor, weight in graph[node]:
        new_cost = cost + weight
        if new_cost < dist[neighbor]:
            dist[neighbor] = new_cost
            if weight == 0: dq.appendleft((new_cost, neighbor))
            else: dq.append((new_cost, neighbor))
```

#### Topological Sort Variants
| Variant | Sort key | Use when |
|---------|---------|---------|
| Standard Kahn's | In-degree 0 → queue | Task ordering, cycle detection |
| Leaf-peeling (centroid) | Degree 1 → queue | Minimum height trees |
| Two-dimensional | Row + column leaf-peel | Matrix dependency ordering |
| Unique ordering check | Queue size == 1 at each step | Sequence reconstruction |
| Critical path DP | Longest path in DAG | Parallel courses duration |
| Color value DAG DP | `dp[node][color]` max | Color accumulation along path |

#### URL Shortener Architecture
- **Encoding:** Snowflake ID → Base62 (0-9, a-z, A-Z) → 7-character short code
- **Redirect:** 301 (permanent, client caches) vs 302 (temporary, server logs every redirect)
- **DB schema:** `(short_code PK, long_url, user_id, created_at, expiry)`
- **Read scaling:** Redis cache (LRU) → read replicas → primary DB
- **Analytics:** 302 redirect → Kafka "redirect_events" → Flink → ClickHouse
- **Bloom filter:** prevents DB lookups for non-existent short codes

#### Key-Value Store Architecture
- **Write path:** WAL → MemTable → SSTable (LSM Tree)
- **Consistent hashing:** VNodes per server; add/remove = minimal key movement
- **Replication:** N=3; write to coordinator + N-1 replicas
- **Quorum:** W + R > N for strong consistency; W=1, R=1 for availability
- **Vector clocks:** track causality for conflict detection
- **Anti-entropy:** Merkle trees for identifying diverged replicas
- **Gossip:** O(log N) convergence for membership and state propagation

---

### Slot 8 — Heaps & Tries

#### Heap Patterns
| Pattern | Heap type | Core operation | Use when |
|---------|----------|---------------|---------|
| Top-K frequent | Min-heap size K | Push + pop when > K | K most frequent elements |
| K-way merge | Min-heap one per stream | Pop min; push next | Merge K sorted lists/streams |
| Kth largest | Max-heap (negate) or quickselect | Pop K times or O(n) avg | Single Kth element |
| Two-heap median | Max-heap lo + min-heap hi | Balance sizes; lo holds extra | Running or window median |
| Lazy deletion | Heap + `removed` counter | Clean top before reading | Sliding window eviction |
| Greedy IPO | Min-heap (capital) + max-heap (profit) | Move affordable; pop max profit | Capital-constrained selection |
| Task Scheduler | Counter formula | `(max_freq-1)*(n+1) + count_max` | Minimum intervals with cooldown |
| Furthest Building | Min-heap size L | Pop min (use bricks); push new (use ladder) | Resource-constrained greedy |

**Two-heap MedianFinder:**
```python
def addNum(self, num):
    heapq.heappush(self.lo, -num)
    heapq.heappush(self.hi, -heapq.heappop(self.lo))
    if len(self.hi) > len(self.lo):
        heapq.heappush(self.lo, -heapq.heappop(self.hi))
```

#### Trie Variants
| Variant | Node stores | Build time | Query time | Use when |
|---------|------------|-----------|-----------|---------|
| Standard | `children`, `is_end` | O(n×L) | O(L) | Prefix queries, word search |
| Map Sum | Accumulated `#sum` | O(L) | O(L) | Prefix sum of values |
| Wildcard DFS | `is_end` | O(n×L) | O(26^L) worst | Pattern matching with '.' |
| Grid DFS + Trie | `is_end` (clear on find) | O(n×L) | O(m×n×4^L) | Word Search II |
| Binary XOR | Bit children [0,1] | O(n×30) | O(30) | Max XOR of two numbers |
| Reversed stream | `is_end` | O(n×L) | O(active) | Stream suffix matching |
| Suffix-wrapped | `suffix#word` | O(n×L²) | O(L) | Prefix + suffix search |

#### Newsfeed / Social Network Architecture
- **Fanout model:** Hybrid — write for normal users (<1M followers), read for celebrities (≥1M)
- **Feed store:** Redis sorted set `feed:{user_id}`, score = timestamp, capped at 1000, 7-day TTL
- **Social graph:** Cassandra dual-table (by followee_id + by follower_id)
- **Notifications:** Kafka "user_actions" → dedup (Redis counter + TTL) → FCM/APNS/email
- **Trending:** Flink sliding window → top-K min-heap → Redis sorted set every 5 min
- **Ranked feed:** score = recency × affinity × engagement; A/B test vs chronological baseline
- **5 questions:** fanout model / feed store / ranking / notifications / scaling

---

### Slot 9 — Greedy & Backtracking

#### Greedy Patterns
| Pattern | Sort key | Data structure | Key insight |
|---------|---------|---------------|------------|
| Merge intervals | Start time | List | Merge when `curr.start ≤ last.end` |
| Non-overlapping | End time | None | Count `curr.start < last_end` |
| Activity selection | End time | None | Keep earliest-ending; maximises future |
| Course Schedule III | Deadline | Max-heap | Evict longest when `total > deadline` |
| Jump Game I | — | None | Track `max_reach`; fail if `i > max_reach` |
| Jump Game II | — | None | Window expansion; count crossings |
| Gas Station | — | None | Running tank; reset start on negative |
| Max Profit Jobs | End time | DP + binary search | `dp[i] = max(dp[i-1], p + dp[j])` |
| Partition Labels | Last occurrence | None | `end = max(end, last[c])`; seal at `i == end` |
| Remove K Digits | — | Monotone stack | Pop when `stack[-1] > curr AND k > 0` |
| Candy | — | Two arrays | Left + right pass; max per child |
| Max Events | Start day | Min-heap (end day) | Attend earliest-ending each day |

#### Backtracking Templates
```python
# Subsets — collect at every node
def backtrack(start, path):
    result.append(path[:])
    for i in range(start, n):
        path.append(nums[i]); backtrack(i+1, path); path.pop()

# Combinations — collect at depth k
def backtrack(start, path):
    if len(path) == k: result.append(path[:]); return
    for i in range(start, n - (k-len(path)) + 1):
        path.append(nums[i]); backtrack(i+1, path); path.pop()

# Combination Sum — allow reuse (recurse with i, not i+1)
def backtrack(start, path, rem):
    if rem == 0: result.append(path[:]); return
    for i in range(start, n):
        if candidates[i] > rem: break
        path.append(candidates[i]); backtrack(i, path, rem-candidates[i]); path.pop()

# Permutations — visited array; loop from 0
def backtrack(path):
    if len(path) == n: result.append(path[:]); return
    for i in range(n):
        if visited[i]: continue
        visited[i] = True; path.append(nums[i]); backtrack(path); path.pop(); visited[i] = False

# Permutations II — sort + skip duplicate at same depth
# skip if i > 0 AND nums[i] == nums[i-1] AND NOT visited[i-1]

# N-Queens — row by row; col + diag sets
def backtrack(row):
    if row == n: record(); return
    for col in range(n):
        if col in cols or (row-col) in d1 or (row+col) in d2: continue
        place(row, col); backtrack(row+1); remove(row, col)
```

#### Message Queue / Pub/Sub Architecture
- **Kafka:** consumer groups (queue OR pub/sub), log retention, replay, high throughput, pull model
- **RabbitMQ:** push model, exchanges/bindings routing, request/reply, traditional task queues
- **SQS:** fully managed FIFO, no replay, simpler operational model
- **Delivery guarantees:** at-most-once (pre-commit) → at-least-once (post-commit + idempotency) → exactly-once (Kafka transactions)
- **Outbox Pattern:** write event to DB outbox table in same transaction; worker publishes asynchronously
- **CQRS + Event Sourcing:** commands → event store → projections rebuild read models
- **DLQ:** after N retries → DLQ; preserve original payload + error + timestamps; alert on depth
- **Backpressure:** rate limiting / prefetch limit / circuit breaker / Kafka pull model
- **6 questions:** delivery guarantee / consumers / replay / failure handling / backpressure / scaling

---

## System Design Full Decision Frameworks

### URL Shortener
```
Shorten → Snowflake ID → Base62 encode → store (short_code, long_url, user_id)
Redirect → Redis cache hit? → Yes: 302 → No: DB lookup → cache + 302
Analytics → Kafka "redirect_events" → ClickHouse
Scale → CDN for redirects; read replicas for DB; Bloom filter for non-existent codes
```

### Key-Value Store
```
Single node: WAL → MemTable → SSTable; Bloom filter for reads; leveled compaction
Distributed: consistent hashing + VNodes; N=3 RF; quorum R+W>N
Consistency: vector clocks for conflicts; gossip for membership; Merkle trees for anti-entropy
```

### Newsfeed
```
Post → MySQL shard + S3 → Kafka "new_posts" → fanout workers → Redis sorted sets
Celebrity → ZADD user_posts:{id} (no fanout); merge at read time
Feed read → ZREVRANGE + celebrity merge + batch fetch + rank
Notifications → Kafka → dedup → FCM/APNS/email
```

### Message Queue System
```
Single consumer? → SQS (simple) or RabbitMQ (complex routing)
Multiple consumers? → Kafka consumer groups
Need replay? → Kafka with log retention
Delivery? → at-least-once + idempotent consumer (default)
Failures? → DLQ after N retries; alert on DLQ depth
Scale? → partition count bounds parallelism; HPA on consumer lag
```

---

## Master Flashcard Index — 105 Cards (Days 61–90)

### Week 9 (Days 57–63): Graphs Foundations + Dijkstra
35 cards — standard Dijkstra, Bellman-Ford K-stops, topological sort, URL Shortener basics
→ See `week-09/weekly-review.md`

### Week 10 (Days 64–70): Graphs Advanced + Key-Value Store
35 cards — Floyd-Warshall, 0-1 BFS, DAG DP, bitmask BFS, functional graphs, KV store internals
→ See `week-10/weekly-review.md`

### Week 11 (Days 71–77): Heaps & Tries Foundations + Newsfeed
35 cards — Top-K, two-heap median, IPO, Trie insert/search, Word Search II, wildcard DFS
→ See `week-11/weekly-review.md`

### Week 12 (Days 78–84): Heaps & Tries Advanced + Greedy Start
35 cards — XOR Trie, Skyline, interval greedy, Jump Game, subsets, Kafka fundamentals
→ See `week-12/weekly-review.md`

### Week 13 (Days 85–90): Backtracking Synthesis + MQ Framework
30 cards — permutations, N-Queens, Palindrome Partitioning, Candy, Max Events, full MQ decision framework
→ See `week-13/weekly-review.md`

---

## Self-Assessment Checklist

### Slot 7 — Graphs Advanced
- [ ] Can implement Dijkstra from memory (min-heap, lazy skip)
- [ ] Can implement Bellman-Ford K-stops with prev-copy pattern
- [ ] Can implement Floyd-Warshall O(n³) triple loop
- [ ] Can implement 0-1 BFS with deque (appendleft vs append)
- [ ] Can explain URL Shortener: Base62, 301 vs 302, Bloom filter
- [ ] Can explain KV Store: LSM Tree, consistent hashing, quorum, vector clocks

### Slot 8 — Heaps & Tries
- [ ] Can implement two-heap MedianFinder with invariant maintenance
- [ ] Can implement K-way merge min-heap (with list_idx tie-breaking)
- [ ] Can implement IPO two-heap greedy from memory
- [ ] Can implement Trie insert/search/startsWith from scratch
- [ ] Can implement Word Search II with Trie pruning (clear is_end)
- [ ] Can explain Newsfeed: hybrid fanout, Redis sorted set, celebrity problem, 5 questions

### Slot 9 — Greedy & Backtracking
- [ ] Can implement activity-selection greedy (sort by end, keep earliest-ending)
- [ ] Can implement Course Schedule III max-heap swap
- [ ] Can implement subsets/combinations/permutations backtracking templates
- [ ] Can implement N-Queens with column + diagonal set tracking
- [ ] Can implement Candy two-pass greedy
- [ ] Can explain MQ system design: Kafka vs RabbitMQ, delivery guarantees, DLQ, 6 questions

---

## Problems Needing Drill
| Slot | Problem | Pattern | Retry date |
|------|---------|---------|------------|
| (learner fills in) | | | |

---

## Phase 1 Progress
- Month 1 (Days 1–30): Arrays, Strings, Linked Lists, Trees — ✅ Complete
- Month 2 (Days 31–60): Binary Search, Sliding Window, DP Foundations — ✅ Complete
- Month 3 (Days 61–90): Graphs, Heaps, Tries, Greedy, Backtracking — ✅ Complete
- Month 4 (Days 91–120): DP Advanced — Next
