# Day 69 — Graphs Advanced: 0-1 BFS on Grids & DFS on Equation Graphs
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Minimum Cost to Make a Valid Path encodes turning directions as edge weights 0 or 1 — a canonical 0-1 BFS on a grid. Evaluate Division maps equations to a weighted directed graph and answers queries with DFS/BFS (product along the path = the quotient). Reachable Nodes in Subdivided Graph uses Dijkstra on the original graph to determine how many sub-nodes between edges can be visited within a step budget.

---

## DSA (2 hours)
### Pattern: 0-1 BFS on directed grid + DFS on equation multiplication graph + Dijkstra with sub-node counting

**Minimum Cost to Make a Valid Path in a Grid (LC 1368):**
Each cell has a direction arrow (→, ←, ↑, ↓). Moving in the arrow's direction costs 0; moving in any other direction costs 1 (you change the arrow). Find the minimum cost to go from (0,0) to (m-1, n-1). 0-1 BFS: follow the arrow (cost 0, `addFirst`) or override it (cost 1, `addLast`).

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
| 1 | Min Cost Valid Path in Grid | 1368 | Medium | 0-1 BFS; follow arrow = cost 0; override = cost 1 | Grid cells as nodes; 4 directions; `addFirst` if direction matches arrow |
| 2 | Evaluate Division | 399 | Medium | Weighted directed graph + DFS/BFS product path | Edge `a→b` weight = val; `b→a` weight = 1/val; DFS from src to dst; multiply weights |
| 3 | Reachable Nodes in Subdivided Graph | 882 | Hard | Dijkstra + sub-node counting per edge | `dist[v]` = min steps to original node v; sub-nodes reachable from each side = `maxMoves - dist[endpoint]` |

---

### Code Skeleton
```java
import java.util.*;

class Solution {

    // Min Cost Valid Path in Grid (LC 1368) — 0-1 BFS
    public static int minCost(int[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        // directions indexed by cell value: 1=right, 2=left, 3=down, 4=up
        int[][] dirMap = {{0,1},{0,-1},{1,0},{-1,0}};
        int[][] dist = new int[rows][cols];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        dist[0][0] = 0;
        Deque<int[]> dq = new ArrayDeque<>();
        dq.addLast(new int[]{0, 0, 0}); // (cost, row, col)
        while (!dq.isEmpty()) {
            int[] cur = dq.pollFirst();
            int cost = cur[0], r = cur[1], c = cur[2];
            if (cost > dist[r][c]) continue;
            for (int d = 0; d < 4; d++) {
                int nr = r + dirMap[d][0], nc = c + dirMap[d][1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                    int moveCost = (grid[r][c] == d + 1) ? 0 : 1;
                    int newCost = cost + moveCost;
                    if (newCost < dist[nr][nc]) {
                        dist[nr][nc] = newCost;
                        if (moveCost == 0) dq.addFirst(new int[]{newCost, nr, nc});
                        else dq.addLast(new int[]{newCost, nr, nc});
                    }
                }
            }
        }
        return dist[rows - 1][cols - 1];
    }

    // Evaluate Division (LC 399)
    public static double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        Map<String, Map<String, Double>> graph = new HashMap<>();
        for (int i = 0; i < equations.size(); i++) {
            String a = equations.get(i).get(0), b = equations.get(i).get(1);
            double val = values[i];
            graph.computeIfAbsent(a, x -> new HashMap<>()).put(b, val);
            graph.computeIfAbsent(b, x -> new HashMap<>()).put(a, 1.0 / val);
        }
        double[] result = new double[queries.size()];
        for (int i = 0; i < queries.size(); i++) {
            String src = queries.get(i).get(0), dst = queries.get(i).get(1);
            result[i] = dfsDivision(src, dst, new HashSet<>(), graph);
        }
        return result;
    }

    private static double dfsDivision(String src, String dst, Set<String> visited, Map<String, Map<String, Double>> graph) {
        if (!graph.containsKey(src) || !graph.containsKey(dst)) return -1.0;
        if (src.equals(dst)) return 1.0;
        visited.add(src);
        for (Map.Entry<String, Double> e : graph.get(src).entrySet()) {
            String nb = e.getKey();
            double weight = e.getValue();
            if (!visited.contains(nb)) {
                double result = dfsDivision(nb, dst, visited, graph);
                if (result != -1.0) return weight * result;
            }
        }
        return -1.0;
    }

    // Reachable Nodes in Subdivided Graph (LC 882)
    public static int reachableNodes(int[][] edges, int maxMoves, int n) {
        List<int[]>[] graph = new ArrayList[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        for (int[] e : edges) {
            graph[e[0]].add(new int[]{e[1], e[2] + 1}); // cost = cnt+1
            graph[e[1]].add(new int[]{e[0], e[2] + 1});
        }
        // Dijkstra from node 0
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[0] = 0;
        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        heap.offer(new int[]{0, 0});
        while (!heap.isEmpty()) {
            int[] top = heap.poll();
            int d = top[0], u = top[1];
            if (d > dist[u]) continue;
            for (int[] nb : graph[u]) {
                int v = nb[0], w = nb[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    heap.offer(new int[]{dist[v], v});
                }
            }
        }
        // Count reachable original nodes
        int reachable = 0;
        for (int v = 0; v < n; v++) {
            if (dist[v] <= maxMoves) reachable++;
        }
        // Count reachable sub-nodes per original edge
        for (int[] e : edges) {
            int u = e[0], v = e[1], cnt = e[2];
            int fromU = (dist[u] <= maxMoves) ? Math.max(0, maxMoves - dist[u]) : 0;
            int fromV = (dist[v] <= maxMoves) ? Math.max(0, maxMoves - dist[v]) : 0;
            reachable += Math.min(cnt, fromU + fromV);
        }
        return reachable;
    }
}
```

