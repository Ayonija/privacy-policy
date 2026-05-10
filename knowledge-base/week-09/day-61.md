# Day 61 — Graphs Advanced: Dijkstra Standard & Bellman-Ford K-Stops
**Week 09 | Phase 1: DSA Mastery | Month 3**

## Focus
Dijkstra finds the shortest path in a non-negative weighted graph using a min-heap. Path with Max Probability flips Dijkstra to a max-heap to maximise a product. Cheapest Flights with K Stops requires Bellman-Ford — Dijkstra cannot enforce a hop constraint.

---

## DSA (2 hours)
### Pattern: Dijkstra (min-heap) + Max-Product Dijkstra + Bellman-Ford with K constraint

**Network Delay Time (LC 743):**
Weighted directed graph. Find the minimum time for a signal from node `k` to reach ALL nodes. Run Dijkstra from `k`. The answer is `max(dist)`. If any node is unreachable, return -1.

Standard Dijkstra template:
1. `dist = {k: 0}` (or `inf` for all others)
2. Min-heap `(0, k)`
3. Pop `(d, u)` → if `d > dist[u]`, skip (stale entry)
4. For each neighbour `v` with weight `w`: if `dist[u] + w < dist[v]`, update and push

**Path with Maximum Probability (LC 1514):**
Find the path that maximises the product of edge probabilities from `start` to `end`. Flip Dijkstra: use a max-heap `(prob, node)` (negate to use Python's min-heap). Instead of adding distances, multiply probabilities. A path is "better" if its product is higher. Because probabilities are in [0, 1], multiplying can only decrease or maintain — standard Dijkstra convergence still holds.

**Cheapest Flights Within K Stops (LC 787):**
Find the cheapest flight from `src` to `dst` with at most `k` stops (k+1 edges). **Dijkstra fails** here because a path with more edges but lower cost so far might be skipped as "visited" before the constraint is checked. Use Bellman-Ford: relax all edges exactly `k+1` times (one per flight leg), copying from the previous iteration's array to prevent using more than one edge per round.

**Trigger condition:**
- "shortest/fastest to reach all nodes from a source, non-negative weights" → Dijkstra from source; answer = max(dist)
- "path that maximises a probability product" → Dijkstra with max-heap; multiply instead of add
- "cheapest path with at most K hops/stops" → Bellman-Ford; copy prev array to enforce K-step limit

**Time complexity:** LC 743: O((V+E) log V) | LC 1514: O((V+E) log V) | LC 787: O(K × E)
**Space complexity:** O(V+E) / O(V+E) / O(V)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Network Delay Time | 743 | Medium | Dijkstra from source; answer = max dist | Skip stale heap entries; if any node unreachable → -1 |
| 2 | Path with Maximum Probability | 1514 | Medium | Max-Dijkstra (negate prob or max-heap) | Multiply probabilities; max-heap; 0.0 default; update if new_prob > old |
| 3 | Cheapest Flights Within K Stops | 787 | Hard | Bellman-Ford with K+1 relaxation rounds | Copy `prev` array before each round to enforce hop count; K stops = K+1 edges |

---

### Code Skeleton
```python
import heapq
from collections import defaultdict

# Network Delay Time (LC 743)
def networkDelayTime(times, n, k):
    graph = defaultdict(list)
    for u, v, w in times:
        graph[u].append((v, w))
    dist = {i: float('inf') for i in range(1, n + 1)}
    dist[k] = 0
    heap = [(0, k)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]: continue   # stale entry
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))
    ans = max(dist.values())
    return ans if ans < float('inf') else -1

# Path with Maximum Probability (LC 1514)
def maxProbability(n, edges, succProb, start, end):
    graph = defaultdict(list)
    for (u, v), p in zip(edges, succProb):
        graph[u].append((v, p))
        graph[v].append((u, p))
    prob = [0.0] * n
    prob[start] = 1.0
    heap = [(-1.0, start)]   # max-heap via negation
    while heap:
        neg_p, u = heapq.heappop(heap)
        p = -neg_p
        if p < prob[u]: continue   # stale
        for v, edge_p in graph[u]:
            new_p = p * edge_p
            if new_p > prob[v]:
                prob[v] = new_p
                heapq.heappush(heap, (-new_p, v))
    return prob[end]

# Cheapest Flights Within K Stops (LC 787) — Bellman-Ford
def findCheapestPrice(n, flights, src, dst, k):
    dist = [float('inf')] * n
    dist[src] = 0
    for _ in range(k + 1):   # k stops = k+1 edges
        prev = dist[:]        # copy to prevent chaining within one round
        for u, v, w in flights:
            if prev[u] != float('inf') and prev[u] + w < dist[v]:
                dist[v] = prev[u] + w
    return dist[dst] if dist[dst] != float('inf') else -1
```

---

### Edge Cases to Trace Before Coding
- LC 743: single node (n=1, k=1) → return 0; disconnected graph → some node stays at inf → return -1
- LC 1514: start == end → return 1.0; end is unreachable → return 0.0 (initial default)
- LC 787: k=0 means no stops → only direct flights from src → dst are valid; no valid path → return -1; src == dst → return 0

---

### Interview Pattern Drill

| Algorithm | Heap content | Update condition | Stale-entry check |
|-----------|-------------|-----------------|------------------|
| Dijkstra (min cost) | `(dist, node)` | `dist[u] + w < dist[v]` | `d > dist[u]` → skip |
| Max Dijkstra (max prob) | `(-prob, node)` | `p * edge_p > prob[v]` | `p < prob[u]` → skip |
| Bellman-Ford K-stops | no heap | `prev[u] + w < dist[v]` | copy `prev` before each round |

**Dijkstra correctness invariant:** Once a node is popped from the heap with distance d, d is the true shortest distance — because all shorter paths would have been popped first (non-negative weights). This invariant breaks with negative weights or hop constraints.

---

## System Design (1 hour)
### Topic: URL Shortener — Requirements, Encoding, and API Design

**What is a URL shortener?**
A service (e.g., bit.ly, tinyurl.com) that maps a long URL to a short code and redirects users. The short URL is compact (6–8 characters) and unique.

**Functional requirements:**
1. `POST /shorten` — given a long URL, return a short URL
2. `GET /{code}` — redirect to the original long URL
3. (Optional) Custom aliases, expiry, analytics

**Non-functional requirements:**
- High read-to-write ratio: ~100:1 (people read short URLs far more often than they create)
- Low latency reads: redirect should be < 10 ms p99
- Uniqueness: no two long URLs share the same short code (unless intentionally aliased)
- Durability: the mapping must never be lost

**Capacity estimation (for 100 M URLs/day write rate):**
- 100 M new URLs/day = ~1,160 writes/sec
- 10 B reads/day (100:1) = ~115,000 reads/sec
- Each record: `{short_code, long_url, created_at, expiry, user_id}` ≈ 500 bytes
- Storage: 100 M × 500 bytes = 50 GB/day; 5 years = ~90 TB → sharding needed

**Base62 encoding:**
A 6-character Base62 code (`[a-z, A-Z, 0–9]`) gives 62⁶ ≈ 56 billion unique codes — enough for years at the write rates above.

```
Alphabet: 0-9, A-Z, a-z  (62 chars)
Encode integer → Base62 string:
  while id > 0:
    code = alphabet[id % 62] + code
    id = id // 62
  return code.zfill(6)  # pad to 6 chars
```

**Two shortening strategies:**
1. **ID-based:** Increment a global counter (DB auto-increment). Encode the ID to Base62. Simple and collision-free, but the ID sequence is predictable (enumerable).
2. **Hash-based:** MD5 or SHA-256 the long URL; take the first 6 characters in Base62. Risk: hash collisions (two URLs mapping to the same short code). Resolve with retry (append a suffix and re-hash until unique). Adds complexity.

**Recommendation:** Use ID-based with a distributed ID generator (Snowflake or a Redis counter) to avoid DB write bottleneck from a single auto-increment counter.

**API design:**
```
POST /api/shorten
  Body: { "url": "https://example.com/very-long-path?foo=bar" }
  Response: { "short_url": "https://short.ly/aB3cD9" }

GET /aB3cD9
  Response: HTTP 301 Moved Permanently
            Location: https://example.com/very-long-path?foo=bar
```

**301 vs 302 redirect:**
| | 301 Permanent | 302 Temporary |
|-|--------------|---------------|
| Browser caches | Yes — browser skips the short URL server next time | No — always hits the short URL server |
| Analytics | No — server never sees the request again after first visit | Yes — every redirect is logged |
| Recommendation | Use 302 if you need analytics or may change the mapping | Use 301 if you want to offload traffic |

**Interview talking point:** "For bit.ly, we'd use 302 because analytics (click counts, referrer data) is core to the product. For a static URL in a print ad where the target never changes, 301 is fine to offload the redirect server."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 743 Network Delay Time (target: 15 min)
- **Medium 2:** LC 1514 Path with Maximum Probability (target: 15 min)
- **Hard:** LC 787 Cheapest Flights Within K Stops (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you designed a system where read performance was far more critical than write performance — how did you architect for the read path?
- Leadership principle: Customer Obsession

---

## Flashcards

| Q | A |
|---|---|
| How do you skip stale entries in Dijkstra's algorithm? | When popping `(d, u)` from the heap, check `if d > dist[u]: continue` — a better path to u was already processed; this entry is outdated |
| How does Path with Maximum Probability adapt Dijkstra? | Use a max-heap (negate probabilities for Python's min-heap); multiply probabilities instead of adding distances; update if `current_prob * edge_prob > stored_prob[v]` |
| Why does Dijkstra fail for Cheapest Flights Within K Stops? | Dijkstra marks a node as "settled" on first pop, but a node might be reachable via fewer hops at a higher cost — that path is needed if the K-hop limit hasn't been exhausted. Dijkstra prunes it incorrectly. |
| How does Bellman-Ford enforce the K-stop constraint? | Run exactly K+1 relaxation rounds; copy `prev = dist[:]` before each round so updates only propagate one hop per round — prevents "chaining" multiple hops within a single round |
| When should you use 301 vs 302 for a URL shortener redirect? | 302 (temporary): browser always contacts the server → click analytics captured. 301 (permanent): browser caches the redirect → no analytics after first visit, but lower server load. Use 302 if analytics matter. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/25 min, no hints)
- [ ] Rewrote Dijkstra template and Bellman-Ford K-stops template from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section — can explain Base62 and 301 vs 302 cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
