# Day 54 — Graphs: Union-Find & Critical Connections
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Union-Find (Disjoint Set Union) with path compression and union-by-rank achieves near-O(1) per operation and solves connected-component problems cleanly. Tarjan's bridge-finding algorithm identifies critical edges whose removal disconnects the graph.

---

## DSA (2 hours)
### Pattern: Union-Find (Path Compression + Union by Rank) + Tarjan's Bridge Detection

**Number of Provinces (LC 547):**
Build a Union-Find over n cities. For each pair of directly connected cities (matrix[i][j] == 1), union them. The number of remaining distinct root representatives = number of provinces.

**Redundant Connection (LC 684):**
Process edges one by one. Union each edge's endpoints. If two endpoints already share the same root → this edge creates a cycle → it's the redundant connection. Return the last such edge (problem guarantees exactly one).

**Critical Connections in a Network (LC 1192):**
Tarjan's bridge-finding. DFS on the undirected graph, tracking:
- `disc[v]`: DFS discovery timestamp of node v
- `low[v]`: the lowest discovery time reachable from the subtree rooted at v (via any back-edge)

An edge `(u, v)` (where v is a child of u in DFS) is a **bridge** if `low[v] > disc[u]`. This means there is no back-edge from v's subtree to u or any of u's ancestors — removing this edge disconnects the graph.

**Union-Find template:**
```python
parent = list(range(n))
rank = [0] * n

def find(x):
    while parent[x] != x:
        parent[x] = parent[parent[x]]   # path compression (halving)
        x = parent[x]
    return x

def union(x, y):
    px, py = find(x), find(y)
    if px == py: return False   # already connected
    if rank[px] < rank[py]: px, py = py, px
    parent[py] = px
    if rank[px] == rank[py]: rank[px] += 1
    return True
```

**Trigger condition:**
- "count connected components" → Union-Find; union all edges; count distinct roots
- "detect if adding an edge creates a cycle" → union(u, v) returns False → cycle found
- "find edges whose removal disconnects the graph" → Tarjan's bridge detection; `low[v] > disc[u]`

**Time complexity:** LC 547/684: O(n × α(n)) ≈ O(n) | LC 1192: O(V+E)
**Space complexity:** O(n) / O(V+E)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Number of Provinces | 547 | Medium | Union-Find on adjacency matrix | Union directly-connected pairs; count distinct roots (nodes where `find(i) == i`) |
| 2 | Redundant Connection | 684 | Medium | Union-Find cycle detection | First edge where both endpoints already have the same root is the redundant connection |
| 3 | Critical Connections in a Network | 1192 | Hard | Tarjan's bridge algorithm | Edge (u,v) is bridge if `low[v] > disc[u]` — v cannot reach u's ancestor without this edge |

---

### Code Skeleton
```python
from collections import defaultdict

# Number of Provinces (LC 547)
def findCircleNum(isConnected):
    n = len(isConnected)
    parent = list(range(n))
    rank = [0] * n
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    def union(x, y):
        px, py = find(x), find(y)
        if px == py: return
        if rank[px] < rank[py]: px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]: rank[px] += 1
    for i in range(n):
        for j in range(i+1, n):
            if isConnected[i][j]: union(i, j)
    return sum(1 for i in range(n) if find(i) == i)

# Redundant Connection (LC 684)
def findRedundantConnection(edges):
    n = len(edges)
    parent = list(range(n+1))
    rank = [0] * (n+1)
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    def union(x, y):
        px, py = find(x), find(y)
        if px == py: return False
        if rank[px] < rank[py]: px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]: rank[px] += 1
        return True
    for u, v in edges:
        if not union(u, v): return [u, v]

# Critical Connections in a Network (LC 1192) — Tarjan's bridges
def criticalConnections(n, connections):
    graph = defaultdict(list)
    for u, v in connections:
        graph[u].append(v)
        graph[v].append(u)
    disc = [-1] * n
    low  = [-1] * n
    timer = [0]
    bridges = []

    def dfs(node, parent):
        disc[node] = low[node] = timer[0]
        timer[0] += 1
        for nb in graph[node]:
            if nb == parent: continue   # don't go back the same edge
            if disc[nb] == -1:
                dfs(nb, node)
                low[node] = min(low[node], low[nb])
                if low[nb] > disc[node]:
                    bridges.append([node, nb])
            else:
                low[node] = min(low[node], disc[nb])

    dfs(0, -1)
    return bridges
```

---

### Edge Cases to Trace Before Coding
- LC 547: single city → 1 province; all cities connected → 1 province; no connections → n provinces
- LC 684: problem guarantees exactly one redundant edge exists; edges given as 1-indexed
- LC 1192: parallel edges between same two nodes (multigraph) — the simple "skip parent" check will incorrectly skip the parallel edge; use edge index tracking instead of parent node tracking for robustness. The skeleton above skips the parent node (simpler, works for simple graphs)

