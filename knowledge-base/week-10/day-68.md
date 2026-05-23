# Day 68 — Graphs Advanced: 0-1 BFS & Binary Search on Graphs
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
0-1 BFS (deque BFS) solves single-source shortest path on graphs with edge weights of only 0 or 1 in O(V+E) — avoiding the O((V+E) log V) overhead of Dijkstra. Find Safest Path combines binary search on the answer with BFS reachability. Second Minimum Time is BFS with careful state tracking of the minimum and second-minimum distances.

---

## DSA (2 hours)
### Pattern: 0-1 BFS (deque) + Binary Search on Graph Answer + BFS with Two-Distance State

**Minimum Obstacle Removal to Reach Corner (LC 2290):**
Grid with 0 (empty) and 1 (obstacle). Move from top-left to bottom-right; removing an obstacle costs 1. Find minimum obstacles to remove. Edge weights: moving to a 0-cell costs 0, moving to a 1-cell costs 1. Classic 0-1 BFS: use a deque; weight-0 edges go to the front (`appendleft`), weight-1 edges go to the back (`append`). No heap needed — O(m×n).

**Find the Safest Path in a Grid (LC 2812):**
Each cell has a "safeness" score = Manhattan distance to the nearest thief. Find the path from top-left to bottom-right that maximises the minimum safeness score (minimax problem). Strategy: binary search on the minimum safeness threshold T. For each T, check if a path exists using only cells with safeness ≥ T (BFS/DFS reachability). Binary search range: [0, max_possible_safeness].

Precompute safeness: multi-source BFS from all thief positions simultaneously.

**Second Minimum Time to Reach Destination (LC 2045):**
Graph with uniform edge weights (travel time = `time`), traffic lights that flip every `change` seconds. Find the second minimum time to reach node n from node 1. A path can revisit nodes. Key insight: the second minimum time must be either `shortest + 2*time` (take a detour of 2 extra edges on any path) or `shortest_2` (an actually longer distinct path). BFS where state tracks both `dist1[v]` and `dist2[v]` (minimum and second minimum distance to each node). A distance is accepted into `dist2[v]` if it's strictly greater than `dist1[v]`.

**Trigger condition:**
- "minimum cost path where edge costs are 0 or 1" → 0-1 BFS with deque (appendleft for 0, append for 1)
- "find path maximising the minimum value / safeness along the way" → binary search on threshold + BFS reachability check
- "second minimum distance in a graph" → BFS tracking two distances per node (min and second-min)

**Time complexity:** LC 2290: O(m×n) | LC 2812: O(m×n log(m×n)) | LC 2045: O((V+E) log V)
**Space complexity:** O(m×n) for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Obstacle Removal | 2290 | Medium | 0-1 BFS (deque) | Weight = cell value (0 or 1); `appendleft` for 0, `append` for 1; dist = min obstacles removed |
| 2 | Find Safest Path in Grid | 2812 | Medium | Multi-source BFS + binary search on min safeness | Precompute safeness via multi-source BFS from thieves; binary search threshold T; check path exists |
| 3 | Second Minimum Time | 2045 | Hard | BFS with `dist1` and `dist2` per node | Accept into `dist2` if `d > dist1[v]`; simulate traffic light delays; answer at node n using `dist2` |

---

