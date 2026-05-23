# Day 46 — Trees: Construction from Traversals & Vertical Order
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
Reconstruct binary trees from two traversal arrays by exploiting the invariant that preorder/postorder identify the root and inorder separates left/right subtrees. Vertical order traversal combines BFS with column/row metadata sorting.

---

## DSA (2 hours)
### Pattern: Divide-and-Conquer Tree Construction + BFS with Coordinate Metadata

**Construct BT from Preorder + Inorder (LC 105):**
Preorder[0] is always the root. Find that root's index in the inorder array — everything to the left is the left subtree; everything to the right is the right subtree. Recurse with matching sub-arrays. Use a `{value: index}` map of the inorder array for O(1) lookups, making the overall algorithm O(n) instead of O(n²).

**Construct BT from Inorder + Postorder (LC 106):**
Postorder[-1] is always the root. Same split strategy using inorder. Recurse with the rightmost slice of postorder for the right subtree first (to match natural traversal order).

**Vertical Order Traversal (LC 987):**
BFS, tracking `(col, row)` for each node. Left child: `(col-1, row+1)`. Right child: `(col+1, row+1)`. Collect all `(col, row, val)` triples. Sort by `(col, row, val)`. Group by column, collecting sorted values per column.

**Trigger condition:**
- "build tree from preorder + inorder" → preorder[0] = root; inorder split by root index
- "build tree from inorder + postorder" → postorder[-1] = root; same split
- "vertical order / column grouping" → BFS with column tracking; sort by (col, row, val)

**Time complexity:** LC 105/106: O(n) | LC 987: O(n log n) for sort
**Space complexity:** O(n) for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Construct BT from Preorder and Inorder | 105 | Medium | Divide-and-conquer; inorder root split | preorder[0] = root; inorder index map for O(1) split; recurse left/right with matching sub-slices |
| 2 | Construct BT from Inorder and Postorder | 106 | Medium | Same divide-and-conquer; root is postorder[-1] | postorder[-1] = root; same inorder map; left subtree size determines postorder split point |
| 3 | Vertical Order Traversal | 987 | Hard | BFS with (col, row) metadata + sort | Collect (col, row, val) triples; sort by (col, row, val); group by col — handles ties by value |

---

