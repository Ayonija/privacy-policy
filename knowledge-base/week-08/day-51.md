# Day 51 — Graphs: DFS on Grids & Connected Components
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Introduce graph DFS on 2-D grids — the most common graph representation in interviews — and build to the hard problem of merging islands via Union-Find with area tracking.

---

## DSA (2 hours)
### Pattern: Grid DFS (Flood Fill) + Union-Find Island Merging

**Number of Islands (LC 200):**
Iterate every cell. When you find an unvisited land cell (`'1'`), increment the island count and launch a DFS/BFS that marks all connected land cells as visited (sink them to `'0'` or use a separate `visited` set). Four-directional movement.

**Max Area of Island (LC 695):**
Same DFS structure. Instead of counting islands, accumulate the number of cells visited in the current DFS call. Track the global maximum area.

**Making A Large Island (LC 827):**
Two-phase approach:
1. Label each existing island with a unique ID (2, 3, 4, …) and record its size in a `{id: size}` map using DFS.
2. For each water cell (`0`), collect the distinct island IDs among its 4 neighbours, sum their sizes + 1 (for the water cell being converted), and update the global maximum.

If there are no water cells at all, the answer is the total land area.

**Trigger condition:**
- "count/explore connected regions in a grid" → DFS/BFS from each unvisited land cell; mark visited inline
- "area of largest connected region" → DFS returning cell count; track global max
- "largest island after flipping one cell" → label islands with DFS, then probe each water cell

**Time complexity:** O(m×n) for all | **Space complexity:** O(m×n) worst case for DFS stack on a full grid

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Number of Islands | 200 | Medium | Grid DFS flood fill | Sink each land cell during DFS to avoid revisiting; iterate entire grid counting DFS launches |
| 2 | Max Area of Island | 695 | Medium | Grid DFS with area accumulation | DFS returns the count of cells visited in current component; track global max |
| 3 | Making A Large Island | 827 | Hard | Two-phase: DFS label + water cell probe | Label islands with unique IDs + size map; for each 0, sum adjacent distinct island sizes + 1 |

---