### Code Skeleton
```java
import java.util.*;

class Solution {

    // Minimum Obstacle Removal (LC 2290) — 0-1 BFS
    public static int minimumObstacles(int[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        int[][] dist = new int[rows][cols];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        dist[0][0] = 0;
        Deque<int[]> dq = new ArrayDeque<>();
        dq.addLast(new int[]{0, 0, 0}); // (cost, row, col)
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
        while (!dq.isEmpty()) {
            int[] cur = dq.pollFirst();
            int cost = cur[0], r = cur[1], c = cur[2];
            if (cost > dist[r][c]) continue;
            if (r == rows - 1 && c == cols - 1) return cost;
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                    int newCost = cost + grid[nr][nc]; // 0 or 1
                    if (newCost < dist[nr][nc]) {
                        dist[nr][nc] = newCost;
                        if (grid[nr][nc] == 0) {
                            dq.addFirst(new int[]{newCost, nr, nc}); // 0-cost → front
                        } else {
                            dq.addLast(new int[]{newCost, nr, nc});  // 1-cost → back
                        }
                    }
                }
            }
        }
        return dist[rows - 1][cols - 1];
    }

    // Find Safest Path in Grid (LC 2812)
    public static int maximumSafenessFactor(List<List<Integer>> gridList) {
        int n = gridList.size();
        int[][] grid = new int[n][n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                grid[i][j] = gridList.get(i).get(j);

        // Multi-source BFS from all thieves to compute safeness
        int[][] safeness = new int[n][n];
        for (int[] row : safeness) Arrays.fill(row, -1);
        Deque<int[]> dq = new ArrayDeque<>();
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 1) {
                    safeness[r][c] = 0;
                    dq.addLast(new int[]{r, c});
                }
            }
        }
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
        while (!dq.isEmpty()) {
            int[] cur = dq.pollFirst();
            int r = cur[0], c = cur[1];
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && safeness[nr][nc] == -1) {
                    safeness[nr][nc] = safeness[r][c] + 1;
                    dq.addLast(new int[]{nr, nc});
                }
            }
        }
        // Binary search on minimum safeness threshold
        int lo = 0, hi = 0;
        for (int[] row : safeness) for (int v : row) hi = Math.max(hi, v);
        while (lo < hi) {
            int mid = (lo + hi + 1) / 2;
            if (canReach(safeness, mid, n, dirs)) lo = mid;
            else hi = mid - 1;
        }
        return lo;
    }

    private static boolean canReach(int[][] safeness, int threshold, int n, int[][] dirs) {
        if (safeness[0][0] < threshold || safeness[n-1][n-1] < threshold) return false;
        boolean[][] visited = new boolean[n][n];
        visited[0][0] = true;
        Deque<int[]> q = new ArrayDeque<>();
        q.addLast(new int[]{0, 0});
        while (!q.isEmpty()) {
            int[] cur = q.pollFirst();
            int r = cur[0], c = cur[1];
            if (r == n - 1 && c == n - 1) return true;
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && !visited[nr][nc] && safeness[nr][nc] >= threshold) {
                    visited[nr][nc] = true;
                    q.addLast(new int[]{nr, nc});
                }
            }
        }
        return false;
    }

    // Second Minimum Time (LC 2045)
    public static int secondMinimum(int n, int[][] edges, int time, int change) {
        List<Integer>[] graph = new ArrayList[n + 1];
        for (int i = 0; i <= n; i++) graph[i] = new ArrayList<>();
        for (int[] e : edges) {
            graph[e[0]].add(e[1]);
            graph[e[1]].add(e[0]);
        }
        int[] dist1 = new int[n + 1];
        int[] dist2 = new int[n + 1];
        Arrays.fill(dist1, Integer.MAX_VALUE);
        Arrays.fill(dist2, Integer.MAX_VALUE);
        dist1[1] = 0;
        Deque<int[]> queue = new ArrayDeque<>();
        queue.addLast(new int[]{0, 1}); // (time_elapsed, node)
        while (!queue.isEmpty()) {
            int[] cur = queue.pollFirst();
            int t = cur[0], u = cur[1];
            for (int v : graph[u]) {
                // compute travel time considering traffic lights
                int cycle = t / change;
                int tDepart = (cycle % 2 == 1) ? (cycle + 1) * change : t;
                int arrive = tDepart + time;
                if (arrive < dist1[v]) {
                    dist2[v] = dist1[v];
                    dist1[v] = arrive;
                    queue.addLast(new int[]{arrive, v});
                } else if (dist1[v] < arrive && arrive < dist2[v]) {
                    dist2[v] = arrive;
                    queue.addLast(new int[]{arrive, v});
                }
            }
        }
        return dist2[n];
    }
}
```

---

### STAR Interview Framework

> **0-1 BFS (Deque):** brute-force Dijkstra O((V+E) log V) → this approach O(V+E) time, O(V+E) space

