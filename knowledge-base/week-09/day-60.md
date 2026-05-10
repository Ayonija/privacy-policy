# Day 60 — Graphs & CDN/DNS/HTTP: Slot 6 Full Synthesis
**Week 09 | Phase 1: DSA Mastery | Month 2**

## Focus
Slot 6 close-out: synthesise graph traversal with BFS maze-exit search, DFS reachability, and Dijkstra-style grid navigation. Synthesise the complete CDN/DNS/HTTP decision framework across all 10 days.

---

## DSA (2 hours)
### Pattern: BFS Boundary Exit + DFS Reachability + Dijkstra on Weighted Grid

**Nearest Exit from Entrance in Maze (LC 1926):**
BFS from the entrance. An "exit" is any empty cell (`.`) on the boundary that is NOT the entrance. Standard BFS — the first time a boundary empty cell (not the start) is dequeued, return the current distance.

**Keys and Rooms (LC 841):**
DFS/BFS from room 0. Collect all keys found in each visited room; use them to unlock and visit more rooms. If all rooms are visited → return True.

**Swim in Rising Water (LC 778):**
Grid where `grid[i][j]` = time when cell (i,j) becomes passable. Find the minimum time T such that a path exists from (0,0) to (n-1,n-1) where all cells on the path have `grid[r][c] ≤ T`. Dijkstra-style: use a min-heap with `(max_time_on_path, row, col)`. Greedily pick the cell with the lowest bottleneck time. The answer is the bottleneck value when you first dequeue `(n-1, n-1)`.

**Trigger condition:**
- "shortest path from start to any boundary exit" → BFS; check boundary condition on dequeue
- "can all nodes be visited starting from one source?" → DFS/BFS reachability check; track visited count
- "minimum bottleneck path in a weighted grid" → Dijkstra (min-heap); bottleneck = max edge weight on path

**Time complexity:** LC 1926: O(m×n) | LC 841: O(V+E) | LC 778: O(n² log n)
**Space complexity:** O(m×n) / O(V+E) / O(n²)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Nearest Exit from Entrance in Maze | 1926 | Medium | BFS shortest path to any boundary non-entrance | BFS from entrance; first boundary '.' cell that isn't the entrance = exit |
| 2 | Keys and Rooms | 841 | Medium | DFS/BFS reachability from room 0 | Collect keys as you visit rooms; visit when key is acquired; all rooms visited → True |
| 3 | Swim in Rising Water | 778 | Hard | Dijkstra on grid (min bottleneck path) | Heap stores (max_time_so_far, r, c); pop minimum; first pop of (n-1,n-1) = answer |

---

### Code Skeleton
```python
from collections import deque
import heapq

# Nearest Exit from Entrance in Maze (LC 1926)
def nearestExit(maze, entrance):
    rows, cols = len(maze), len(maze[0])
    er, ec = entrance
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    queue = deque([(er, ec, 0)])
    maze[er][ec] = '+'   # mark entrance visited
    while queue:
        r, c, dist = queue.popleft()
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and maze[nr][nc] == '.':
                # Check if this is a boundary exit (and not the entrance)
                if nr == 0 or nr == rows-1 or nc == 0 or nc == cols-1:
                    return dist + 1
                maze[nr][nc] = '+'   # mark visited
                queue.append((nr, nc, dist + 1))
    return -1

# Keys and Rooms (LC 841)
def canVisitAllRooms(rooms):
    visited = set([0])
    stack = [0]
    while stack:
        room = stack.pop()
        for key in rooms[room]:
            if key not in visited:
                visited.add(key)
                stack.append(key)
    return len(visited) == len(rooms)

# Swim in Rising Water (LC 778) — Dijkstra / min-heap
def swimInWater(grid):
    n = len(grid)
    heap = [(grid[0][0], 0, 0)]   # (max_time_on_path, row, col)
    visited = set()
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    while heap:
        t, r, c = heapq.heappop(heap)
        if (r, c) in visited: continue
        visited.add((r, c))
        if r == n-1 and c == n-1: return t
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < n and 0 <= nc < n and (nr, nc) not in visited:
                heapq.heappush(heap, (max(t, grid[nr][nc]), nr, nc))
    return -1
```

---

### Edge Cases to Trace Before Coding
- LC 1926: entrance is at a corner — it's on the boundary but NOT an exit; entrance surrounded by walls (no paths) → return -1; 1×1 maze → no exit possible (entrance = only cell) → return -1
- LC 841: only room 0 → return True (trivially all rooms visited); room 0 has no keys and there are other rooms → return False
- LC 778: single cell grid → return `grid[0][0]`; path only exists through high-value cells → Dijkstra correctly picks the min-max path

---

### Interview Pattern Drill — Slot 6 Complete Reference

| Pattern | Trigger phrase | Algorithm | Complexity |
|---------|---------------|-----------|-----------|
| Grid DFS flood fill | Connected components, max area | DFS + mutate in-place | O(m×n) |
| Multi-source BFS | Distance from nearest source (0-cells, rotten) | BFS, all sources at t=0 | O(m×n) |
| BFS shortest path | Minimum steps in unweighted graph | BFS | O(V+E) |
| BFS implicit graph | Word ladder, genetic mutation | BFS + generate neighbours | O(V × branch) |
| DFS 3-colour | Cycle detection in directed graph | DFS, GRAY = in stack | O(V+E) |
| Kahn's topological sort | Task ordering, safe nodes | BFS from in-degree 0 | O(V+E) |
| Union-Find | Connected components, cycle detection, bridges | Path compression + rank | O(α(n)) per op |
| Tarjan's bridges | Critical connections | DFS, `low[v] > disc[u]` | O(V+E) |
| BFS 2-colouring | Bipartite check, 2-partition | BFS, alternate colours | O(V+E) |
| Boundary DFS | Enclosed regions, enclaves | DFS from all border cells | O(m×n) |
| State-space BFS | Sliding puzzle, min moves | BFS on encoded state | O(S × branch) |
| Dijkstra on grid | Min bottleneck path, weighted grid | Min-heap | O(n² log n) |
| DFS backtracking (DAG) | All paths source to target | DFS + push/pop | O(2ⁿ × n) |
| Reverse graph + Kahn's | Safe nodes, eventual terminals | Build reverse, Kahn's | O(V+E) |

