# Day 57 — Graphs: DFS Path Finding & Two-Level Topological Sort
**Week 09 | Phase 1: DSA Mastery | Month 2**

## Focus
DFS backtracking collects ALL paths in a DAG. Find Eventual Safe States classifies nodes by cycle membership using reverse-graph topological sort. Sort Items by Groups requires two simultaneous topological sorts — the most complex graph problem in this slot.

---

## DSA (2 hours)
### Pattern: DFS All-Paths Backtracking + Reverse-Graph Safe-Node Detection + Two-Level Topological Sort

**All Paths From Source to Target (LC 797):**
DAG (directed acyclic graph) — no cycles. DFS from source (node 0) with backtracking. Maintain a current path list; when the target is reached, copy it to results. Backtrack (pop) after returning from each child to restore the path for sibling branches.

**Find Eventual Safe States (LC 802):**
A node is "safe" if all paths from it lead to a terminal node (a node with no outgoing edges) — equivalently, it is not part of a cycle. Build the reverse graph (reverse all edge directions) and apply Kahn's topological sort: terminal nodes in the original graph become source nodes in the reverse graph. Nodes that can be processed in this sort (reach in-degree 0 in the reverse graph) are safe.

**Sort Items by Groups (LC 1203):**
Items belong to groups (or each unassigned item forms its own group of size 1). Two-level topological sort:
1. Reassign items with group = -1 to unique group IDs (starting from m)
2. Build an item-level dependency graph AND a group-level dependency graph (derived from cross-group item dependencies)
3. Kahn's topological sort on both graphs; if either has a cycle → return []
4. Interleave: for each group in the group topological order, emit its items in the item topological order

**Trigger condition:**
- "enumerate all paths from source to target" → DFS backtracking on a DAG; push/pop path list
- "which nodes are guaranteed to reach a terminal?" → reverse graph + Kahn's; safe = nodes that are processed
- "sort items respecting both within-group and cross-group dependencies" → two-level topological sort

**Time complexity:** LC 797: O(2ⁿ×n) worst case | LC 802: O(V+E) | LC 1203: O(V+E)
**Space complexity:** O(V) for LC 802/1203 | O(n × path_length) for LC 797

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | All Paths From Source to Target | 797 | Medium | DFS backtracking on DAG | Push before recurse, pop after return; copy path (not reference) on reaching target |
| 2 | Find Eventual Safe States | 802 | Medium | Reverse graph + Kahn's topological sort | Reverse edges; in-degree 0 in reverse = terminal in original; processed nodes are safe |
| 3 | Sort Items by Groups Respecting Dependencies | 1203 | Hard | Two-level topological sort (items + groups) | Separate item-graph from group-graph; Kahn's on both; interleave by group topological order |

---

### Code Skeleton
```python
from collections import defaultdict, deque

# All Paths From Source to Target (LC 797)
def allPathsSourceTarget(graph):
    result = []
    def dfs(node, path):
        if node == len(graph) - 1:
            result.append(list(path))   # copy, not reference
            return
        for nb in graph[node]:
            path.append(nb)
            dfs(nb, path)
            path.pop()                  # backtrack
    dfs(0, [0])
    return result

# Find Eventual Safe States (LC 802)
def eventualSafeNodes(graph):
    n = len(graph)
    reverse_graph = defaultdict(list)
    in_degree = [0] * n   # in-degree in the REVERSE graph
    for u in range(n):
        for v in graph[u]:
            reverse_graph[v].append(u)
            in_degree[u] += 1          # u has an outgoing edge in original → in-degree in reverse
    # Start with terminal nodes in original (no outgoing edges = in_degree 0 in reverse? No...)
    # Wait: terminal nodes in original have no outgoing edges → in_degree[node] == 0
    queue = deque(i for i in range(n) if in_degree[i] == 0)
    safe = set()
    while queue:
        node = queue.popleft()
        safe.add(node)
        for nb in reverse_graph[node]:
            in_degree[nb] -= 1
            if in_degree[nb] == 0:
                queue.append(nb)
    return sorted(safe)

# Sort Items by Groups Respecting Dependencies (LC 1203)
def sortItems(n, m, group, beforeItems):
    # Step 1: assign unique group IDs to ungrouped items
    for i in range(n):
        if group[i] == -1:
            group[i] = m
            m += 1

    # Step 2: build item graph and group graph
    item_graph   = defaultdict(list)
    group_graph  = defaultdict(set)   # set to avoid duplicate edges
    item_indeg   = [0] * n
    group_indeg  = [0] * m

    for item in range(n):
        for dep in beforeItems[item]:
            item_graph[dep].append(item)
            item_indeg[item] += 1
            if group[dep] != group[item]:
                if item not in group_graph[group[dep]]:
                    group_graph[group[dep]].add(group[item])
                    group_indeg[group[item]] += 1

    # Step 3: Kahn's topological sort helper
    def kahn(nodes, indeg, adj):
        q = deque(v for v in nodes if indeg[v] == 0)
        order = []
        while q:
            v = q.popleft()
            order.append(v)
            for nb in adj[v]:
                indeg[nb] -= 1
                if indeg[nb] == 0: q.append(nb)
        return order if len(order) == len(nodes) else []

    item_order  = kahn(range(n), item_indeg, item_graph)
    group_order = kahn(range(m), group_indeg, group_graph)
    if not item_order or not group_order: return []

    # Step 4: group items by group ID in item-topological order
    group_to_items = defaultdict(list)
    for item in item_order:
        group_to_items[group[item]].append(item)

    result = []
    for g in group_order:
        result.extend(group_to_items[g])
    return result
```

