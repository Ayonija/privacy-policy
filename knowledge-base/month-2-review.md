# Month 2 Review — Days 31–60
**Phase 1: DSA Mastery | Months 1–3**

## Phase
Phase 1 — DSA Mastery. Month 2 covers Slots 4–6 (Days 31–60): Hashing deep dive, Trees & BST, and Graphs. System Design arc: DB Indexing → Caching → CDN/DNS/HTTP.

---

## Patterns Mastered (Comfort ≥ 4/5)
Rate yourself 1–5 after completing this review. Mark ≥ 4 here once achieved.

- [ ] **HashMap canonical key grouping** — Group Anagrams, streak detection
- [ ] **HashMap prefix sum + count map** — Subarray Sum = K, Continuous Subarray Sum
- [ ] **LRU Cache** — DLL + HashMap, O(1) get/put
- [ ] **Frequency bucket sort** — Top-K Frequent Elements, Max Frequency Stack
- [ ] **LFU Cache** — freq→OrderedDict + min_freq variable
- [ ] **Tree DFS gain propagation** — Binary Tree Maximum Path Sum
- [ ] **BFS level order** — snapshot `level_size = len(queue)` pattern
- [ ] **BST inorder invariant** — Kth smallest, Validate BST, Recover BST
- [ ] **LCA patterns** — BST (O(h) value comparison) vs. General BT (post-order DFS)
- [ ] **Union-Find template** — path compression + union-by-rank, near-O(1)
- [ ] **Grid DFS flood fill** — Number of Islands, sinking pattern
- [ ] **Multi-source BFS** — Rotting Oranges, 01 Matrix
- [ ] **Kahn's topological sort** — Course Schedule II, safe nodes
- [ ] **BFS 2-colouring** — Bipartite check, Possible Bipartition
- [ ] **Boundary DFS exclusion** — Surrounded Regions, Enclaves

---

## Patterns Needing Drill
List patterns you could not solve in 20 min without hints — target these in the week-09 onwards review cycle.

| Pattern | Problem where you struggled | Retry date |
|---------|---------------------------|------------|
| (learner fills in) | | |

---

## DSA Difficulty Ramp — Month 2 Summary
All days in Month 2 (Days 31–90) follow the **Medium + Medium + Hard** format.

| Slot | Days | DSA Focus | Hard problems practised |
|------|------|-----------|------------------------|
| 4 | 31–40 | Hashing, Prefix Sum, HashMap Design | LC 76, 895, 1074, 146, 432, 30, 41, 336, 460, 381 |
| 5 | 41–50 | Trees & BST | LC 124, 297, 99, 662, 968, 987, 1373, 685, 1028, 834 |
| 6 | 51–60 | Graphs | LC 827, 127, 329, 1192, 839, 773, 1203, 1345, 765, 778 |

---

## System Design Topics Mastered
Rate yourself 1–5 after this review.

**Slot 4 — DB Indexing:**
- [ ] B+Tree structure (branching factor 200 → height 4 for 1B rows), search/insert/split
- [ ] Clustered vs. non-clustered, covering index, composite index prefix rule
- [ ] Hash indexes: equality-only, no range queries; InnoDB AHI
- [ ] Index selectivity, query planner decisions, stale statistics (ANALYZE)
- [ ] Inverted index: posting lists, AND query intersection, full-text search
- [ ] Partial indexes, when NOT to index
- [ ] Complete index decision tree (full-text → B+Tree → composite → partial → covering)

**Slot 5 — Caching:**
- [ ] Cache placement hierarchy (browser → CDN → app → Redis → DB)
- [ ] Cache-aside pattern; hit-rate vs. cache-size curve
- [ ] Redis data structures: String, List, Set, Sorted Set, Hash, HyperLogLog
- [ ] Redis vs. Memcached decision table
- [ ] Eviction policies: LRU, LFU, TTL, allkeys-lru, volatile-lru
- [ ] Write strategies: write-through, write-back, write-around
- [ ] Cache invalidation: TTL, event-driven, URL versioning, delete-on-write
- [ ] Consistent hashing + virtual nodes; Redis Cluster (16 384 hash slots)
- [ ] Cache stampede: mutex, PER (probabilistic early expiry), background refresh
- [ ] Redis Sentinel vs Cluster; Redis Pub/Sub vs Streams