---

## System Design (1 hour)
### Topic: Full CDN + DNS + HTTP + Real-Time Synthesis — Decision Framework

**Complete decision framework for web infrastructure:**

```
User makes a request to your service?
  │
  ├─ DNS Resolution
  │    ├─ Use Anycast? → Yes for global CDN providers; DNS-based GeoDNS for others
  │    ├─ TTL? → 60s during migrations; 3600s normally
  │    └─ DNSSEC? → Enable for critical domains to prevent cache poisoning
  │
  ├─ Transport Security
  │    ├─ Always HTTPS/TLS 1.3
  │    ├─ HSTS for strict enforcement
  │    └─ TLS terminated at CDN edge (not origin) for performance
  │
  ├─ CDN Strategy
  │    ├─ Static assets (CSS/JS/images) → CDN + URL versioning + 1-year cache
  │    ├─ Public API responses → CDN + short TTL (60–300 s)
  │    ├─ Private / personalised → no CDN caching (Cache-Control: private)
  │    └─ Flash crowd / viral content → Origin Shield essential
  │
  ├─ HTTP Protocol
  │    ├─ HTTP/2 for most services (multiplexing, header compression)
  │    ├─ HTTP/3 for mobile-heavy / unreliable networks (QUIC avoids TCP HOL)
  │    └─ WebSocket / SSE for real-time push (bidirectional vs server-only)
  │
  ├─ Cache Strategy
  │    ├─ Write-through → strong consistency (user data)
  │    ├─ Write-back → high write throughput (counters, analytics)
  │    ├─ Cache-Control + ETag → freshness + conditional requests
  │    └─ CDN purge API or URL versioning → instant or TTL invalidation
  │
  └─ Real-Time Communication
       ├─ Bidirectional (chat, gaming) → WebSocket
       ├─ Server-push only (live feed, notifications) → SSE
       └─ Fallback / simple → Long Polling
```

**The 5 questions to answer cold for any CDN/DNS/HTTP system design:**
1. **Where is content cached?** Browser / CDN edge / Origin Shield — pick the right layers
2. **How fresh must the content be?** Determines TTL length and invalidation strategy
3. **Is the data user-specific?** Yes → `Cache-Control: private`; No → `public`, CDN-cacheable
4. **How is the protocol stack optimised?** HTTPS everywhere; HTTP/2 default; HTTP/3 for mobile
5. **What does real-time communication look like?** Request-response (HTTP), push (SSE), bidirectional (WebSocket)

**Interview synthesis talking point:** "For a news website with 50 M daily readers: DNS via Anycast CDN. Static pages cached at CDN edge with 5-minute TTL + CDN purge on publish. CSS/JS/images with URL versioning at 1-year cache. Breaking news pushed to open browsers via SSE. All traffic on HTTP/2; mobile app uses HTTP/3. A 5-minute TTL means at most 5 minutes of staleness after a news update — acceptable for news but not for financial data."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1926 Nearest Exit (target: 15 min)
- **Medium 2:** LC 841 Keys and Rooms (target: 15 min)
- **Hard:** LC 778 Swim in Rising Water (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Walk through a system you built or improved that touched multiple infrastructure layers (CDN, DNS, HTTP, caching, or real-time) — describe the design decisions and trade-offs you made at each layer.
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does Nearest Exit BFS distinguish between the entrance and a valid exit? | Both may be on the boundary; the entrance cell is marked visited immediately and skipped; any other boundary empty cell encountered during BFS is a valid exit |
| How does Keys and Rooms determine if all rooms are reachable? | DFS/BFS from room 0, collecting keys as rooms are visited; at the end, `len(visited) == len(rooms)` — if not all rooms were unlocked, return False |
| How does Swim in Rising Water use a min-heap differently from standard BFS? | Heap stores (max_cell_value_on_path, row, col); pop the minimum-bottleneck cell; answer = heap value when (n-1, n-1) is first popped — this is Dijkstra for min bottleneck |
| What is the complete 5-question CDN/DNS/HTTP decision checklist? | (1) Where is content cached? (2) How fresh must it be? (3) Is data user-specific? (4) HTTP/1.1 vs 2 vs 3? (5) What real-time pattern (HTTP/SSE/WebSocket)? |
| When choose SSE vs WebSocket for a real-time feature? | SSE: server-push only (live scores, notifications, dashboards) — HTTP-native, auto-reconnect, simpler. WebSocket: bidirectional (chat, collaborative editing, gaming) — lower overhead per message, full-duplex |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/25 min, no hints)
- [ ] Rewrote Union-Find template AND Dijkstra grid template from memory (two hardest patterns)
- [ ] Stated time + space complexity aloud for each solution
- [ ] Recited the Slot 6 pattern reference table trigger conditions from memory
- [ ] Completed system design synthesis — recited the 5-question decision framework cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
