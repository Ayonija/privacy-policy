# Day 50 — Trees & Caching: Slot 5 Full Synthesis
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
Slot 5 close-out: synthesise tree traversal techniques with Subtree Matching (DFS equality), All Nodes Distance K (graph-style BFS on trees), and Sum of Distances in Tree (rerooting — the most advanced DFS technique in this slot). Synthesise the complete caching decision framework.

---

## DSA (2 hours)
### Pattern: Tree DFS Matching + BFS from Interior Node + Rerooting DFS

**Subtree of Another Tree (LC 572):**
For each node in the main tree, check if the subtree rooted there is identical to `subRoot`. `isSameTree(s, t)`: both null → True; one null → False; values match AND both children match recursively. O(m×n) naive; can be optimised by comparing Merkle-style serialised strings (O(m+n)).

**All Nodes Distance K in Binary Tree (LC 863):**
Two-phase approach:
1. DFS to add parent pointers to each node (converting the tree into an undirected graph)
2. BFS from `target` node for exactly `K` steps, respecting visited set to avoid cycles

The parent pointer is the key insight — without it you can only traverse downward; with it you can traverse upward and sideways.

**Sum of Distances in Tree (LC 834):**
Classic rerooting technique. Two DFS passes on an unweighted undirected tree:
- Pass 1 (root at node 0): compute `count[v]` = size of subtree rooted at `v`; compute `answer[0]` = sum of all distances from root to every other node
- Pass 2 (reroot to each child): when moving root from parent `p` to child `v`, `count[v]` nodes get 1 closer and `n - count[v]` nodes get 1 farther:
  `answer[v] = answer[p] - count[v] + (n - count[v])`

**Trigger condition:**
- "does tree S contain a subtree identical to tree T?" → check `isSameTree` at every node of S; or serialise + string match
- "all nodes exactly K hops from a target node" → add parent pointers, then BFS from target
- "sum of distances from every node to all others" → rerooting DFS; two-pass O(n)

**Time complexity:** LC 572: O(m×n) | LC 863: O(n) | LC 834: O(n)
**Space complexity:** O(h) for LC 572/863, O(n) for LC 834

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Subtree of Another Tree | 572 | Medium | DFS matching at every root | isSameTree helper; call at every node; early termination once found |
| 2 | All Nodes Distance K in Binary Tree | 863 | Medium | Parent pointer DFS + BFS from target | Parent pointers make tree traversable as undirected graph; BFS explores K steps in all directions |
| 3 | Sum of Distances in Tree | 834 | Hard | Rerooting DFS — two-pass | Pass 1: subtree sizes + root's total distance. Pass 2: reroot formula `answer[v] = answer[parent] - count[v] + (n - count[v])` |

---

### Code Skeleton
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# Subtree of Another Tree (LC 572)
def isSubtree(root, subRoot):
    def isSameTree(s, t):
        if not s and not t: return True
        if not s or not t:  return False
        return s.val == t.val and isSameTree(s.left, t.left) and isSameTree(s.right, t.right)
    if not root: return False
    if isSameTree(root, subRoot): return True
    return isSubtree(root.left, subRoot) or isSubtree(root.right, subRoot)

# All Nodes Distance K (LC 863)
from collections import defaultdict, deque
def distanceK(root, target, k):
    # Phase 1: add parent pointers via DFS
    parent = {}
    def add_parents(node, par):
        if not node: return
        parent[node] = par
        add_parents(node.left, node)
        add_parents(node.right, node)
    add_parents(root, None)

    # Phase 2: BFS from target for exactly k steps
    visited = {target}
    queue = deque([target])
    dist = 0
    while queue and dist < k:
        for _ in range(len(queue)):
            node = queue.popleft()
            for neighbour in (node.left, node.right, parent[node]):
                if neighbour and neighbour not in visited:
                    visited.add(neighbour)
                    queue.append(neighbour)
        dist += 1
    return [node.val for node in queue]