**Slot 6 — CDN, DNS, HTTP:**
- [ ] CDN fundamentals: PoP, Anycast, Origin Shield, cache fill flow
- [ ] DNS resolution: Root → TLD → Authoritative; record types (A, CNAME, MX, NS, TXT); TTL
- [ ] HTTPS/TLS 1.3: 3 guarantees, handshake; HSTS; status codes
- [ ] HTTP caching: Cache-Control directives; ETag conditional requests; CDN invalidation strategies
- [ ] HTTP/2: multiplexing, HPACK, server push; TCP HOL blocking limitation
- [ ] HTTP/3/QUIC: per-stream flow control, 0-RTT, built-in TLS
- [ ] DNSSEC, DoH, DoT: what each protects against
- [ ] WebSocket vs SSE vs Long Polling: when to use each
- [ ] Complete CDN+DNS+HTTP request lifecycle (5-step synthesis)

---

## Month Flashcard Master Deck
Link to each weekly review for this month's flashcard decks:
- [Week 05 Review](week-05/weekly-review.md) — 35 cards (Days 29–35, Hashing Slot 4 opening)
- [Week 06 Review](week-06/weekly-review.md) — 35 cards (Days 36–42, Hashing synthesis + Trees opening)
- [Week 07 Review](week-07/weekly-review.md) — 35 cards (Days 43–49, Trees & BST core)
- [Week 08 Review](week-08/weekly-review.md) — 35 cards (Days 50–56, Trees synthesis + Graphs opening)

**Total cards available for Month 2: 140 cards (4 weeks × 35)**

**5 must-know flashcards from each slot:**

*Slot 4 — Hashing:*
- Subarray Sum = K: prefix sum `count_map[prefix - k]` pattern
- LRU Cache: O(1) get/put via DLL + HashMap; move to head on access
- LFU Cache: freq→OrderedDict + min_freq; OrderedDict for LRU tie-break
- All O'one: DLL of frequency nodes; O(1) inc/dec/getMin/getMax
- Contiguous Array: replace 0→-1; prefix sum first-seen map for longest subarray

*Slot 5 — Trees & BST:*
- Max Path Sum: `max(child_gain, 0)` + nonlocal global max
- LCA of Binary Tree: both subtrees non-null → current node is LCA
- Validate BST: DFS passing `(low, high)` bounds
- Rerooting (Sum of Distances): `answer[v] = answer[p] - count[v] + (n - count[v])`
- BST Delete: 3 cases; two-child → inorder successor replaces value

*Slot 6 — Graphs:*
- Union-Find template: `find()` with path compression; `union()` with rank
- Tarjan's bridge: `low[v] > disc[u]` → bridge
- Kahn's cycle detection: `len(result) < V` after BFS → cycle exists
- Multi-source BFS: all sources at level 0 simultaneously
- Cache stampede: stale-while-revalidate or mutex lock or PER

---

## OA / Mock Interview Stats
*(Month 4+ — not applicable for Month 2)*
- Contests attempted: —
- OAs completed: —
- Mock interviews done: —

---

## Next Month Goals — Month 3 (Days 61–90)

**DSA (Slots 7–9):**
1. **Slot 7 (Days 61–70):** Graphs advanced — Dijkstra, Bellman-Ford, Floyd-Warshall, Topological Sort advanced. System Design: URL Shortener, Key-Value Store.
2. **Slot 8 (Days 71–80):** Heaps & Tries — Top-K patterns, Merge K Lists, Prefix tree search. System Design: Newsfeed / Social Network.
3. **Slot 9 (Days 81–90):** Greedy & Backtracking — Interval problems, Subsets, Permutations, N-Queens. System Design: Message Queues, Pub/Sub.

**Specific targets for Month 3:**
1. Dijkstra from scratch — min-heap template with distance array, should be writeable in 8 minutes
2. Trie — insert/search/startsWith from scratch without looking up the template
3. Interval merge, insert, overlap check — three canonical interval problems in one session
4. Backtracking — subsets, combinations, permutations with the same recursive template
5. System design: design a URL shortener end-to-end including DB schema, encoding choice, redirect flow, and scaling strategy

**Study cadence:**
- Continue 3 problems/day (Medium + Medium + Hard), 20 min timed per problem
- Weekly review on Day 63, 70, 77, 84
- Monthly review on Day 90 (Slot 9 synthesis)
- Log all problems unsolved in 20 min to `knowledge-base/revision-log.md`; revisit on weekly review days

---

## Strength / Gap Self-Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| HashMap prefix sum | — | — |
| LRU / LFU cache implementation | — | — |
| Tree DFS (gain propagation) | — | — |
| BST operations (validate, kth, recover, delete) | — | — |
| Union-Find template | — | — |
| Grid DFS flood fill | — | — |
| Kahn's topological sort | — | — |
| Tarjan's bridge finding | — | — |
| BFS on implicit state graphs | — | — |
| Boundary DFS (enclosed regions) | — | — |

*(Learner fills in comfort scores and marks patterns for drill)*
