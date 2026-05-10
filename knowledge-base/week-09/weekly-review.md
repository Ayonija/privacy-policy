# Week 09 Review — Days 57–63
**Phase 1: DSA Mastery | Month 3 opening / Month 2 close**

## Week Span
Days 57–60: Slot 6 close-out (Graphs synthesis — all-paths DFS, BFS with state grouping, boundary DFS, Dijkstra on grid)
Days 61–63: Slot 7 opening (Graphs advanced — Dijkstra standard, grid Dijkstra, topological sort variants)

System Design: Days 57–60: CDN Anycast, Origin Shield, DNS security, WebSocket/SSE/Long Polling. Days 61–63: URL Shortener (requirements, DB schema, scaling).

---

## Patterns This Week

### DSA Patterns

**All Paths DFS backtracking (Day 57 — LC 797):**
DFS from source with path list; on reaching target, append `list(path)` (copy, not reference); backtrack with `pop()` after returning from each child.

**Reverse graph + Kahn's safe-node detection (Day 57 — LC 802):**
Build reverse graph; in-degree in reverse = out-degree in original; Kahn's from in-degree-0 nodes; processed nodes = safe in original graph.

**Two-level topological sort (Day 57 — LC 1203):**
Item graph + group graph (derived from cross-group item dependencies); Kahn's on both; interleave items within each group in group-topological order.

**BFS with bank-validated substitutions (Day 58 — LC 433):**
Try each position × 4 chars; valid if in bank set; BFS levels = mutation count.

**Clone Graph with HashMap (Day 58 — LC 133):**
Register clone in `{original: clone}` map BEFORE recursing into neighbours — handles cycles by returning existing clone on re-visit.

**BFS + value-grouped index map with clear (Day 58 — LC 1345):**
Group indices by value; when visiting, process all same-value indices in one step, then `val_to_idx[arr[i]].clear()` — prevents O(n²) re-processing.

**Boundary DFS exclusion (Day 59 — LC 130, 1020):**
DFS from ALL border cells to mark reachable; remaining unmarked cells = enclosed (flip or count them).

**Union-Find greedy component analysis (Day 59 — LC 765):**
Union couple IDs for each adjacent seat pair; min swaps = total_pairs − number_of_components.

**BFS maze exit + DFS reachability (Day 60 — LC 1926, 841):**
BFS shortest path to boundary non-entrance; DFS/BFS from room 0 collecting keys; compare visited count to total rooms.

**Dijkstra on weighted grid (Day 60 — LC 778):**
Heap = `(max_cell_value_on_path, r, c)`; pop minimum; answer when `(n-1, n-1)` first dequeued.

**Standard Dijkstra (Day 61 — LC 743):**
Min-heap `(dist, node)`; skip stale entries `d > dist[u]`; answer = `max(dist.values())`.

**Max-probability Dijkstra (Day 61 — LC 1514):**
Max-heap (negate for Python); multiply probabilities; update if `cur_prob * edge_prob > prob[v]`.

**Bellman-Ford K-stops (Day 61 — LC 787):**
K+1 relaxation rounds; copy `prev = dist[:]` before each round to prevent chaining.

**Grid Dijkstra min-bottleneck (Day 62 — LC 1631):**
`max(cur_effort, abs(h1-h2)) < effort[nr][nc]` → update and push.

**Union-Find min-edge in component (Day 62 — LC 2492):**
Find 1's component root; scan all edges in that component for minimum weight.

**Min-heap boundary BFS 3D trapping (Day 62 — LC 407):**
Push all boundary cells; pop lowest; water at neighbour = `max(0, level - height[nr][nc])`; push neighbour with `max(level, neighbour_height)`.

**Leaf-peeling topological sort (Day 63 — LC 310):**
Initialize with all degree-1 nodes; remove leaves in rounds; stop when `remaining ≤ 2`; return remaining nodes.

**Kahn's with ingredient resolution (Day 63 — LC 2115):**
Available supplies start; recipes unlock when all ingredients satisfied; result = recipes that complete.

**Two-dimension topological sort (Day 63 — LC 2392):**
Independent Kahn's for rows and columns; `row_pos[v]` and `col_pos[v]` → `matrix[row_pos[v]][col_pos[v]] = v`.

