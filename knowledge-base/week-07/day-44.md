# Day 44 — Trees: Lowest Common Ancestor & Maximum Width
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
LCA of BST exploits sort order for O(h) decisions. LCA of a general binary tree uses the "both non-null → current node is LCA" post-order pattern. Maximum Width uses BFS index arithmetic to measure level spans.

---

## DSA (2 hours)
### Pattern: LCA Traversal + BFS Index Arithmetic

**Lowest Common Ancestor of a BST (LC 235):**
Exploit the BST invariant. At each node:
- If both p and q are less than the current node → LCA is in the left subtree
- If both p and q are greater → LCA is in the right subtree
- Otherwise (one ≤ current ≤ other, or current == p or q) → current node IS the LCA

This is O(h) — no need to traverse both subtrees.

**Lowest Common Ancestor of a Binary Tree (LC 236):**
Post-order DFS. For each node, recursively search left and right:
- If both sides return non-null → this node is the LCA (p is in one subtree, q in the other)
- If only one side is non-null → propagate that non-null result upward (the LCA is deeper)
- If this node equals p or q → return this node (it is an ancestor of the other)

**Maximum Width of Binary Tree (LC 662):**
BFS with index tracking: root = index 0. Left child of node at index `i` = `2*i`; right child = `2*i + 1`. At each level, width = `last_index - first_index + 1`. Normalise indices at the start of each level (subtract the first index) to prevent 64-bit integer overflow for deep trees.

**Trigger condition:**
- "LCA in BST" → traverse left/right based on value comparison, no full DFS needed
- "LCA in general binary tree" → post-order: return self if p or q found; LCA is the node where both subtrees return non-null
- "width of a binary tree level" → BFS with positional indices; normalise to avoid overflow

**Time complexity:** LC 235: O(h) | LC 236: O(n) | LC 662: O(n)
**Space complexity:** O(h) for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Lowest Common Ancestor of a BST | 235 | Medium | BST value comparison | No full DFS — traverse left if both < node, right if both > node, else current is LCA |
| 2 | Lowest Common Ancestor of a Binary Tree | 236 | Medium | Post-order DFS, propagate non-null | Both subtrees return non-null → current is LCA; one returns non-null → propagate it |
| 3 | Maximum Width of Binary Tree | 662 | Hard | BFS with positional indices | Index left child = 2*i, right child = 2*i+1; normalise at level start to prevent overflow |

---