**S:** "Given an m×n grid where cells contain 0 (free) or 1 (obstacle), find minimum obstacles to remove to reach bottom-right from top-left. Edge weights are binary (0 or 1). Dijkstra works but adds unnecessary O(log(m×n)) heap overhead."
**T:** "Need O(m×n) — linear in grid size — by exploiting the binary edge-weight structure with a deque instead of a heap."
**A (60% of answer time):**
1. *Classify:* "'Minimum cost path where edge costs are 0 or 1' — 0-1 BFS signal: replace min-heap with a deque."
2. *Init:* "dist[0][0] = 0, all others = MAX_VALUE; deque = [(0, 0, 0)]; 4-directional movement."
3. *Loop/Step:* "For each (cost, r, c): for each neighbour (nr, nc): newCost = cost + grid[nr][nc]; if newCost < dist[nr][nc]: update and addFirst if grid[nr][nc]==0, addLast if grid[nr][nc]==1."
4. *Termination:* "Each cell processed at most twice (once from front, once stale — skip via cost > dist guard); terminates at bottom-right."
5. *Gotcha:* "Must check `if (cost > dist[r][c]) continue` at the top of the loop — stale entries accumulate in the deque because we don't have a decrease-key operation. Without this guard, you process obsolete states and get wrong (too-high) distances."
**R:** "O(m×n) time, O(m×n) space. Removes the log overhead of Dijkstra — for a 1000×1000 grid: ~10^6 operations vs ~20×10^6 with a heap."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Dijkstra with min-heap | Arbitrary non-negative weights | Works correctly but O((V+E) log V) — 20× slower than 0-1 BFS for binary weights |
| Standard BFS | Unweighted graph (all edges cost 1) | Edge weights 0 and 1 violate BFS's equal-cost assumption — produces wrong distances |

---

> **Binary Search on Answer + Multi-Source BFS Reachability:** brute-force O(m×n × max_safeness) → this approach O(m×n log(m×n)) time, O(m×n) space

**S:** "Given an n×n grid with thieves, find the path from top-left to bottom-right that maximises the minimum Manhattan distance to any thief (minimax path). Naive: try all paths and track minimum safeness — exponential."
**T:** "Need O(n² log n) by binary searching on the answer (minimum safeness threshold) and checking reachability via BFS for each candidate."
**A (60% of answer time):**
1. *Classify:* "'Maximise the minimum value along a path' — binary search on threshold + feasibility check signal."
2. *Init:* "Multi-source BFS from all thieves simultaneously → safeness[r][c] for each cell. Binary search lo=0, hi=max(safeness)."
3. *Loop/Step:* "For each mid: canReach(safeness, mid) — BFS from (0,0) using only cells with safeness ≥ mid; if reachable → lo = mid; else → hi = mid - 1."
4. *Termination:* "Binary search converges in O(log(max_safeness)) ≤ O(log(n)) iterations; each iteration is O(n²) BFS."
5. *Gotcha:* "Check safeness[0][0] < threshold before BFS — if the starting cell itself is below threshold, return false immediately. Forgetting this causes BFS to start from an invalid cell and return false correctly but wastes O(n²) work."
**R:** "O(n² log n) time, O(n²) space. n=400 grid: ~640K operations per binary search step × log(400) ≈ 9 steps = ~5.7M total operations."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Dijkstra with max-safeness priority | Direct optimisation without binary search | Works (O(n² log n)) but more complex to implement; binary search + simple BFS is cleaner |
| DFS for reachability check | Same complexity | DFS works but BFS is easier to reason about for BFS-like level structures |

---

> **BFS Two-Distance Tracking:** brute-force O(V! path enumeration) → this approach O((V+E) log V) time, O(V+E) space

