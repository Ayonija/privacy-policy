# Day 63 — Graphs Advanced: Topological Sort Variants & Two-Level Constraints
**Week 09 | Phase 1: DSA Mastery | Month 3**

## Focus
Minimum Height Trees uses a "peeling leaves" topological sort to find the centroid(s) of a tree. Find All Possible Recipes applies Kahn's algorithm where ingredients are edges — a recipe is made when all its ingredients are available. Build Matrix with Conditions requires two independent topological sorts (one for rows, one for columns) and then placing values at their intersecting coordinates.

---

## DSA (2 hours)
### Pattern: Leaf-Peeling Topological Sort + Kahn's Prerequisite Resolution + Two-Dimension Topological Sort

**Minimum Height Trees (LC 310):**
The roots that minimise tree height are the centroid(s) — at most 2, always the middle nodes of the longest path. Algorithm: repeatedly remove leaf nodes (degree 1) until 1 or 2 nodes remain — exactly like Kahn's topological sort on an undirected tree. Maintain a set of current leaves; remove them, reduce neighbours' degrees; add any newly-created leaves. Stop when `remaining ≤ 2`.

**Find All Possible Recipes (LC 2115):**
Recipes may depend on ingredients and on other recipes. Build a dependency graph: if recipe A requires recipe B, add edge B → A. Apply Kahn's: start with all ingredients that are available (in-degree 0). When a recipe's in-degree reaches 0 (all its dependencies are resolved), it can be made — add it to results.

**Build a Matrix With Conditions (LC 2392):**
Place values 1..k in a k×k matrix such that:
- Row conditions: value a appears in a row above value b
- Column conditions: value a appears in a column left of value b

Run Kahn's on the row conditions → get row order. Run Kahn's on the column conditions → get column order. If either has a cycle → return empty matrix. For each value v, its position = (row_pos[v], col_pos[v]). Build the matrix.

