# Day 66 — Graphs Advanced: Dijkstra Counting + BFS Alternating Edges + BFS on Hypergraph
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Number of Restricted Paths counts paths that satisfy a Dijkstra-derived ordering constraint. Shortest Path with Alternating Colors encodes edge colour into BFS state `(node, last_colour)`. Bus Routes models routes as nodes in a hypergraph: the BFS is on routes, not stops — reaching a route unlocks all its stops simultaneously.

---

## DSA (2 hours)
### Pattern: Dijkstra + count DP on restricted ordering + BFS with edge-type state + BFS on route hypergraph

**Number of Restricted Paths From First to Last Node (LC 1786):**
A path from node 1 to node n is "restricted" if, for each consecutive pair (u, v), `distToLastNode(u) > distToLastNode(v)` — i.e., each step gets closer to n (strictly). First, run Dijkstra from node n (reverse direction) to get `dist[]` for all nodes. Then count the number of restricted paths from 1 to n using DP on the DAG defined by the `dist` ordering (DFS/memoization or topological DP).

**Shortest Path in a Graph With Alternating Colors (LC 1129):**
Edges are red or blue. Find the shortest path from node 0 to all other nodes using alternating colours (no two consecutive edges of the same colour). BFS state = `(node, last_colour)`. From each state, only extend with edges of the opposite colour. Initialize queue with both `(0, RED)` and `(0, BLUE)` with 0 steps.

**Bus Routes (LC 815):**
Given routes (each route = a list of stops), find minimum bus transfers to reach target stop from source stop. Key insight: don't BFS over stops directly — BFS over ROUTES. Two routes are adjacent if they share a stop. When you board a route, you can reach all its stops for free (no additional transfer). BFS levels = number of transfers.

Structure:
1. Build `stop → routes` map
2. BFS queue starts with all routes containing the source stop (level 0)
3. For each route in queue, visit all its stops; for each stop, enqueue all routes through that stop (if not yet boarded)
4. Return level when target stop is reached

**Trigger condition:**
- "count paths where each step is 'closer' to destination (by shortest path dist)" → Dijkstra from dest; DP on DAG defined by dist ordering
- "shortest path alternating between edge types" → BFS with `(node, last_type)` state; initialize both types at source
- "minimum transfers through groups (routes) where reaching a group gives access to all members" → BFS on groups (routes), not elements (stops)

**Time complexity:** LC 1786: O((V+E) log V) | LC 1129: O(V+E) | LC 815: O(stops × routes)
**Space complexity:** O(V+E) for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Number of Restricted Paths | 1786 | Medium | Dijkstra from n + DP count on dist-ordered DAG | Dijkstra from node n; count paths using DFS+memo where `dist[u] > dist[v]` |
| 2 | Shortest Path Alternating Colors | 1129 | Medium | BFS state = `(node, last_colour)` | Init queue with `(0, RED)` and `(0, BLUE)`; only extend with opposite colour |
| 3 | Bus Routes | 815 | Hard | BFS on routes (hypergraph); transfers = BFS levels | `stop→routes` map; BFS over routes; boarding a route gives all its stops for free |

---

### Code Skeleton
```python
import heapq
from collections import deque, defaultdict
from functools import lru_cache

# Number of Restricted Paths (LC 1786)
def countRestrictedPaths(n, edges):
    MOD = 10**9 + 7
    graph = defaultdict(list)
    for u, v, w in edges:
        graph[u].append((v, w))
        graph[v].append((u, w))
    # Dijkstra from node n
    dist = [float('inf')] * (n + 1)
    dist[n] = 0
    heap = [(0, n)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]: continue
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))
    # Count restricted paths from 1 to n using DFS + memoization
    memo = {}
    def dfs(u):
        if u == n: return 1
        if u in memo: return memo[u]
        count = 0
        for v, _ in graph[u]:
            if dist[u] > dist[v]:   # restricted condition: getting closer to n
                count = (count + dfs(v)) % MOD
        memo[u] = count
        return count
    return dfs(1)

# Shortest Path with Alternating Colors (LC 1129)
def shortestAlternatingPaths(n, redEdges, blueEdges):
    RED, BLUE = 0, 1
    graph = defaultdict(list)
    for u, v in redEdges:
        graph[u].append((v, RED))
    for u, v in blueEdges:
        graph[u].append((v, BLUE))
    ans = [-1] * n
    ans[0] = 0
    visited = {(0, RED), (0, BLUE)}
    queue = deque([(0, RED, 0), (0, BLUE, 0)])   # (node, last_colour, steps)
    while queue:
        node, colour, steps = queue.popleft()
        for nb, edge_colour in graph[node]:
            if edge_colour != colour and (nb, edge_colour) not in visited:
                visited.add((nb, edge_colour))
                if ans[nb] == -1: ans[nb] = steps + 1
                queue.append((nb, edge_colour, steps + 1))
    return ans

# Bus Routes (LC 815)
def numBusesToDestination(routes, source, target):
    if source == target: return 0
    stop_to_routes = defaultdict(set)
    for i, route in enumerate(routes):
        for stop in route:
            stop_to_routes[stop].add(i)
    visited_routes = set()
    visited_stops = {source}
    queue = deque()
    # Enqueue all routes containing source
    for route_idx in stop_to_routes[source]:
        queue.append((route_idx, 1))
        visited_routes.add(route_idx)
    while queue:
        route_idx, transfers = queue.popleft()
        for stop in routes[route_idx]:
            if stop == target: return transfers
            if stop not in visited_stops:
                visited_stops.add(stop)
                for next_route in stop_to_routes[stop]:
                    if next_route not in visited_routes:
                        visited_routes.add(next_route)
                        queue.append((next_route, transfers + 1))
    return -1
```

