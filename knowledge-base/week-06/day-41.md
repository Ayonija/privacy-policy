# Day 41 — Trees: DFS Traversals & Maximum Path Sum
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Master iterative-style recursive DFS and the gain-propagation pattern — where each node returns the best one-sided gain upward while the global answer can bend through any node in both directions.

---

## DSA (2 hours)
### Pattern: DFS Depth Tracking & Gain Propagation

**Binary Tree Right Side View (LC 199):**
DFS right-first: visit the right child before the left at every node. Maintain a `depth` parameter. The first time you reach a new depth `d` (i.e., `d == len(result)`), that is the rightmost visible node — append its value. This single DFS pass collects all right-side nodes in O(n).

**Count Good Nodes in Binary Tree (LC 1448):**
DFS passing the maximum value seen on the root-to-current-node path. A node is "good" if `node.val >= max_so_far`. Propagate `max(max_so_far, node.val)` downward. Count all good nodes across the tree.

**Binary Tree Maximum Path Sum (LC 124):**
Each node can be the "apex" (turning point) of a path that bends left → node → right. DFS computes:
- `left_gain  = max(0, dfs(left))`  — take left branch only if positive
- `right_gain = max(0, dfs(right))` — same for right
- Update global max with `node.val + left_gain + right_gain` (the apex-split path)
- Return `node.val + max(left_gain, right_gain)` to parent — path can only go one direction upward

**Trigger condition:**
- "rightmost/leftmost node at each depth" → DFS with depth index, right-before-left (or left-before-right)
- "count nodes satisfying a root-to-node path constraint" → DFS with running max/min/sum
- "maximum sum of any path (any start, any end) in a binary tree" → gain-propagation DFS + global max

**Time complexity:** O(n) for all | **Space complexity:** O(h) — O(log n) balanced, O(n) skewed

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Binary Tree Right Side View | 199 | Medium | DFS right-first + depth index | Visit right before left; first node seen at each new depth is the rightmost visible node |
| 2 | Count Good Nodes in Binary Tree | 1448 | Medium | DFS with running max | Pass `max_so_far` down; node is good when `node.val >= max_so_far` |
| 3 | Binary Tree Maximum Path Sum | 124 | Hard | Gain-propagation DFS + nonlocal max | Return one-sided gain to parent; update global answer with two-sided split at current node |

---

### Code Skeleton
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# Binary Tree Right Side View (LC 199)
def rightSideView(root):
    result = []
    def dfs(node, depth):
        if not node: return
        if depth == len(result):        # first node reached at this depth (right-first)
            result.append(node.val)
        dfs(node.right, depth + 1)     # right child first
        dfs(node.left,  depth + 1)
    dfs(root, 0)
    return result

# Count Good Nodes in Binary Tree (LC 1448)
def goodNodes(root):
    def dfs(node, max_so_far):
        if not node: return 0
        good = 1 if node.val >= max_so_far else 0
        new_max = max(max_so_far, node.val)
        return good + dfs(node.left, new_max) + dfs(node.right, new_max)
    return dfs(root, root.val)

# Binary Tree Maximum Path Sum (LC 124)
def maxPathSum(root):
    max_sum = [root.val]   # list for mutable closure
    def dfs(node):
        if not node: return 0
        left_gain  = max(dfs(node.left),  0)   # ignore negative branches
        right_gain = max(dfs(node.right), 0)
        # can this node be the apex of the best path?
        max_sum[0] = max(max_sum[0], node.val + left_gain + right_gain)
        return node.val + max(left_gain, right_gain)   # one direction up to parent
    dfs(root)
    return max_sum[0]
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Gain Propagation DFS (Maximum Path Sum) in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a binary tree with integer node values (including negatives) and asked to find the maximum sum of any path — where a path is any sequence of connected nodes, not necessarily from root to leaf, and each node can appear at most once. The brute-force approach of enumerating every path between every pair of nodes is O(n²) for the enumeration alone, and it's not clear how to implement it cleanly."