### Code Skeleton
```python
dirs = [(0,1),(0,-1),(1,0),(-1,0)]

# Number of Islands (LC 200)
def numIslands(grid):
    rows, cols = len(grid), len(grid[0])
    count = 0
    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1': return
        grid[r][c] = '0'   # sink to mark visited
        for dr, dc in dirs:
            dfs(r + dr, c + dc)
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                dfs(r, c)
                count += 1
    return count

# Max Area of Island (LC 695)
def maxAreaOfIsland(grid):
    rows, cols = len(grid), len(grid[0])
    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != 1: return 0
        grid[r][c] = 0
        return 1 + sum(dfs(r+dr, c+dc) for dr, dc in dirs)
    return max((dfs(r, c) for r in range(rows) for c in range(cols)), default=0)

# Making A Large Island (LC 827)
def largestIsland(grid):
    n = len(grid)
    island_size = {}   # id → size
    island_id = 2      # start labeling from 2 (0 = water, 1 = unlabeled land)

    def dfs(r, c, iid):
        if r < 0 or r >= n or c < 0 or c >= n or grid[r][c] != 1: return 0
        grid[r][c] = iid
        return 1 + sum(dfs(r+dr, c+dc, iid) for dr, dc in dirs)

    for r in range(n):
        for c in range(n):
            if grid[r][c] == 1:
                island_size[island_id] = dfs(r, c, island_id)
                island_id += 1

    result = max(island_size.values(), default=0)  # all land, no water cells

    for r in range(n):
        for c in range(n):
            if grid[r][c] == 0:
                seen = set()
                size = 1   # the flipped cell
                for dr, dc in dirs:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < n and 0 <= nc < n and grid[nr][nc] > 1:
                        iid = grid[nr][nc]
                        if iid not in seen:
                            size += island_size[iid]
                            seen.add(iid)
                result = max(result, size)
    return result
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Grid DFS / Flood Fill patterns in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a 2-D grid of 1s (land) and 0s (water) and asked to count the number of distinct islands. The grid can be up to 300×300 = 90,000 cells. A brute-force approach of checking every cell against every other cell for connectivity is O(n⁴) — completely intractable."

**Task:** "My goal was to count connected components of land cells in O(m×n) time using DFS flood fill — visiting each cell at most once."

**Action:** Walk the interviewer through these steps:
1. *Outer loop — scan every cell:* "I iterate every `(r, c)` in the grid. When I find a land cell (`grid[r][c] == '1'`), I increment the island count and launch a DFS from that cell."
2. *DFS — sink and recurse:* "On entry to `dfs(r, c)`: first I check bounds and that the cell is still land. Then I mutate `grid[r][c] = '0'` — 'sinking' the cell to mark it visited. Then I recurse in all 4 directions."
3. *Why mutate in-place:* "Mutating the grid to '0' on entry is equivalent to a visited set but uses zero extra memory. Each cell can only be sunk once — it's O(m×n) total work across all DFS calls."
4. *Count:* "Each DFS launch from the outer loop corresponds to one island. The count of DFS launches is the number of islands."
5. *Four-directional movement:* "I use `dirs = [(0,1),(0,-1),(1,0),(-1,0)]` — horizontal and vertical only. Diagonal connectivity would require 8 directions."

**Result:** "O(m×n) time — each cell is visited at most twice (once to discover it, once to sink it). O(m×n) space in the worst case for the recursion stack on a fully-land grid. For a 300×300 grid: 90,000 operations instead of 8.1 billion for brute force."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer DFS flood fill |
|-------------|---------------------------|--------------------------|
| BFS flood fill | Stack overflow risk for very deep grids | Equivalent O(m×n); BFS uses a queue (no stack depth issue); trade-off is slightly more code |
| Union-Find | When you need to query connectivity dynamically | More complex; O(α(n)) per operation; overkill for static grid counting |
| DFS flood fill | Standard | O(m×n) time, zero extra space for visited tracking |

**Why mutate in-place rather than using a separate visited set:** A `visited` set adds O(m×n) extra memory and O(1) lookup overhead per cell. Mutating in-place achieves the same O(1) lookup with zero extra space — the grid itself becomes the visited marker.

---

### Edge Cases to Trace Before Coding
- LC 200: grid with all water → 0; grid with all land → 1 island
- LC 695: no land cells → return 0 (use `default=0` in max())
- LC 827: no water cells → return total land area (handle before water-cell probe loop); single cell grid `[[0]]` → return 1 (flip the only cell)

---

### Interview Pattern Drill
| Graph representation | Adjacency structure | Visited tracking |
|---------------------|--------------------|--------------------|
| Grid (m×n) | 4 directions: `[(0,1),(0,-1),(1,0),(-1,0)]` | Mutate cell in-place or use `visited` set |
| Adjacency list | `graph[u]` = list of neighbours | `visited` set of nodes |
| Implicit (word graph) | Generate neighbours on-the-fly | `visited` set of states |

---

## System Design (1 hour)
### Topic: CDN Fundamentals — Why, What, and How

**What is a CDN?**
A Content Delivery Network is a globally distributed network of edge servers (Points of Presence — PoPs) that cache content close to end users. Without a CDN, every request travels from the user's browser to the origin server — potentially crossing continents.

**Latency reduction (why CDNs exist):**
```
No CDN:   User in Tokyo → Origin in US East → ~180ms round-trip
With CDN: User in Tokyo → PoP in Tokyo → ~2ms round-trip (cache hit)
```
CDNs typically have 50–200+ PoPs worldwide (Cloudflare: 300+, AWS CloudFront: 400+).

**What CDNs cache:**
- Static assets: HTML, CSS, JS, images, videos, fonts
- API responses (with appropriate Cache-Control headers)
- Streaming video segments (HLS/DASH chunks)
- NOT: authenticated user-specific data, real-time data

**How requests are routed to the nearest PoP:**
1. **DNS-based routing:** CDN's DNS returns the IP of the nearest PoP based on the requester's IP geolocation. Simple but limited by DNS caching.
2. **Anycast routing:** Multiple PoPs share the same IP address; BGP routing protocol delivers packets to the nearest one. Used by Cloudflare — no DNS TTL issues.

**CDN cache fill flow:**
```
1. User requests resource → hits nearest PoP
2. PoP checks local cache
   - Cache HIT  → return cached response (sub-millisecond)
   - Cache MISS → PoP fetches from origin (or Origin Shield)
