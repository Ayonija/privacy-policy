# Week 10 Review — Days 64–70
**Phase 1: DSA Mastery | Month 3 | Slot 7 (Days 61–70)**

## Week Span
Days 64–70: Slot 7 close-out (Graphs advanced — Floyd-Warshall, 2D DP, implicit BFS, topological DP, 0-1 BFS, multi-source Dijkstra)
System Design: Key-Value Store (single-node LSM, distributed design, consistency, fault tolerance, compaction, synthesis with URL Shortener)

---

## Patterns This Week

### DSA Patterns

**Floyd-Warshall all-pairs shortest path (Day 64 — LC 1334):**
Triple nested loop; `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])` for all intermediate k; O(n³); use for n ≤ ~200.

**Dijkstra + count DP (Day 64 — LC 1976):**
Maintain `ways[v]` alongside `dist[v]`; on shorter path: set `ways[v] = ways[u]`; on equal path: `ways[v] += ways[u]`; answer modulo 10⁹+7.

**2D Bellman-Ford time constraint (Day 64 — LC 1928):**
State = `(node, time)`; `dp[t][v] = min(dp[t][v], dp[t-w][u] + fee[v])`; answer = `min(dp[t][n-1])` for t in range.

**BFS on implicit state space + bitmask (Day 65 — LC 864):**
State = `(r, c, keys_bitmask)`; total states = m×n×2^k; lock check = `(keys >> lock_idx) & 1`; goal = all bits set.

**Boustrophedon board BFS (Day 65 — LC 909):**
BFS on cell numbers; convert cell# to (r,c) with boustrophedon formula (alternate row direction); resolve snake/ladder before enqueueing.

**Knight BFS with symmetry (Day 65 — LC 1197):**
Use `(abs(x), abs(y))`; bound to `r >= -1, c >= -1`; 8 directions.

**Dijkstra + restricted path count DP (Day 66 — LC 1786):**
Dijkstra from dest; DFS+memoization on dist-ordering DAG from source; count paths where `dist[u] > dist[v]`.

**BFS with edge-type state (Day 66 — LC 1129):**
State = `(node, last_colour)`; init with both colours at source; extend only with opposite colour.

**BFS on route hypergraph (Day 66 — LC 815):**
BFS over routes, not stops; boarding a route unlocks all its stops; `stop→routes` map; BFS level = transfers.

**Kahn's uniqueness check (Day 67 — LC 444):**
Queue must have exactly 1 node at each step; if ever `len(queue) > 1` → not unique topological order.

**Topological DP critical path (Day 67 — LC 2050):**
`time[v] = duration[v] + max(time[prereqs])`; process in Kahn's order; answer = `max(time)`.

**Color value DP on DAG (Day 67 — LC 1857):**
`dp[v][c]` = max count of colour c on any path ending at v; propagate via Kahn's; cycle → `-1` if `processed < n`.

**0-1 BFS on grid (Day 68 — LC 2290):**
Deque; weight = `grid[nr][nc]` (0 or 1); `appendleft` for 0-cost, `append` for 1-cost; O(m×n).

**Binary search + multi-source BFS reachability (Day 68 — LC 2812):**
Precompute safeness via multi-source BFS from thieves; binary search threshold T; check path with cells ≥ T.

**BFS two-distance tracking (Day 68 — LC 2045):**
`dist1[v]` and `dist2[v]`; accept into `dist2` if `dist1[v] < new_d < dist2[v]`; simulate traffic light delay.

**0-1 BFS on directed grid (Day 69 — LC 1368):**
Direction match = cost 0 (`appendleft`), override = cost 1 (`append`); direction indexed 1–4.

**Equation weighted graph DFS (Day 69 — LC 399):**
`a→b` weight val; `b→a` weight 1/val; DFS from src to dst, multiplying weights; -1.0 if unreachable.

**Dijkstra + sub-node counting (Day 69 — LC 882):**
Dijkstra on original graph; `from_u = max(0, maxMoves - dist[u])`; sub-nodes per edge = `min(cnt, from_u + from_v)`.

**Two-DFS closest node (Day 70 — LC 2359):**
Get `dist1[]` and `dist2[]` by walking functional graph edges; minimize `max(dist1[v], dist2[v])`.

**In-degree analysis for minimum starting set (Day 70 — LC 1557):**
Nodes with in-degree 0 = no path leads to them; they must be starting vertices; answer = `[v for v if in_degree[v] == 0]`.

**Multi-source Dijkstra Y-shape subgraph (Day 70 — LC 2203):**
Dijkstra from src1, src2 (forward); Dijkstra from dest (reverse graph); minimize `d1[v] + d2[v] + d3[v]` over all v.

---

## Flashcard Deck — 35 Cards

### Day 64 — Floyd-Warshall + Dijkstra Count DP + 2D Bellman-Ford

**1.** What is Floyd-Warshall and when do you use it?
Triple nested loop over all k (intermediate node): `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`. O(n³). Use for all-pairs shortest path when n ≤ 200.

