# Day 64 — Graphs Advanced: Floyd-Warshall & DP on Dijkstra State
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Find the City uses Floyd-Warshall (all-pairs shortest path) to count reachable neighbours within a threshold. Number of Ways to Arrive combines Dijkstra with a DP count array to count shortest paths. Min Cost to Reach Destination in Time adds a time dimension to Bellman-Ford — effectively Bellman-Ford where each node state is `(node, time_spent)`.

---

## DSA (2 hours)
### Pattern: Floyd-Warshall (all-pairs) + Dijkstra + Count DP + Bellman-Ford with time dimension

**Find the City With the Smallest Number of Neighbors (LC 1334):**
For each city, count how many other cities can be reached within `distanceThreshold`. Return the city with the fewest reachable neighbours; break ties by choosing the largest city index. Run Floyd-Warshall (O(n³)) to compute all-pairs shortest paths, then count reachable neighbours per city.

Floyd-Warshall template:
```
for k in range(n):           # intermediate node
    for i in range(n):
        for j in range(n):
            if dist[i][k] + dist[k][j] < dist[i][j]:
                dist[i][j] = dist[i][k] + dist[k][j]
```
Initialize `dist[i][i] = 0`, `dist[i][j] = edge weight` if edge exists, else `inf`.

**Number of Ways to Arrive at Destination (LC 1976):**
Standard Dijkstra for shortest path, but also maintain `ways[v]` = number of ways to reach v in the shortest distance. When relaxing edge `(u, v, w)`:
- If `dist[u] + w < dist[v]`: update `dist[v]`, set `ways[v] = ways[u]`
- If `dist[u] + w == dist[v]`: add `ways[v] += ways[u]` (another shortest path found)

**Min Cost to Reach Destination in Time (LC 1928):**
Graph with edge travel times and per-city fees. Reach city `n-1` within `maxTime`. State: `(current_city, time_used)`. This is a 2D Dijkstra: `dist[city][time] = minimum fee`. Or use Bellman-Ford: `dp[t][v]` = min cost to reach v in exactly t time units. Relax up to `maxTime` times.

**Trigger condition:**
- "how many cities can each city reach within a limit?" → Floyd-Warshall; then count per city
- "count the number of shortest paths from source to destination" → Dijkstra + `ways[]` array; update `ways` on equal-distance relaxation
- "minimum cost path with a time/hop budget" → 2D DP/Bellman-Ford with `(node, time_spent)` state

**Time complexity:** LC 1334: O(n³) | LC 1976: O((V+E) log V) | LC 1928: O(maxTime × E)
**Space complexity:** O(n²) / O(V+E) / O(maxTime × V)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Find City Smallest Neighbors | 1334 | Medium | Floyd-Warshall all-pairs + threshold count | O(n³) fine for n ≤ 100; count reachable per city; tie-break = larger city index |
| 2 | Number of Ways to Arrive | 1976 | Medium | Dijkstra + count DP on equal-distance relaxation | `ways[v] = ways[u]` on shorter; `ways[v] += ways[u]` on equal; answer modulo 10⁹+7 |
| 3 | Min Cost to Reach Dest in Time | 1928 | Hard | 2D Bellman-Ford `dp[t][v]` = min fee | State = (city, time); relax up to maxTime; answer = min over all times at dest |

---