**S:** "Given a graph with n nodes and uniform edge weight `time`, traffic lights flipping every `change` seconds, find the second minimum time to reach node n from node 1. Brute force enumerates all paths — exponential."
**T:** "Need O((V+E) log V) by BFS that tracks both the minimum and second-minimum arrival time per node."
**A (60% of answer time):**
1. *Classify:* "'Second minimum distance in a graph' — BFS with two distance slots per node: dist1[v] and dist2[v]."
2. *Init:* "dist1[] = MAX, dist2[] = MAX; dist1[1] = 0; queue = [(0, node1)]."
3. *Loop/Step:* "For each (t, u): compute departure time accounting for red lights (if in red phase, wait until next green); arrive = depart + time; accept into dist1[v] if < dist1[v], or into dist2[v] if dist1[v] < arrive < dist2[v]."
4. *Termination:* "Each node accepted into dist2 at most once; terminates; answer = dist2[n]."
5. *Gotcha:* "Traffic light handling: if `(t / change) % 2 == 1` (red phase), must wait until `(t/change + 1) * change` before departing. Missing this produces wrong departure times. Also: only accept into dist2 if strictly greater than dist1 — ties don't count as a 'second minimum'."
**R:** "O((V+E) log V) time (priority queue for correct ordering with variable wait times), O(V+E) space. Handles n=10,000 nodes correctly with traffic light simulation."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Simple BFS (no wait simulation) | Uniform edge weights without traffic lights | Ignores red-light delays; produces wrong times when waiting is required |
| DFS all paths | Find all paths to enumerate second min | Exponential; BFS with two-distance slots is O((V+E) log V) |

---

### Edge Cases to Trace Before Coding
- LC 2290: all cells are 0 → dist[n-1][m-1] = 0; only path goes through many obstacles → 0-1 BFS finds minimum cost correctly
- LC 2812: no thieves → all safeness = inf (or max manhattan) → any path is safe → answer = 0 (trivial path); single cell with a thief at start → safeness[0][0] = 0 → return 0
- LC 2045: single direct edge → second minimum = direct path + backtrack + forward again = `time * 3` adjusted for traffic; n=2 is the classic example — second min time = `time + 2*time` with possible green-light wait

---

### Interview Pattern Drill

| Pattern | Data structure | Edge weight handling | Complexity vs Dijkstra |
|---------|---------------|---------------------|----------------------|
| Standard Dijkstra | Min-heap | Any non-negative weight | O((V+E) log V) |
| 0-1 BFS | Deque | 0 → `addFirst`; 1 → `addLast` | O(V+E) — no heap |
| Binary search + BFS | Deque/queue per check | Threshold filter on cells | O(E log(max_val)) |
| BFS two-distance | Deque | Standard BFS | O(V+E) × constant |

**0-1 BFS correctness:** The deque maintains the invariant that all cost-d nodes appear before cost-(d+1) nodes. Appending weight-0 edges to the front preserves this: a weight-0 step doesn't increase d, so the new node should be processed at the same "level." This is equivalent to Dijkstra on {0,1}-weight graphs but without the log overhead.

---

## System Design (1 hour)
### Topic: Key-Value Store — LSM Compaction and Bloom Filters (Deep Dive)

