# Week 07 Weekly Review — Days 43–49
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Week Summary
This week covered the core of the Trees & BST slot, from BST validation and structural operations through advanced topics including tree DP, complete tree properties, and BST serialization.

**Topics covered:**
- BST validation, inorder invariant, Recover BST (Day 43)
- LCA in BST and general trees, Maximum Width with BFS indices (Day 44)
- Path collection DFS, root-to-leaf accumulation, greedy camera placement (Day 45)
- Tree construction from traversal pairs, vertical order traversal (Day 46)
- BST delete, minimum absolute difference, Maximum Sum BST subtree (Day 47)
- Tree DP (House Robber III), Complete Tree count, Redundant Connection II (Day 48)
- BST serialization, Duplicate Subtrees, Preorder recovery (Day 49)

**System design arc:** Completed the full Caching unit: Memcached vs Redis, eviction policies (LRU/LFU/TTL), write strategies (write-through/back/around), cache invalidation (TTL, event-driven, versioning, delete-on-write), distributed caching (consistent hashing, Redis Cluster), cache stampede solutions (mutex, PER, background refresh), Redis at scale (Sentinel, Cluster, Streams).

---

## 35 Flashcards — Full Week Review

### Day 43 — BST: Validate, Kth Smallest, Recover
| Q | A |
|---|---|
| How do you validate a BST without sorting? | DFS passing `(low, high)` bounds; left child gets `(low, node.val)`, right child gets `(node.val, high)`; invalid if `not (low < node.val < high)` |
| How does inorder traversal enable O(n) kth-smallest in a BST? | Inorder visits nodes in ascending order; decrement a counter on each visit; the node where counter hits 0 is the k-th smallest |
| How many inversions does Recover BST produce, and what do they represent? | At most 2 inversions in inorder; `first = prev at first inversion` (larger swapped node), `second = current at last inversion` (smaller swapped node); swap their values |
| What does Redis have that Memcached lacks? | Persistence (RDB/AOF), rich data structures (Sorted Set, Hash, List, Set, Stream), built-in HA (Sentinel/Cluster), pub/sub, Lua scripting |
| When is Memcached a better choice than Redis? | Pure string caching with no persistence, need multi-threaded CPU scaling on one large box, values are simple blobs ≤ 1 MB, no need for HA or data structures |

### Day 44 — LCA & Maximum Width
| Q | A |
|---|---|
| How does LCA of BST avoid a full DFS? | Compare both p.val and q.val to the current node; traverse left if both are smaller, right if both are larger; otherwise the current node is the LCA — O(h) |
| What does LCA of Binary Tree return from each DFS call? | The node itself if it equals p or q; propagates the non-null child result if only one subtree contains p or q; returns the current node when both subtrees return non-null |
| Why must you normalise BFS indices at the start of each level in Maximum Width? | Left-child indices double per level (2*i); without normalisation, a depth-30 tree produces indices > 2³⁰, causing integer overflow and corrupting width calculations |
| What is the key weakness of LFU cache eviction? | Frequency inflation — keys popular long ago accumulate high counts and resist eviction even after going cold; newly inserted keys start at frequency 1 and are evicted first |
| What Redis maxmemory-policy is best for a general-purpose cache? | `allkeys-lru` — evicts the least-recently-used key from ALL keys; combine with per-key TTLs for freshness |

### Day 45 — Tree Path Problems & Binary Tree Cameras
| Q | A |
|---|---|
| What must you do after recursing both children in Path Sum II? | Pop the current node from the path (backtrack) — restores the path to its state before entering this node so sibling branches see the correct path |
| Why does Sum Root to Leaf Numbers not need backtracking? | The accumulated number is an integer passed by value — each recursive call gets its own copy; no shared mutable state |
| In Binary Tree Cameras, what does a null child return and why? | Returns 1 (covered) — null nodes do not need monitoring; treating them as "already covered" prevents placing unnecessary cameras at every leaf |
| What is the difference between write-through and write-back caching? | Write-through: write to DB and cache synchronously (no data loss, higher write latency). Write-back: write to cache only, flush DB asynchronously (low latency, risk of data loss if cache crashes) |
| When should you use write-around caching? | When data is written once and may never be re-read — bulk imports, audit logs, media uploads; avoids polluting cache with data that won't be needed again |

### Day 46 — Tree Construction & Vertical Order
| Q | A |
|---|---|
| How do you find the left subtree size in Construct BT from Preorder+Inorder? | `left_size = inorder_idx[root_val] - in_start`; this tells how many nodes are in the left subtree and determines the split points in both preorder and inorder |
| What is the root in Construct BT from Inorder+Postorder? | `postorder[post_end]` (last element of the current postorder slice); same inorder split strategy as preorder+inorder |
| What makes LC 987 Vertical Order Traversal harder than basic column grouping? | When two nodes share the same (col, row), they must be sorted by value — sort key is `(col, row, val)` not just `(col, row)` |
| What is the safest cache invalidation pattern when writes are rare but reads must be fresh? | Cache-aside + delete-on-write: update DB, then `cache.delete(key)` — avoids race conditions from cache.set overwriting newer values |
| Why delete the cache key on write instead of setting the new value? | Setting can lose a race: two concurrent writes could update DB in order W1→W2 but set cache in order W2→W1, leaving stale W1 in cache; deleting is always safe |