### Code Skeleton
```java
import java.util.*;

class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Construct BT from Preorder + Inorder (LC 105)
    public static TreeNode buildTreePreIn(int[] preorder, int[] inorder) {
        Map<Integer, Integer> inorderIdx = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) inorderIdx.put(inorder[i], i);
        return buildPreIn(preorder, inorderIdx, 0, preorder.length - 1, 0, inorder.length - 1);
    }
    private static TreeNode buildPreIn(int[] preorder, Map<Integer, Integer> inorderIdx,
                                       int preStart, int preEnd, int inStart, int inEnd) {
        if (preStart > preEnd) return null;
        int rootVal = preorder[preStart];
        TreeNode root = new TreeNode(rootVal);
        int mid = inorderIdx.get(rootVal);         // root's position in inorder
        int leftSize = mid - inStart;
        root.left  = buildPreIn(preorder, inorderIdx, preStart + 1, preStart + leftSize, inStart, mid - 1);
        root.right = buildPreIn(preorder, inorderIdx, preStart + leftSize + 1, preEnd,   mid + 1,  inEnd);
        return root;
    }

    // Construct BT from Inorder + Postorder (LC 106)
    public static TreeNode buildTreeInPost(int[] inorder, int[] postorder) {
        Map<Integer, Integer> inorderIdx = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) inorderIdx.put(inorder[i], i);
        return buildInPost(inorder, postorder, inorderIdx, 0, inorder.length - 1, 0, postorder.length - 1);
    }
    private static TreeNode buildInPost(int[] inorder, int[] postorder, Map<Integer, Integer> inorderIdx,
                                        int inStart, int inEnd, int postStart, int postEnd) {
        if (inStart > inEnd) return null;
        int rootVal = postorder[postEnd];
        TreeNode root = new TreeNode(rootVal);
        int mid = inorderIdx.get(rootVal);
        int leftSize = mid - inStart;
        root.left  = buildInPost(inorder, postorder, inorderIdx, inStart, mid - 1, postStart, postStart + leftSize - 1);
        root.right = buildInPost(inorder, postorder, inorderIdx, mid + 1,  inEnd,  postStart + leftSize, postEnd - 1);
        return root;
    }

    // Vertical Order Traversal (LC 987)
    public static List<List<Integer>> verticalTraversal(TreeNode root) {
        if (root == null) return new ArrayList<>();
        List<int[]> nodes = new ArrayList<>();  // [col, row, val]
        Deque<Object[]> queue = new ArrayDeque<>();
        queue.addLast(new Object[]{root, 0, 0});   // (node, col, row)
        while (!queue.isEmpty()) {
            Object[] item = queue.pollFirst();
            TreeNode node = (TreeNode) item[0];
            int col = (int) item[1], row = (int) item[2];
            nodes.add(new int[]{col, row, node.val});
            if (node.left  != null) queue.addLast(new Object[]{node.left,  col - 1, row + 1});
            if (node.right != null) queue.addLast(new Object[]{node.right, col + 1, row + 1});
        }
        nodes.sort((a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] != b[1] ? a[1] - b[1] : a[2] - b[2]);
        List<List<Integer>> result = new ArrayList<>();
        Map<Integer, List<Integer>> colMap = new TreeMap<>();
        for (int[] n : nodes) {
            colMap.computeIfAbsent(n[0], k -> new ArrayList<>()).add(n[2]);
        }
        result.addAll(colMap.values());
        return result;
    }
}
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Tree Construction from Traversal Arrays in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given preorder and inorder traversal arrays of a binary tree with unique values and asked to reconstruct the original tree. The brute-force approach of scanning the full inorder array to find the root at each recursion level is O(n²) — for a tree with 100,000 nodes that's 10 billion operations, which times out."

**Task:** "My goal was to reconstruct the tree in O(n) by precomputing a value-to-index map of the inorder array so every root-lookup is O(1)."

**Action:** Walk the interviewer through these steps:
1. *Key invariant — preorder:* "The first element of any preorder slice is always the root of that subtree. For preorder `[3, 9, 20, 15, 7]`, root = 3."
2. *Key invariant — inorder:* "Everything to the left of the root in the inorder array is the left subtree; everything to the right is the right subtree. For inorder `[9, 3, 15, 20, 7]`, root 3 is at index 1; left subtree = `[9]`; right subtree = `[15, 20, 7]`."
3. *Build the index map:* "Before any recursion, I build `inorder_idx = {value: index}` for the full inorder array — O(n) one-time cost. Each root lookup is O(1) during recursion."
4. *Compute left subtree size:* "`left_size = inorder_idx[root_val] - in_start`. This tells me how many nodes are in the left subtree, which determines the preorder split point."
5. *Recurse:* "Left child: preorder `[pre_start+1, pre_start+left_size]`, inorder `[in_start, mid-1]`. Right child: preorder `[pre_start+left_size+1, pre_end]`, inorder `[mid+1, in_end]`. Each call processes a disjoint slice — O(n) total."

**Result:** "O(n) time with the HashMap, O(n) space. Without the HashMap it's O(n²). For 100,000 nodes: O(n) ≈ 100,000 operations vs O(n²) ≈ 10 billion — the difference between sub-millisecond and several seconds."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer HashMap + divide-and-conquer |
|-------------|---------------------------|----------------------------------------|
| Scan inorder linearly at each level | n ≤ 1,000 | O(n²) — acceptable for small trees, times out for n = 100,000 |
| Iterative reconstruction with stack | Stack-safe for very deep trees | Complex; divide-and-conquer is cleaner and equivalent in time |
| HashMap + divide-and-conquer | Standard | O(n) time, O(n) space; the canonical solution |

**Why NOT linear scan:** O(n²) on n = 10⁵ is 10¹⁰ ops at 10⁹ ops/sec ≈ 10 seconds. The HashMap lookup is the single most impactful optimization — a 100,000× speedup for the largest inputs.

---

### Edge Cases to Trace Before Coding
- LC 105: single-element tree → `preorder = inorder = [x]` → one-node tree; all values in one subtree (no left children) → left_size = 0, so left recursion terminates immediately
- LC 106: single element; verify postorder slicing — right subtree uses postorder from `post_start + left_size` to `post_end - 1` (excluding the root at post_end)
- LC 987: two nodes at same `(col, row)` → sort by `val` to break ties (this is the critical difference from "normal" vertical order in LC 314, which only sorts by col then row)

---

### Interview Pattern Drill
| Construction input | Root identity | Left/right split method |
|-------------------|--------------|------------------------|
| Preorder + Inorder | preorder[0] | Inorder index of root |
| Inorder + Postorder | postorder[-1] | Inorder index of root |
| Preorder + Postorder | preorder[0] (or postorder[-1]) | Left subtree root = preorder[1]; find in postorder |

---

## System Design (1 hour)
### Topic: Cache Invalidation — Strategies and Trade-offs

**The cache invalidation problem:** Cache holds a copy of data. When the source of truth (DB) changes, the cache copy becomes stale. How do you ensure the cache reflects up-to-date data without defeating the purpose of caching?

**Strategy 1 — TTL-based expiry (simplest):**
Set a short TTL on every cache entry. Accept that data can be stale by up to TTL seconds.
```
cache.set(key, val, ex=60)   # stale for at most 60 seconds
```
- Good for: news feeds, product listings, public APIs — users tolerate a few seconds of staleness
- Bad for: bank balances, inventory counts, anything needing strong consistency

**Strategy 2 — Event-driven invalidation:**
On every DB write, publish an event. A subscriber deletes (or updates) the relevant cache key.
```
# on write path:
db.update(user_id, new_email)
events.publish("user.updated", user_id)

