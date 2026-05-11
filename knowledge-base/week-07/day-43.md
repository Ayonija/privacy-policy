# Day 43 — BST: Validate, Kth Smallest & Recover
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
BST properties: inorder traversal produces a sorted sequence. Use this invariant to validate BST with range bounds, extract the k-th element without full sort, and detect the two swapped nodes that break a BST.

---

## DSA (2 hours)
### Pattern: BST Inorder Invariant & Bound Propagation

**Validate Binary Search Tree (LC 98):**
DFS with `(low, high)` bounds. At each node, verify `low < node.val < high`. Pass updated bounds down: left child gets `(low, node.val)`; right child gets `(node.val, high)`. Using `float('-inf')` and `float('inf')` as initial bounds handles the root cleanly.

**Kth Smallest Element in a BST (LC 230):**
BST inorder traversal (left → node → right) visits nodes in ascending sorted order. Use a counter; decrement on each visit; when the counter reaches 0, the current node is the answer. Works iteratively too with an explicit stack.

**Recover Binary Search Tree (LC 99):**
Exactly two nodes were swapped. During inorder traversal, a swap creates at most two "inversions" (places where the current node's value < the previous node's value). Track `first` and `second`:
- `first` = the larger node at the first inversion (prev node)
- `second` = the smaller node at the latest inversion (current node)
After traversal, swap `first.val` and `second.val`.

*Edge case:* if the two swapped nodes are adjacent in inorder, there is only one inversion — `second` is updated only once.

**Trigger condition:**
- "is this a valid BST?" → range-bound DFS, propagate `(low, high)` bounds
- "kth element in BST without extra sort" → inorder traversal with counter
- "two nodes in a BST were swapped — restore it" → inorder traversal tracking two inversions

**Time complexity:** O(n) for all | **Space complexity:** O(h) — O(log n) balanced, O(n) skewed

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Validate Binary Search Tree | 98 | Medium | DFS with range bounds | Pass `(low, high)` bounds; left child: upper bound = node.val; right child: lower bound = node.val |
| 2 | Kth Smallest Element in a BST | 230 | Medium | Inorder traversal + counter | Inorder = sorted; decrement counter; return current node when counter hits 0 |
| 3 | Recover Binary Search Tree | 99 | Hard | Inorder traversal, track two inversions | First inversion: `first = prev`; any subsequent inversion: `second = current`; swap values at the end |

---

