# Day 56 — Graphs: Multi-Source BFS, Reverse BFS & State-Space Search
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Multi-source BFS solves "distance from any source" problems by initialising the queue with all sources simultaneously. Reverse BFS (searching from the boundary inward) avoids exponential-space DFS. Sliding Puzzle demonstrates BFS on a compact state space encoded as a string.

---

## DSA (2 hours)
### Pattern: Multi-Source BFS + Reverse BFS + BFS on State Space

**01 Matrix (LC 542):**
Goal: for each cell, find its distance to the nearest 0. Multi-source BFS: initialise the queue with ALL 0-cells (distance 0). BFS propagates distances outward — each 1-cell's distance is set when it is first reached (guaranteed shortest).

**Pacific Atlantic Water Flow (LC 417):**
Instead of simulating water flow downward (exponential paths), reverse the problem: water can flow uphill from the ocean boundaries inward. BFS/DFS from all Pacific-border cells (top row + left column) uphill; separately from all Atlantic-border cells (bottom row + right column) uphill. The answer = cells reachable from both directions.

**Sliding Puzzle (LC 773):**
The 2×3 board has at most 6! = 720 unique states. Encode each state as a string. BFS from the initial state; neighbours are all boards reachable by one 0-swap. Goal = `"123450"`. Return BFS levels when goal is reached (minimum swaps), or -1 if unreachable.

**Trigger condition:**
- "distance from each cell to the nearest [source]" → multi-source BFS, all sources at level 0
- "find cells reachable by opposite-direction flow from two borders" → reverse BFS from both borders; intersection
- "minimum moves to reach a goal state in a small state space" → BFS on encoded states (string/tuple)

**Time complexity:** LC 542: O(m×n) | LC 417: O(m×n) | LC 773: O(S × 6) where S ≤ 720
**Space complexity:** O(m×n) / O(m×n) / O(S)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | 01 Matrix | 542 | Medium | Multi-source BFS from all 0-cells | Init queue with all zeros at dist 0; BFS propagates distances to all 1s optimally |
| 2 | Pacific Atlantic Water Flow | 417 | Medium | Reverse BFS from both ocean borders | BFS uphill (not downhill) from each border; cells in both reachable sets are the answer |
| 3 | Sliding Puzzle | 773 | Hard | BFS on state space (string-encoded board) | Encode board as string; BFS from initial state; goal = "123450"; visited = set of seen strings |

---

### Code Skeleton
```python
from collections import deque

# 01 Matrix (LC 542)
def updateMatrix(mat):
    rows, cols = len(mat), len(mat[0])
    dist = [[0]*cols for _ in range(rows)]
    queue = deque()
    for r in range(rows):
        for c in range(cols):
            if mat[r][c] == 0:
                queue.append((r, c))
            else:
                dist[r][c] = float('inf')   # 1-cells start as unvisited
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    while queue:
        r, c = queue.popleft()
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and dist[nr][nc] > dist[r][c] + 1:
                dist[nr][nc] = dist[r][c] + 1
                queue.append((nr, nc))
    return dist

# Pacific Atlantic Water Flow (LC 417)
def pacificAtlantic(heights):
    rows, cols = len(heights), len(heights[0])
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    def bfs(starts):
        visited = set(starts)
        queue = deque(starts)
        while queue:
            r, c = queue.popleft()
            for dr, dc in dirs:
                nr, nc = r+dr, c+dc
                if 0 <= nr < rows and 0 <= nc < cols and (nr,nc) not in visited \
                        and heights[nr][nc] >= heights[r][c]:   # water flows uphill in reverse
                    visited.add((nr, nc))
                    queue.append((nr, nc))
        return visited

    pacific  = [(0, c) for c in range(cols)] + [(r, 0) for r in range(rows)]
    atlantic = [(rows-1, c) for c in range(cols)] + [(r, cols-1) for r in range(rows)]

    p_reach = bfs(pacific)
    a_reach = bfs(atlantic)
    return [[r, c] for r in range(rows) for c in range(cols) if (r,c) in p_reach and (r,c) in a_reach]

# Sliding Puzzle (LC 773)
def slidingPuzzle(board):
    # flatten 2×3 board to string; 0 = blank
    start = "".join(str(board[r][c]) for r in range(2) for c in range(3))
    goal  = "123450"
    # neighbours of each position in the flat string (2x3 grid)
    neighbours = {0:[1,3], 1:[0,2,4], 2:[1,5], 3:[0,4], 4:[1,3,5], 5:[2,4]}

    if start == goal: return 0
    queue = deque([(start, 0)])
    visited = {start}
    while queue:
        state, moves = queue.popleft()
        zero = state.index('0')
        for nb in neighbours[zero]:
            new_state = list(state)
            new_state[zero], new_state[nb] = new_state[nb], new_state[zero]
            new_str = "".join(new_state)
            if new_str == goal: return moves + 1
            if new_str not in visited:
                visited.add(new_str)
                queue.append((new_str, moves + 1))
    return -1
```

