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
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# LCA of BST (LC 235)
def lowestCommonAncestorBST(root, p, q):
    node = root
    while node:
        if p.val < node.val and q.val < node.val:
            node = node.left
        elif p.val > node.val and q.val > node.val:
            node = node.right
        else:
            return node   # split point — current node is LCA

# LCA of Binary Tree (LC 236)
def lowestCommonAncestor(root, p, q):
    if not root or root == p or root == q:
        return root
    left  = lowestCommonAncestor(root.left,  p, q)
    right = lowestCommonAncestor(root.right, p, q)
    if left and right:   # p and q are in different subtrees
        return root
    return left or right  # LCA is deeper on whichever side found a node

# Maximum Width of Binary Tree (LC 662)
from collections import deque
def widthOfBinaryTree(root):
    if not root: return 0
    max_width = 0
    queue = deque([(root, 0)])     # (node, index)
    while queue:
        level_size = len(queue)
        _, first_idx = queue[0]
        for _ in range(level_size):
            node, idx = queue.popleft()
            idx -= first_idx        # normalise to prevent overflow
            if node.left:  queue.append((node.left,  2 * idx))
            if node.right: queue.append((node.right, 2 * idx + 1))
        max_width = max(max_width, idx + 1)   # idx is last node's normalised index
    return max_width
```

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
- STAR prompt: Describe a time you had to balance recency vs. frequency of use when prioritising resources — analogous to choosing between LRU and LFU eviction.
- Leadership principle: Dive Deep

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
