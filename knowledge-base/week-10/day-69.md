# Day 69 — Graphs Advanced: 0-1 BFS on Grids & DFS on Equation Graphs
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Minimum Cost to Make a Valid Path encodes turning directions as edge weights 0 or 1 — a canonical 0-1 BFS on a grid. Evaluate Division maps equations to a weighted directed graph and answers queries with DFS/BFS (product along the path = the quotient). Reachable Nodes in Subdivided Graph uses Dijkstra on the original graph to determine how many sub-nodes between edges can be visited within a step budget.

---

## DSA (2 hours)
### Pattern: 0-1 BFS on directed grid + DFS on equation multiplication graph + Dijkstra with sub-node counting

**Minimum Cost to Make a Valid Path in a Grid (LC 1368):**
Each cell has a direction arrow (→, ←, ↑, ↓). Moving in the arrow's direction costs 0; moving in any other direction costs 1 (you change the arrow). Find the minimum cost to go from (0,0) to (m-1, n-1). 0-1 BFS: follow the arrow (cost 0, `appendleft`) or override it (cost 1, `append`).

**Evaluate Division (LC 399):**
Given equations like `a / b = 2.0`, answer queries like `a / c = ?`. Build a weighted directed graph: edge `a → b` with weight `val`; edge `b → a` with weight `1/val`. To answer `a / c`: DFS/BFS from a to c, multiplying edge weights along the path. If a or c is not in the graph, return -1.

**Reachable Nodes in Subdivided Graph (LC 882):**
Each original edge `(u, v, cnt)` has `cnt` virtual sub-nodes inserted between u and v. You start at node 0 with `maxMoves` steps. Count total reachable original nodes + reachable sub-nodes.

