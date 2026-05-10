# Day 62 — Graphs Advanced: Dijkstra on Grids & Priority Queue BFS
**Week 09 | Phase 1: DSA Mastery | Month 3**

## Focus
Path With Minimum Effort extends Dijkstra to grids where edge weight = height difference between adjacent cells (minimax path). Minimum Score of a Path uses Union-Find to find the minimum score reachable between two cities. Trapping Rain Water II uses a min-heap boundary BFS: water level at any interior cell is bounded by the minimum boundary cell that encloses it.

---

## DSA (2 hours)
### Pattern: Grid Dijkstra (min bottleneck) + Union-Find reachability + Priority Queue Boundary BFS

**Path With Minimum Effort (LC 1631):**
Move through a grid; effort = max absolute height difference along the path. Minimise the effort of any path from top-left to bottom-right. This is a minimax path problem — identical structure to Swim in Rising Water (Day 60) but with `|grid[r][c] - grid[nr][nc]|` as the edge weight. Dijkstra: heap stores `(max_effort_so_far, row, col)`. Update if `max(cur_effort, abs_diff) < dist[nr][nc]`.

**Minimum Score of a Path Between Two Cities (LC 2492):**
Find the minimum weight edge on ANY path between city 1 and city n (not the minimum cost path — any path is allowed). Key insight: any cycle is reachable. If 1 and n are in the same connected component, the minimum edge weight in that component is the answer. Use Union-Find to determine the component of node 1, then find the minimum edge weight among all edges in that component.

**Trapping Rain Water II (LC 407):**
3D version of Trapping Rain Water. Water level at each interior cell is the minimum of the surrounding boundary that traps it. Algorithm: push all boundary cells into a min-heap. Process cells in height order (lowest first). For each popped cell, visit its 4 neighbours — the water level at a neighbour is `max(current_cell_height, neighbour_height)`. If not visited, compute trapped water = `max(0, heap_water_level - neighbour_height)`. Push neighbour with `max(current_water_level, neighbour_height)`.