---

### Edge Cases to Trace Before Coding
- LC 1786: node 1 is unreachable from n's Dijkstra → `dist[1] = inf` → dfs(1) returns 0; only one restricted path → answer is 1
- LC 1129: no blue edges → node 0 can reach via red edges starting with RED; all edges same colour → alternating impossible → return -1 for unreachable nodes
- LC 815: source == target → return 0 immediately; source not on any route → return -1; target reachable from source's route directly → return 1

---

### Interview Pattern Drill

| Problem type | Graph nodes | BFS/DFS over | "Cost" unit |
|-------------|------------|-------------|------------|
| Standard graph shortest path | Cities/nodes | Edges | Edge weight |
| Alternating edge colours | `(node, colour)` pairs | Edges of opposite colour | Steps |
| Route-based (bus/train) | Routes | Route adjacency (shared stops) | Transfers |
| Restricted path count | Nodes sorted by dist | Directed DAG edges | Path count (mod) |

---

## System Design (1 hour)
### Topic: Key-Value Store — Consistency Models and Vector Clocks

**The consistency problem in distributed KV stores:**
With N replicas of each key, writes may reach different replicas at different times. A reader querying two different replicas may get different values for the same key — which is correct?

**Consistency models (weakest to strongest):**

| Model | Guarantee | Example |
|-------|-----------|---------|
| Eventual consistency | All replicas converge eventually | DynamoDB (default), Cassandra |
| Read-your-writes | A client always reads its own writes | Session-pinned reads |
| Monotonic read | A client never reads an older value than it saw before | Read from same replica |
| Causal consistency | Writes causally related appear in order | Vector clocks |
| Strong consistency | All reads see the last committed write | ZooKeeper, etcd |

**Vector clocks (causal consistency):**
A vector clock is a map `{node_id: version}`. Each write increments the writer's counter. When a write is propagated, the receiving replica merges clocks by taking the max of each component.

```
Client writes key K on Node A:  {A: 1}
Client writes key K on Node B:  {A: 1, B: 1}  (saw A's write first)
Node C receives A's write:      {A: 1}
Node C receives B's write:      {A: 1, B: 1}  → C knows B happened after A
```

Conflict detection: if two versions have incomparable vector clocks (neither dominates the other), they are concurrent writes — a conflict requiring resolution (last-writer-wins by timestamp, or application-level merge).

**Read repair:**
When a read is sent to R replicas and they return different values:
1. The coordinator picks the value with the highest vector clock (most recent)
2. It asynchronously writes the correct value back to stale replicas
3. Future reads get the correct value

**Last-writer-wins (LWW):**
Simpler than vector clocks: each write carries a timestamp; the write with the highest timestamp wins. Drawback: clock skew between nodes can cause a newer write to have a lower timestamp → lost update. Cassandra uses LWW by default.

**Interview talking point:** "DynamoDB provides eventual consistency by default (fast reads from any replica) and strong consistency optionally (reads from a quorum of replicas). For a shopping cart, eventual consistency is acceptable — if a user adds an item and immediately reads their cart, they might see the old version for a few milliseconds, which is fine. For a bank balance, use strong consistency — a user should never see a debit succeeded but balance unchanged."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1786 Number of Restricted Paths (target: 20 min)
- **Medium 2:** LC 1129 Shortest Path Alternating Colors (target: 15 min)
- **Hard:** LC 815 Bus Routes (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to resolve conflicting information from two sources — how did you determine which was correct?
- Leadership principle: Earn Trust

---

## Flashcards

| Q | A |
|---|---|
| How do you count restricted paths after running Dijkstra from destination? | DFS+memoization from source: from node u, sum paths to each neighbour v where `dist[u] > dist[v]` (restricted = getting strictly closer to destination); memoize to avoid recomputation |
| What is the BFS state for Shortest Path with Alternating Colors? | `(node, last_colour)` — the colour of the last edge used. Initialize queue with `(0, RED, 0)` and `(0, BLUE, 0)`. Only extend via edges of the OPPOSITE colour. |
| Why does Bus Routes BFS over routes instead of stops? | Boarding a route gives you all its stops "for free" — they're all the same distance. BFS over routes means 1 BFS level = 1 additional bus transfer. BFS over stops would overcount: two stops on the same bus are 1 transfer, not 2. |
| What is eventual consistency and when is it appropriate? | All replicas converge to the same value eventually (no timing guarantee). Appropriate when temporary staleness is acceptable — social feeds, shopping carts, analytics. NOT appropriate for financial transactions or inventory counts. |
| How do vector clocks detect concurrent (conflicting) writes? | Two versions V1 `{A:2, B:1}` and V2 `{A:1, B:2}` are concurrent if neither dominates the other (V1[A] > V2[A] but V1[B] < V2[B]). Concurrent versions require conflict resolution (LWW or app-level merge). |

---

## Checklist
- [ ] Solved all 3 problems (timed — 20/15/25 min, no hints)
- [ ] Rewrote alternating-colour BFS state and bus routes BFS structure from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain eventual vs strong consistency and vector clocks cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
