# Day 70 — Graphs Advanced: Slot 7 Full Synthesis
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Slot 7 close-out: synthesise advanced graph algorithms — closest node reachability (DFS), minimum vertex set to reach all nodes (in-degree analysis), and multi-source Dijkstra for minimum weighted subgraph. Synthesise the complete URL Shortener and Key-Value Store decision framework.

---

## DSA (2 hours)
### Pattern: DFS Reachability + In-degree Source Analysis + Multi-source Dijkstra Synthesis

**Find Closest Node to Given Two Nodes (LC 2359):**
Given a directed graph where each node has at most one outgoing edge (functional graph), find a node reachable from both node1 and node2 that minimises `max(dist(node1, meeting), dist(node2, meeting))`. Traverse from node1 using DFS/BFS to compute `dist1[]`; traverse from node2 to compute `dist2[]`. For each node v where both `dist1[v]` and `dist2[v]` are finite, compute `max(dist1[v], dist2[v])`. Return the node with minimum such value; break ties by index.

**Find Minimum Number of Vertices to Reach All Nodes (LC 1557):**
In a directed graph, find the smallest set of vertices from which all other nodes are reachable. Answer: all nodes with in-degree 0. If a node has any incoming edge, it can be reached from another node — we don't need to include it in the starting set. The minimum set = exactly the nodes with in-degree 0.

**Minimum Weighted Subgraph with the Required Paths (LC 2203):**
Find the minimum total edge weight subgraph that contains a path from `src1` to `dest` AND a path from `src2` to `dest`. The two paths may share edges (the shared portion is only counted once). The optimal subgraph has the two paths converging at some "meeting node" v, forming a Y-shape. For each candidate v:
- `d1[v]` = dist from src1 to v (Dijkstra on original graph from src1)
- `d2[v]` = dist from src2 to v (Dijkstra on original graph from src2)
- `d3[v]` = dist from v to dest (Dijkstra on REVERSE graph from dest)
- Total cost through v = `d1[v] + d2[v] + d3[v]`

Minimize over all v.

**Trigger condition:**
- "closest node reachable from two sources in a functional graph" → two DFS traversals; minimize max(dist1, dist2)
- "minimum set of starting vertices to reach all nodes" → in-degree analysis; answer = nodes with in-degree 0
- "minimum cost subgraph containing two paths with possible shared edges" → three Dijkstras (src1, src2, reverse from dest); minimize `d1[v] + d2[v] + d3[v]` over all v

**Time complexity:** LC 2359: O(V) | LC 1557: O(V+E) | LC 2203: O((V+E) log V)
**Space complexity:** O(V) / O(V+E) / O(V+E)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Find Closest Node to Given Two Nodes | 2359 | Medium | Two DFS traversals; minimize max(dist1, dist2) | Functional graph; no heap needed; detect cycles with visited set |
| 2 | Min Vertices to Reach All Nodes | 1557 | Medium | In-degree analysis | Nodes with in-degree 0 = no one can reach them; must be starting vertices |
| 3 | Minimum Weighted Subgraph | 2203 | Hard | Three Dijkstras (src1, src2, reverse from dest) | Y-shape optimal subgraph; minimize `d1[v] + d2[v] + d3[v]` for meeting node v |

---

