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
- STAR prompt: Describe a time you identified a bottleneck in a system and introduced a caching layer or equivalent optimisation to relieve it — walk through the symptom, solution, and what metric you measured to confirm the fix worked.
- Leadership principle: Invent and Simplify

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