---

### Edge Cases to Trace Before Coding
- LC 542: all cells are 0 → all distances stay 0; all cells are 1 → impossible by problem constraints (at least one 0 guaranteed)
- LC 417: single cell grid → it touches both Pacific and Atlantic borders → it's in the result
- LC 773: board already solved → return 0; unreachable state (e.g., `[[1,2,3],[5,4,0]]`) → BFS exhausts all states → return -1

---

### Interview Pattern Drill
| Pattern | Queue initialisation | Visited tracking | Stopping condition |
|---------|---------------------|-----------------|-------------------|
| Multi-source BFS | ALL sources at level 0 | Mark on enqueue | When all cells processed |
| Reverse BFS | Border cells of both sources | Separate visited sets | BFS exhausted |
| State-space BFS | Initial state string | Set of seen strings | Goal state dequeued |

---

### STAR Interview Framework

> **Multi-Source BFS:** brute-force O(m×n) per source cell → this approach O(m×n) total time, O(m×n) space

**S:** "Given an m×n grid with multiple 0-source cells. Naive single-source BFS from each 0 is O(sources × m×n) — too slow when sources are dense."
**T:** "Need O(m×n) total by treating all sources as a single level-0 BFS frontier."
**A (60%):**
1. *Classify:* "'Distance from each cell to nearest source' → multi-source BFS. All sources enqueued simultaneously at distance 0."
2. *Init:* "Scan grid; enqueue ALL 0-cells at distance 0; set non-zero cells to `inf`."
3. *Loop/Step:* "Standard BFS; update `dist[nr][nc] = dist[r][c] + 1` only when `dist[nr][nc] > dist[r][c] + 1`."
4. *Termination:* "Each cell's distance is set the first time it is reached — BFS guarantees shortest distance."
5. *Gotcha:* "Mark cells visited on enqueue (not dequeue) — prevents the same cell being enqueued multiple times."
**R:** "O(m×n) time and space. For a 1000×1000 grid with 100K source cells: single pass vs 100K × 1M = 100B ops brute force."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Dijkstra | Weighted grid with varying costs | Unweighted — BFS is O(m×n) vs O(m×n log(m×n)) |
| Single-source BFS per cell | Exactly one source | Correct but O(n² × m×n) for n sources |

---

## System Design (1 hour)
### Topic: CDN + DNS + HTTP — Complete Request Lifecycle Synthesis

**Full lifecycle of `https://api.example.com/feed` from a user's browser:**

```
Step 1 — DNS Resolution (20–50 ms first time, ~0 ms cached)
  Browser cache → OS cache → Recursive resolver (ISP / 8.8.8.8)
  → Root NS → .com TLD NS → Authoritative NS for example.com
  → Returns CDN's Anycast IP (e.g., Cloudflare)

Step 2 — TCP + TLS to nearest CDN PoP (5–20 ms)
  TCP 3-way handshake with nearest PoP
  TLS 1.3 handshake (1 RTT)
  Connection established

Step 3 — HTTP/2 Request to CDN Edge
  GET /feed HTTP/2
  Host: api.example.com
  Authorization: Bearer <token>
  Cache-Control: no-cache   ← forced fresh if user refreshed

Step 4 — CDN Cache Lookup
  Cache HIT (public, cacheable response):
    CDN returns response in < 1 ms
    Sets: Cache-Control: public, max-age=60, s-maxage=300
  Cache MISS:
    CDN forwards request to Origin Shield (if configured)
      → Origin Shield hits origin only if not cached there either
    Origin processes: DB query + computation (~50 ms)
    Origin returns response; CDN caches it

Step 5 — HTTP Response to Browser
  200 OK + JSON body
  Headers: ETag, Cache-Control, Content-Encoding: gzip
  Browser caches per Cache-Control headers

Step 6 — Subsequent Requests
  Cache HIT in browser: 0 ms
  CDN HIT: < 5 ms
  CDN MISS but Origin Shield HIT: ~10 ms
  Full miss to origin: ~100 ms
```

