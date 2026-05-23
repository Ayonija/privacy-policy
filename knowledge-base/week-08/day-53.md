# Day 53 — Graphs: Directed Graph Cycle Detection & Topological Sort
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Directed graph cycle detection uses DFS with a "currently in stack" flag. Topological sort is either post-order DFS reversed, or Kahn's BFS from zero-in-degree nodes. Longest Increasing Path applies memo DFS on a DAG formed by value comparisons.

---

## DSA (2 hours)
### Pattern: Directed Graph DFS (3-Colour Marking) + Topological Sort (Kahn's) + DAG Longest Path with Memo

**Course Schedule (LC 207):**
Build a directed graph from prerequisites. Detect a cycle — if a cycle exists, it's impossible to finish all courses. DFS 3-colour marking: `WHITE=0` (unvisited), `GRAY=1` (in current DFS stack), `BLACK=2` (fully processed). A back-edge to a GRAY node = cycle.

**Course Schedule II (LC 210):**
Same cycle detection, but also produce the topological order. Kahn's algorithm (BFS): start with all nodes of in-degree 0; remove them from the graph; add their neighbours to the queue when their in-degree becomes 0. Order in which nodes are dequeued = a valid topological order.

**Longest Increasing Path in a Matrix (LC 329):**
Each cell is a node; add a directed edge from cell A to cell B if B is a 4-directional neighbour with a strictly larger value. The resulting graph is a DAG (no cycles possible — values strictly increase). Memoised DFS from each cell: `memo[r][c]` = length of longest increasing path starting at `(r, c)`. Return the global maximum.