# cache invalidation service:
on "user.updated" event → cache.delete(f"user:{user_id}")
```
- Good for: strong consistency requirements; data that must be fresh immediately after a write
- Bad for: complex fan-out (one DB write invalidates thousands of cache keys — e.g., a tweet invalidates all followers' feeds)

**Strategy 3 — Cache key versioning:**
Embed a version number in the cache key. On write, increment the version in the DB. Clients always read the key with the current version number; old keys expire naturally.
```
# key format: "user:{id}:v{version}"
version = db.get_version(user_id)      # read current version
val = cache.get(f"user:{user_id}:v{version}")
```
- Good for: immutable key semantics; CDN cache busting (CSS/JS files with hash in filename)
- Bad for: complex version management at scale

**Strategy 4 — Write-through with immediate invalidation:**
On write: update DB, then `cache.set(key, new_val)` (not delete). No stale window because the cache is updated atomically with the DB write.
- Combines write-through + no invalidation needed (cache always current)
- Risk: race condition if two concurrent writes update DB and cache in different orders → one stale write wins in cache

**Cache-aside + delete-on-write (most common safe pattern):**
```
write(key, val):
    db.write(key, val)
    cache.delete(key)   # DELETE, not set — next read will repopulate from DB
```
Deleting (rather than setting) avoids a race where a stale cache.set could overwrite a more recent DB write.

**Interview talking point:** "If asked how Twitter handles cache invalidation when a celebrity with 10 M followers tweets, answer: fan-out-on-write is too expensive (10 M cache writes per tweet). Instead, use fan-out-on-read for celebrities: when a follower loads their feed, merge the celebrity's recent tweets on the fly from a small dedicated cache for that celebrity. For regular users (< 10 K followers), fan-out-on-write is fine — their follower list is small."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Insist on the Highest Standards**

**STAR Story — Trade-off between data freshness and system performance**

**Situation:** Our platform displayed real-time product availability counts on the product detail page — "Only 3 left in stock!" These counts were cached in Redis with a 10-minute TTL. The business had seen multiple customer complaints of clicking "Add to Cart" on a product showing stock, only to receive an out-of-stock error at checkout — a frustrating experience that correlated with a 22% cart abandonment rate for those affected sessions.

**Task:** I owned the inventory display layer and was asked to reduce the discrepancy between displayed stock and actual stock to under 30 seconds, without increasing database load by more than 20%.

**Action:** I audited when stock counts changed in the 10-minute windows. The majority of discrepancies occurred during flash sales and restock events — brief periods of rapid inventory change that happened predictably. I designed a two-layer invalidation strategy: TTL-based for normal periods (10-minute TTL, acceptable staleness) plus event-driven invalidation on significant stock changes. I added a Kafka consumer that listened to inventory update events and only fired `cache.delete(product:{id})` when the stock count changed by more than 10% or dropped below 10 units — the two thresholds that correlated with customer-visible discrepancies. This kept the event fan-out small: 98% of inventory changes were small fluctuations that didn't warrant immediate cache invalidation. I load-tested the Kafka consumer at 5× peak event rate to verify it wouldn't bottleneck under a flash sale spike.

**Result:** Stock-count discrepancy at checkout dropped from an average of 8 minutes (half the TTL) to under 25 seconds for the triggering cases. Cart abandonment due to out-of-stock errors fell from 22% to 6% for affected sessions. Database read load increased by only 11% (within the 20% budget) because the invalidation threshold filtered 98% of events. The TTL continued to handle normal traffic with zero overhead.

---

## Flashcards

| Q | A |
|---|---|
| How do you find the left subtree size in Construct BT from Preorder+Inorder? | `left_size = inorder_idx[root_val] - in_start`; this tells you how many nodes are in the left subtree, and you slice both preorder and inorder accordingly |
| What is the root in Construct BT from Inorder+Postorder? | `postorder[post_end]` (last element of the current postorder slice); everything to its left in inorder is the left subtree |
| What makes LC 987 Vertical Order Traversal harder than a basic column grouping? | When two nodes share the same (col, row), they must be sorted by value — the sort key is `(col, row, val)` not just `(col, row)` |
| What is the safest cache invalidation pattern when writes are rare but reads must be fresh? | Cache-aside + delete-on-write: update DB, then `cache.delete(key)` — the next read misses and repopulates from DB; avoids race conditions from cache.set overwriting newer values |
| Why do you delete the cache key on write instead of setting the new value? | Setting can lose a race: two concurrent writes W1 and W2 could commit to DB in order W1→W2 but set cache in order W2→W1, leaving stale W1 in cache; deleting is safe because the next reader always fetches from DB |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