**Where each technology plugs in:**
| Layer | Technology | Responsibility |
|-------|-----------|---------------|
| Name resolution | DNS (Anycast/GeoDNS) | Route to nearest CDN PoP |
| Transport security | TLS 1.3 | Encrypt all data in transit |
| Application protocol | HTTP/2 or HTTP/3 | Multiplex requests |
| Edge caching | CDN PoP | Serve hot content without touching origin |
| Origin protection | Origin Shield | Collapse cache misses to one origin request |
| Cache freshness | Cache-Control, ETag | Define TTL and conditional refresh |
| Cache invalidation | URL versioning / CDN purge API | Instant or TTL-based invalidation |

**5-second interview answer for "what happens when a user opens example.com?":**
"DNS resolves the domain to a CDN PoP IP via Anycast. The browser connects with TLS. The CDN checks its cache — a hit serves the content in milliseconds from the edge; a miss fetches from the origin, caches it, and serves. All subsequent users at the same PoP hit the cache."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Are Right, A Lot

**Full STAR Story — "End-to-End Latency Audit":**
**S (20%):** "At a consumer app company, our P95 page load time was 4.1 seconds — users were abandoning at a 38% rate on first visit. Leadership assumed the database was the bottleneck, but no one had traced the full request lifecycle."
**T:** "I was asked to diagnose and fix the latency issue within two weeks, with a target of sub-2-second P95."
**A (60% — 'I' not 'we'):** "(1) I instrumented the full request path — DNS, TCP handshake, TLS negotiation, CDN hit/miss, server processing, and DB query — using distributed tracing headers at each layer. (2) I found that 62% of latency was in TLS negotiation (1.8s) because we were using RSA-2048 instead of ECDHE — not the DB. (3) I migrated to ECDHE cipher suites and enabled TLS 1.3 session resumption, cutting TLS to 120ms. (4) I also found that static assets had no CDN caching headers — I added `Cache-Control: public, max-age=31536000` with URL versioning, offloading 80% of asset requests to CDN edge."
**R (20%):** "P95 page load dropped from 4.1s to 1.4s — 66% improvement. Abandonment rate fell from 38% to 21%. CDN offloading reduced origin bandwidth by 73%, saving $8,400/month."
*Works for: Are Right, A Lot, Dive Deep, Customer Obsession.*

---

## Flashcards

| Q | A |
|---|---|
| How does 01 Matrix's multi-source BFS initialise differently from single-source BFS? | ALL zero-cells are enqueued simultaneously at distance 0; BFS propagates outward — each 1-cell's distance is set the first time it is reached from any zero |
| How does Pacific Atlantic Water Flow avoid simulating water flow downhill? | Reverse the problem: BFS uphill (neighbour height ≥ current height) from Pacific-border cells and Atlantic-border cells separately; cells reachable from both are the answer |
| Why can the Sliding Puzzle state space use BFS effectively? | The state space is finite and small (6! = 720 unique boards); encoding the board as a string makes state comparison O(6) and visited set O(1) per lookup |
| What is an Origin Shield in a CDN architecture? | A mid-tier CDN layer between edge PoPs and the origin server; PoP cache misses go to the Origin Shield first — if it has the content, only one origin request is needed regardless of how many PoPs missed |
| What are the 3 places a response can be cached in the CDN lifecycle? | (1) Browser cache (private, per-user). (2) CDN PoP edge cache (shared, geographic). (3) Origin Shield cache (shared, central) |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section (synthesis — trace the full lifecycle aloud)
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