**2.** How do you count shortest paths with Dijkstra?
Maintain `ways[v]`: on shorter path, `ways[v] = ways[u]`; on equal path, `ways[v] += ways[u]`. Answer modulo 10⁹+7.

**3.** How does 2D Bellman-Ford handle a time budget?
`dp[t][v]` = min cost to reach v at time t. Transition: `dp[t][v] = min(dp[t][v], dp[t-w][u] + fee[v])` for each edge (u,v,w). Answer = `min(dp[t][n-1])`.

**4.** What is the LSM Tree write path?
(1) Write to WAL (append-only, durable). (2) Write to MemTable (sorted in-memory). (3) Flush to L0 SSTable when MemTable full. (4) Background compaction merges SSTables across levels.

**5.** What role does a Bloom filter play in an SSTable-based KV store?
Before reading each SSTable: check its Bloom filter. "Definitely absent" → skip the SSTable disk read (no false negatives). Reduces read amplification for missing keys to O(1) disk reads.

### Day 65 — BFS Implicit State + Bitmask + Snakes and Ladders

**6.** What is the state space for Shortest Path to Get All Keys?
`(row, col, keys_bitmask)`. Total states = m×n×2^k where k ≤ 6. Lock check: `(keys >> lock_idx) & 1`; key collection: `keys |= (1 << key_idx)`.

**7.** What is the boustrophedon conversion from cell number to (row, col)?
`s -= 1; row = s // n; col = s % n; if row % 2 == 1: col = n-1-col; row = n-1-row`. Even rows go left-to-right, odd rows right-to-left; row 0 is at the top.

**8.** How do you exploit symmetry in Minimum Knight Moves?
Use `(abs(x), abs(y))` — by symmetry all four quadrants need the same moves. Bound search to `r >= -1, c >= -1` to avoid unbounded expansion.

**9.** How does consistent hashing reduce resharding?
Keys assigned to nearest clockwise node on a hash ring. Adding a node only re-shards keys between the new node and its predecessor — O(keys/N) moved vs O(keys) in naive `key % N`.

**10.** What is the Quorum condition R + W > N?
With N replicas, if W nodes ack writes and R nodes respond to reads, then R+W > N ensures the read set and write set overlap — at least one node has the latest write.

### Day 66 — Restricted Path Count + Alternating Colors + Bus Routes

**11.** How does restricted path counting use Dijkstra from destination?
Run Dijkstra from dest; restricted path condition: each step from u to v satisfies `dist[u] > dist[v]` (getting closer to dest). Count via DFS+memoization on this dist-induced DAG.

**12.** What is the BFS state for alternating color shortest paths?
`(node, last_colour)`. Initialize with `(0, RED, 0)` and `(0, BLUE, 0)`. Extend only via edges of the OPPOSITE colour. Answer = min steps at each node across both colour states.

**13.** Why does Bus Routes BFS over routes not stops?
All stops on a route are equidistant (free within one bus). BFS level = bus transfers. If you BFS over stops, two stops on the same bus appear as distance 2 when they should be distance 1 (same transfer).

**14.** What is eventual consistency and when is it appropriate?
All replicas converge to the same value eventually (no timing guarantee). Appropriate for: social feeds, shopping carts, user preferences. Not appropriate for: financial balances, inventory.

**15.** How do vector clocks detect concurrent writes?
If neither V1 nor V2 dominates (V1[A] > V2[A] but V1[B] < V2[B]), they are concurrent → conflict. Resolve via LWW (last-writer-wins) or application merge.

### Day 67 — Unique Topo Sort + Critical Path + Color DP

**16.** How do you verify a topological order is unique?
Kahn's BFS: at each step, the in-degree-0 queue must have exactly 1 node. If ever `len(queue) > 1` → multiple valid orders → not uniquely reconstructable.

**17.** Critical path DP formula (Parallel Courses III)?
`finish_time[v] = duration[v] + max(finish_time[prereq] for prereq in prereqs[v])`. Process in topological order. Answer = `max(finish_time)`.

**18.** Color value DP propagation in directed graph?
`dp[v][c] = max(dp[v][c], dp[u][c] + (1 if color[v] == c else 0))` for each predecessor u. Initialize `dp[v][color[v]] = 1`. Return -1 if cycle detected (processed < n).

**19.** What is gossip protocol and why is it fault-tolerant?
Nodes periodically exchange cluster state with random peers; O(log N) rounds to propagate. Fault-tolerant: no single coordinator — any subset of alive nodes can continue gossiping.

**20.** What does Merkle tree anti-entropy accomplish?
Two nodes compare Merkle tree roots; equal → all data matches. Differing roots → traverse to find differing subtrees → sync only those key ranges (not the full dataset).

### Day 68 — 0-1 BFS + Binary Search + Second Minimum