3. PoP caches the response (respecting Cache-Control headers)
4. Subsequent users at same PoP get cache hit
```

**Static vs. dynamic content:**
| Content type | Cacheable? | Typical TTL |
|-------------|-----------|-------------|
| Images, fonts | Yes | 1 day – 1 year |
| HTML pages | Partially | 0 – 5 min |
| JS/CSS with version hash | Yes | 1 year (immutable) |
| Personalised API responses | No | — |
| Real-time data | No | — |

**Interview talking point:** "If asked how a video streaming service (like YouTube) serves 1 billion daily video views, answer: pre-position video chunks at CDN PoPs globally. Each video is encoded into multiple segments (HLS); segments are cached at edge nodes close to viewers. The CDN absorbs 95 %+ of traffic — only cache misses reach the origin. The origin only deals with uploads, transcoding, and metadata — not playback bandwidth."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Invent and Simplify**

**STAR Story — Reducing latency by moving computation closer to where it was needed**

**Situation:** Our mobile app displayed a personalised product feed that required fetching catalogue metadata, pricing, and inventory status — all from separate microservices located in our US-East datacenter. Mobile users in Europe and Asia-Pacific were experiencing 2.8–4.2 second feed load times due to 12–18 round-trip API calls, each crossing the Atlantic or Pacific (180–280 ms RTT per round trip).

**Task:** I was the mobile backend lead and was asked to reduce feed load time to under 1 second for international users without adding engineering complexity to the mobile clients or the microservices themselves.

**Action:** I analysed the API call pattern: 80% of the data in the feed (catalogue metadata, category information, non-personalised content) was identical across users and changed at most a few times per day. Only inventory status and personalised rankings were user-specific. I proposed a two-layer solution: first, deploy a GraphQL gateway with a feed aggregation endpoint to the CDN edge (Cloudflare Workers) — this reduced 12 round trips to 1 round trip by doing fan-out at the edge, close to the user. Second, I implemented fragment-level caching at the edge: catalogue metadata for each product was cached at the edge with a 1-hour TTL, tagged with `product:{id}` for instant purge on price changes. Only the personalised ranking layer made a round trip to the US-East origin. I built this using Cloudflare Workers KV for edge caching and Cloudflare D1 for lightweight edge-side data. I measured the improvement using synthetic monitoring from Frankfurt, Tokyo, and São Paulo at 15-minute intervals.

**Result:** Feed load time dropped from 2.8–4.2 seconds to 650–820 ms for European and Asia-Pacific users — a 4× improvement. Edge cache hit rate for catalogue fragments was 91% at peak. Origin requests from the CDN dropped by 78%, reducing our US-East origin load. Mobile session length increased by 22% for international users in the following month, which the product team attributed directly to the latency improvement.

---

## Flashcards

| Q | A |
|---|---|
| How do you count connected components in a grid using DFS? | Iterate all cells; for each unvisited land cell, launch DFS to mark all connected land cells visited; increment count once per DFS launch |
| How does DFS grid traversal avoid revisiting cells efficiently? | Mutate the cell in-place (set `grid[r][c] = 0`) immediately on entry — no separate visited set needed; saves O(m×n) space |
| How does Making A Large Island probe each water cell? | Collect the set of distinct island IDs among the 4 neighbours (dedup with a set); sum their sizes from the island_size map, add 1 for the flipped cell |
| What is a CDN PoP and how does request routing reach it? | PoP = Point of Presence (edge datacenter); routing via DNS geolocation (CDN returns nearest PoP's IP) or Anycast (multiple PoPs share one IP; BGP delivers to nearest) |
| What is the CDN cache miss flow? | PoP checks cache → miss → PoP fetches from Origin Shield (if present) or origin server → caches the response per Cache-Control headers → returns to user |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
