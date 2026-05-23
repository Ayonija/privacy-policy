# Day 55 — Graphs: Bipartite Checking & Similar String Groups
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Bipartite graphs can be 2-coloured — BFS/DFS detects odd-length cycles which prevent this. Possible Bipartition frames a partitioning problem as graph 2-colourability. Similar String Groups uses Union-Find on pairwise string similarity checks.

---

## DSA (2 hours)
### Pattern: Graph 2-Colouring (Bipartite Check) + Union-Find on Similarity Relation

**Is Graph Bipartite? (LC 785):**
Try to 2-colour the graph: assign colour 0 to the start node; assign colour 1 to all its neighbours; alternate colours as BFS/DFS expands. If any node's neighbour already has the same colour → odd-length cycle → not bipartite.

**Possible Bipartition (LC 886):**
Build a "dislike graph": add an edge between every pair of people who dislike each other. Then check if this graph is bipartite — group A = colour 0, group B = colour 1. If any dislike edge connects two people in the same group → impossible.

**Similar String Groups (LC 839):**
Two strings are "similar" if they are identical OR differ in exactly 2 positions (those positions form a valid transposition — swapping those two characters). Union-Find: iterate all pairs; union similar pairs; count the number of connected components.

**Trigger condition:**
- "can you split nodes into two groups with no conflicts within a group?" → bipartite check (BFS 2-colour)
- "group strings where transitive similarity connects them" → Union-Find on all pairs with similarity predicate

**Time complexity:** LC 785: O(V+E) | LC 886: O(V+E) | LC 839: O(n²×L) where L = string length
**Space complexity:** O(V+E) for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Is Graph Bipartite? | 785 | Medium | BFS 2-colouring | Alternate colours; if a neighbour has the same colour as current node → not bipartite |
| 2 | Possible Bipartition | 886 | Medium | Build dislike graph + bipartite check | Dislike pairs = edges; graph 2-colourable → partitionable; not bipartite → impossible |
| 3 | Similar String Groups | 839 | Hard | Union-Find on pairwise string similarity | Two strings similar if equal or exactly 2 chars differ; union similar pairs; count components |

---

### Code Skeleton
```python
from collections import defaultdict, deque

# Is Graph Bipartite? (LC 785)
def isBipartite(graph):
    n = len(graph)
    color = [-1] * n   # -1 = uncoloured
    for start in range(n):
        if color[start] != -1: continue   # already coloured
        queue = deque([start])
        color[start] = 0
        while queue:
            node = queue.popleft()
            for nb in graph[node]:
                if color[nb] == -1:
                    color[nb] = 1 - color[node]   # alternate colour
                    queue.append(nb)
                elif color[nb] == color[node]:
                    return False   # same colour on both ends of an edge
    return True

# Possible Bipartition (LC 886)
def possibleBipartition(n, dislikes):
    graph = defaultdict(list)
    for a, b in dislikes:
        graph[a].append(b)
        graph[b].append(a)
    color = [-1] * (n + 1)
    for start in range(1, n + 1):
        if color[start] != -1: continue
        queue = deque([start])
        color[start] = 0
        while queue:
            node = queue.popleft()
            for nb in graph[node]:
                if color[nb] == -1:
                    color[nb] = 1 - color[node]
                    queue.append(nb)
                elif color[nb] == color[node]:
                    return False
    return True

# Similar String Groups (LC 839)
def numSimilarGroups(strs):
    n = len(strs)
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

    def similar(a, b):
        diff = [(ca, cb) for ca, cb in zip(a, b) if ca != cb]
        return len(diff) == 0 or (len(diff) == 2 and diff[0] == diff[1][::-1])

    for i in range(n):
        for j in range(i+1, n):
            if similar(strs[i], strs[j]):
                union(i, j)

    return sum(1 for i in range(n) if find(i) == i)
```

---

### Edge Cases to Trace Before Coding
- LC 785: disconnected graph — must check all connected components (outer loop over all start nodes); graph with all self-loops → trivially bipartite if no cross-edges
- LC 886: no dislike pairs → trivially partitionable (everyone in group A); single person → True
- LC 839: all strings identical → all in one group → 1 component; strings with length 1 → can only be similar if equal (no 2-char swap possible)

---

### Interview Pattern Drill
| Problem type | Algorithm | Key step |
|-------------|-----------|---------|
| 2-partition with conflict edges | BFS 2-colouring | Same colour on edge ends → reject |
| Transitive grouping | Union-Find | Define similarity predicate; union if similar |
| Connected components | DFS/BFS counting | Launch once per unvisited node |

---

### STAR Interview Framework

> **Graph 2-Colouring (Bipartite Check):** brute-force O(2^V subsets) → this approach O(V+E) time, O(V) space

**S:** "Given V nodes and E conflict edges, need to determine if nodes can be split into 2 groups with no conflicts within a group. Brute-force subset enumeration is O(2^V)."
**T:** "Need O(V+E) bipartite check using BFS 2-colouring."
**A (60%):**
1. *Classify:* "'Can you 2-partition nodes with conflict edges?' → bipartite check. Conflict graph = dislike graph."
2. *Init:* "`color = [-1] * n`; iterate all uncoloured nodes (handles disconnected graph)."
3. *Loop/Step:* "BFS from each uncoloured start node; assign colour 0 to start; assign `1 - color[node]` to each uncoloured neighbour; if neighbour already has same colour → odd cycle → not bipartite."
4. *Termination:* "If all nodes are coloured without conflict, graph is bipartite."
5. *Gotcha:* "Must iterate ALL nodes in the outer loop — disconnected components won't be reached from a single BFS start."
**R:** "O(V+E) time, O(V) space. Handles disconnected graphs and self-loops correctly."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| DFS 2-colouring | Same complexity, preferred recursively | Stack overflow risk for deep graphs |
| Union-Find | Transitive grouping (not partitioning) | Can't detect odd cycles directly |

