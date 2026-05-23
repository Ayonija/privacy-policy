# Day 59 — Graphs: Border DFS, Enclave Counting & Greedy Swaps
**Week 09 | Phase 1: DSA Mastery | Month 2**

## Focus
Surrounded Regions and Number of Enclaves share a pattern: DFS/BFS from the boundary to mark unreachable cells, then count or flip what remains. Couples Holding Hands applies Union-Find to a greedy pairing problem where the minimum swap count derives directly from component structure.

---

## DSA (2 hours)
### Pattern: Boundary-DFS Exclusion + Union-Find Greedy Component Analysis

**Surrounded Regions (LC 130):**
Any 'O' cell connected to the border cannot be captured. Instead of flooding from interior 'O' cells, reverse the problem: DFS from all border 'O' cells, mark reachable 'O's as safe (e.g., 'S'). After all border DFS, flip all remaining 'O' (surrounded) to 'X'; flip 'S' back to 'O'.

**Number of Enclaves (LC 1020):**
An "enclave" is a land cell (1) that cannot reach the grid boundary. Same reverse pattern: DFS/BFS from all boundary land cells, marking reachable land. Count all remaining unvisited land cells (true enclaves).

**Couples Holding Hands (LC 765):**
`n` couples (numbered 0–n-1) sit in 2n seats. Seat pair [2i, 2i+1] must hold a couple. Union-Find: for each adjacent seat pair (2i, 2i+1), union the couple IDs of the two people sitting there (`person // 2`). The minimum swaps = n/2 − (number of connected components formed). Each component of k couples needs k-1 swaps to arrange correctly.

**Trigger condition:**
- "which interior regions are enclosed / which cells can't reach the boundary?" → DFS from ALL border cells to mark reachable; count/flip what's unmarked
- "minimum swaps to pair elements in adjacent slots" → Union-Find on couple IDs; swaps = total_pairs − components

**Time complexity:** LC 130/1020: O(m×n) | LC 765: O(n × α(n))
**Space complexity:** O(m×n) for DFS stack / O(n) for Union-Find

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Surrounded Regions | 130 | Medium | Boundary DFS + flip remaining | DFS from all border 'O' cells to mark safe; flip remaining 'O' → 'X'; restore safe 'O' |
| 2 | Number of Enclaves | 1020 | Medium | Boundary DFS + count unmarked | DFS from all boundary 1-cells; count land cells that were never reached |
| 3 | Couples Holding Hands | 765 | Hard | Union-Find on couple IDs → component count | Union seat-pair couple IDs; min swaps = n/2 − number of components |

---

### Code Skeleton
```python
# Surrounded Regions (LC 130)
def solve(board):
    if not board: return
    rows, cols = len(board), len(board[0])
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != 'O': return
        board[r][c] = 'S'   # mark as safe
        for dr, dc in dirs:
            dfs(r+dr, c+dc)

    # Mark border-connected O cells as safe
    for r in range(rows):
        dfs(r, 0); dfs(r, cols-1)
    for c in range(cols):
        dfs(0, c); dfs(rows-1, c)

    # Flip: remaining O → X (surrounded); S → O (restore safe)
    for r in range(rows):
        for c in range(cols):
            if board[r][c] == 'O':   board[r][c] = 'X'
            elif board[r][c] == 'S': board[r][c] = 'O'

# Number of Enclaves (LC 1020)
def numEnclaves(grid):
    rows, cols = len(grid), len(grid[0])
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != 1: return
        grid[r][c] = 0   # mark visited by sinking
        for dr, dc in dirs:
            dfs(r+dr, c+dc)

    for r in range(rows):
        dfs(r, 0); dfs(r, cols-1)
    for c in range(cols):
        dfs(0, c); dfs(rows-1, c)

    return sum(grid[r][c] for r in range(rows) for c in range(cols))

# Couples Holding Hands (LC 765)
def minSwapsCouples(row):
    n = len(row) // 2   # number of couple pairs
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

    components = n   # start with n components
    for i in range(0, 2 * n, 2):
        c1, c2 = row[i] // 2, row[i+1] // 2   # couple IDs for the two people in this seat pair
        if find(c1) != find(c2):
            union(c1, c2)
            components -= 1   # merging two components into one

    return n - components
```