---

## Flashcard Deck — 35 Cards

### Day 57 — All Paths DFS + Safe Nodes + Two-Level Topo Sort

**1.** How does All Paths DFS avoid reference bugs?
`result.append(list(path))` — `list(path)` creates a shallow copy; without it, all entries point to the same mutable list.

**2.** How does Find Eventual Safe States use a reverse graph?
Reverse all edges; Kahn's BFS from in-degree-0 nodes (= original terminals); dequeued nodes = safe in original.

**3.** What are the two graphs in Sort Items by Groups topological sort?
Item-level graph (item→item deps) + group-level graph (group→group, derived from cross-group item deps); both must be cycle-free.

**4.** What is Anycast routing?
Multiple PoPs share one IP; BGP routes each packet to the nearest PoP at the network layer — no DNS TTL delay, instant failover on PoP failure.

**5.** What problem does Origin Shield solve?
Without Shield: cache misses at all edge PoPs hit origin simultaneously (thundering herd). With Shield: misses go to 1–3 Shield PoPs; O(1) origin requests for popular content.

### Day 58 — BFS State Grouping + Clone Graph + Jump Game IV

**6.** How does Minimum Genetic Mutation generate BFS neighbours?
For each of 8 positions, try `{A, C, G, T}`; add candidate if in bank set and not yet visited.

**7.** How does Clone Graph handle cycles without infinite recursion?
Register clone in `{original: clone}` map BEFORE recursing neighbours — re-visit returns the existing clone immediately.

**8.** What breaks if you don't clear `val_to_idx` in Jump Game IV?
Re-enqueuing visited same-value indices causes O(n²) redundant work; clearing after first visit ensures each group is processed at most once.

**9.** What does DNSSEC protect against and what does it NOT protect?
Prevents cache poisoning (forged records fail signature check); does NOT encrypt queries — eavesdropping still possible.

**10.** DoH vs DoT: key difference?
Both encrypt DNS queries. DoH wraps in HTTPS on port 443 (hard to block). DoT uses TLS on port 853 (distinguishable, can be firewall-blocked).

### Day 59 — Boundary DFS + Union-Find Greedy

**11.** How does Surrounded Regions avoid incorrectly capturing safe 'O' cells?
DFS from all border 'O' cells, marking connected 'O' as safe ('S'); only remaining unvisited 'O' (truly surrounded) are flipped to 'X'.

**12.** How does Number of Enclaves differ from Number of Islands?
Enclaves = land cells that CANNOT reach the boundary; DFS from all boundary land cells first; remaining unvisited land cells = enclaves.

**13.** How does Union-Find derive minimum swaps in Couples Holding Hands?
Union couple IDs of each adjacent seat pair; a component of k couples needs k-1 swaps; total = total_pairs − number_of_components.

**14.** WebSocket vs SSE: when to use each?
WebSocket: bidirectional (chat, gaming, collaborative edit). SSE: server-to-client only (live scores, notifications, dashboards) — HTTP-native, auto-reconnect, simpler.

**15.** How do you horizontally scale WebSocket servers?
Redis Pub/Sub or Kafka: any server receiving a message publishes to broker; all servers subscribe and push to local WebSocket connections for the target user.

### Day 60 — Graph Synthesis + CDN/DNS/HTTP Decision Framework

**16.** How does Nearest Exit BFS distinguish entrance from a valid exit?
Mark entrance visited immediately; any other boundary empty cell encountered during BFS is a valid exit — return current distance + 1.

**17.** How does Keys and Rooms determine all rooms are reachable?
DFS/BFS from room 0, collecting keys; at end, `len(visited) == len(rooms)` — unvisited rooms = unreachable → return False.

**18.** How does Swim in Rising Water use a min-heap differently from BFS?
Heap stores `(max_cell_value_on_path, r, c)`; answer = heap value when `(n-1, n-1)` is first popped — Dijkstra for minimum bottleneck.

**19.** State the 5-question CDN/DNS/HTTP decision checklist.
(1) Where is content cached? (2) How fresh must it be? (3) Is data user-specific? (4) HTTP/1.1 vs 2 vs 3? (5) What real-time pattern (HTTP/SSE/WebSocket)?

