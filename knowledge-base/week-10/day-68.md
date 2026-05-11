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
- STAR prompt: Describe a time you optimised a system that had acceptable peak performance but poor worst-case performance — how did you identify and address the tail latency?
- Leadership principle: Dive Deep

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