### Code Skeleton
```java
class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Validate BST (LC 98)
    public static boolean isValidBST(TreeNode root) {
        return dfs(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }
    private static boolean dfs(TreeNode node, long low, long high) {
        if (node == null) return true;
        if (!(low < node.val && node.val < high)) return false;
        return dfs(node.left, low, node.val) && dfs(node.right, node.val, high);
    }

    // Kth Smallest in BST (LC 230)
    private int count = 0;
    private int result = 0;
    public int kthSmallest(TreeNode root, int k) {
        inorder(root, k);
        return result;
    }
    private void inorder(TreeNode node, int k) {
        if (node == null || count >= k) return;
        inorder(node.left, k);
        count++;
        if (count == k) {
            result = node.val;
            return;
        }
        inorder(node.right, k);
    }

    // Recover BST (LC 99) — O(n) time, O(h) space
    TreeNode first = null, second = null, prev = null;
    public void recoverTree(TreeNode root) {
        inorderRecover(root);
        int tmp = first.val;
        first.val = second.val;
        second.val = tmp;
    }
    private void inorderRecover(TreeNode node) {
        if (node == null) return;
        inorderRecover(node.left);
        if (prev != null && prev.val > node.val) {
            if (first == null) {
                first = prev;     // larger node at first inversion
            }
            second = node;        // smaller node (update on every inversion)
        }
        prev = node;
        inorderRecover(node.right);
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 98: `[5, 1, 4, null, null, 3, 6]` — node 4 violates `(5, inf)` bound; single-node tree → valid; `INT_MIN`/`INT_MAX` values — use `float('-inf')/float('inf')` not integer bounds
- LC 230: k = n (last element = largest); use iterative version if recursion depth is a concern for skewed tree
- LC 99: adjacent swap (e.g., positions 2 and 3 in inorder) → only one inversion fires; `second` is set on the first (and only) inversion — code handles this because `second = node` on every inversion, including the first

---

### Interview Pattern Drill
| BST Property | How to use it | Example |
|-------------|---------------|---------|
| Inorder = sorted | Visit left → node → right for sorted output | Kth smallest, sorted merge |
| Range bounds | Pass `(low, high)` constraints down | Validate BST, insert position |
| Two inversions in inorder | Two nodes were swapped | Recover BST |
| BST height | log₂(n) for balanced → determines time complexity | Binary search, LCA in BST |

---

## System Design (1 hour)
### Topic: Redis vs. Memcached — When to Use Each

**Redis:**
- Single-threaded event loop (fast, predictable latency; no lock contention)
- Rich data structures: String, List, Set, Sorted Set, Hash, Stream, HyperLogLog, Bitmap, Geo
- Optional persistence: RDB snapshots + AOF (append-only file)
- Replication: master-replica with async replication
- Pub/Sub and Streams (message queue capabilities)
- Lua scripting for atomic multi-step operations
- Cluster mode for horizontal sharding

**Memcached:**
- Multi-threaded — scales to multiple CPU cores natively
- Simple key-value only (string keys, string/binary values)
- No persistence — memory only; all data is lost on restart
- No replication built-in (must be handled at the client level with consistent hashing)
- Extremely memory-efficient slab allocator (no per-key overhead)
- Maximum key size 250 bytes; maximum value size 1 MB

**Head-to-head decision table:**
| Dimension | Redis | Memcached |
|-----------|-------|-----------|
| Data structures | Rich (7+ types) | String/binary only |
| Persistence | RDB + AOF optional | None |
| Replication/HA | Built-in Sentinel/Cluster | Client-side only |
| CPU scaling | Single-threaded per shard | Multi-threaded |
| Memory overhead | ~80 bytes per key | ~56 bytes per key |
| Pub/Sub | Yes | No |
| Atomic multi-step ops | Lua scripts, transactions | No |
| Max value size | 512 MB | 1 MB |

**When to choose Memcached:**
- Pure caching workload with no need for persistence or data structures
- Need to squeeze maximum throughput from many CPU cores on a single machine
- Values are simple blobs (HTML fragments, serialised JSON) that don't need server-side manipulation
- Team already operates Memcached and Redis adds no benefit

**When to choose Redis (almost always):**
- Need any rich data structure (sorted sets for leaderboards, sets for unique counts, etc.)
- Need persistence (cache-as-database pattern, session store that survives restarts)
- Need pub/sub or Streams for lightweight messaging
- Need Lua scripting for atomic operations (rate limiter, distributed lock)
- Need built-in HA with Sentinel or Cluster

**Interview talking point:** "In almost all new projects, I'd choose Redis — it's a strict superset of Memcached's functionality for caching, and its rich data structures often eliminate the need for a separate DB for derived data. The only reason I'd choose Memcached today is if I had a single large cache-only box and needed to saturate all CPU cores, or if the team had existing Memcached infrastructure and no use case for Redis's extra features."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you chose a simpler tool over a more feature-rich one — or vice versa — and how you made the call. Relate to the Redis vs. Memcached trade-off.
- Leadership principle: Bias for Action

---

## Flashcards

| Q | A |
|---|---|
| How do you validate a BST without sorting? | DFS passing `(low, high)` bounds; left child gets `(low, node.val)`, right child gets `(node.val, high)`; node is invalid if `not (low < node.val < high)` |
| How does inorder traversal enable O(n) kth-smallest in a BST? | Inorder visits nodes in ascending order; decrement a counter on each visit; the node where counter hits 0 is the k-th smallest |
| How many inversions does Recover BST produce, and what do they represent? | At most 2 inversions in inorder (a swap between distant nodes produces 2; adjacent swap produces 1); `first = prev at first inversion`, `second = current at last inversion`; swap their values |
| What does Redis have that Memcached lacks? | Persistence (RDB/AOF), rich data structures (Sorted Set, Hash, List, Set, Stream), built-in HA (Sentinel/Cluster), pub/sub, Lua scripting |
| When is Memcached a better choice than Redis? | Pure string caching with no persistence, need multi-threaded CPU scaling on one large box, values are simple blobs ≤ 1 MB, no need for HA or data structures |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