### Code Skeleton
```java
class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // LCA of BST (LC 235)
    public static TreeNode lowestCommonAncestorBST(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode node = root;
        while (node != null) {
            if (p.val < node.val && q.val < node.val) {
                node = node.left;
            } else if (p.val > node.val && q.val > node.val) {
                node = node.right;
            } else {
                return node;   // split point — current node is LCA
            }
        }
        return null;
    }

    // LCA of Binary Tree (LC 236)
    public static TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode left  = lowestCommonAncestor(root.left,  p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left != null && right != null) return root;   // p and q are in different subtrees
        return left != null ? left : right;  // LCA is deeper on whichever side found a node
    }

    // Maximum Width of Binary Tree (LC 662)
    public static int widthOfBinaryTree(TreeNode root) {
        if (root == null) return 0;
        int maxWidth = 0;
        java.util.Deque<long[]> queue = new java.util.ArrayDeque<>();
        queue.addLast(new long[]{0, 0}); // placeholder; we'll store [node_index] pairs
        // Use a queue of [node, index] pairs
        java.util.Deque<Object[]> q = new java.util.ArrayDeque<>();
        q.addLast(new Object[]{root, 0L});
        while (!q.isEmpty()) {
            int size = q.size();
            long firstIdx = ((Long) ((Object[]) q.peekFirst())[1]);
            long lastIdx = firstIdx;
            for (int i = 0; i < size; i++) {
                Object[] pair = q.pollFirst();
                TreeNode node = (TreeNode) pair[0];
                long idx = (Long) pair[1] - firstIdx;  // normalise to prevent overflow
                lastIdx = idx;
                if (node.left  != null) q.addLast(new Object[]{node.left,  2 * idx});
                if (node.right != null) q.addLast(new Object[]{node.right, 2 * idx + 1});
            }
            maxWidth = Math.max(maxWidth, (int)(lastIdx + 1));
        }
        return maxWidth;
    }
}
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining LCA in a General Binary Tree in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a binary tree and two nodes p and q, and asked to find their lowest common ancestor — the deepest node that has both p and q as descendants. Unlike a BST, there's no ordering property to exploit, so I can't navigate left or right based on values. Brute-force would be to find the root-to-p path and the root-to-q path, then find the last common node — O(n) time but O(n) extra space for the path lists."

**Task:** "My goal was to solve this in O(n) time and O(h) space using a single post-order DFS pass without storing any paths."

**Action:** Walk the interviewer through these steps:
1. *Post-order DFS:* "I use post-order traversal — process children before the current node — because I need to know what's in the subtrees before making a decision at the current node."
2. *Base case:* "If the current node is null, return null. If the current node equals p or q, return it immediately — we don't need to search further; we've found one of the targets."
3. *Query left and right:* "I recursively call `dfs(left)` and `dfs(right)`. Each returns either null (neither p nor q in that subtree) or a non-null node (p, q, or their LCA)."
4. *Decision logic:* "If both `left` and `right` are non-null → p is in one subtree and q is in the other → the current node is their LCA, return it. If only one side is non-null → the LCA is in that subtree → propagate the non-null result upward. If both are null → return null."
5. *Why this handles one-is-ancestor-of-the-other:* "If p is an ancestor of q, when we reach p we return it immediately (base case 2), without searching further. The left/right calls above p will bubble that p node all the way to the root — the correct answer."

**Result:** "O(n) time — each node visited exactly once. O(h) space for the recursion stack — O(log n) for balanced trees. No extra path storage needed. The three-case return logic is the key pattern: null/null → null; one non-null → propagate; both non-null → current node is LCA."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer post-order DFS |
|-------------|---------------------------|--------------------------|
| Store root-to-node paths, find last common node | If paths are needed for other purposes | O(n) extra space for paths; two DFS passes |
| Euler tour + range minimum query | Ultra-fast repeated LCA queries on same tree | O(n log n) preprocessing; overkill for single queries |
| Post-order DFS (single pass) | Single or few LCA queries | O(n) time, O(h) space, no preprocessing |

**Why NOT path storage:** Two separate DFS passes (one for p, one for q) plus path comparison is correct but uses O(n) space for paths. Post-order DFS finds the LCA in one pass with O(h) space.

---

### Edge Cases to Trace Before Coding
- LC 235: one node IS the ancestor of the other (e.g., p = root) → the while loop hits `p.val == node.val` and returns immediately; both nodes equal root → returns root
- LC 236: p or q does not exist in tree — problem guarantees both exist; one is ancestor of the other → the first-found node is returned upward and becomes the answer
- LC 662: perfect binary tree → indices blow up exponentially; normalise by subtracting first index at every level start (this is why `idx -= first_idx` is inside the level loop, applied immediately to the popped index)

---

### Interview Pattern Drill
| Problem | DFS direction | What is returned | When answer is found |
|---------|--------------|-----------------|---------------------|
| LCA of BST | Iterative left/right | The LCA node | When values straddle or equal current node |
| LCA of BT | Post-order | Non-null result bubbles up | When both left and right are non-null |
| Max Width | BFS with indices | Width of each level | After each level completes |

---

## System Design (1 hour)
### Topic: Cache Eviction Policies — LRU, LFU, TTL, and Hybrids

**Why eviction policies matter:**
A cache is bounded in size. When it is full and a new key must be inserted, the eviction policy decides which existing entry to remove. The wrong policy evicts hot data, driving up miss rates.

**LRU — Least Recently Used:**
Evict the entry that was accessed (read or written) least recently. Works on the intuition that recently used data is likely to be used again (temporal locality).
- Implementation: doubly-linked list + HashMap (like LC 146).
- Best for: workloads with strong temporal locality (e.g., web page cache, session cache).
- Weakness: a "scan" pattern (sequential one-time access to many keys) evicts useful data for keys that are only read once.

**LFU — Least Frequently Used:**
Evict the entry with the lowest access count. Tie-break with LRU (evict least-recently-used among equal-frequency entries).
- Implementation: three HashMaps (like LC 460).
- Best for: workloads with clear hot/cold data (some keys always popular, others rarely accessed).
- Weakness: newly inserted keys start with frequency 1 — will be evicted before old keys that were hot long ago but are no longer accessed (frequency inflation problem).

**TTL — Time To Live:**
Each entry expires after a fixed duration regardless of access pattern. Simplest strategy.
- Best for: data with natural freshness windows (API responses, computed feeds, rate-limit windows).
- Implementation: Redis `EXPIRE key seconds` on any key type.
- Weakness: no intelligence — a key accessed 10 000 times expires at the same time as one accessed once.

**FIFO — First In, First Out:**
Evict the oldest inserted entry. Rarely used in caches; useful when cache should be a bounded sliding window of recent events.

**Hybrid: TTL + LRU (most production systems):**
Set a TTL to ensure freshness + use LRU as the eviction policy within the TTL window. Redis supports this with `maxmemory-policy allkeys-lru` combined with per-key TTLs.

**Redis maxmemory-policy options:**
| Policy | Eviction pool | Strategy |
|--------|--------------|----------|
| `noeviction` | — | Returns error on write when full |
| `allkeys-lru` | All keys | LRU across all keys |
| `volatile-lru` | Keys with TTL | LRU only among expiring keys |
| `allkeys-lfu` | All keys | LFU across all keys (Redis 4.0+) |
| `volatile-lfu` | Keys with TTL | LFU among expiring keys |
| `allkeys-random` | All keys | Random eviction |
| `volatile-ttl` | Keys with TTL | Evict soonest-to-expire first |

**Redis LFU implementation detail:** Redis uses an approximated LFU (Morris counter) — each key has a 24-bit field: 16 bits for last-accessed timestamp (minute resolution), 8 bits for frequency counter that decays over time. This uses ~24 bytes per key, not a full sorted structure.

**Interview talking point:** "If asked what eviction policy to use for a content feed cache with 10 M keys, answer: `allkeys-lru` with a 10-minute TTL on each key. LRU keeps recently requested feeds warm; TTL ensures even popular feeds are refreshed so they don't serve stale content indefinitely. If some keys are permanently hot (e.g., viral content), `allkeys-lfu` may perform better — profile both."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Dive Deep**

**STAR Story — Balancing recency vs. frequency when prioritising limited resources**

**Situation:** Our team maintained an API gateway that cached JWT validation results for incoming tokens. The cache held a maximum of 500,000 tokens at any given time using LRU eviction. We began observing a pattern where legitimate long-session users — enterprise customers who kept sessions open for 8+ hours — were getting randomly logged out during business hours. Investigation revealed that a batch import job was generating thousands of short-lived one-off API tokens every hour, flooding the LRU cache and evicting the long-session enterprise tokens that were being used continuously.

**Task:** I was asked to investigate the root cause and propose a solution that kept both enterprise long-session users and high-volume batch jobs working correctly without increasing the cache size budget.

**Action:** I pulled cache access logs for 72 hours and built a frequency-vs-recency scatter plot for cached tokens. The pattern was clear: enterprise tokens had very high access frequency (hundreds of hits per session) but moderate recency (last hit minutes ago). Batch tokens had access frequency of 1 and very recent timestamps. LRU was perfectly preserving the batch tokens — they had just been inserted — while evicting the enterprise tokens that were in active use. I prototyped a switch to LFU eviction policy in a staging environment. I then realised LFU alone wasn't enough: new enterprise tokens would still start at frequency 1 and be vulnerable during their first minute of existence. I added a hybrid: use LFU for keys with frequency > 5 (proven to be in active use), and LRU as a fallback for new entries below that threshold. I implemented this as a two-tier eviction policy in our custom caching layer and load-tested it with both enterprise session patterns and batch import patterns simultaneously.

**Result:** Enterprise session cache evictions dropped from 340 per hour to zero during the following week. Batch import continued to work correctly. Enterprise user "unexpected logout" support tickets dropped from 12 per week to 0. The two-tier eviction policy added 3% memory overhead but eliminated the entire class of problem.

---

## Flashcards

| Q | A |
|---|---|
| How does LCA of BST avoid a full DFS? | Compare both p.val and q.val to the current node; traverse left if both are smaller, right if both are larger; otherwise the current node is the LCA — O(h) |
| What does LCA of Binary Tree return from each DFS call? | The node itself if it equals p or q; the non-null child result if only one subtree contains p or q; the current node if both subtrees return non-null |
| Why must you normalise BFS indices at the start of each level in Maximum Width? | Left-child indices double each level (2*i); without normalisation, a depth-30 tree produces indices > 2³⁰, overflowing 32-bit integers and corrupting width calculations |
| What is the key weakness of LFU cache eviction? | Frequency inflation — keys that were popular long ago accumulate high counts and resist eviction even after becoming cold; newly inserted keys start at frequency 1 and are evicted first |
| What Redis maxmemory-policy is best for a general-purpose cache? | `allkeys-lru` — evicts the least-recently-used key from ALL keys (not just expiring ones); combine with per-key TTLs for freshness |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