---

### Edge Cases to Trace Before Coding
- LC 130: grid with no 'O' → no change; all 'O' with border cells → none are captured (all are safe); single-row or single-column grid → all 'O' are on the border → none captured
- LC 1020: grid with no land cells → 0 enclaves; grid where all land is connected to boundary → 0 enclaves
- LC 765: already correctly arranged → 0 swaps; n=1 couple, 2 seats → if not together, 1 swap; couple IDs: person 0,1 = couple 0; person 2,3 = couple 1; etc.

---

### Interview Pattern Drill
| Pattern | Border initialisation | What to count/flip |
|---------|----------------------|-------------------|
| Surrounded Regions | All border 'O' cells | Remaining 'O' → 'X'; safe 'S' → 'O' |
| Number of Enclaves | All border land cells | Remaining land cell count |
| Island count (standard) | All unvisited land cells | Number of DFS launches |

---

### STAR Interview Framework

> **Boundary DFS Exclusion (Surrounded Regions / Enclaves):** brute-force flood-fill from each interior cell O(m²×n²) → this approach O(m×n) time, O(m×n) space

**S:** "Given an m×n grid with 'O' cells, identify which interior 'O' cells are fully surrounded by 'X'. Naive approach: flood-fill from each 'O' inward — O(m²×n²) for dense grids."
**T:** "Need O(m×n) by reversing the problem: flood from the border inward."
**A (60%):**
1. *Classify:* "'Which interior cells cannot reach the boundary?' → reverse: DFS from ALL border cells to mark what CAN reach the boundary; everything else is enclosed."
2. *Init:* "Mark all border 'O' cells as safe ('S'); seed DFS from each."
3. *Loop/Step:* "DFS propagates 'S' marking inward — all 'O' cells connected to a border 'O' become 'S'."
4. *Termination:* "After all border DFS: remaining 'O' cells are truly surrounded → flip to 'X'; 'S' → restore to 'O'."
5. *Gotcha:* "Don't DFS from interior 'O' cells — only border cells. Interior-to-border tracking is the slow O(m²×n²) approach."
**R:** "O(m×n) time, O(m×n) space (DFS stack). Single grid scan after DFS. Handles all edge cases: single-row/column grids have all cells on border → no captures."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Interior flood-fill | Tiny grids, simple correctness check | O(m²×n²) for dense grids |
| Union-Find with virtual border node | Need component counts | DFS is simpler and same complexity |

---

## System Design (1 hour)
### Topic: WebSockets, Long Polling & Server-Sent Events — Real-Time Communication Patterns

**The problem with HTTP request-response:**
HTTP is inherently pull-based — the client asks, the server answers, the connection closes. For real-time features (chat, live score, collaborative edit, live auction), the server needs to PUSH data to the client without waiting for the client to ask.

**Long Polling:**
Client sends HTTP request → server holds the response open until data is available (or timeout) → server responds → client immediately sends the next request.
```
Client: GET /updates (connection held open ~30s)
Server: [30s later] 200 OK + new data
Client: GET /updates (immediately re-opens)
```
- Pros: works over standard HTTP, no special infrastructure
- Cons: one TCP connection per client per "poll cycle"; high overhead for frequent updates; server holds many open connections

**WebSocket:**
Upgraded TCP connection. Both parties can send messages at any time after the initial HTTP upgrade handshake.
```
Client → Server: GET /ws HTTP/1.1
                 Upgrade: websocket
                 Connection: Upgrade
Server → Client: 101 Switching Protocols
[connection now bidirectional, persistent]
Client → Server: {"type": "message", "text": "hi"}
Server → Client: {"type": "message", "text": "reply"}
```
- Pros: true bidirectional; low overhead per message (small framing header); real-time
- Cons: requires WebSocket-aware load balancers (sticky sessions or pub/sub for horizontal scaling); not natively cacheable; firewall/proxy issues on port 80/443 with aggressive proxies

