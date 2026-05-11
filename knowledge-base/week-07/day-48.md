# Day 48 — Trees: Tree DP, Complete Tree Count & Redundant Edge
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
Tree DP (House Robber III) demonstrates the canonical "return two choices per node" pattern. Count Complete Tree Nodes uses binary search on tree height for a sub-linear O(log²n) solution. Redundant Connection II combines union-find with special-case analysis for directed graphs.

---

## DSA (2 hours)
### Pattern: Tree DP (Two-Choice Return) + Binary Search on Tree Height + Union-Find on Trees

**House Robber III (LC 337):**
Tree DP: at each node, you have two choices — rob this node or skip it.
- `rob(node)` = node.val + skip(node.left) + skip(node.right)
- `skip(node)` = max(rob(node.left), skip(node.left)) + max(rob(node.right), skip(node.right))

DFS returns `(rob_val, skip_val)` for each subtree. The answer for the root = `max(rob_val, skip_val)`.

**Count Complete Tree Nodes (LC 222):**
A complete binary tree has all levels full except possibly the last (filled left to right). Key insight: compute the height of the leftmost path and the rightmost path. If equal, the tree is a perfect binary tree → `2^h - 1` nodes. If not equal, recurse on both subtrees. Recursion depth is O(log n); each level computes height in O(log n) → total O(log² n).

**Redundant Connection II (LC 685):**
In a directed graph with n nodes (1 to n), n edges are given to form a rooted tree PLUS one extra edge. The extra edge creates either:
1. A node with two parents (in-degree 2)
2. A cycle (if in-degree check finds no double-parent)
3. Both (a node has two parents AND one candidate edge creates a cycle)

Strategy:
- Find any node with in-degree 2 → two candidate edges (the two edges pointing to it)
- Try removing the later candidate; check if the remaining graph forms a valid tree using union-find
- If removing the later candidate fixes it → return it; else return the earlier candidate
- If no double-parent node → cycle only → return the last edge that forms a cycle (standard union-find)

**Trigger condition:**
- "maximum sum of nodes with no two adjacent (tree)" → tree DP, return (rob, skip) pair per node
- "count nodes in complete binary tree faster than O(n)" → height check on leftmost/rightmost path, recurse
- "find the extra edge in a directed graph making an invalid rooted tree" → in-degree check + union-find

**Time complexity:** LC 337: O(n) | LC 222: O(log² n) | LC 685: O(n log n)
**Space complexity:** O(h) for LC 337/222, O(n) for LC 685

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | House Robber III | 337 | Medium | Tree DP with (rob, skip) return | Each node returns two values; parent combines children's values based on its own rob/skip choice |
| 2 | Count Complete Tree Nodes | 222 | Medium | Binary search on tree height | Left-height == right-height → perfect subtree → 2^h - 1; else recurse; O(log² n) |
| 3 | Redundant Connection II | 685 | Hard | In-degree + union-find | Find double-parent node (2 candidates); try removing each; if none → cycle-only → return last cycle edge |

---

### Code Skeleton
```java
class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // House Robber III (LC 337)
    public static int rob(TreeNode root) {
        int[] result = dfsRob(root);
        return Math.max(result[0], result[1]);
    }
    // returns [robThis, skipThis]
    private static int[] dfsRob(TreeNode node) {
        if (node == null) return new int[]{0, 0};
        int[] left  = dfsRob(node.left);
        int[] right = dfsRob(node.right);
        int robThis  = node.val + left[1] + right[1];
        int skipThis = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        return new int[]{robThis, skipThis};
    }

    // Count Complete Tree Nodes (LC 222)
    public static int countNodes(TreeNode root) {
        if (root == null) return 0;
        int lh = height(root, true);    // leftmost path height
        int rh = height(root, false);   // rightmost path height
        if (lh == rh) {
            return (1 << lh) - 1;   // 2^h - 1: perfect binary tree
        }
        return 1 + countNodes(root.left) + countNodes(root.right);
    }
    private static int height(TreeNode node, boolean goLeft) {
        int h = 0;
        while (node != null) {
            h++;
            node = goLeft ? node.left : node.right;
        }
        return h;
    }

    // Redundant Connection II (LC 685)
    public static int[] findRedundantDirectedConnection(int[][] edges) {
        int n = edges.length;
        int[] parent = new int[n + 1];
        for (int i = 0; i <= n; i++) parent[i] = i;   // union-find parent array

        // Step 1: find node with in-degree 2 (if any)
        int[] inDegree = new int[n + 1];
        for (int[] e : edges) inDegree[e[1]]++;

        int cand1 = -1, cand2 = -1;
        for (int i = n - 1; i >= 0; i--) {   // iterate backwards to find LATER candidate first
            if (inDegree[edges[i][1]] == 2) {
                if (cand2 == -1) {
                    cand2 = i;   // later edge (try removing this first)
                } else {
                    cand1 = i;   // earlier edge
                }
            }
        }

        // Step 2: try removing cand2 and see if remaining graph is valid
        for (int i = 0; i < n; i++) {
            if (i == cand2) continue;   // skip candidate 2
            int u = edges[i][0], v = edges[i][1];
            int pu = find(parent, u), pv = find(parent, v);
            if (pu == pv) {
                // cycle found
                return cand1 != -1 ? edges[cand1] : edges[i];
            }
            parent[pu] = pv;
        }
        return edges[cand2];   // removing cand2 made graph valid → cand2 is the answer
    }
    private static int find(int[] parent, int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];   // path compression
            x = parent[x];
        }
        return x;
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 337: single-node tree → `max(node.val, 0)` = node.val; linear chain tree → alternating rob/skip pattern
- LC 222: single level (only root) → lh = rh = 1 → returns `2^1 - 1 = 1`; perfectly full tree → returns answer in O(log n) not O(n)
- LC 685: exactly one edge forms a cycle (no double-parent) → cand1 = cand2 = None → union-find returns the cycle-forming edge; both conditions present → cand1 is returned (the earlier edge pointing to the double-parent node, since cand2's removal breaks the cycle)

---

### Interview Pattern Drill
| Pattern | What DFS returns | How children's values combine |
|---------|-----------------|-------------------------------|
| Tree DP (rob/skip) | `(rob, skip)` tuple | `rob = val + l_skip + r_skip`; `skip = max(l) + max(r)` |
| Subtree aggregation | Single aggregated value | Combine left + node + right |
| Postorder validity | `(valid, min, max, sum)` | Check bounds + propagate |

---

## System Design (1 hour)
### Topic: Cache Stampede & Thundering Herd — Prevention Strategies

**The problem:**
A popular cache key expires at time T. At T+1, hundreds (or thousands) of concurrent requests all get a cache miss simultaneously and all query the DB to recompute the value. The DB is suddenly hit with 1000× its normal load for that key — potentially causing a cascade failure.

**Strategy 1 — Mutex / Single-Flight (request coalescing):**
Only one request is allowed to rebuild the cache at a time. All other requests for the same key wait (block or return stale).
```python
# pseudocode with distributed lock
def get_with_lock(key):
    val = cache.get(key)
    if val: return val
    lock = redis.set(f"lock:{key}", 1, nx=True, ex=5)  # acquire lock
    if lock:
        val = db.query(key)
        cache.set(key, val, ex=300)
        redis.delete(f"lock:{key}")
        return val
    else:
        # Another request is rebuilding — return stale or wait briefly
        time.sleep(0.05)
        return cache.get(key)   # try again