---

### Interview Pattern Drill
| Problem type | Union-Find or DFS? | Key step |
|-------------|-------------------|---------|
| Connected components, static | Union-Find | Count distinct `find(i) == i` |
| Cycle detection in undirected | Union-Find | `union(u,v)` returns False |
| Cycle detection in directed | DFS 3-colour | GRAY node = back-edge |
| Bridge finding | DFS (Tarjan's) | `low[v] > disc[u]` |

---

## System Design (1 hour)
### Topic: HTTP Caching Headers — Cache-Control, ETag, and CDN Cache Invalidation

**Why HTTP caching matters:**
Serving a cached response takes ~0.5 ms. Serving an uncached response (DB query + computation) takes ~50–200 ms. For a site with 10 M daily requests, a 90 % cache hit rate saves ~9 M DB queries per day.

**Cache-Control directives:**
| Directive | Meaning |
|-----------|---------|
| `max-age=N` | Cache the response for N seconds from request time |
| `s-maxage=N` | Overrides max-age for shared caches (CDN); browser uses max-age |
| `no-cache` | Cache the response but ALWAYS revalidate with server before using it |
| `no-store` | Never cache — response contains sensitive data |
| `private` | Only browser can cache (not CDNs) — for personalised responses |
| `public` | Can be cached by any cache including CDN |
| `immutable` | Content will never change — browser won't revalidate even on refresh |
| `stale-while-revalidate=N` | Serve stale content for N seconds while re-fetching in background |

**Conditional requests — ETag:**
ETag = a hash or version ID of the response body, sent by the server.
```
# First request:
GET /api/products/123
← Response: 200 OK, ETag: "abc123", body: {...}

# Second request (client sends the ETag):
GET /api/products/123
If-None-Match: "abc123"
← Response: 304 Not Modified (no body) — saves bandwidth
```
**Last-Modified** is the date-based alternative: `If-Modified-Since: Wed, 10 May 2026 12:00:00 GMT`.

**CDN cache invalidation strategies:**
1. **TTL expiry (simplest):** Wait for `Cache-Control: max-age` to expire. Acceptable if staleness window ≤ TTL.
2. **URL versioning (best for static assets):** Embed a content hash in the URL: `main.abc123.js`. Old URL is immutable — no invalidation needed. New content = new URL.
3. **CDN API purge:** Cloudflare, CloudFront, Fastly all expose an API to purge specific URLs or cache tags instantly. Useful for content that changes unpredictably.
4. **Cache tags / surrogate keys:** Tag related cache entries with a logical key (e.g., `product:42`); purge all entries with that tag when product 42 changes.

**Strategy selection:**
| Asset type | Strategy | TTL |
|-----------|---------|-----|
| CSS/JS/Images with hash | URL versioning + `immutable` | 1 year |
| API responses | Short TTL + ETag revalidation | 60 s |
| HTML pages | Short TTL + CDN purge on deploy | 5 min |
| Private user data | `Cache-Control: private, no-store` | — |

**Interview talking point:** "If asked how to cache a product catalogue page at CDN with instant invalidation when a product changes, answer: use cache tags. Each product page is tagged with `product:{id}` when cached at the CDN. When a product is updated in the DB, the write path calls the CDN purge API with tag `product:{id}`. All PoPs purge matching pages within seconds. The alternative — short TTL — accepts up to N seconds of staleness; the trade-off is simpler (no API calls) but can show stale prices or inventory."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you identified a single point of failure in a system (analogous to a bridge edge) and removed it — what was the impact?
- Leadership principle: Insist on the Highest Standards

---

## Flashcards

| Q | A |
|---|---|
| How does Union-Find with path compression and union-by-rank achieve near-O(1) per operation? | Path compression flattens the tree on each `find()` call; union-by-rank attaches smaller trees under taller ones; combined gives amortised O(α(n)) — inverse Ackermann, effectively constant |
| How does Redundant Connection use Union-Find to find the cycle edge? | Process edges in order; `union(u, v)` returns False when u and v are already in the same component; that edge is the one creating the cycle — return it |
| What does `low[v] > disc[u]` mean in Tarjan's bridge detection? | v's subtree has no back-edge that reaches u or any of u's ancestors — removing edge (u,v) disconnects the graph; it is a bridge |
| What is the difference between `Cache-Control: no-cache` and `no-store`? | `no-cache`: cache the response but always revalidate with the server before serving it (a 304 Not Modified avoids retransmitting the body). `no-store`: never cache the response at all — for sensitive data |
| What is URL versioning for static assets and why is it preferred over CDN purge? | Embed a content hash in the URL (e.g., `app.abc123.css`); old URLs are immutable (1-year cache); new content gets a new URL — zero invalidation delay, zero CDN API calls, no stale-content risk |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory (especially Union-Find template)
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