### Code Skeleton
```java
import java.util.*;

class Solution {

    // Find City With Smallest Neighbors (LC 1334)
    public static int findTheCity(int n, int[][] edges, int distanceThreshold) {
        int[][] dist = new int[n][n];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        for (int i = 0; i < n; i++) dist[i][i] = 0;
        for (int[] e : edges) {
            dist[e[0]][e[1]] = e[2];
            dist[e[1]][e[0]] = e[2];
        }
        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    if (dist[i][k] != Integer.MAX_VALUE && dist[k][j] != Integer.MAX_VALUE
                            && dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }
        int ans = -1;
        int minCount = Integer.MAX_VALUE;
        for (int i = 0; i < n; i++) {
            int count = 0;
            for (int j = 0; j < n; j++) {
                if (i != j && dist[i][j] <= distanceThreshold) count++;
            }
            if (count <= minCount) { // <= keeps the largest index on ties
                minCount = count;
                ans = i;
            }
        }
        return ans;
    }

    // Number of Ways to Arrive (LC 1976)
    public static int countPaths(int n, int[][] roads) {
        int MOD = 1_000_000_007;
        List<int[]>[] graph = new ArrayList[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        for (int[] r : roads) {
            graph[r[0]].add(new int[]{r[1], r[2]});
            graph[r[1]].add(new int[]{r[0], r[2]});
        }
        long[] dist = new long[n];
        long[] ways = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[0] = 0;
        ways[0] = 1;
        PriorityQueue<long[]> heap = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        heap.offer(new long[]{0, 0}); // (dist, node)
        while (!heap.isEmpty()) {
            long[] top = heap.poll();
            long d = top[0];
            int u = (int) top[1];
            if (d > dist[u]) continue;
            for (int[] nb : graph[u]) {
                int v = nb[0], w = nb[1];
                long newD = dist[u] + w;
                if (newD < dist[v]) {
                    dist[v] = newD;
                    ways[v] = ways[u];
                    heap.offer(new long[]{newD, v});
                } else if (newD == dist[v]) {
                    ways[v] = (ways[v] + ways[u]) % MOD;
                }
            }
        }
        return (int) ways[n - 1];
    }

    // Min Cost to Reach Destination in Time (LC 1928) — 2D Bellman-Ford
    public static int minCost(int maxTime, int[][] edges, int[] passingFees) {
        int n = passingFees.length;
        // dp[t][v] = minimum fee to reach city v using exactly t time
        int[][] dp = new int[maxTime + 1][n];
        for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE);
        dp[0][0] = passingFees[0];
        for (int t = 1; t <= maxTime; t++) {
            for (int[] e : edges) {
                int u = e[0], v = e[1], w = e[2];
                if (t >= w) {
                    if (dp[t - w][u] != Integer.MAX_VALUE) {
                        dp[t][v] = Math.min(dp[t][v], dp[t - w][u] + passingFees[v]);
                    }
                    if (dp[t - w][v] != Integer.MAX_VALUE) {
                        dp[t][u] = Math.min(dp[t][u], dp[t - w][v] + passingFees[u]);
                    }
                }
            }
        }
        int ans = Integer.MAX_VALUE;
        for (int t = 0; t <= maxTime; t++) {
            ans = Math.min(ans, dp[t][n - 1]);
        }
        return ans == Integer.MAX_VALUE ? -1 : ans;
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 1334: n=2 → each city can reach 1 other; tie-break returns city 1 (larger index); isolated node (no edges) → reachable count = 0 → likely the answer city
- LC 1976: only one path exists → ways[n-1] = 1; multiple equal-length paths → ways accumulates; answer modulo 10⁹+7
- LC 1928: maxTime = 0 → can only be at city 0; start == end (n=1) → return passingFees[0]; no path within maxTime → return -1

---

### Interview Pattern Drill

| Algorithm | Use case | Time complexity | Key data structure |
|-----------|---------|----------------|-------------------|
| Floyd-Warshall | All-pairs shortest path; n ≤ 200 | O(n³) | 2D dist array |
| Dijkstra | Single-source shortest path; non-negative weights | O((V+E) log V) | Min-heap |
| Dijkstra + count DP | Count shortest paths | O((V+E) log V) | Min-heap + ways[] |
| 2D Bellman-Ford | Shortest path with resource constraint (time, hops) | O(budget × E) | 2D dp table |

---

## System Design (1 hour)
### Topic: Key-Value Store — Single-Node Design (MemTable, WAL, SSTable)

**What is a key-value store?**
A distributed store that maps arbitrary byte-string keys to arbitrary byte-string values. Examples: Redis (in-memory), RocksDB (disk-based), DynamoDB (distributed). All CRUD operations are: `get(key)`, `put(key, value)`, `delete(key)`.

**Why not just use a HashMap in memory?**
- Capacity: memory is limited; data must spill to disk
- Durability: process crash loses all in-memory state
- Large data sets: need efficient disk I/O

**LSM Tree (Log-Structured Merge Tree) — the standard disk-based KV design:**

```
Write path:
  1. Write to Write-Ahead Log (WAL) — append-only file on disk; ensures durability
  2. Write to MemTable — sorted in-memory data structure (Red-Black tree or skip list)
  3. When MemTable exceeds size limit (e.g., 64 MB): flush to disk as SSTable (Sorted String Table)