```
- Pros: DB sees exactly 1 query per cache miss for a given key
- Cons: other requests block or see a brief stale response; lock implementation requires care

**Strategy 2 — Probabilistic Early Expiry (PER):**
Before the TTL actually expires, re-compute with a probability that increases as expiration approaches. Spreads re-computation load over time instead of all at once.
```python
import random, math, time
def get_with_per(key, beta=1.0):
    val, expiry = cache.get_with_expiry(key)
    now = time.time()
    # re-compute early with probability proportional to time-to-expiry
    if not val or now - beta * math.log(random.random()) > expiry:
        val = db.query(key)
        cache.set(key, val, ex=300)
    return val
```
- Pros: no locks; distributed; no single point of coordination
- Cons: some requests do extra work slightly early; implementation is less intuitive

**Strategy 3 — Background refresh (proactive re-computation):**
A background job re-computes and updates the cache entry shortly before it expires, so it never actually expires for live traffic.
```
# background worker:
while True:
    for key in tracked_hot_keys:
        if cache.ttl(key) < 30:   # re-compute when 30 seconds remain
            val = db.query(key)
            cache.set(key, val, ex=300)
    sleep(5)
```
- Pros: zero misses for tracked keys; no blocking
- Cons: must know which keys are "hot" in advance; over-fetches cold keys

**Strategy 4 — Stale-While-Revalidate:**
Return the stale (expired) value immediately, and trigger a background recompute asynchronously. Used extensively in HTTP caches (`Cache-Control: stale-while-revalidate`).
- Pros: zero additional latency for the client; recompute is off the critical path
- Cons: clients may receive stale data for one request cycle after expiry

**Comparison:**
| Strategy | User-visible latency | DB load on expiry | Implementation complexity |
|---------|---------------------|-------------------|--------------------------|
| Mutex | Slightly elevated (wait) | 1 query | Medium (distributed lock) |
| PER | Normal | Spread over time | Low (random + math) |
| Background refresh | Zero | Proactive | Medium (key tracking) |
| Stale-while-revalidate | Zero (stale) | 1 query async | Low |

**Interview talking point:** "For a high-traffic product page cache, I'd use stale-while-revalidate for most data (acceptable to show 5 seconds of stale content) combined with a distributed mutex for critical data like inventory counts, where serving stale data could cause overselling. For keys that are always hot (home page, trending items), background refresh eliminates misses entirely."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time your system experienced a cascade failure or surge when a shared resource became unavailable — what happened, what did you do, and what did you change to prevent recurrence?
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| In House Robber III tree DP, what does each DFS call return? | A tuple `(rob_this, skip_this)` — the best sum if this node is robbed vs. skipped; parent uses these to compute its own rob/skip choices |
| How does Count Complete Tree Nodes achieve O(log² n) instead of O(n)? | At each node, compute left-path height and right-path height in O(log n); if equal, the subtree is perfect → return 2^h - 1 without recursing into it; otherwise recurse on both children |
| What are the two cases in Redundant Connection II? | Case 1: a node has in-degree 2 (two candidate edges — try removing the later one first). Case 2: a cycle with no double-parent node (return the last edge that forms the cycle via union-find) |
| What is a cache stampede and what causes it? | When a popular cached key expires, all concurrent requests miss simultaneously and all hit the DB at once — potentially causing DB overload and cascade failure |
| What is the probabilistic early re-expiry (PER) strategy for preventing stampedes? | Each request checks whether to re-compute early with probability proportional to remaining TTL; as expiry approaches, more requests proactively refresh — spreading DB load across time |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