# Sum of Distances in Tree (LC 834)
def sumOfDistancesInTree(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    count  = [1] * n    # subtree size (including self)
    answer = [0] * n

    # Pass 1: root at node 0 — compute subtree sizes and answer[0]
    def dfs1(node, parent):
        for child in graph[node]:
            if child == parent: continue
            dfs1(child, node)
            count[node] += count[child]
            answer[node] += answer[child] + count[child]
    dfs1(0, -1)

    # Pass 2: reroot — propagate answers from parent to child
    def dfs2(node, parent):
        for child in graph[node]:
            if child == parent: continue
            # moving root from node to child:
            # count[child] nodes each get 1 closer; (n - count[child]) each get 1 farther
            answer[child] = answer[node] - count[child] + (n - count[child])
            dfs2(child, node)
    dfs2(0, -1)

    return answer
```

---

### Edge Cases to Trace Before Coding
- LC 572: `subRoot` is a single leaf node — `isSameTree` handles this; `root` is null but `subRoot` is not → return False immediately
- LC 863: `k = 0` → return `[target.val]`; target is the root → BFS explores all directions correctly (parent[root] = None, guarded by `if neighbour`)
- LC 834: single-node tree (`n=1`) → all distances are 0 → `answer = [0]`; path graph (every node has degree ≤ 2) → rerooting still works; trace the formula manually for n=3 star graph before coding

---

### Interview Pattern Drill: Slot 5 Complete Reference

| Pattern | Trigger phrase | Complexity | Key implementation |
|---------|---------------|-----------|-------------------|
| DFS right-first | Right side view | O(n) | First node at depth = rightmost |
| Gain propagation | Max path sum | O(n) | `max(child_gain, 0)` + global max |
| BFS level snapshot | Level order, zigzag | O(n) | `level_size = len(queue)` |
| BFS serialize | Encode tree → string | O(n) | Null sentinels; queue-based deserialize |
| BST bounds DFS | Validate BST | O(h) | Pass `(low, high)` down |
| Inorder traversal | Kth smallest, min diff | O(n) | Inorder = sorted for BST |
| Two-inversion inorder | Recover BST | O(n) | first = prev at 1st inversion; second = curr at last |
| LCA post-order | LCA of binary tree | O(n) | Both non-null → return current |
| BFS with indices | Max width, vertical order | O(n log n) | Normalise index at level start |
| DFS backtrack | Path Sum II | O(n) | Push before, pop after (backtrack) |
| Greedy postorder | Binary Tree Cameras | O(n) | 3-state return (0/1/2); +1 if root=0 |
| Divide & conquer | Construct from traversals | O(n) | inorder index map; root identity |
| Tree DP | House Robber III | O(n) | Return `(rob, skip)` tuple |
| Rerooting DFS | Sum of Distances | O(n) | Two passes; `ans[v] = ans[p] - cnt[v] + (n-cnt[v])` |
| Parent pointers + BFS | Distance K in tree | O(n) | DFS to add parents; BFS K steps |
| BST serialization | Serialize/deserialize BST | O(n) | Preorder only; bounds-based deserialize |

---

## System Design (1 hour)
### Topic: Full Caching System Design Synthesis — Decision Framework

**Complete caching decision framework:**

```
New performance problem or system design question involving data access?
  │
  ├─ Is data mostly READ (>80% reads)?
  │    └─ YES → cache it
  │         ├─ Is data user-specific? → in-memory distributed cache (Redis) keyed by user ID
  │         └─ Is data public/shared? → CDN + distributed cache
  │
  ├─ How frequently does the data CHANGE?
  │    ├─ Rarely (hours/days) → long TTL (3600s – 86400s)
  │    ├─ Often (seconds/minutes) → short TTL (10s – 300s) OR event-driven invalidation
  │    └─ Real-time → skip cache OR serve stale with stale-while-revalidate
  │
  ├─ What data STRUCTURE best models this data?
  │    ├─ Simple value → String
  │    ├─ Ranked list → Sorted Set
  │    ├─ Unique set of members → Set / HyperLogLog (at scale)
  │    ├─ Object with multiple fields → Hash
  │    └─ Queue / activity feed → List / Stream
  │
  ├─ What WRITE strategy?
  │    ├─ Strong consistency required → write-through (update DB + cache together)
  │    ├─ Write-heavy, small loss OK → write-back (cache only; async DB flush)
  │    └─ Write-once, read-rarely → write-around (DB only; cache on first read)
  │
  ├─ What EVICTION policy?
  │    ├─ General cache → allkeys-lru
  │    ├─ Some keys must never evict → volatile-lru (only evict TTL keys)
  │    ├─ Clear hot/cold split → allkeys-lfu
  │    └─ Data must expire at known time → volatile-ttl
  │
  └─ What SCALE/HA topology?
       ├─ Dataset fits one node, need HA → Redis Sentinel (1 master + replicas + 3 Sentinels)
       ├─ Dataset too large for one node → Redis Cluster (16 384 hash slots, 3+ masters)
       └─ Simple cache, multi-core machine → Memcached (multi-threaded, string-only)
```

**The 5 caching questions you must answer cold in an interview:**

1. **What data should be cached?** High read frequency, slow to compute or fetch, acceptable staleness window.

2. **When does cached data go stale?** Determines TTL length or whether event-driven invalidation is needed.

3. **How do you prevent cache stampede?** Mutex lock for critical keys; probabilistic early expiry for general keys; background refresh for permanently hot keys.

4. **What happens when the cache goes down?** Cache-aside falls back to DB automatically (increased latency, not failure). Write-back risks data loss — must have DB durability guarantees.

5. **How do you scale the cache?** Redis Sentinel for HA on a single shard; Redis Cluster for horizontal scale; consistent hashing at the client layer for Memcached.

**Synthesis interview talking point:** "Designing a cache for an e-commerce product catalogue with 10 M SKUs, 99 % read traffic, and eventual consistency acceptable: Redis Cluster with `allkeys-lru` policy; product data cached as Hash (HSET `product:{id}` field value); 1-hour TTL with delete-on-write invalidation on product updates; Redis Cluster for HA and scale across 3 shards × 2 replicas. For the home page and category pages, add a CDN layer with 5-minute TTL — the cache serves the cache."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock session — pick one problem from each difficulty
- **Medium 1:** LC 572 Subtree of Another Tree (target: 15 min)
- **Medium 2:** LC 863 All Nodes Distance K (target: 20 min)
- **Hard:** LC 834 Sum of Distances in Tree (target: 30 min)

After each problem: state time complexity, space complexity, and one edge case aloud before submitting.

---

## Behavioral (30 min)
- STAR prompt: Walk through a caching problem you've encountered end-to-end — from noticing the symptom (high latency, DB load) through diagnosing the cause, selecting the caching strategy, and measuring the result — mirroring the full decision framework.
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does Subtree of Another Tree check equality at each node? | Calls `isSameTree(s, t)`: both null → True; one null → False; else `s.val == t.val and isSameTree(s.left, t.left) and isSameTree(s.right, t.right)` |
| What are the two phases of All Nodes Distance K? | Phase 1: DFS to add parent pointers (makes tree into undirected graph). Phase 2: BFS from target for K steps using parent/left/right as 3 possible neighbours |
| What is the rerooting formula for Sum of Distances in Tree? | `answer[child] = answer[parent] - count[child] + (n - count[child])` — moving root to child: `count[child]` nodes each gain 1 distance unit saved; `n - count[child]` nodes each gain 1 more |
| What is the complete 5-question caching checklist? | (1) What data to cache? (2) When does it go stale? (3) How to prevent stampede? (4) What happens if cache goes down? (5) How to scale the cache? |
| When do you choose Redis Sentinel vs Redis Cluster? | Sentinel: dataset fits one node, need HA only. Cluster: dataset too large for one node, need horizontal sharding + built-in HA |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/20/30 min, no hints)
- [ ] Rewrote rerooting DFS skeleton from memory (most complex pattern of the slot)
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced rerooting formula manually on a 5-node example tree
- [ ] Completed system design section + recited the 5 caching questions cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