### Day 47 — BST Operations & Maximum Sum BST
| Q | A |
|---|---|
| What is the inorder successor of a node in a BST and how do you find it? | The smallest node in the right subtree — traverse to `root.right` then follow `.left` until null |
| In Delete Node BST, why do you delete the successor from the right subtree after copying its value? | You placed the successor's value in the current node; the original successor node must be removed from the right subtree to maintain BST structure |
| How does Maximum Sum BST return range information for parent validity checks? | Returns `(min_val, max_val)` of the subtree; null nodes return `(+inf, -inf)` so the parent's `l_max < node.val < r_min` check always passes for absent children |
| What is the key property of consistent hashing that naive modular hashing lacks? | On node addition/removal, only `~1/N` keys migrate — not all keys; naive `hash(key) % N` remaps every key when N changes |
| What are Redis Cluster hash slots and why 16 384? | Each key maps to a slot via `CRC16(key) % 16384`; slots are distributed among master nodes; 16 384 balances granularity against the size of the slot→node mapping (2 KB) |

### Day 48 — Tree DP, Complete Tree, Redundant Edge
| Q | A |
|---|---|
| In House Robber III tree DP, what does each DFS call return? | A tuple `(rob_this, skip_this)` — the best sum if this node is robbed vs. skipped; the parent uses these values to compute its own rob/skip choices |
| How does Count Complete Tree Nodes achieve O(log² n) instead of O(n)? | At each node, compute left-path and right-path heights in O(log n); if equal, subtree is perfect → return 2^h - 1 without recursing; otherwise recurse on both sides |
| What are the two cases in Redundant Connection II? | Case 1: a node has in-degree 2 — two candidate edges; try removing the later one first. Case 2: a cycle with no double-parent — return the last edge that forms the cycle (union-find) |
| What is a cache stampede and what causes it? | A popular key expires; all concurrent requests miss simultaneously and all query the DB at once — potentially causing cascade DB failure |
| What is probabilistic early re-expiry (PER) for preventing stampedes? | Each request re-computes early with probability proportional to remaining TTL; as expiry approaches, more requests refresh proactively — spreading DB load across time |

### Day 49 — BST Serialization, Duplicate Subtrees, Preorder Recovery
| Q | A |
|---|---|
| Why can a BST be serialized without null markers? | The BST property means preorder values alone uniquely determine structure — deserializer uses value bounds `(low, high)` to decide left vs. right placement |
| How does Find Duplicate Subtrees uniquely identify a subtree? | Postorder canonical string `"left_str,right_str,node.val"`; two subtrees with identical shape and values produce the same string; HashMap tracks occurrence count |
| In Recover Tree from Preorder, how do you determine a node's parent? | Count leading dashes for depth `d`; pop stack until `len(stack) == d`; top is the parent; assign as left if None, else right |
| What is the minimum safe Redis Sentinel deployment and why? | 3 Sentinel processes — quorum requires a majority; with 2 Sentinels, a network split could cause two simultaneous master promotions (split-brain) |
| What is the key difference between Redis Pub/Sub and Redis Streams? | Pub/Sub: fire-and-forget, no persistence, message lost if no subscriber. Streams: messages persisted, consumer groups enable reliable at-least-once delivery with `XACK` |

---

## Pattern Quick Reference — Week 07

| Pattern | Trigger phrase | Complexity |
|---------|---------------|-----------|
| BST range bounds DFS | Validate BST, insert in BST | O(h) — O(log n) balanced |
| Inorder = sorted | Kth smallest, min diff, merge | O(n) time, O(h) space |
| Two-inversion tracking | Recover BST, find swapped pair | O(n) time, O(h) space |
| LCA post-order DFS | Ancestor of two nodes in BT | O(n) time, O(h) space |
| BFS with positional indices | Width, vertical order | O(n) time, O(n) space |
| Tree DP (rob/skip tuple) | House Robber III, any two-choice tree | O(n) time, O(h) space |
| Postorder canonical string | Duplicate subtrees, tree equality | O(n²) naive, O(n) with hash |

---

## Checklist
- [ ] Reviewed all 35 flashcards
- [ ] Able to state trigger conditions for all patterns without looking
- [ ] Rewrote at least 3 skeletons from memory (choose: Validate BST, LCA of BT, House Robber III, Recover BST, Duplicate Subtrees)
- [ ] Identified which problems you could not solve in 20 min and logged them
