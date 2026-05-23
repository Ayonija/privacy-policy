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
```java
import java.util.*;

class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Subtree of Another Tree (LC 572)
    public static boolean isSubtree(TreeNode root, TreeNode subRoot) {
        if (root == null) return false;
        if (isSameTree(root, subRoot)) return true;
        return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
    }
    private static boolean isSameTree(TreeNode s, TreeNode t) {
        if (s == null && t == null) return true;
        if (s == null || t == null)  return false;
        return s.val == t.val && isSameTree(s.left, t.left) && isSameTree(s.right, t.right);
    }

    // All Nodes Distance K (LC 863)
    public static List<Integer> distanceK(TreeNode root, TreeNode target, int k) {
        // Phase 1: add parent pointers via DFS
        Map<TreeNode, TreeNode> parent = new HashMap<>();
        addParents(root, null, parent);

        // Phase 2: BFS from target for exactly k steps
        Set<TreeNode> visited = new HashSet<>();
        visited.add(target);
        Deque<TreeNode> queue = new ArrayDeque<>();
        queue.addLast(target);
        int dist = 0;
        while (!queue.isEmpty() && dist < k) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.pollFirst();
                for (TreeNode neighbour : new TreeNode[]{node.left, node.right, parent.get(node)}) {
                    if (neighbour != null && !visited.contains(neighbour)) {
                        visited.add(neighbour);
                        queue.addLast(neighbour);
                    }
                }
            }
            dist++;
        }
        List<Integer> result = new ArrayList<>();
        for (TreeNode node : queue) result.add(node.val);
        return result;
    }
    private static void addParents(TreeNode node, TreeNode par, Map<TreeNode, TreeNode> parent) {
        if (node == null) return;
        parent.put(node, par);
        addParents(node.left, node, parent);
        addParents(node.right, node, parent);
    }

    // Sum of Distances in Tree (LC 834)
    public static int[] sumOfDistancesInTree(int n, int[][] edges) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
        for (int[] e : edges) {
            graph.get(e[0]).add(e[1]);
            graph.get(e[1]).add(e[0]);
        }
        int[] count  = new int[n];   // subtree size (including self)
        int[] answer = new int[n];
        Arrays.fill(count, 1);

        // Pass 1: root at node 0 — compute subtree sizes and answer[0]
        dfs1(0, -1, graph, count, answer);

        // Pass 2: reroot — propagate answers from parent to child
        dfs2(0, -1, n, graph, count, answer);

        return answer;
    }
    private static void dfs1(int node, int par, List<List<Integer>> graph, int[] count, int[] answer) {
        for (int child : graph.get(node)) {
            if (child == par) continue;
            dfs1(child, node, graph, count, answer);
            count[node] += count[child];
            answer[node] += answer[child] + count[child];
        }
    }
    private static void dfs2(int node, int par, int n, List<List<Integer>> graph, int[] count, int[] answer) {
        for (int child : graph.get(node)) {
            if (child == par) continue;
            // moving root from node to child:
            // count[child] nodes each get 1 closer; (n - count[child]) each get 1 farther
            answer[child] = answer[node] - count[child] + (n - count[child]);
            dfs2(child, node, n, graph, count, answer);
        }
    }
}
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Rerooting DFS (Sum of Distances in Tree) in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an unweighted tree of n nodes and asked to return an array where `result[i]` = sum of distances from node i to every other node. The brute-force approach of running a separate BFS from each of the n nodes is O(n²) — for n = 30,000 nodes that's 900 million operations, which times out."

**Task:** "My goal was to solve this in O(n) using a two-pass rerooting DFS that computes all n answers from a single rooting of the tree."

**Action:** Walk the interviewer through these steps:
1. *Pass 1 — root at node 0:* "I DFS from node 0. For each node v, I compute `count[v]` = total nodes in v's subtree (including itself), and `answer[0]` += `answer[child] + count[child]` (each node in the child's subtree is one step farther from node 0 than from the child itself)."
2. *Why `answer[0]` is special:* "After Pass 1, `answer[0]` is the sum of distances from node 0 to all other nodes — exactly the BFS result for node 0. I now want to 'reroot' this to compute every other node's answer without another BFS."
3. *Pass 2 — reroot from parent to child:* "Moving the root from node p to child node v: `count[v]` nodes each get 1 step closer (they were one step farther from p through v's subtree). `n - count[v]` nodes each get 1 step farther. The formula: `answer[v] = answer[p] - count[v] + (n - count[v])`."
4. *DFS pass 2:* "I DFS again from node 0, applying the reroot formula for each child. Each child's answer is computed in O(1) from its parent's answer."
5. *Correctness intuition:* "The formula captures exactly what happens when you pick up the root and place it on a child: the subtree 'below' the new root is n - count[v] nodes (which now have to go one more hop), and the subtree 'above' is count[v] nodes (which saved one hop)."

**Result:** "O(n) time, O(n) space. Two DFS passes. For n = 30,000 that's 60,000 operations instead of 900 million — the difference between milliseconds and seconds."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer rerooting DFS |
|-------------|---------------------------|--------------------------|
| BFS from every node | n ≤ 1,000 | O(n²) — times out for n ≥ 10,000 |
| Floyd-Warshall | Small graphs (n ≤ 500) | O(n³) space and time — much worse |
| Rerooting DFS (two-pass) | Standard | O(n) time, O(n) space; the canonical solution |

**Why NOT BFS from every node:** For n = 30,000, O(n²) = 900 M ops at 10⁸ ops/sec = 9 seconds. The rerooting approach does the same work in O(n) = 60,000 ops in under 1 ms.

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

**Leadership principle: Dive Deep**

**STAR Story — End-to-end caching problem: diagnosis, strategy selection, measurement**

**Situation:** Six weeks after a major content recommendation feature launched, our backend's average response time drifted from 180 ms to 740 ms. No alarm fired during the drift — it was slow enough to stay within our loose 95th-percentile SLO. A customer success team member noticed users were complaining about the site "feeling sluggish" and escalated to engineering.

**Task:** I was the senior engineer on call and took ownership of the investigation. My goal was to identify the exact cause of the latency regression, implement a fix, and verify improvement — all within one business day to prevent further user impact.

**Action:** I started with the observability dashboard: 95th-percentile DB query time had gone from 12 ms to 410 ms on the recommendations query. I ran `EXPLAIN ANALYZE` on the slow query — it was a 6-table JOIN to compute personalised recommendations, running a sequential scan on a 40 M-row events table. The events table had grown 4× since launch (from 10 M rows when the index was last analysed to 40 M rows now). Running `ANALYZE recommendations_events` refreshed the statistics — the query planner immediately switched to an index scan, reducing query time from 410 ms to 22 ms. But 40 M rows was still growing, and personalised recommendations for the same user rarely changed within a session. I designed a cache-aside layer: recommendations for each user cached in Redis as a JSON string with a 5-minute TTL. A write-through hook invalidated the cache on explicit user preference changes. I load-tested at 3,000 req/sec and verified a 94% hit rate within 2 minutes of warm-up.

**Result:** After deploying the ANALYZE fix + Redis caching, average response time dropped from 740 ms to 95 ms — a 7.8× improvement. DB query load fell by 88% during peak hours. I added `ANALYZE` to the nightly maintenance job so statistics would auto-refresh. I also set a DB response-time alerting threshold at 50 ms (previously had none for the recommendations service), so any future drift would be caught within minutes instead of weeks.

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