---

## System Design (1 hour)
### Topic: HTTP/2 and HTTP/3 — Multiplexing, QUIC, and Head-of-Line Blocking

**HTTP/1.1 limitations:**
- One request at a time per TCP connection (pipeline model is broken in practice)
- Browsers open 6 TCP connections per domain to parallelise — wasteful
- Text-based headers — verbose and uncompressed (a typical request header is 500–2000 bytes)

**HTTP/2 improvements:**
1. **Binary framing:** requests and responses are encoded in binary frames — more compact and less error-prone to parse
2. **Multiplexing:** multiple requests/responses share a single TCP connection simultaneously via streams. A stream is an independent, bidirectional sequence of frames; stream IDs distinguish them
3. **Header compression (HPACK):** maintains a dynamic header table; only sends the diff between successive requests — reduces header overhead by 85–90 %
4. **Server push:** server can proactively send resources (e.g., CSS, JS) it knows the client will need, before the client asks
5. **Stream prioritisation:** clients can assign weights to streams to hint which responses are more urgent

**HTTP/2 remaining limitation:**
TCP-level head-of-line (HOL) blocking. If one TCP packet is lost, ALL HTTP/2 streams on that connection are blocked until the packet is retransmitted. Multiplexing solves application-level HOL but not transport-level HOL.

**HTTP/3 (QUIC):**
HTTP/3 replaces TCP with QUIC (Quick UDP Internet Connections) — a custom transport protocol built on UDP.
- Each QUIC stream is independently flow-controlled — a lost packet only blocks the stream it belongs to, not others
- Connection IDs survive IP address changes (mobile users switching WiFi → cellular without re-handshaking)
- **0-RTT connection establishment:** for previously visited servers, HTTP/3 can send data in the very first packet (no handshake round-trip)
- TLS 1.3 is built into QUIC — no separate TLS layer

**Comparison:**
| Dimension | HTTP/1.1 | HTTP/2 | HTTP/3 (QUIC) |
|-----------|----------|--------|--------------|
| Transport | TCP | TCP | QUIC (UDP) |
| Multiplexing | No | Yes | Yes |
| TCP HOL blocking | Yes | Yes | No |
| Header compression | No | HPACK | QPACK |
| 0-RTT reconnection | No | No | Yes |
| Adoption | Universal | ~65 % of sites | ~30 % of sites (2025) |

**Interview talking point:** "If asked why Google switched to HTTP/3 for YouTube, answer: video streaming over unreliable mobile networks suffers from TCP HOL blocking — one dropped packet pauses ALL video data on that connection. HTTP/3 with QUIC ensures a single lost packet only blocks its own stream; other video segments continue uninterrupted. Combined with 0-RTT for returning users, HTTP/3 reduces initial load latency by 10–30 % on high-latency connections."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Bias for Action

**Full STAR Story — "Decoupling the Blocking Dependency":**
**S (20%):** "At a logistics startup, our order status updates and our shipment tracking pipeline shared a single synchronous RPC call to a third-party carrier API. When the carrier's API had a 2-minute brownout, ALL order status updates stalled — including those for carriers completely unrelated to the brownout."
**T:** "I needed to decouple unrelated services so that one carrier's failure could not block others — while keeping the fix scoped to a single sprint."
**A (60% — 'I' not 'we'):** "(1) I identified that the synchronous RPC was the shared bottleneck — equivalent to a single TCP connection with HOL blocking. (2) I replaced the synchronous call with a per-carrier async job queue: each carrier got its own isolated queue, so a slow or failing carrier only blocked its own queue. (3) I added a circuit breaker per carrier with a 30-second trip threshold and exponential backoff on retry. (4) I deployed behind a feature flag and ran both paths in parallel for 48 hours to validate latency and error rates."
**R (20%):** "Next carrier brownout: zero cross-carrier impact. P99 order status update latency dropped from 3.2s to 280ms. The isolation pattern was adopted for 3 other third-party integrations within the following quarter."
*Works for: Bias for Action, Invent and Simplify, Customer Obsession.*

---

## Flashcards

| Q | A |
|---|---|
| How do you check graph bipartiteness with BFS? | 2-colour the graph: assign opposite colours to BFS neighbours; if any neighbour already has the same colour as the current node → odd-length cycle → not bipartite |
| Why must the bipartite check outer loop iterate all nodes (not just node 0)? | The graph may be disconnected; components not reachable from node 0 must also be checked for bipartiteness independently |
| How does Similar String Groups define string similarity, and what is the O(n²×L) cost? | Two strings are similar if identical or differ in exactly 2 positions that form a swap; O(n²) pairs × O(L) comparison per pair; Union-Find groups transitive similarities |
| What TCP problem does HTTP/2 multiplexing NOT solve? | Transport-level head-of-line (HOL) blocking — if a TCP packet is lost, all HTTP/2 streams on that connection stall until retransmission; multiplexing only removes application-level HOL blocking |
| What transport protocol does HTTP/3 use instead of TCP, and what does it gain? | QUIC (built on UDP); each stream is independently flow-controlled so one lost packet blocks only its own stream; 0-RTT for reconnection; built-in TLS 1.3 |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