**Trigger condition:**
- "minimum of the maximum edge weight along a path" → Dijkstra; heap = `(max_weight_on_path, node)`, update = `max(cur, edge_weight) < dist[v]`
- "minimum edge on ANY path (not minimum-cost path)" → Union-Find to find component of source; scan all edges in component for minimum weight
- "how much water can be trapped in a 3D elevation grid" → min-heap boundary BFS (Prim's-like); water level propagates inward from lowest boundary

**Time complexity:** LC 1631: O(m×n log(m×n)) | LC 2492: O(E × α(n)) | LC 407: O(m×n log(m×n))
**Space complexity:** O(m×n) / O(V+E) / O(m×n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Path With Minimum Effort | 1631 | Medium | Grid Dijkstra (min of max edge) | `max(cur_effort, abs(h1-h2))` on each step; update if less than stored effort |
| 2 | Minimum Score of Path | 2492 | Medium | Union-Find; min edge in component | ANY path allowed; find 1's component; min edge weight within it |
| 3 | Trapping Rain Water II | 407 | Hard | Min-heap boundary BFS (3D trapping) | Push all boundary cells; process lowest first; water level = max(incoming, neighbour height) |

---

### Code Skeleton
```python
import heapq
from collections import defaultdict

# Path With Minimum Effort (LC 1631)
def minimumEffortPath(heights):
    rows, cols = len(heights), len(heights[0])
    effort = [[float('inf')] * cols for _ in range(rows)]
    effort[0][0] = 0
    heap = [(0, 0, 0)]   # (max_effort, row, col)
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    while heap:
        e, r, c = heapq.heappop(heap)
        if e > effort[r][c]: continue
        if r == rows-1 and c == cols-1: return e
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols:
                new_e = max(e, abs(heights[r][c] - heights[nr][nc]))
                if new_e < effort[nr][nc]:
                    effort[nr][nc] = new_e
                    heapq.heappush(heap, (new_e, nr, nc))
    return effort[rows-1][cols-1]

# Minimum Score of Path (LC 2492) — Union-Find
def minScore(n, roads):
    parent = list(range(n + 1))
    rank = [0] * (n + 1)
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
    for u, v, _ in roads:
        union(u, v)
    root = find(1)
    ans = float('inf')
    for u, v, d in roads:
        if find(u) == root:   # edge is in 1's component
            ans = min(ans, d)
    return ans

# Trapping Rain Water II (LC 407)
def trapRainWater(heightMap):
    if not heightMap or len(heightMap) < 3 or len(heightMap[0]) < 3:
        return 0
    rows, cols = len(heightMap), len(heightMap[0])
    visited = [[False] * cols for _ in range(rows)]
    heap = []
    # Push all boundary cells
    for r in range(rows):
        for c in range(cols):
            if r == 0 or r == rows-1 or c == 0 or c == cols-1:
                heapq.heappush(heap, (heightMap[r][c], r, c))
                visited[r][c] = True
    water = 0
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    while heap:
        level, r, c = heapq.heappop(heap)
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and not visited[nr][nc]:
                visited[nr][nc] = True
                water += max(0, level - heightMap[nr][nc])
                heapq.heappush(heap, (max(level, heightMap[nr][nc]), nr, nc))
    return water
```

---

### Edge Cases to Trace Before Coding
- LC 1631: single cell → return 0; grid where all heights equal → return 0; only one path exists (linear grid) → Dijkstra still works correctly
- LC 2492: node 1 and node n in different components → problem guarantees path exists; minimum edge = the overall minimum edge in 1's component (could be any edge, not necessarily on the shortest path)
- LC 407: 1×n or m×1 grid → all boundary → return 0; grid where interior cells are all mountains taller than boundary → no water trapped

---

### Interview Pattern Drill

| Problem type | Heap content | Update condition | Invariant |
|-------------|-------------|-----------------|-----------|
| Min-cost path (Dijkstra) | `(cost, node)` | `cost + w < dist[v]` | Cumulative sum |
| Min-bottleneck path | `(max_edge, node)` | `max(cur, edge) < dist[v]` | Maximum on path |
| Max-probability path | `(-prob, node)` | `cur * edge_p > prob[v]` | Product on path |
| 3D water trapping | `(water_level, r, c)` | Always process lowest boundary first | Water level = max(incoming, cell height) |

---

## System Design (1 hour)
### Topic: URL Shortener — Database Schema, Redirect Flow, and Write Path

**Database schema:**
```sql
CREATE TABLE urls (
    id          BIGINT PRIMARY KEY,          -- auto-increment (or Snowflake ID)
    short_code  VARCHAR(8) UNIQUE NOT NULL,  -- Base62-encoded ID
    long_url    TEXT NOT NULL,
    user_id     BIGINT,
    created_at  TIMESTAMP DEFAULT NOW(),
    expires_at  TIMESTAMP,                   -- NULL = never expires
    click_count BIGINT DEFAULT 0
);
CREATE INDEX idx_short_code ON urls(short_code);
```

Why `id` as primary key and `short_code` as unique index:
- Writes: INSERT by auto-increment id → short_code derived from id after insert (or use Snowflake ID before insert)
- Reads: lookup by `short_code` → single index scan → O(log n)
- Click counts: UPDATE with separate analytics table for high-write counts to avoid locking the main table

**Write path (shorten a URL):**
```
Client → Load Balancer → App Server
  1. App server receives long URL
  2. (Optional) check if long URL already shortened → lookup by long_url → return existing code
  3. Generate new Snowflake ID (or increment Redis counter)
  4. Base62-encode the ID → short_code
  5. INSERT (id, short_code, long_url, user_id, created_at) into DB
  6. Write {short_code → long_url} to Redis cache (TTL = 24h or forever for popular URLs)
  7. Return short URL to client
```

**Read path (redirect):**
```
Client → Load Balancer → App Server
  1. App server extracts short_code from URL path
  2. Check Redis cache: if hit → 302 redirect immediately (< 1 ms)
  3. Cache miss → query DB by short_code index
  4. If found: write to cache, return 302 redirect
  5. If not found: return 404
  6. (Async) increment click_count in analytics table
```

**Why Redis for caching:**
- Redirect is a pure key-value lookup: `{short_code → long_url}`
- Redis GET: sub-millisecond
- With cache, 99 %+ of reads never touch the DB

**Handling custom aliases:**
- User supplies a desired short code (e.g., "my-product")
- Check if short_code already exists in DB → if yes, reject or suggest alternative
- If no, insert with the provided short_code (bypass the ID→Base62 encoding step)

**Expiry handling:**
- Set `expires_at` on the record
- On read: check `expires_at` in app layer; if expired → return 410 Gone
- Background job: batch-DELETE expired records nightly to keep table size bounded

**Interview talking point:** "The write path has one DB write and one Redis write per new URL. The read path hits only Redis for cached entries — zero DB reads for popular URLs. The ratio 100:1 reads to writes means caching is high-leverage: even a 50 % cache hit rate halves DB read load; at 95 % cache hit rate, the DB handles only 1 in 20 reads."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1631 Path With Minimum Effort (target: 18 min)
- **Medium 2:** LC 2492 Minimum Score of Path (target: 15 min)
- **Hard:** LC 407 Trapping Rain Water II (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to design a data storage schema under uncertain scaling requirements — how did you balance simplicity now vs future growth?
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| How does Path With Minimum Effort differ from Network Delay Time in the Dijkstra update? | Network Delay: `dist[u] + w < dist[v]` (sum). Minimum Effort: `max(cur_effort, abs(h1-h2)) < effort[v]` (minimax — track the maximum height diff on the path, not the sum) |
| When should you use Union-Find instead of Dijkstra for a path problem? | When the question asks about connectivity or the minimum/maximum of ANY edge on any path (not the minimum cost path), Union-Find is simpler: find the component of the source, scan all edges in it |
| What is the key invariant in the Trapping Rain Water II heap approach? | Always process the lowest boundary cell first (min-heap). Water level at any interior cell = max(boundary cell that enclosed it, cell's own height). The heap ensures we process "outer walls" before inner cells |
| What is the URL shortener write path? | 1) Receive long URL 2) Generate Snowflake/counter ID 3) Base62-encode → short_code 4) INSERT to DB 5) Write to Redis cache 6) Return short URL |
| Why is Redis GET sufficient for URL shortener reads, and what is the fallback? | Redis stores `{short_code → long_url}` as a string KV; GET is O(1) sub-millisecond. Cache miss → DB query by `idx_short_code` index → write back to cache for future requests |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/15/25 min, no hints)
- [ ] Rewrote Path With Minimum Effort and Trapping Rain Water II skeletons from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw the write path and read path cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