**Task:** "My goal was to solve this in O(n) time with a single DFS pass by using a gain-propagation pattern — each node computes the best it can contribute to its parent while simultaneously evaluating whether it could be the apex of the best global path."

**Action:** Walk the interviewer through these steps:
1. *Two roles per node:* "Each node plays two roles: (a) it might be the 'apex' — the highest point of the optimal path, with the path bending left–node–right. (b) It contributes a 'one-sided gain' up to its parent — the path can only continue in one direction upward."
2. *Compute gains:* "I compute `left_gain = max(0, dfs(node.left))` and `right_gain = max(0, dfs(node.right))`. The `max(0, ...)` prunes negative subtrees — if a branch loses value, don't extend the path into it."
3. *Update global answer:* "At each node, the candidate apex path = `node.val + left_gain + right_gain`. I update a nonlocal `max_sum` with this candidate."
4. *Return to parent:* "I return `node.val + max(left_gain, right_gain)` — only the best single direction can continue upward; a path bending both ways cannot extend further."
5. *Base case and all-negative:* "I initialise `max_sum = [root.val]`, not 0 — because all nodes could be negative and the answer is still the single least-negative node. Returning 0 for None nodes (base case) combined with the `max(0, ...)` clamp handles this correctly."

**Result:** "O(n) time, O(h) space for the recursion stack — O(log n) for balanced trees. For a tree of 30,000 nodes this runs in milliseconds. The gain-propagation pattern generalises to diameter, maximum path with k edges, and sum-product variants."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer gain-propagation DFS |
|-------------|---------------------------|----------------------------------|
| Enumerate all root-to-leaf paths | Tree paths must start at root | Misses paths that start and end in the middle of the tree |
| Two-pointer on sorted paths | — | Paths in a tree have no sorted structure; inapplicable |
| Gain propagation DFS | Any tree, any path | O(n) single pass; generalises to many path-sum variants |

**Why NOT enumerate all paths:** A path from any node to any node can start and end anywhere — you can't enumerate them without O(n²) work. Gain propagation handles this by letting each node evaluate its apex role in O(1) during the DFS.

---

### Edge Cases to Trace Before Coding
- LC 199: single node → `[root.val]`; fully left-skewed tree → every depth has exactly one node (the leftmost) but it IS the rightmost too
- LC 1448: root is always good (initialise `max_so_far = root.val`); descending tree where every child < parent → only root is good
- LC 124: all negative values → answer = single least-negative node (do NOT return 0 from dfs; the outer `max_sum` starts at `root.val`); single-node tree → `root.val`

---

### Interview Pattern Drill
| Pattern | What DFS returns | Where answer lives | Example |
|---------|-----------------|-------------------|---------|
| Depth tracking | void | `result[depth]` | Right Side View |
| Running max/min | subtree count | accumulated return | Good Nodes |
| Gain propagation | one-sided max gain | nonlocal `max_sum` | Max Path Sum, Diameter |

---

## System Design (1 hour)
### Topic: Caching Fundamentals — Why, Where, and What to Measure

**Why caches exist:**
Disk I/O ≈ 10 ms. RAM access ≈ 100 ns. A DB query with disk read ≈ 50–200 ms. An in-memory cache hit ≈ 0.5 ms. For a service with 10 000 req/sec reading 80% cached data, caching eliminates 8 000 DB queries per second — turning a scaling crisis into a solved problem.

**Cache placement hierarchy:**
```
Request
  ↓
Browser / CDN (edge cache)       ← static assets, public API responses (hours–days TTL)
  ↓
Reverse Proxy (Nginx)            ← short-lived response cache (seconds–minutes)
  ↓
Application in-process cache     ← Python dict, Guava cache (single server, fast)
  ↓
Distributed cache                ← Redis / Memcached (shared across all app servers)
  ↓
Database query cache             ← deprecated in MySQL 8.0; avoid relying on it
  ↓
Disk / persistent storage
```

**Key metrics:**
- **Hit rate** = hits / (hits + misses). A 90 % hit rate means 90 % of reads never reach the DB.
- **Miss penalty** = extra latency when a miss forces a DB read + cache write.
- **Hit-rate vs. cache-size curve:** hit rate improves logarithmically with capacity — doubling from 1 GB → 2 GB might improve hit rate 85 % → 88 %, but 4 GB → 8 GB might gain < 1 %. Profile before provisioning.