**Server-Sent Events (SSE):**
One-directional: server pushes to client over a persistent HTTP connection. Client uses the `EventSource` API.
```javascript
const es = new EventSource('/api/events');
es.onmessage = (e) => console.log(e.data);
```
- Pros: HTTP-based (CDN + proxy friendly); automatic reconnection; simpler than WebSocket
- Cons: server-to-client only (client still uses regular HTTP for writes); HTTP/1.1 limits to 6 SSE connections per domain (HTTP/2 removes this limit)

**Decision framework:**
| Use case | Pattern | Why |
|---------|---------|-----|
| Chat / gaming (bidirectional) | WebSocket | Full-duplex needed |
| Live dashboard, notifications (server push) | SSE | Simpler, HTTP-compatible, auto-reconnect |
| Fallback for environments blocking WebSocket | Long Polling | Works on any HTTP/1.1 |
| High-frequency real-time (collaborative editing) | WebSocket | Sub-100ms latency |

**Horizontal scaling of WebSockets:**
WebSocket is stateful — a specific connection is held to a specific server. For horizontal scaling:
- Use a pub/sub system (Redis Pub/Sub, Kafka) so any server can publish to any connection
- Server receiving a WebSocket message publishes to Redis; all servers subscribe and push to their local connections matching the target user

**Interview talking point:** "For a live sports score app with 1 M concurrent viewers, SSE is the right choice — the server only pushes score updates (no client messages). SSE is HTTP-based, so CDNs and load balancers handle it natively. The score feed server publishes updates to Redis Pub/Sub; all app servers subscribe and push to their connected SSE clients. WebSocket would be overkill and harder to scale."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Are Right, A Lot

**Full STAR Story — "Choosing SSE Over WebSocket":**
**S (20%):** "At a sports data company, we were building live score updates for 1.2M concurrent users during peak events. Engineering leadership pushed for a WebSocket implementation because it felt 'more real-time.' I believed SSE was the right tool."
**T:** "I needed to advocate for the simpler SSE approach with data, and deliver a scalable real-time score feed within 3 weeks."
**A (60% — 'I' not 'we'):** "(1) I ran a load test comparing SSE and WebSocket for unidirectional server-push at 100K concurrent connections — SSE had 18% lower memory overhead per connection and required no sticky session configuration. (2) I presented the data showing WebSocket required a pub/sub layer (Redis) plus custom session affinity, adding 2 weeks of infrastructure work and an ongoing ops burden — SSE worked natively with our existing HTTP/2 load balancer. (3) I built the SSE prototype in 4 days: a score-update publisher pushed events to Redis Pub/Sub; all app servers subscribed and forwarded events to their local SSE clients. (4) I ran a 72-hour soak test at 200K simulated concurrent connections with zero dropped events."
**R (20%):** "Launched on time. P99 score update latency: 87ms under 1.2M concurrent connections. Infrastructure cost was 40% less than the WebSocket estimate. The pattern was adopted for two other real-time features."
*Works for: Are Right, A Lot, Invent and Simplify, Frugality.*

---

## Flashcards

| Q | A |
|---|---|
| How does Surrounded Regions avoid incorrectly capturing border-connected 'O' cells? | Reverse the problem: DFS from all border 'O' cells and mark connected 'O' as safe ('S'); only the remaining unvisited 'O' cells (truly interior and surrounded) are flipped to 'X' |
| How does Number of Enclaves differ from Number of Islands? | Enclaves are land cells that CANNOT reach the boundary; start DFS from all boundary land cells to mark reachable land; remaining unvisited land cells are the enclaves |
| How does the Union-Find approach derive minimum swaps for Couples Holding Hands? | Union the couple IDs of each adjacent seat pair; a component of k couples requires k-1 swaps; total swaps = total_pairs - number_of_components |
| What is the key difference between WebSocket and Server-Sent Events (SSE)? | WebSocket: bidirectional persistent connection — both client and server can send at any time. SSE: server-to-client only — client sends regular HTTP requests to write, uses SSE to receive server pushes |
| How do you horizontally scale WebSocket servers? | Use a pub/sub broker (Redis Pub/Sub or Kafka) — any server receiving a message publishes to the broker; all servers subscribe and push to their local WebSocket connections matching the target user |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
