# Week 08 Weekly Review — Days 50–56
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Week Summary
This week completed the Trees & BST synthesis (Day 50) and opened the Graphs slot (Days 51–56).

**Topics covered:**
- Trees synthesis: Subtree matching, All Nodes Distance K, Sum of Distances in Tree (Day 50)
- Grid DFS: connected components, max area, island merging (Day 51)
- BFS shortest path: multi-source, binary matrix, Word Ladder implicit graph (Day 52)
- Directed graphs: cycle detection (3-colour DFS), topological sort (Kahn's), DAG longest path (Day 53)
- Union-Find: provinces, redundant connection, Tarjan's bridge detection (Day 54)
- Bipartite checking: 2-colouring BFS, Possible Bipartition, Similar String Groups (Day 55)
- BFS variants: 01 Matrix multi-source, Pacific Atlantic reverse BFS, Sliding Puzzle state space (Day 56)

**System design arc:** Completed Caching synthesis (Day 50). Started CDN/DNS/HTTP unit: CDN fundamentals & PoP routing (Day 51), DNS resolution hierarchy (Day 52), HTTP/HTTPS & TLS handshake (Day 53), HTTP caching headers & ETag (Day 54), HTTP/2 & HTTP/3 QUIC (Day 55), full CDN+DNS+HTTP request lifecycle synthesis (Day 56).

---

## 35 Flashcards — Full Week Review

### Day 50 — Trees Slot 5 Synthesis
| Q | A |
|---|---|
| How does Subtree of Another Tree check equality at each node? | Calls `isSameTree(s, t)`: both null → True; one null → False; else `s.val == t.val and isSameTree(s.left, t.left) and isSameTree(s.right, t.right)` |
| What are the two phases of All Nodes Distance K? | Phase 1: DFS to add parent pointers (makes tree traversable as undirected graph). Phase 2: BFS from target for K steps using parent/left/right as 3 possible neighbours |
| What is the rerooting formula for Sum of Distances in Tree? | `answer[child] = answer[parent] - count[child] + (n - count[child])` — child nodes gain 1, parent-side nodes lose 1 |
| What is the complete 5-question caching checklist for an interview? | (1) What data to cache? (2) When does it go stale? (3) How to prevent stampede? (4) What happens if cache goes down? (5) How to scale the cache? |
| When choose Redis Sentinel vs Redis Cluster? | Sentinel: dataset fits one node, HA only. Cluster: dataset too large for one node, horizontal sharding + built-in HA |

### Day 51 — Grid DFS & Island Merging
| Q | A |
|---|---|
| How do you count connected components in a grid using DFS? | Iterate all cells; for each unvisited land cell, launch DFS to mark all connected land as visited; increment count once per DFS launch |
| How does DFS grid traversal avoid revisiting cells efficiently? | Mutate the cell in-place (set `grid[r][c] = 0`) immediately on entry — no separate visited set needed |
| How does Making A Large Island probe each water cell? | Collect the set of distinct island IDs among the 4 neighbours (dedup); sum their sizes from the island_size map, add 1 for the flipped cell |
| What is a CDN PoP and how does request routing reach it? | PoP = Point of Presence (edge datacenter); routing via DNS geolocation or Anycast (multiple PoPs share one IP; BGP delivers to nearest) |
| What is the CDN cache miss flow? | PoP checks cache → miss → fetch from Origin Shield or origin → cache the response → return to user |

### Day 52 — BFS Shortest Path
| Q | A |
|---|---|
| How does multi-source BFS for Rotting Oranges differ from single-source? | All rotten cells are enqueued at time 0; BFS levels represent time — rot spreads in parallel from all sources simultaneously |
| Why mark cells visited on enqueue (not dequeue) in BFS? | Marking on dequeue allows the same cell to be enqueued multiple times before being processed; marking on enqueue ensures each cell enters the queue exactly once |
| How does Word Ladder generate neighbours in the implicit graph? | For each of M character positions, substitute all 26 letters; check against the word set (O(1)); valid unseen words become BFS neighbours |
| What are the 4 steps in a full DNS resolution? | (1) Resolver → Root (returns TLD server). (2) Resolver → TLD (returns authoritative server). (3) Resolver → Authoritative (returns IP). (4) Resolver caches and returns IP |
| Why lower DNS TTL before a production migration? | Resolvers cache for the full TTL; lowering TTL ensures old records expire quickly after the switch so traffic drains within seconds, not hours |

### Day 53 — Directed Graph Cycle Detection & Topological Sort
| Q | A |
|---|---|
| How does DFS 3-colour cycle detection work in a directed graph? | WHITE (unvisited) → GRAY (in current DFS path) → BLACK (done); a back-edge to a GRAY node means a cycle exists |
| What does Kahn's algorithm return when a directed cycle exists? | The result list has fewer than V nodes — nodes in the cycle never reach in-degree 0 and are never dequeued |
| Why does Longest Increasing Path need no visited set? | Each step requires a strictly larger value — it's impossible to return to a previously visited cell; memo caches the answer per cell |
| What are the three guarantees HTTPS/TLS provides? | (1) Confidentiality (symmetric encryption). (2) Authentication (CA-signed server certificate). (3) Integrity (MAC detects tampering) |
| What is HSTS and what attack does it prevent? | HTTP Strict Transport Security — browser must use HTTPS for all future requests to this domain; prevents SSL stripping attacks that downgrade HTTPS to HTTP |

### Day 54 — Union-Find & Critical Connections
| Q | A |
|---|---|
| How does Union-Find with path compression and union-by-rank achieve near-O(1)? | Path compression flattens the tree on each find(); union-by-rank attaches smaller trees under taller ones; combined gives O(α(n)) amortised — inverse Ackermann |
| How does Redundant Connection use Union-Find? | Process edges in order; the first edge where `union(u, v)` returns False (endpoints already share a root) is the cycle-forming redundant connection |
| What does `low[v] > disc[u]` mean in Tarjan's bridge detection? | v's subtree has no back-edge reaching u or earlier — removing edge (u,v) disconnects the graph; it is a bridge |
| What is the difference between `Cache-Control: no-cache` and `no-store`? | `no-cache`: cache but always revalidate (304 avoids retransmitting body). `no-store`: never cache at all — for sensitive data |
| What is URL versioning for static assets and why is it preferred over CDN purge? | Embed content hash in URL (e.g., `app.abc123.css`); old URLs are immutable (1-year cache); new content = new URL — zero invalidation delay, zero API calls |

### Day 55 — Bipartite Checking & Similar Strings
| Q | A |
|---|---|
| How do you check graph bipartiteness with BFS? | 2-colour: assign opposite colours to BFS neighbours; if any neighbour has the same colour as current → odd cycle → not bipartite |
| Why must the bipartite check outer loop iterate all nodes? | The graph may be disconnected; components unreachable from node 0 must also be checked independently |
| How does Similar String Groups define similarity, and what is the O(n²×L) cost? | Similar if identical or differ in exactly 2 positions that form a swap; O(n²) pairs × O(L) comparison per pair; Union-Find groups transitive similarities |
| What TCP problem does HTTP/2 multiplexing NOT solve? | Transport-level head-of-line blocking — one lost TCP packet blocks ALL HTTP/2 streams on that connection |
| What transport protocol does HTTP/3 use and what does it gain? | QUIC (built on UDP); each stream is independently flow-controlled; 0-RTT for reconnection; built-in TLS 1.3 |

### Day 56 — Multi-Source BFS & State Space
| Q | A |
|---|---|
| How does 01 Matrix's multi-source BFS initialise? | ALL zero-cells are enqueued at distance 0; BFS propagates outward — each 1-cell's distance is the level at which it is first reached |
| How does Pacific Atlantic Water Flow avoid simulating downhill flow? | Reverse: BFS uphill from Pacific-border cells and Atlantic-border cells separately; cells reachable from both are the answer |
| Why can Sliding Puzzle state space use BFS effectively? | State space is finite and small (6! = 720 boards); encode board as string; BFS finds minimum swaps to reach goal state |
| What is an Origin Shield in a CDN? | Mid-tier layer between edge PoPs and origin; PoP misses hit Origin Shield first — only one origin request needed regardless of how many PoPs missed |
| What are the 3 cache layers in the CDN request lifecycle? | (1) Browser cache (private, per-user). (2) CDN PoP edge cache (geographic, shared). (3) Origin Shield cache (central, shared) |

---

## Pattern Quick Reference — Week 08

| Pattern | Trigger phrase | Complexity |
|---------|---------------|-----------|
| Grid DFS flood fill | Connected components, enclosed regions | O(m×n) time, O(m×n) space |
| Multi-source BFS | Distance from nearest source | O(m×n) time, O(m×n) space |
| BFS shortest path | Minimum steps in unweighted graph | O(V+E) time, O(V) space |
| BFS implicit graph | Word transformations, state reachability | O(V×branching) |
| DFS 3-colour (directed) | Cycle detection, safe nodes | O(V+E) time, O(V) space |
| Kahn's topological sort | Task ordering with prerequisites | O(V+E) time, O(V) space |
| Union-Find (path compress + rank) | Connected components, cycle detection | O(α(n)) per op |
| Tarjan's bridges | Critical edges in undirected graph | O(V+E) time, O(V) space |
| BFS 2-colouring | Bipartite check, 2-partition | O(V+E) time, O(V) space |
| Reverse BFS | Pacific Atlantic, boundary reachability | O(m×n) time, O(m×n) space |

---

## Checklist
- [ ] Reviewed all 35 flashcards
- [ ] Able to write Union-Find template from memory (parent + rank + find + union)
- [ ] Able to state the difference between DFS 3-colour and Kahn's for cycle/topological sort
- [ ] Identified which problems you could not solve in 20 min and logged them