**The read amplification problem:**
Every write creates a new SSTable (or adds to L0). Over time, thousands of SSTables accumulate. A read for key K must search SSTables from newest to oldest — potentially reading many files before finding the key (or confirming it doesn't exist). This is read amplification.

**Compaction:**
Periodically merge SSTables to reduce their count and remove obsolete/deleted entries:
```
Before compaction: [L0: SST1, SST2, SST3, SST4]
After compaction:  [L1: SST_merged]  (sorted, deduplicated, tombstones removed)
```

**Leveled compaction (used in LevelDB, RocksDB):**
```
L0: 4 SSTables (unsorted across files; may overlap)
L1: 1 merged SSTable (10 MB; non-overlapping key ranges)
L2: 10 merged SSTables (100 MB each; non-overlapping)
L3: ...
```
- L0 → L1: triggered when L0 has ≥ 4 files; merge all L0 files + overlapping L1 files
- L1 → L2: triggered when L1 exceeds size limit; select one L1 SSTable + overlapping L2 SSTables; merge

**Write amplification:** Each byte written to L0 may be rewritten multiple times during compaction (L0→L1→L2→...). Write amplification factor ≈ 10–30× for leveled compaction.

**Size-tiered compaction (Cassandra default):**
Merge SSTables of similar size. Lower write amplification but higher space amplification (more duplicate keys exist between levels). Better for write-heavy workloads.

**Bloom filter (deep dive):**
A bit array of size m with k hash functions. To insert key K: compute `h1(K), h2(K), ..., hk(K)` and set those bit positions. To query K: check all k positions — if any is 0, K is definitely absent (no false negatives). If all are 1, K is probably present (false positive rate ≈ `(1 - e^(-kn/m))^k`).

For a KV store with 1 billion keys and 1% false positive rate:
- Optimal m ≈ 9.6 bits/key = 1.2 GB for 10⁹ keys
- Optimal k ≈ 7 hash functions

Each SSTable maintains its own Bloom filter in memory. Before a disk read: check the filter. If "definitely absent" → skip. Otherwise → read SSTable and confirm.

**Interview talking point:** "Without Bloom filters, a read for a non-existent key in RocksDB would scan every SSTable's index and possibly the data file. With a per-SSTable Bloom filter (9.6 bits/key, 1% FP rate), 99% of non-existent key lookups resolve in memory without any disk read. For a 1 TB dataset, this is the difference between O(log n) disk reads and O(1) disk reads for absent keys."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 2290 Minimum Obstacle Removal (target: 18 min)
- **Medium 2:** LC 2812 Find Safest Path in Grid (target: 20 min)
- **Hard:** LC 2045 Second Minimum Time (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
**Full STAR Story — "0-1 BFS: Eliminating Tail Latency by Removing the Unnecessary Heap":**
**S (20%):** "At a mapping startup, our real-time routing service had acceptable average latency (40ms) but p99 latency of 380ms — unacceptable for in-app turn-by-turn navigation. Our graph had binary-cost edges (toll/no-toll roads), but we were using full Dijkstra with a priority heap."
**T:** "I was assigned to reduce p99 latency below 100ms without changing the product's routing quality — same optimal paths, lower overhead."
**A (60% — 'I' not 'we'):** "(1) I profiled the routing hot path and identified that 60% of CPU cycles were spent on heap operations for what were effectively 0/1 edge weights — toll vs non-toll. (2) I replaced the priority queue with a deque-based 0-1 BFS: weight-0 edges (non-toll) appended to front, weight-1 edges (toll) appended to back — identical optimal paths, zero heap overhead. (3) I benchmarked on a 500,000-node road graph: 0-1 BFS processed each query in 4ms vs 28ms for Dijkstra — a 7× improvement. (4) I validated correctness by running both implementations on 10,000 randomly sampled routes and confirming identical optimal distances on 100% of cases before deploying."
**R (20%):** "p99 latency dropped from 380ms to 61ms — well under the 100ms target. Average latency fell from 40ms to 6ms. The fix required changing 30 lines of code. Routing service capacity effectively increased 4× without new infrastructure."
*Works for LP questions on: Dive Deep, Frugality, Insist on the Highest Standards.*

---

## Flashcards

| Q | A |
|---|---|
| How does 0-1 BFS achieve O(V+E) instead of O((V+E) log V)? | Uses a deque instead of a heap. Weight-0 edges push the new node to the front (`addFirst` — same "level" as current); weight-1 edges push to the back (`addLast` — next "level"). Maintains sorted order without a heap's log overhead. |
| What is the algorithm for Find Safest Path? | (1) Multi-source BFS from all thieves to compute per-cell safeness. (2) Binary search on threshold T. (3) For each T, BFS/DFS from top-left using only cells with safeness ≥ T. Return the largest T where a path exists. |
| How does BFS track the second minimum time? | Maintain `dist1[v]` and `dist2[v]`. Accept a new time t at v into `dist1[v]` if `t < dist1[v]` (also update `dist2[v] = old dist1[v]`). Accept into `dist2[v]` if `dist1[v] < t < dist2[v]`. Answer = `dist2[n]`. |
| What is leveled compaction and what is its trade-off? | SSTables organized in levels (L0→L1→L2...); L0 files merge into L1, L1 into L2, etc. Reduces read amplification (fewer SSTables to search per level). Trade-off: high write amplification — each byte rewritten ~10–30× across compaction levels. |
| What are the two key properties of a Bloom filter? | (1) No false negatives: if the filter says "absent," the key is definitely not there. (2) Small false positive rate (typically 1%): if the filter says "present," the key is probably there. Used to skip SSTable reads for absent keys. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/20/25 min, no hints)
- [ ] Rewrote 0-1 BFS deque pattern and binary search + reachability check from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain leveled compaction and Bloom filter trade-offs cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