### Code Skeleton
```java
import java.util.*;

class Solution {

    // Find Closest Node to Given Two Nodes (LC 2359)
    public static int closestMeetingNode(int[] edges, int node1, int node2) {
        int n = edges.length;
        int[] dist1 = getDist(node1, edges, n);
        int[] dist2 = getDist(node2, edges, n);
        int ans = -1;
        int minMax = Integer.MAX_VALUE;
        for (int v = 0; v < n; v++) {
            if (dist1[v] != -1 && dist2[v] != -1) {
                int mx = Math.max(dist1[v], dist2[v]);
                if (mx < minMax) {
                    minMax = mx;
                    ans = v;
                }
            }
        }
        return ans;
    }

    private static int[] getDist(int start, int[] edges, int n) {
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        int node = start, d = 0;
        while (node != -1 && dist[node] == -1) {
            dist[node] = d;
            d++;
            node = edges[node];
        }
        return dist;
    }

    // Min Vertices to Reach All Nodes (LC 1557)
    public static List<Integer> findSmallestSetOfVertices(int n, List<List<Integer>> edges) {
        Set<Integer> hasIncoming = new HashSet<>();
        for (List<Integer> e : edges) hasIncoming.add(e.get(1));
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (!hasIncoming.contains(i)) result.add(i);
        }
        return result;
    }

    // Minimum Weighted Subgraph (LC 2203)
    public static long minimumWeight(int n, int[][] edges, int src1, int src2, int dest) {
        List<int[]>[] graph = new ArrayList[n];
        List<int[]>[] revGraph = new ArrayList[n];
        for (int i = 0; i < n; i++) {
            graph[i] = new ArrayList<>();
            revGraph[i] = new ArrayList<>();
        }
        for (int[] e : edges) {
            graph[e[0]].add(new int[]{e[1], e[2]});
            revGraph[e[1]].add(new int[]{e[0], e[2]});
        }
        long[] d1 = dijkstra(src1, graph, n);    // dist from src1 to all nodes
        long[] d2 = dijkstra(src2, graph, n);    // dist from src2 to all nodes
        long[] d3 = dijkstra(dest, revGraph, n); // dist from dest to all nodes (reverse graph)

        long ans = Long.MAX_VALUE;
        for (int v = 0; v < n; v++) {
            if (d1[v] != Long.MAX_VALUE && d2[v] != Long.MAX_VALUE && d3[v] != Long.MAX_VALUE) {
                ans = Math.min(ans, d1[v] + d2[v] + d3[v]);
            }
        }
        return ans == Long.MAX_VALUE ? -1 : ans;
    }

    private static long[] dijkstra(int start, List<int[]>[] adj, int n) {
        long[] dist = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[start] = 0;
        PriorityQueue<long[]> heap = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        heap.offer(new long[]{0, start});
        while (!heap.isEmpty()) {
            long[] top = heap.poll();
            long d = top[0]; int u = (int) top[1];
            if (d > dist[u]) continue;
            for (int[] nb : adj[u]) {
                int v = nb[0], w = nb[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    heap.offer(new long[]{dist[v], v});
                }
            }
        }
        return dist;
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 2359: node1 == node2 → return node1 (dist max is 0); one node is in a cycle the other can't reach → only the non-cycle node pair contributes; no common reachable node → return -1
- LC 1557: all nodes have in-degree > 0 → impossible (problem guarantees a valid DAG where all are reachable from some source) — actually return empty list; node 0 always in result if no edges point to it
- LC 2203: src1 == src2 → reduces to single-source Dijkstra + dest; src1 or src2 unreachable from dest → return -1; meeting node v = src1 → `d1[v] = 0`; meeting node v = dest → `d3[v] = 0`

---

### Slot 7 Complete Algorithm Reference

| Algorithm | Graph type | Heap / Data structure | Time | Use when |
|-----------|-----------|----------------------|------|---------|
| Dijkstra | Non-negative weighted | Min-heap | O((V+E) log V) | Single-source shortest path |
| Max-prob Dijkstra | Probability graph | Max-heap (negate) | O((V+E) log V) | Maximize product probability |
| Bellman-Ford K-stops | Any (negative OK) | Array + iteration | O(K × E) | K-hop constraint |
| Floyd-Warshall | All-pairs | 2D array | O(n³) | n ≤ 200, all-pairs |
| 0-1 BFS | {0,1} edge weights | Deque | O(V+E) | Grid with binary costs |
| Dijkstra + count DP | Non-negative | Min-heap + ways[] | O((V+E) log V) | Count shortest paths |
| Multi-source Dijkstra | Non-negative | Min-heap × 3 runs | O((V+E) log V) | Y-shape subgraph min cost |
| Bitmask BFS | State graph | Deque | O(m×n×2^k) | Collect items with unlock |
| Kahn's topological sort | DAG | Queue | O(V+E) | Task ordering, cycle detection |
| Critical path DP | DAG | Queue + DP array | O(V+E) | Minimum makespan |
| Leaf-peeling topological | Undirected tree | Deque | O(V) | Centroid finding |
| Reverse graph + Kahn's | Directed | Queue | O(V+E) | Safe node detection |
| Two-dimension topo sort | DAG × 2 | Queue × 2 | O(V+E) | 2D ordering constraints |

---

## System Design (1 hour)
### Topic: URL Shortener + Key-Value Store — Full Decision Framework

**Master decision tree for URL shortener:**
```
Design a URL shortener?
  │
  ├─ Write path
  │    ├─ ID generation → Snowflake ID (avoid single-point DB counter)
  │    ├─ Encoding → Base62, 6 chars = 56 B unique codes
  │    ├─ Storage → MySQL (simple schema, read-optimised B+Tree)
  │    └─ Cache write → Redis after INSERT
  │
  ├─ Read path (redirect)
  │    ├─ Hit rate target → 99 %+ from Redis
  │    ├─ Cache miss → Read Replica → write to Redis
  │    ├─ Invalid codes → Bloom filter (no false negatives) → 404 without DB touch
  │    └─ Redirect type → 302 (analytics) or 301 (offload)
  │
  ├─ Scaling
  │    ├─ Read scale → read replicas (3–5 behind load balancer)
  │    ├─ Write scale → shard by hash(short_code) % N shards when > 10K writes/sec
  │    └─ Global → CDN for static redirect responses (edge caches 301s)
  │
  └─ Analytics
       ├─ Click events → Kafka topic per click
       └─ Aggregation → Flink/Spark stream → ClickHouse / analytics DB