**20.** When should you use Origin Shield vs just edge caching?
Flash crowds / viral content: Origin Shield prevents a thundering herd of edge PoPs hitting origin simultaneously; use Shield when popular content could cause origin overload.

### Day 61 — Standard Dijkstra + Max-Prob Dijkstra + Bellman-Ford K-stops

**21.** How do you skip stale entries in Dijkstra?
On pop `(d, u)`: `if d > dist[u]: continue` — a shorter path was already processed; this entry is stale.

**22.** How does Path with Max Probability adapt Dijkstra?
Max-heap (negate for Python); multiply probabilities instead of summing; update if `p * edge_p > prob[v]`.

**23.** Why does Dijkstra fail for Cheapest Flights K Stops?
Dijkstra marks a node settled on first pop; a path with more hops but lower cost may be pruned before exhausting K stops.

**24.** How does Bellman-Ford enforce K stops?
Copy `prev = dist[:]` before each of K+1 rounds; relaxations use `prev[u]` to prevent chaining multiple hops in one round.

**25.** URL shortener: 301 vs 302 redirect?
302: browser always contacts server → analytics captured. 301: browser caches redirect → no analytics after first visit. Use 302 if click analytics matter.

### Day 62 — Grid Dijkstra + Union-Find Min Edge + 3D Water Trapping

**26.** How does Path With Minimum Effort differ from Network Delay Time in Dijkstra update?
Network Delay: `dist[u] + w < dist[v]` (sum). Min Effort: `max(cur_effort, abs(h1-h2)) < effort[v]` (minimax).

**27.** When to use Union-Find instead of Dijkstra for a path problem?
When question asks about ANY path (not minimum-cost path) — find component of source; scan all edges in it for min/max edge weight.

**28.** What is the invariant in Trapping Rain Water II heap approach?
Always process the lowest boundary cell first; water level at interior cell = max(enclosing boundary height, cell's own height).

**29.** URL shortener write path — 5 steps?
(1) Receive long URL (2) Generate Snowflake/counter ID (3) Base62-encode → short_code (4) INSERT to DB (5) Write to Redis cache.

**30.** Why use Bloom filter in a URL shortener?
Checks if short_code definitely NOT in DB in O(1); invalid codes (bots, typos) skip DB entirely — zero false negatives.

### Day 63 — Leaf Peeling + Kahn's Recipes + Two-Dimension Topo Sort

**31.** How does leaf-peeling find a tree's centroid(s)?
Repeatedly remove all degree-1 nodes in rounds; decrement neighbour degrees; add newly-created leaves; stop when ≤ 2 nodes remain.

**32.** How does Find All Possible Recipes initialise differently from standard Kahn's?
Available supplies start with in-degree 0 (already available); recipes start with in-degree = number of ingredients. Processing supplies decrements recipe in-degrees.

**33.** What does an empty return from topo_sort mean in Build Matrix?
A cycle in the row or column conditions was detected — impossible to satisfy all ordering constraints; return `[]`.

**34.** Why does a URL shortener use Kafka for click analytics?
115,000 redirects/sec writing directly to DB creates hot-row contention on popular URLs; Kafka buffers events; stream processor aggregates before writing to analytics DB.

**35.** URL shortener sharding strategy?
Shard by hash(short_code) % n_shards; Snowflake ID generator encodes shard_id in bits 10–21; read replicas per shard.

---

## Patterns Mastered This Week
Rate yourself 1–5 after this review.

- [ ] Dijkstra template (skip stale, min-heap) — LC 743
- [ ] Max-probability Dijkstra (negate, multiply) — LC 1514
- [ ] Bellman-Ford K-stops (copy prev array) — LC 787
- [ ] Grid Dijkstra minimax (max effort, min-heap) — LC 1631
- [ ] 3D water trapping (boundary min-heap BFS) — LC 407
- [ ] Leaf-peeling topological sort (centroid finding) — LC 310
- [ ] Two-dimension topological sort — LC 2392
- [ ] All-paths DFS backtracking — LC 797
- [ ] Reverse graph + Kahn's safe nodes — LC 802
- [ ] BFS value-grouping with bucket clear — LC 1345

---

## Problems Needing Drill
| Pattern | Problem | Retry date |
|---------|---------|------------|
| (learner fills in) | | |