**Trigger condition:**
- "find the tree's centroid / root that minimises height" → leaf-peeling (Kahn's on undirected tree); stop when ≤ 2 nodes remain
- "which items can be produced given dependencies and available inputs?" → Kahn's where inputs start with in-degree 0; produced items unlock dependents
- "arrange items in a 2D grid respecting row-order and column-order constraints separately" → two independent topological sorts; use position indices to place

**Time complexity:** LC 310: O(V+E) | LC 2115: O(V+E) | LC 2392: O(k²)
**Space complexity:** O(V+E) for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Height Trees | 310 | Medium | Leaf-peeling topological sort | Remove leaves repeatedly; last 1–2 nodes are centroids; initialize queue with all degree-1 nodes |
| 2 | Find All Possible Recipes | 2115 | Medium | Kahn's with available ingredients as sources | Available ingredients start with in-degree 0; recipes unlock when all ingredients resolved |
| 3 | Build Matrix With Conditions | 2392 | Hard | Two independent topological sorts (rows + cols) | Kahn's on row constraints → row positions; Kahn's on col constraints → col positions; place at intersection |

---

### Code Skeleton
```python
from collections import defaultdict, deque

# Minimum Height Trees (LC 310)
def findMinHeightTrees(n, edges):
    if n == 1: return [0]
    graph = defaultdict(set)
    for u, v in edges:
        graph[u].add(v)
        graph[v].add(u)
    leaves = deque(i for i in range(n) if len(graph[i]) == 1)
    remaining = n
    while remaining > 2:
        leaf_count = len(leaves)
        remaining -= leaf_count
        for _ in range(leaf_count):
            leaf = leaves.popleft()
            for nb in graph[leaf]:
                graph[nb].remove(leaf)
                if len(graph[nb]) == 1:
                    leaves.append(nb)
    return list(leaves)

# Find All Possible Recipes (LC 2115)
def findAllRecipes(recipes, ingredients, supplies):
    recipe_set = set(recipes)
    graph = defaultdict(list)
    in_degree = defaultdict(int)
    for recipe, ing_list in zip(recipes, ingredients):
        for ing in ing_list:
            graph[ing].append(recipe)
            in_degree[recipe] += 1
    # Supplied ingredients start with in-degree 0 (already available)
    queue = deque(s for s in supplies if s not in in_degree or in_degree[s] == 0)
    result = []
    while queue:
        item = queue.popleft()
        if item in recipe_set:
            result.append(item)
        for dep in graph[item]:
            in_degree[dep] -= 1
            if in_degree[dep] == 0:
                queue.append(dep)
    return result

# Build a Matrix With Conditions (LC 2392)
def buildMatrix(k, rowConditions, colConditions):
    def topo_sort(conditions):
        graph = defaultdict(list)
        in_degree = [0] * (k + 1)
        for u, v in conditions:
            graph[u].append(v)
            in_degree[v] += 1
        queue = deque(i for i in range(1, k + 1) if in_degree[i] == 0)
        order = []
        while queue:
            node = queue.popleft()
            order.append(node)
            for nb in graph[node]:
                in_degree[nb] -= 1
                if in_degree[nb] == 0:
                    queue.append(nb)
        return order if len(order) == k else []   # cycle → empty

    row_order = topo_sort(rowConditions)
    col_order = topo_sort(colConditions)
    if not row_order or not col_order: return []

    row_pos = {val: i for i, val in enumerate(row_order)}
    col_pos = {val: i for i, val in enumerate(col_order)}
    matrix = [[0] * k for _ in range(k)]
    for val in range(1, k + 1):
        matrix[row_pos[val]][col_pos[val]] = val
    return matrix
```

---

### Edge Cases to Trace Before Coding
- LC 310: n=1 → return [0] immediately; n=2 → return both nodes; star graph (one centre, all others leaves) → centre is the only centroid; path graph of even length → two centroids
- LC 2115: a recipe depends on another recipe which depends on a third → chain works as long as earlier recipes appear in the queue first; ingredient not in supplies and not a recipe → its dependent recipe can never be made
- LC 2392: cycle in row conditions → `topo_sort` returns [] → return []; value appears only in one condition set → still gets a position (its in-degree reaches 0 normally)

---

### Interview Pattern Drill

| Pattern | Initialization | Termination condition | What the result means |
|---------|---------------|----------------------|-----------------------|
| Standard Kahn's | All nodes with in-degree 0 | Queue empty; check `len(result) == V` for cycle | Topological order of all nodes |
| Leaf-peeling Kahn's | All nodes with degree 1 | `remaining ≤ 2` | Centroid node(s) |
| Prerequisite Kahn's | Available supplies (no dependencies) | Queue empty; result = producible items | Items that can be unlocked |
| Two-level Kahn's | In-degree 0 per dimension | Both sorts complete without cycle | Row and column positions → grid placement |

---

## System Design (1 hour)
### Topic: URL Shortener — Scaling, Read Replicas, and Caching Strategy

**Scaling reads (the dominant traffic):**
At 115,000 redirects/sec, a single MySQL instance cannot handle reads alone. Solution: add read replicas.

```
Write path: App → Primary DB (writes)
Read path:  App → Redis cache (hit) → done
            App → Redis cache (miss) → Read Replica → cache + redirect
```

**Read replica architecture:**
- Primary DB accepts all writes; replication lag is typically < 1 ms for this workload
- 3–5 read replicas behind a load balancer for redirect queries
- Cache-aside pattern: on miss, read from replica, write to Redis (TTL = 24 h for new codes, longer for popular ones)

**Caching policy for URL shortener:**
```
Hot URLs (top 10 %): TTL = 7 days (or no TTL)
Normal URLs: TTL = 24 hours
Expired URLs: cache the 410 Gone response briefly (1 min TTL) to prevent DB hammering on expired hot codes
```

**Bloom filter for non-existent codes:**
Before any DB lookup, check a Bloom filter of all valid short codes. If the filter says "definitely not present," return 404 immediately without touching DB or cache. Reduces load from invalid URL scanning / bots.

```
Bloom filter size for 1 B URLs at 1 % FP rate ≈ 1.2 GB — fits in memory
```

**Sharding the DB (for write scale):**
If write throughput grows beyond a single primary's capacity:
- Shard by the first character of short_code (62 shards) or by hash of short_code % n_shards
- Snowflake ID generator assigned per shard (shard_id baked into bits 10–21 of the ID)
- Read replicas per shard

**URL Shortener complete architecture:**
```
[Client]
    │
    ▼
[CDN / Load Balancer]
    │
    ▼
[App Servers] ──► [Redis cluster (cache-aside)]
    │                     │ (miss)
    │                     ▼
    └──► [Primary DB] ◄── [Read Replicas]
              │
              └──► [Analytics DB] (async click events)
```

**Analytics at scale:**
Don't write click counts to the main DB on every redirect — this creates a hot-row problem. Instead:
1. App server emits click events to Kafka on each redirect
2. Stream processor (Flink/Spark) aggregates counts in micro-batches
3. Write aggregated counts to analytics DB (Cassandra or ClickHouse) every 10 seconds

**Interview talking point:** "The URL shortener is a textbook read-heavy system. With a 99 % Redis hit rate, the DB only sees 1 % of redirect traffic — roughly 1,150 reads/sec vs the raw 115,000. The primary DB only sees the ~1,160 writes/sec for new URLs. That's comfortably within a single M5.xlarge RDS instance. Sharding is only needed beyond ~10,000 writes/sec."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 310 Minimum Height Trees (target: 18 min)
- **Medium 2:** LC 2115 Find All Possible Recipes (target: 18 min)
- **Hard:** LC 2392 Build Matrix with Conditions (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to scale a system from single-node to distributed — what was the inflection point that triggered the change?
- Leadership principle: Invent and Simplify

---

## Flashcards

| Q | A |
|---|---|
| How does leaf-peeling find the centroid(s) of a tree? | Repeatedly remove all current leaves (degree-1 nodes) in rounds; decrement their neighbours' degrees; add newly-created leaves; stop when ≤ 2 nodes remain — those are the centroid(s) that minimise tree height |
| How does Find All Possible Recipes differ from standard topological sort initialisation? | Standard Kahn's starts with nodes whose in-degree is 0. Here, available supplies start with in-degree 0; recipes start with in-degree = number of ingredients. As ingredients are "produced," recipe in-degrees decrement |
| What happens if topo_sort returns an empty list in Build Matrix with Conditions? | A cycle was detected in the row or column conditions — it is impossible to satisfy all ordering constraints. Return an empty matrix `[]` |
| How does a Bloom filter improve URL shortener read performance? | A Bloom filter answers "is this short_code definitely NOT in the DB?" in O(1) with zero false negatives. Invalid codes (bots, typos) skip the DB entirely, preventing unnecessary DB load |
| Why use Kafka for URL shortener analytics instead of direct DB writes? | Redirect events (115,000/sec) writing click counts directly to DB would create hot-row contention on popular URLs. Kafka buffers the events; a stream processor aggregates them before writing to the analytics DB |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/18/25 min, no hints)
- [ ] Rewrote leaf-peeling and Kahn's templates from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw the full URL shortener architecture cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