**Trigger condition:**
- "can you complete all tasks given prerequisites?" → directed graph cycle detection (DFS 3-colour or Kahn's)
- "give a valid ordering of tasks respecting dependencies" → topological sort
- "longest increasing path in a grid" → DAG memo DFS, no need for visited set (values prevent revisits)

**Time complexity:** O(V+E) for LC 207/210 | O(m×n) for LC 329
**Space complexity:** O(V+E) for LC 207/210 | O(m×n) for LC 329

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Course Schedule | 207 | Medium | DFS 3-colour cycle detection | GRAY = in current stack; back-edge to GRAY = cycle; no cycle → True |
| 2 | Course Schedule II | 210 | Medium | Kahn's topological sort | BFS from in-degree-0 nodes; dequeue order = topological order; cycle → empty result |
| 3 | Longest Increasing Path in a Matrix | 329 | Hard | Memo DFS on implicit DAG | Edges go from lower to higher value; memo[r][c] = longest path from here; no visited set needed (strict increase prevents revisits) |

---

### Code Skeleton
```python
from collections import defaultdict, deque

# Course Schedule (LC 207) — DFS 3-colour
def canFinish(numCourses, prerequisites):
    graph = defaultdict(list)
    for a, b in prerequisites:
        graph[b].append(a)   # b must be done before a

    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * numCourses

    def dfs(node):
        if color[node] == GRAY: return False   # back-edge = cycle
        if color[node] == BLACK: return True   # already processed
        color[node] = GRAY
        for nb in graph[node]:
            if not dfs(nb): return False
        color[node] = BLACK
        return True

    return all(dfs(i) for i in range(numCourses))

# Course Schedule II (LC 210) — Kahn's BFS topological sort
def findOrder(numCourses, prerequisites):
    graph = defaultdict(list)
    in_degree = [0] * numCourses
    for a, b in prerequisites:
        graph[b].append(a)
        in_degree[a] += 1

    queue = deque(i for i in range(numCourses) if in_degree[i] == 0)
    order = []
    while queue:
        course = queue.popleft()
        order.append(course)
        for nb in graph[course]:
            in_degree[nb] -= 1
            if in_degree[nb] == 0:
                queue.append(nb)
    return order if len(order) == numCourses else []

# Longest Increasing Path in a Matrix (LC 329)
def longestIncreasingPath(matrix):
    rows, cols = len(matrix), len(matrix[0])
    memo = {}
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    def dfs(r, c):
        if (r, c) in memo: return memo[(r, c)]
        best = 1
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and matrix[nr][nc] > matrix[r][c]:
                best = max(best, 1 + dfs(nr, nc))
        memo[(r, c)] = best
        return best

    return max(dfs(r, c) for r in range(rows) for c in range(cols))
```

---

### Edge Cases to Trace Before Coding
- LC 207: no prerequisites → all courses finishable → return True; self-loop `[[0,0]]` → GRAY→GRAY = cycle → False
- LC 210: multiple valid topological orders exist — Kahn's returns one; if cycle → return `[]` (check `len(order) == numCourses`)
- LC 329: all values identical → no valid moves from any cell → answer is 1; single cell → answer is 1

---

### Interview Pattern Drill
| Algorithm | Cycle detection | Topological order | Complexity |
|-----------|----------------|-------------------|-----------|
| DFS 3-colour | GRAY node in stack | Post-order reversed | O(V+E) |
| Kahn's BFS | `len(result) < V` after BFS | Dequeue order | O(V+E) |
| Memo DFS (DAG longest path) | Not applicable (strict comparison prevents cycles) | Implicit (DFS from each node) | O(V+E) |

---

### STAR Interview Framework

> **Directed Graph Cycle Detection + Topological Sort:** brute-force O(V! orderings) → this approach O(V+E) time, O(V) space

**S:** "Given V tasks and E prerequisite edges. Naive enumeration of all orderings is O(V!) — impossible at V=40."
**T:** "Need O(V+E) valid ordering (or cycle detection) using the graph's inherent structure."
**A (60%):**
1. *Classify:* "Prerequisite edges form a directed graph — check for DAG before sorting."
2. *Init:* "Build adjacency list and in-degree array; seed queue with all in-degree-0 nodes."
3. *Loop/Step:* "Dequeue node u; decrement in-degree of each neighbour v; enqueue v when in-degree hits 0."
4. *Termination:* "If `len(result) == V`, no cycle — result is a valid topological order."
5. *Gotcha:* "If `len(result) < V` after BFS completes, a cycle exists — return [] or False."
**R:** "O(V+E) time, O(V) space. For 40 services with 90 edges: microseconds vs V!=8×10^47 for brute force."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| DFS 3-colour | Need cycle detection only, no ordering needed | Less natural for producing order; same complexity |
| Reverse post-order DFS | Prefer DFS over BFS | Harder to detect cycle (need GRAY set) |

---

## System Design (1 hour)
### Topic: HTTP & HTTPS Fundamentals — Protocol, TLS, and Status Codes

**HTTP request/response structure:**
```
GET /api/users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Accept: application/json
                            ← blank line separates headers from body
[body — empty for GET]
```
```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=60
                            ← blank line
{"id": 42, "name": "Alice"}
```

**Essential HTTP status codes:**
| Code | Name | When used |
|------|------|-----------|
| 200 | OK | Successful GET/PUT |
| 201 | Created | Successful POST (new resource) |
| 204 | No Content | Successful DELETE (no body) |
| 301 | Moved Permanently | Permanent redirect (cached by browser) |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Conditional GET hit — body omitted |
| 400 | Bad Request | Invalid client input |
| 401 | Unauthorized | Missing or invalid auth token |
| 403 | Forbidden | Authenticated but not authorised |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unhandled server exception |
| 503 | Service Unavailable | Server overloaded or in maintenance |

**HTTPS = HTTP + TLS:**
TLS (Transport Layer Security) adds three guarantees:
1. **Confidentiality:** all data is encrypted with a symmetric key negotiated during the handshake
2. **Authentication:** server presents a certificate signed by a trusted CA (Certificate Authority)
3. **Integrity:** each record includes a MAC — tampering is detectable

**TLS 1.3 handshake (simplified):**
```
Client → Server: ClientHello (supported cipher suites, random)
Server → Client: ServerHello (chosen cipher), Certificate, ServerFinished
Client verifies certificate chain (CA → intermediate → leaf cert)
Client → Server: Key exchange (ECDHE ephemeral key), ClientFinished
Both sides derive the same session key → encrypted data flows
```
TLS 1.3 completes in 1 round-trip (1-RTT). 0-RTT is possible for resumed sessions (but replay attack risk).

**HTTP vs HTTPS in production:**
- All production traffic must use HTTPS — plain HTTP leaks auth tokens, session cookies, API keys
- HSTS (HTTP Strict Transport Security): server sends `Strict-Transport-Security: max-age=31536000` header — browser refuses plain HTTP for that domain for 1 year
- Certificate management: Let's Encrypt provides free 90-day certs with automatic renewal via ACME protocol

**Interview talking point:** "If asked what happens between a user clicking a link and the server sending a response, answer: DNS lookup (covered yesterday) → TCP three-way handshake (SYN/SYN-ACK/ACK) → TLS handshake (1-2 RTTs) → HTTP GET request → Server processes, queries DB/cache → HTTP response with status code and body → Browser renders. Each step has a latency budget: DNS ~20ms, TCP ~10ms (local), TLS ~10ms (TLS 1.3), server processing ~50ms, network ~50ms. Total ≈ 150ms for a reasonably optimised stack."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Think Big

**Full STAR Story — "Dependency-Ordered Service Migration":**
**S (20%):** "At a mid-size fintech, our monolith had 40 services being extracted into microservices with implicit startup dependencies — teams were deploying in wrong order, causing cascading failures across 3 environments."
**T:** "I was responsible for defining the safe deployment order across all 40 services within a 6-week migration window."
**A (60% — 'I' not 'we'):** "(1) I mapped every service's explicit and implicit dependency edges into a directed graph with 40 nodes and ~90 edges. (2) I wrote a script to run Kahn's topological sort on the graph, detecting two circular dependency cycles that teams hadn't noticed. (3) I refactored those cycles by extracting a shared config service that broke both loops. (4) I generated the final deployment order and codified it as a pipeline gate in CI — no service could deploy unless all its topological predecessors had passed health checks."
**R (20%):** "Zero cascading failures across 6 subsequent deployment windows. Migration completed 4 days ahead of schedule. The cycle detection caught bugs that would have caused production outages affecting 12,000 daily active users."
*Works for: Think Big, Deliver Results, Invent and Simplify.*

---

## Flashcards

| Q | A |
|---|---|
| How does DFS 3-colour cycle detection work in a directed graph? | WHITE (unvisited) → GRAY (in current DFS path) → BLACK (done); a back-edge to a GRAY node means a cycle exists |
| What does Kahn's algorithm return when a directed cycle exists? | The result list will have fewer than V nodes — some nodes with cyclic dependencies never reach in-degree 0 and are never dequeued |
| Why does Longest Increasing Path need no visited set? | Each DFS step requires a strictly larger value — it's impossible to return to a previously visited cell, so cycles in the grid graph cannot form; memo caches the answer per cell |
| What are the three guarantees HTTPS/TLS provides? | (1) Confidentiality (symmetric encryption). (2) Authentication (CA-signed server certificate). (3) Integrity (MAC detects tampering) |
| What is HSTS and what attack does it prevent? | HTTP Strict Transport Security — server declares that the browser must use HTTPS for all future requests to this domain; prevents SSL stripping attacks where a MitM downgrades HTTPS to HTTP |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