**21.** How does 0-1 BFS achieve O(V+E)?
Deque instead of heap. Weight-0 edges: `appendleft` (same distance level). Weight-1 edges: `append` (next level). Maintains sorted order without O(log n) heap overhead.

**22.** What is the algorithm for Find Safest Path in Grid?
(1) Multi-source BFS from all thieves → per-cell safeness. (2) Binary search on threshold T. (3) BFS/DFS reachability using only cells with safeness ≥ T. Return largest T where path exists.

**23.** How does BFS track the second minimum time?
`dist1[v]` = minimum; `dist2[v]` = second minimum. Accept `t` into `dist2[v]` if `dist1[v] < t < dist2[v]`. Answer = `dist2[n]`.

**24.** What is leveled compaction and its trade-off?
SSTables organized L0→L1→L2...; each level has non-overlapping key ranges. Reduces read amplification. Trade-off: high write amplification (~10–30× — each byte rewritten across compaction levels).

**25.** Why is leveled compaction preferred over size-tiered for read-heavy workloads?
Leveled maintains non-overlapping key ranges per level → each level contributes at most 1 SSTable to any point lookup. Size-tiered can have many SSTables at a level with overlapping ranges → more reads per query.

### Day 69 — 0-1 BFS Grid + Equation DFS + Subdivided Graph

**26.** How does 0-1 BFS handle direction-cost in Min Cost Valid Path?
Direction match = cost 0 → `appendleft`. Direction override = cost 1 → `append`. Direction index: 1=right, 2=left, 3=down, 4=up (matching grid cell values).

**27.** How do you build and query the Evaluate Division graph?
For `a / b = val`: edge `a→b` weight val, edge `b→a` weight 1/val. DFS from src to dst, multiply edge weights. Return -1.0 if src or dst not in graph, or if dst unreachable.

**28.** How do you count reachable sub-nodes in Subdivided Graph?
After Dijkstra: `from_u = max(0, maxMoves - dist[u])` if u reachable. Sub-nodes on edge (u,v,cnt): `min(cnt, from_u + from_v)` — cap at cnt to avoid double-counting.

**29.** When does a URL shortener prefer SQL over LSM-based KV?
SQL: fixed schema, moderate write rate (< 10K/sec), B+Tree index gives sub-ms lookups, transactions prevent duplicate codes. LSM: write-heavy (> 10K/sec), large datasets, high delete/update rate.

**30.** Why Kafka for URL analytics instead of direct DB writes?
115K redirects/sec → hot-row contention on popular URLs in SQL. Kafka decouples write from read path; stream processor aggregates into analytics DB asynchronously.

### Day 70 — Functional Graph + In-degree + Multi-source Dijkstra Synthesis

**31.** How does Find Closest Node handle cycles in a functional graph?
Track visited set per traversal. Follow `edges[node]` until `edges[node] == -1` (no edge) or `node` already visited (cycle). Assign dist for each visited node.

**32.** Why is the minimum starting vertex set = nodes with in-degree 0?
Any node with in-degree > 0 has at least one node that can reach it — not needed in the starting set. Nodes with in-degree 0 have no incoming edges; no other node leads to them — they must be included.

**33.** Three Dijkstra runs for Minimum Weighted Subgraph?
(1) Dijkstra from src1 on original graph → `d1[]`. (2) Dijkstra from src2 on original graph → `d2[]`. (3) Dijkstra from dest on reverse graph → `d3[]`. Meeting node v: `d1[v] + d2[v] + d3[v]`. Minimize over all v.

**34.** URL Shortener full decision framework — 5 key choices?
(1) ID gen: Snowflake. (2) Encoding: Base62 6-char. (3) Storage: MySQL + read replicas. (4) Cache: Redis (99%+ hit rate). (5) Redirect: 302 for analytics, 301 for static.

**35.** KV Store full decision framework — 5 key choices?
(1) Storage engine: LSM Tree (WAL+MemTable+SSTable). (2) Sharding: consistent hashing + VNodes. (3) Replication: N=3, R=2, W=2. (4) Conflict: vector clocks or LWW. (5) Fault recovery: hinted handoff + Merkle tree anti-entropy.

---

## Patterns Mastered This Week
Rate yourself 1–5 after this review.

- [ ] Floyd-Warshall all-pairs template — LC 1334
- [ ] Dijkstra + count DP (modulo) — LC 1976
- [ ] 0-1 BFS deque template — LC 2290, 1368
- [ ] Bitmask state BFS (keys+locks) — LC 864
- [ ] Boustrophedon conversion — LC 909
- [ ] Topological DP critical path — LC 2050
- [ ] Color value DP on DAG — LC 1857
- [ ] Multi-source Dijkstra (3 runs) — LC 2203
- [ ] In-degree analysis — LC 1557
- [ ] Route hypergraph BFS — LC 815

---

## Problems Needing Drill
| Pattern | Problem | Retry date |
|---------|---------|------------|
| (learner fills in) | | |