---

### Edge Cases to Trace Before Coding
- LC 797: only one path exists → result has one entry; single node graph where node 0 = target → result = [[0]]
- LC 802: all nodes are terminal (no edges) → all are safe; all nodes in one big cycle → none are safe; single isolated node → safe
- LC 1203: item depends on itself (self-loop) → Kahn's never processes it → return []; cross-group dependencies that form a cycle among groups → group Kahn's cycle detected → return []

---

### Interview Pattern Drill
| Pattern | What DFS/BFS processes | What constitutes "safe" / "done" |
|---------|----------------------|----------------------------------|
| All paths DFS | Every node in every root-to-target path | Path reaches target |
| Safe node BFS | Reverse graph, Kahn's from terminals | Node dequeued = safe in original graph |
| Two-level topo sort | Items AND groups separately | Both sorts complete without cycle |

---

## System Design (1 hour)
### Topic: CDN Advanced — Anycast Routing and Origin Shield

**Anycast routing (deep dive):**
In Anycast, multiple physical servers advertise the same IP address to the internet via BGP (Border Gateway Protocol). The internet's routing infrastructure automatically delivers packets to the topologically nearest server advertising that IP.

```
User in Frankfurt → routes to Frankfurt PoP (same Anycast IP as Tokyo, Dallas, etc.)
User in Tokyo    → routes to Tokyo PoP
```

**Why Anycast over DNS-based routing:**
- **DNS-based:** CDN's DNS returns different IPs for different geographic regions. Limited by DNS caching TTL — a user behind a long-TTL resolver may be sent to a suboptimal PoP for hours.
- **Anycast:** routing happens at the network layer, every packet is routed optimally, no TTL delays. Additionally, if one PoP fails, BGP re-routes traffic to the next nearest PoP within seconds.

**Anycast DDoS mitigation:**
Volumetric DDoS attacks are absorbed across all PoPs. A 1 Tbps attack aimed at a single IP is spread across hundreds of PoPs — each absorbs a fraction. This is why Cloudflare (Anycast) can absorb massive DDoS attacks that would overwhelm a single datacenter.

**Origin Shield (Shield PoP):**
Without Origin Shield:
- 300 edge PoPs worldwide
- Cache miss at any PoP → goes directly to origin
- Popular object missed at 100 PoPs simultaneously → 100 origin requests (thundering herd)

With Origin Shield:
- Cache miss at any edge PoP → goes to a designated Shield PoP (1–3 per region)
- Shield checks its own cache; misses from the shield go to origin
- Popular object missed at 100 PoPs → 1–3 origin requests max

**Which CDN providers offer what:**
| Feature | Cloudflare | AWS CloudFront | Fastly | Akamai |
|---------|-----------|---------------|-------|--------|
| Anycast | Yes (all PoPs) | Partial | Yes | No (DNS-based) |
| Origin Shield | Yes (Tiered Caching) | Yes (Origin Shield) | Yes (Shield) | Yes (SureRoute) |
| PoP count | 300+ | 400+ | 50+ | 4000+ |
| Price model | Flat-rate plan | Per-GB + requests | Per-GB | Enterprise |

**Interview talking point:** "If asked how a CDN handles a flash crowd (sudden viral content), answer: when a new video goes viral, the first viewer in each region causes one cache miss to the Origin Shield; the Shield fetches once from origin and caches. All subsequent viewers in that region hit the shield or edge cache. Without a shield, 300 PoPs could simultaneously miss on the same video and hit origin with 300 requests — potentially overwhelming it. With a shield, origin sees at most a handful of requests."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you identified a bottleneck in a multi-stage system and inserted an intermediate layer to absorb load — analogous to an Origin Shield reducing requests from edge to origin.
- Leadership principle: Frugality

---

## Flashcards

| Q | A |
|---|---|
| How does All Paths DFS avoid capturing references instead of copies? | Use `result.append(list(path))` — `list(path)` creates a shallow copy of the current path list; without this, all entries in result would point to the same mutable list |
| How does Find Eventual Safe States determine safety using a reverse graph? | Reverse all edges; apply Kahn's BFS from nodes with in-degree 0 in the reverse graph (= original terminal nodes); nodes dequeued during this BFS are safe in the original graph |
| What is the two-level structure in Sort Items by Groups topological sort? | An item-level graph (item → item dependencies) and a group-level graph (group → group, derived from cross-group item dependencies); both must be topologically sortable (no cycles) |
| What is Anycast routing and how does it differ from DNS-based CDN routing? | Anycast: multiple PoPs share one IP; BGP routes each packet to the nearest PoP at the network layer — no DNS TTL delay, instant failover. DNS-based: CDN's DNS returns PoP-specific IPs; limited by resolver TTL caching |
| What problem does Origin Shield solve, and what is the failure mode without it? | Without Shield: cache misses at all edge PoPs hit origin simultaneously (thundering herd — O(PoPs) origin requests). With Shield: misses go to one Shield PoP; Shield misses hit origin — O(1) origin requests for popular content |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