**Cache-aside (lazy loading) — most common pattern:**
```
read(key):
  val = cache.get(key)
  if val is None:
    val = db.query(key)
    cache.set(key, val, ttl=300)
  return val
```
Data is only loaded on first access — no wasted writes for cold keys.
**Downside:** first request always misses; cache stampede on cold start (many concurrent misses all hitting the DB simultaneously).

**What NOT to cache:**
- Rapidly mutating data (stale reads are harmful) — use short TTL or skip cache
- Data that is already computed in microseconds — cache overhead > gain
- Very large values relative to cache capacity (evicts useful small entries)

**Interview talking point:** "For a social feed serving 100 M DAU, 95 % of traffic is reads. We cache each user's computed feed in Redis with a 5-minute TTL. On write (new post), we fan-out-on-write to followers with < 1 000 followers and lazy-invalidate for celebrity accounts — recomputing on next read avoids writing to millions of cache keys per post."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Invent and Simplify**

**STAR Story — Introducing a caching layer to relieve a bottleneck**

**Situation:** Our product page API was hitting a shared PostgreSQL database on every request to fetch a product's full attribute set — name, description, price, images, category, and 12 custom attributes. Each attribute required a separate JOIN across three tables. The product catalogue had 2 million SKUs, but our top 50,000 products accounted for 92% of traffic. At 8,000 product-page requests per second, the database was handling 96,000 SQL queries per second (12 JOINs × 8,000 requests) and running at 95% CPU, causing cascading timeouts.

**Task:** I was given two weeks to reduce database CPU to below 60% without a schema change. My goal was to identify the right caching strategy and implement it end-to-end, including cache invalidation on price or inventory updates.

**Action:** I profiled the access pattern: 92% of requests hit the same 50,000 products and those products' attributes changed at most a few times per day (price updates) or never (name, description). I designed a cache-aside Redis layer: on a product-page request, first check `Redis.GET product:{id}:attrs`. On a hit, return the cached JSON blob (typically 2–4 KB). On a miss, run the full SQL query, serialize the result to JSON, call `Redis.SET product:{id}:attrs {json} EX 300` (5-minute TTL), and return. For price/inventory updates, I added a write-through invalidation hook: on any UPDATE to the products table, the service called `Redis.DEL product:{id}:attrs` synchronously before committing, ensuring no stale price reads. I sized the Redis instance at 8 GB — 50,000 products × 4 KB average = 200 MB, so there was ample headroom. I load-tested at 10,000 req/sec using k6 and verified the hit rate stabilised above 94% after a 30-second warm-up period.

**Result:** Database CPU dropped from 95% to 34% within the first hour of production rollout. API P99 latency fell from 380 ms to 22 ms for cache hits. The database handled 10,000 req/sec with headroom to spare. Cache invalidation on price updates worked correctly — QA verified zero stale-price incidents across 200 test update cycles. The solution ran unchanged for 14 months before we migrated to a read replica.

---

## Flashcards

| Q | A |
|---|---|
| How does DFS right-side view work without BFS? | Visit right child before left; track depth; the first node reached at each new depth `d` is the rightmost — append to `result[d]` |
| What does Count Good Nodes propagate downward through DFS? | The maximum value seen on the path from root to the current node; a node is "good" if its value ≥ this running maximum |
| In Maximum Path Sum, why take `max(gain, 0)` for each child? | Negative-gain subtrees should be pruned — clamping at 0 means "don't extend the path into loss territory" |
| What is the cache-aside (lazy loading) pattern? | On miss: query DB, populate cache with TTL, return. On hit: return cached value. Cache is only written on demand — no wasted space for unread data |
| What does the hit-rate vs. cache-size curve look like, and why does it matter? | Logarithmic — each doubling of cache size yields diminishing hit-rate gains; use this curve to justify cache sizing decisions rather than guessing |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
