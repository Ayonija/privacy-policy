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
- STAR prompt: Describe a time you had to make a trade-off between data freshness and system performance — analogous to choosing between event-driven invalidation (always fresh) and TTL-based expiry (simpler but stale).
- Leadership principle: Insist on the Highest Standards

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