Read path:
  1. Check MemTable (most recent writes)
  2. Check SSTables from newest to oldest (most recent first wins)
  3. Use Bloom filter per SSTable to skip SSTables that definitely don't have the key
```

**Write-Ahead Log (WAL):**
- Sequential append to a single file — very fast (disk sequential writes >> random writes)
- On crash: replay WAL from last checkpoint to rebuild MemTable
- WAL is deleted/truncated after MemTable is flushed to SSTable

**MemTable:**
- Sorted in-memory structure (usually a Red-Black tree or sorted skip list)
- Supports O(log n) insert and O(log n) lookup
- On flush: serialized in sorted key order → SSTable

**SSTable (Sorted String Table):**
- Immutable sorted file on disk
- Supports efficient binary search (sorted key order)
- Each SSTable has an associated Bloom filter and sparse index for fast lookups
- SSTables are organized into levels (L0, L1, L2...); lower levels are smaller and newer

**Bloom filter per SSTable:**
- Before reading an SSTable: check its Bloom filter
- If filter says "definitely not present" → skip this SSTable entirely
- Reduces SSTable reads to only those likely to contain the key

**Interview talking point:** "A naive KV store does random disk writes — catastrophic for HDD and suboptimal for SSD. The LSM Tree converts all writes to sequential appends (WAL + MemTable flush → SSTable), which maximises disk throughput. The trade-off is read amplification: a read may need to search multiple SSTables. Bloom filters reduce this to O(1) SSTable checks for non-existent keys."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1334 Find City Smallest Neighbors (target: 18 min)
- **Medium 2:** LC 1976 Number of Ways to Arrive (target: 18 min)
- **Hard:** LC 1928 Min Cost Reach Destination in Time (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you chose a well-known algorithm or design pattern over a custom solution — what made the standard approach the right call?
- Leadership principle: Are Right, A Lot

---

## Flashcards

| Q | A |
|---|---|
| What is the Floyd-Warshall algorithm and when do you use it? | Triple nested loop over all intermediate nodes k, then i→j via k. O(n³). Use for all-pairs shortest path when n ≤ ~200; not for single-source queries (use Dijkstra). |
| How does counting shortest paths extend Dijkstra? | Maintain `ways[v]`: when relaxation finds a shorter path, `ways[v] = ways[u]`; when it finds an equal-length path, `ways[v] += ways[u]`; answer modulo 10⁹+7. |
| How does 2D Bellman-Ford handle a time budget constraint? | State = `(node, time_used)`; `dp[t][v]` = min cost to reach v at time t; transition: for each edge (u,v,w), `dp[t][v] = min(dp[t][v], dp[t-w][u] + fee[v])`. |
| Why does LSM Tree use sequential writes instead of random writes? | Sequential disk writes (WAL + MemTable flush → SSTable) are 10–100× faster than random writes. All mutations first hit the append-only WAL, then the in-memory MemTable — no random disk I/O on the write path. |
| What is the role of a Bloom filter in an SSTable-based KV store? | Before reading each SSTable, check its Bloom filter. If the filter says "definitely not present," skip the SSTable — avoids an expensive disk read. Reduces read amplification for missing keys to near O(1). |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/18/25 min, no hints)
- [ ] Rewrote Floyd-Warshall template and Dijkstra+count DP from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain WAL → MemTable → SSTable flow cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