Algorithm:
1. Run Dijkstra from node 0 on the original graph (edge weight = `cnt + 1`) → `dist[v]` = steps to reach original node v
2. For each original edge `(u, v, cnt)`: count sub-nodes reachable from u-side = `min(cnt, max(0, maxMoves - dist[u]))` and from v-side similarly. Sub-nodes on this edge reachable = `min(cnt, from_u + from_v)` (can't double-count if both ends are reachable).
3. Count reachable original nodes: all v where `dist[v] <= maxMoves`.

**Trigger condition:**
- "minimum cost path in grid where following existing direction is free" → 0-1 BFS on grid cells
- "answer ratio queries from a set of known ratios" → weighted directed graph; DFS/BFS multiplying edge weights
- "count nodes reachable within a budget on a graph with virtual intermediate nodes" → Dijkstra on original graph; count sub-nodes per edge from both sides

**Time complexity:** LC 1368: O(m×n) | LC 399: O((V+E) × Q) | LC 882: O((V+E) log V)
**Space complexity:** O(m×n) / O(V+E) / O(V+E)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Min Cost Valid Path in Grid | 1368 | Medium | 0-1 BFS; follow arrow = cost 0; override = cost 1 | Grid cells as nodes; 4 directions; `appendleft` if direction matches arrow |
| 2 | Evaluate Division | 399 | Medium | Weighted directed graph + DFS/BFS product path | Edge `a→b` weight = val; `b→a` weight = 1/val; DFS from src to dst; multiply weights |
| 3 | Reachable Nodes in Subdivided Graph | 882 | Hard | Dijkstra + sub-node counting per edge | `dist[v]` = min steps to original node v; sub-nodes reachable from each side = `maxMoves - dist[endpoint]` |

---

### Code Skeleton
```python
from collections import deque, defaultdict
import heapq

# Min Cost Valid Path in Grid (LC 1368) — 0-1 BFS
def minCost(grid):
    rows, cols = len(grid), len(grid[0])
    # directions indexed by cell value: 1=right, 2=left, 3=down, 4=up
    dir_map = {1: (0,1), 2: (0,-1), 3: (1,0), 4: (-1,0)}
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[0][0] = 0
    dq = deque([(0, 0, 0)])   # (cost, row, col)
    while dq:
        cost, r, c = dq.popleft()
        if cost > dist[r][c]: continue
        for d, (dr, dc) in enumerate(dirs, 1):   # d = direction number
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols:
                move_cost = 0 if grid[r][c] == d else 1
                new_cost = cost + move_cost
                if new_cost < dist[nr][nc]:
                    dist[nr][nc] = new_cost
                    if move_cost == 0:
                        dq.appendleft((new_cost, nr, nc))
                    else:
                        dq.append((new_cost, nr, nc))
    return dist[rows-1][cols-1]

# Evaluate Division (LC 399)
def calcEquation(equations, values, queries):
    graph = defaultdict(dict)
    for (a, b), val in zip(equations, values):
        graph[a][b] = val
        graph[b][a] = 1.0 / val

    def dfs(src, dst, visited):
        if src not in graph or dst not in graph: return -1.0
        if src == dst: return 1.0
        visited.add(src)
        for nb, weight in graph[src].items():
            if nb not in visited:
                result = dfs(nb, dst, visited)
                if result != -1.0:
                    return weight * result
        return -1.0

    return [dfs(src, dst, set()) for src, dst in queries]

# Reachable Nodes in Subdivided Graph (LC 882)
def reachableNodes(edges, maxMoves, n):
    graph = defaultdict(list)
    for u, v, cnt in edges:
        graph[u].append((v, cnt + 1))   # cost = cnt+1 to cross from u to v
        graph[v].append((u, cnt + 1))
    # Dijkstra from node 0
    dist = [float('inf')] * n
    dist[0] = 0
    heap = [(0, 0)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]: continue
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))
    # Count reachable original nodes
    reachable = sum(1 for v in range(n) if dist[v] <= maxMoves)
    # Count reachable sub-nodes per original edge
    edge_map = {(min(u,v), max(u,v)): cnt for u, v, cnt in edges}
    for u, v, cnt in edges:
        from_u = max(0, maxMoves - dist[u]) if dist[u] <= maxMoves else 0
        from_v = max(0, maxMoves - dist[v]) if dist[v] <= maxMoves else 0
        reachable += min(cnt, from_u + from_v)
    return reachable
```

---

### Edge Cases to Trace Before Coding
- LC 1368: 1×1 grid → return 0; all arrows already point toward destination → cost = 0
- LC 399: query where src == dst → return 1.0; query where src is not in any equation → return -1.0; division by itself (a/a) → return 1.0 if a is in graph, else -1.0
- LC 882: maxMoves = 0 → only node 0 is reachable; an edge with cnt = 0 → no sub-nodes; node v unreachable → contributes 0 sub-nodes from its side

---

### Interview Pattern Drill

| Problem type | Graph construction | Traversal | What to accumulate |
|-------------|-------------------|-----------|-------------------|
| Grid path with direction costs | Grid cells as nodes; cost = 0 or 1 per direction | 0-1 BFS | Min cost to reach corner |
| Ratio/equation chain | `a→b` weight val; `b→a` weight 1/val | DFS from src to dst | Product of edge weights |
| Subdivided graph budget | Original graph (edge weight = cnt+1) | Dijkstra from source | Original nodes + sub-nodes within budget |

---

## System Design (1 hour)
### Topic: Key-Value Store vs URL Shortener — Architectural Comparison

**Comparison overview:**

| Dimension | URL Shortener | Key-Value Store |
|-----------|--------------|----------------|
| Data model | `{short_code → long_url}` | `{arbitrary_key → arbitrary_value}` |
| Access pattern | Point lookups only (no scans) | Point lookups + range scans (some) |
| Write behaviour | Append-mostly (new codes); rare deletes | Arbitrary updates, deletes, range updates |
| Read behaviour | High read-to-write ratio (100:1) | Depends on use case |
| Consistency need | Eventual OK (a 302 ms stale redirect is fine) | Configurable (eventual to strong) |
| Scalability unit | Shard by `hash(short_code)`; read replicas | Consistent hashing ring; VNodes |
| Storage layer | SQL (B+Tree index on short_code) | LSM Tree (append-optimised for high writes) |
| Caching | Redis cache-aside (`short_code → url`) | Application-level; depends on use case |
| Analytics | Async via Kafka + Flink | Not primary use case |

**When to use SQL vs LSM Tree:**
- SQL (B+Tree): OLTP workloads with random reads and writes, strong consistency, complex queries, joins. Read performance with indexes is excellent; write performance degrades at high update rates (B+Tree rewrites pages on insert/update).
- LSM Tree: write-heavy workloads (logging, time-series, event sourcing), large data sets, append-mostly. Write performance is excellent (sequential WAL + MemTable); read performance requires Bloom filters + compaction to remain manageable.

**URL shortener choosing SQL:**
- Schema is fixed and simple
- Short codes are unique keys → B+Tree index is perfect
- Read replicas handle the 100:1 read/write ratio
- SQL transactions ensure no duplicate short codes under concurrent inserts
- `SELECT long_url FROM urls WHERE short_code = ?` is a single index lookup — sub-millisecond

**When a URL shortener would need LSM:**
- If short codes expire and are recycled at high rate (many deletes) → LSM's tombstone model is cleaner
- If tracking every click at the DB level (100K+ writes/sec) → LSM's write throughput is needed. But this is better solved with Kafka + a dedicated analytics store (ClickHouse)

**Interview talking point:** "The URL shortener's write path is simple: INSERT one row per new URL. The write rate (1,160/sec) is well within MySQL's range. The read path (115,000 redirects/sec) needs caching — not a different DB. A KV store's power is for write-heavy workloads like IoT sensor data, event logs, or time-series — not for a system where reads dominate and the write rate is modest."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1368 Min Cost Valid Path in Grid (target: 18 min)
- **Medium 2:** LC 399 Evaluate Division (target: 18 min)
- **Hard:** LC 882 Reachable Nodes in Subdivided Graph (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you needed to choose between two technology options (e.g., database, framework, service) — how did you make the trade-off decision?
- Leadership principle: Are Right, A Lot

---

## Flashcards

| Q | A |
|---|---|
| How does 0-1 BFS handle the direction-cost grid in Min Cost Valid Path? | Build a deque. For each neighbour: cost = 0 if the cell's arrow points that way, else 1. `appendleft` for cost 0 (same distance level), `append` for cost 1 (next level). |
| How do you build the weighted graph for Evaluate Division? | For `a / b = val`: add edge `a → b` with weight `val` and edge `b → a` with weight `1/val`. DFS from `src` to `dst`, multiplying edge weights; return -1.0 if unreachable. |
| In Reachable Nodes in Subdivided Graph, how do you count sub-nodes on an edge? | After Dijkstra: `from_u = max(0, maxMoves - dist[u])` if u is reachable; `from_v = max(0, maxMoves - dist[v])` if v is reachable. Sub-nodes on this edge = `min(cnt, from_u + from_v)` (cap at cnt to avoid double-counting). |
| When does a URL shortener prefer SQL over an LSM-based KV store? | SQL is preferable when: (1) schema is fixed and simple, (2) writes are moderate (< 10K/sec), (3) reads need strong consistency, (4) B+Tree index on short_code gives sub-millisecond lookups. LSM is for write-heavy workloads where random-write B+Tree performance degrades. |
| Why is Kafka used for URL analytics instead of writing clicks to the main DB? | Direct click-count updates at 115K/sec would cause hot-row contention on popular URLs in SQL. Kafka decouples the write from the read path; stream processors aggregate clicks into analytics tables asynchronously. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/18/25 min, no hints)
- [ ] Rewrote 0-1 BFS grid template and Evaluate Division DFS from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can articulate URL Shortener vs KV Store architectural differences cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