---

### STAR Interview Framework

> **0-1 BFS on Directed Grid:** brute-force Dijkstra O(m×n log(m×n)) → this approach O(m×n) time, O(m×n) space

**S:** "Given an m×n grid where each cell has a direction arrow. Moving in the arrow's direction costs 0; overriding it costs 1. Find the minimum cost to travel from (0,0) to (m-1, n-1). Dijkstra works but adds O(log(m×n)) heap overhead to every cell expansion."
**T:** "Need O(m×n) — linear — by recognising that override vs follow is exactly a 0/1 edge weight, enabling deque-based 0-1 BFS."
**A (60% of answer time):**
1. *Classify:* "'Minimum cost grid path where one move type is free and another costs 1' — 0-1 BFS on grid cells signal."
2. *Init:* "dirMap = {{0,1},{0,-1},{1,0},{-1,0}} indexed 0–3; cell values 1–4 map to direction index; dist[0][0] = 0; deque = [(0, 0, 0)]."
3. *Loop/Step:* "For each (cost, r, c): for each direction d: moveCost = (grid[r][c] == d+1) ? 0 : 1; newCost = cost + moveCost; if newCost < dist[nr][nc]: addFirst if moveCost==0, addLast if moveCost==1."
4. *Termination:* "Each cell updated at most twice; deque drains; return dist[rows-1][cols-1]."
5. *Gotcha:* "Direction indexing is 1-based in the grid (values 1–4) but 0-based in the dirMap array — so the match is `grid[r][c] == d + 1`, not `grid[r][c] == d`. Off-by-one here makes every move cost 1, breaking the 0-cost case silently."
**R:** "O(m×n) time, O(m×n) space. 1000×1000 grid: ~10^6 operations vs ~20×10^6 with Dijkstra's heap."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Dijkstra with min-heap | Arbitrary non-negative weights | Correct but O(m×n log(m×n)) — unnecessarily slow for binary weights |
| BFS (no deque) | All moves cost 1 | Moves cost 0 or 1; plain BFS treats both as equal-cost and finds wrong paths |

---

> **Equation Weighted Graph + DFS Product Path:** brute-force O(Q × n!) → this approach O((V+E) × Q) time, O(V+E) space

**S:** "Given k equations like a/b = 2.0 and q queries like a/c = ?, answer each query by chaining known ratios. Brute force tries all combinations of equations — O(Q × k!) infeasible."
**T:** "Need O((V+E) × Q) by modelling equations as a weighted directed graph and DFS-ing from source to destination, multiplying edge weights."
**A (60% of answer time):**
1. *Classify:* "'Answer ratio queries by chaining known ratios' — weighted directed graph signal; edge weight = ratio; DFS from src multiplies along the path."
2. *Init:* "For a/b = val: graph[a][b] = val; graph[b][a] = 1/val. For each query (src, dst): DFS from src with product=1.0."
3. *Loop/Step:* "DFS(src, dst, visited, product): if src==dst return product; for each (nb, w) in graph[src]: if nb not visited: result = DFS(nb, dst, visited, product×w); if result != -1 return result."
4. *Termination:* "Finite graph; visited set prevents cycles; returns -1 if dst unreachable."
5. *Gotcha:* "If src or dst is not in the graph at all (never appeared in any equation), return -1 immediately — checking graph.containsKey() before DFS prevents incorrect 1.0 returns when src==dst but the variable is unknown."
**R:** "O((V+E) × Q) time, O(V+E) space. k=40 equations, q=40 queries: ~3,200 operations vs factorial explosion."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Floyd-Warshall pre-compute all pairs | Many queries, small variable set (n ≤ 50) | For V ≤ 40 and Q ≤ 40, DFS per query is fast enough; Floyd-Warshall adds O(V³) preprocessing |
| Union-Find with weights | Detect connectivity, not compute ratios | Weighted union-find can compute ratios but is more complex to implement correctly under time pressure |