```

**Master decision tree for Key-Value Store:**
```
Design a distributed key-value store?
  │
  ├─ Single-node storage engine
  │    ├─ Write-heavy → LSM Tree (WAL + MemTable + SSTable)
  │    ├─ Durability → WAL (append-only; replay on crash)
  │    ├─ Read optimisation → Bloom filter per SSTable + leveled compaction
  │    └─ Read path → MemTable → L0 SSTables → L1/L2 (newest-first)
  │
  ├─ Distribution
  │    ├─ Sharding → Consistent hashing with VNodes (150–200 per physical node)
  │    ├─ Replication → N=3 replicas (standard)
  │    └─ Write quorum → W=2, R=2 (R+W > N = 3 → strong consistency)
  │
  ├─ Consistency
  │    ├─ Strong → W=2, R=2; synchronous replication
  │    ├─ Eventual → W=1, R=1; async replication with read repair
  │    └─ Conflict resolution → vector clocks (causal) or LWW (simple)
  │
  └─ Fault tolerance
       ├─ Node failure → hinted handoff (writes buffered until recovery)
       ├─ Recovery → Merkle tree anti-entropy (sync only changed ranges)
       └─ Failure detection → gossip protocol (phi accrual failure detector)
```

**URL Shortener vs KV Store: 5-question decision guide:**
1. **Is the schema fixed and simple?** → URL Shortener: SQL. KV Store: arbitrary key-value, no schema.
2. **What is the write-to-read ratio?** → URL Shortener: 1:100 (SQL + Redis optimal). KV Store: often 1:1 to 10:1 (LSM optimal).
3. **Do you need range scans?** → URL Shortener: no (point lookups only). Some KV stores (Cassandra) support range scans with sorted keys.
4. **What consistency level is required?** → URL Shortener: eventual is fine (stale redirect is a minor UX issue). KV Store: configurable; financial use cases need strong.
5. **How large is the dataset?** → URL Shortener: 90 TB over 5 years — manageable with sharded SQL. KV Store: can scale to petabytes with LSM + consistent hashing.

**Interview synthesis talking point:** "If asked to design a system to store user preferences for 500 M users at Amazon: I'd use a distributed KV store (DynamoDB). Keys are `{user_id:preference_name}`, values are arbitrary JSON. Each key is sharded by consistent hashing of `user_id`. N=3 replicas, W=1 (fast writes), R=1 (fast reads, eventual consistency is fine for preferences). Bloom filters + in-memory caching for hot users. Compare to the URL shortener: fixed schema, SQL, Redis caching, 100:1 read ratio — very different workloads."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — full Slot 7 synthesis
- **Medium 1:** LC 2359 Find Closest Node (target: 15 min)
- **Medium 2:** LC 1557 Min Vertices to Reach All Nodes (target: 10 min)
- **Hard:** LC 2203 Minimum Weighted Subgraph (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Walk through a distributed system you built or improved — describe how you handled the CAP theorem trade-offs (consistency, availability, partition tolerance).
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| How does Find Closest Node handle cycles in a functional graph? | Track visited nodes per traversal; if the next node in `edges[]` is already visited, stop — the graph has a cycle and we won't reach new nodes beyond it |
| Why are nodes with in-degree 0 the minimum set to reach all nodes? | Any node with an incoming edge is reachable from another node — it doesn't need to be in the starting set. Only nodes with no incoming edges have no potential "parent" — they must be starting vertices. |
| What are the three Dijkstra runs for Minimum Weighted Subgraph? | (1) Dijkstra from src1 on original graph → `d1[]`. (2) Dijkstra from src2 on original graph → `d2[]`. (3) Dijkstra from dest on REVERSE graph → `d3[]`. For each meeting node v: cost = `d1[v] + d2[v] + d3[v]`. |
| What are the 5 key differences between URL Shortener and KV Store architectures? | (1) Schema: fixed SQL vs schema-less. (2) Write-to-read: 1:100 vs 1:1 to 10:1. (3) Storage engine: B+Tree index vs LSM Tree. (4) Sharding: hash(short_code) vs consistent hashing. (5) Consistency: eventual OK vs configurable (strong for financial). |
| State the full Key-Value Store write path (LSM Tree). | (1) Client sends PUT(key, value). (2) Write to WAL (append-only, durability). (3) Write to MemTable (in-memory sorted tree). (4) When MemTable full: flush to new L0 SSTable. (5) Background compaction merges L0→L1→L2... to reduce read amplification. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/10/25 min, no hints)
- [ ] Rewrote multi-source Dijkstra template and in-degree analysis from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Recited the Slot 7 algorithm reference table trigger conditions from memory
- [ ] Completed system design synthesis — recited both decision frameworks cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