---

> **Dijkstra + Sub-Node Counting:** brute-force O(V! path × sub-node enumeration) → this approach O((V+E) log V) time, O(V+E) space

**S:** "Given a graph where each edge (u, v) has cnt virtual sub-nodes inserted between u and v, count how many original nodes and sub-nodes are reachable from node 0 with at most maxMoves steps. Brute force traverses the full subdivided graph — O(total_subnodes) which can be 10^14."
**T:** "Need O((V+E) log V) by running Dijkstra on the original graph only, then counting reachable sub-nodes per edge analytically."
**A (60% of answer time):**
1. *Classify:* "'Count nodes reachable within a step budget on a graph with virtual intermediate nodes' — Dijkstra on original graph + per-edge sub-node accounting."
2. *Init:* "Edge weight = cnt+1 (to traverse u→sub-nodes→v); Dijkstra from node 0 → dist[v] for all original nodes."
3. *Loop/Step:* "For each original edge (u, v, cnt): from_u = max(0, maxMoves - dist[u]); from_v = max(0, maxMoves - dist[v]); sub-nodes on this edge = min(cnt, from_u + from_v)."
4. *Termination:* "Dijkstra terminates; edge loop is O(E); sum up reachable original nodes + sub-nodes."
5. *Gotcha:* "The cap `min(cnt, from_u + from_v)` is essential — if both endpoints are reachable and from_u + from_v > cnt, you'd count sub-nodes that don't exist. Double-counting is the most common bug here."
**R:** "O((V+E) log V) time, O(V+E) space. Original graph with V=3,000 vs subdivided graph with up to 10^9 virtual nodes — Dijkstra on original is the only tractable approach."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| BFS on full subdivided graph | cnt ≤ 10 (small sub-node counts) | cnt up to 10,000; subdivided graph has up to 10^9 virtual nodes — too large |
| DFS with step budget | Small graphs | DFS doesn't guarantee minimum-distance paths; Dijkstra needed for correct counting |

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
**Full STAR Story — "Equation Graph DFS: Choosing the Right Data Store by Chaining Trade-Off Ratios":**
**S (20%):** "At a growth-stage startup, we needed to choose a primary data store for our user activity service: PostgreSQL vs DynamoDB vs Cassandra. Three team members each advocated for a different option; the decision affected 4 engineering teams and 18 months of roadmap. No one had a structured framework — we were going in circles."
**T:** "As the senior engineer on the system design, I owned the final recommendation. Goal: deliver a documented, defensible decision within one week so engineering could start infrastructure provisioning."
**A (60% — 'I' not 'we'):** "(1) I modelled the decision as a ratio-chain: defined 5 weighted criteria (write throughput, query flexibility, operational complexity, cost at scale, strong-consistency need) with relative importance ratios derived from our product requirements. (2) I scored each database option against each criterion using quantified benchmarks — DynamoDB handled 50K writes/sec at our scale; PostgreSQL handled 8K/sec without sharding. (3) I built a decision matrix and traced the product of ratios from requirements to constraints to technology fit — similar to how Evaluate Division chains known ratios to answer unknown queries. (4) I circulated the matrix to all stakeholders, held a 1-hour review, and incorporated two rebuttals (one on Cassandra's operational cost, one on DynamoDB's query limitations) with updated evidence."
**R (20%):** "Decision reached in 6 days: DynamoDB for user activity (write-heavy, eventual consistency acceptable) + PostgreSQL for billing (strong consistency required, complex queries). The clear rationale reduced re-litigation — the decision held for 2 years. Infrastructure provisioning started on time."
*Works for LP questions on: Are Right, A Lot, Have Backbone Disagree and Commit, Earn Trust.*

---

## Flashcards

| Q | A |
|---|---|
| How does 0-1 BFS handle the direction-cost grid in Min Cost Valid Path? | Build a deque. For each neighbour: cost = 0 if the cell's arrow points that way, else 1. `addFirst` for cost 0 (same distance level), `addLast` for cost 1 (next level). |
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
