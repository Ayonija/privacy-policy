# Day 47 — BST: Delete, Minimum Difference & Maximum Sum BST
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
BST delete requires handling three cases with the inorder successor. Minimum Absolute Difference exploits the sorted-inorder invariant. Maximum Sum BST in Binary Tree combines postorder DFS with a validity + range return — the hardest BST subproblem.

---

## DSA (2 hours)
### Pattern: BST Structural Manipulation + Postorder Subtree Aggregation

**Delete Node in a BST (LC 450):**
Three cases:
1. Node has no children → return `None`
2. Node has one child → return that child
3. Node has two children → replace the node's value with its inorder successor (the smallest value in the right subtree = `min(root.right)`). Then delete the successor from the right subtree recursively.

**Minimum Absolute Difference in BST (LC 530):**
Inorder traversal gives values in sorted ascending order. Track the previous node visited (`prev`). The minimum difference = minimum over all adjacent pairs = `min(node.val - prev.val)` for each consecutive (prev, node) pair in inorder.

**Maximum Sum BST in Binary Tree (LC 1373):**
Postorder DFS where each call returns a tuple `(is_valid_bst, min_val, max_val, subtree_sum)`. A node's subtree is a valid BST if:
- Both children subtrees are valid BSTs
- `left.max_val < node.val < right.min_val`
When valid, `sum = left.sum + node.val + right.sum`. Update global max when valid. When invalid, propagate `(False, ...)` upward (but min/max must still propagate correctly for parent validity checks — though once invalid, the parent will also be invalid).

**Trigger condition:**
- "delete a node from BST and maintain BST property" → 3-case delete; two-child case uses inorder successor
- "minimum difference between any two nodes in a BST" → inorder traversal, track prev
- "maximum sum of any valid BST subtree" → postorder DFS with (valid, min, max, sum) tuple

**Time complexity:** O(h) for LC 450, O(n) for LC 530 and LC 1373
**Space complexity:** O(h) for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Delete Node in a BST | 450 | Medium | 3-case structural BST delete | Two-child case: find inorder successor (min of right subtree), replace value, delete successor |
| 2 | Minimum Absolute Difference in BST | 530 | Medium | Inorder traversal + prev tracking | Inorder = sorted; min difference = minimum consecutive pair difference; track prev node |
| 3 | Maximum Sum BST in Binary Tree | 1373 | Hard | Postorder DFS returning (valid, min, max, sum) | Each node checks BST validity against children's ranges; update global max when valid |

---

### Code Skeleton
```java
class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Delete Node in BST (LC 450)
    public static TreeNode deleteNode(TreeNode root, int key) {
        if (root == null) return null;
        if (key < root.val) {
            root.left = deleteNode(root.left, key);
        } else if (key > root.val) {
            root.right = deleteNode(root.right, key);
        } else {
            // Found the node to delete
            if (root.left == null) return root.right;   // no left child
            if (root.right == null) return root.left;   // no right child
            // Two children: replace with inorder successor
            TreeNode successor = root.right;
            while (successor.left != null) {
                successor = successor.left;
            }
            root.val = successor.val;
            root.right = deleteNode(root.right, successor.val);
        }
        return root;
    }

    // Minimum Absolute Difference in BST (LC 530)
    private Integer prevVal = null;
    private int minDiff = Integer.MAX_VALUE;
    public int getMinimumDifference(TreeNode root) {
        inorder530(root);
        return minDiff;
    }
    private void inorder530(TreeNode node) {
        if (node == null) return;
        inorder530(node.left);
        if (prevVal != null) {
            minDiff = Math.min(minDiff, node.val - prevVal);
        }
        prevVal = node.val;
        inorder530(node.right);
    }

    // Maximum Sum BST in Binary Tree (LC 1373)
    private int maxSum = 0;
    // Returns int[]{isValid (1=true,0=false), minVal, maxVal, subtreeSum}
    public int maxSumBST(TreeNode root) {
        dfs1373(root);
        return maxSum;
    }
    private int[] dfs1373(TreeNode node) {
        if (node == null) {
            return new int[]{1, Integer.MAX_VALUE, Integer.MIN_VALUE, 0};
        }
        int[] left  = dfs1373(node.left);
        int[] right = dfs1373(node.right);
        if (left[0] == 1 && right[0] == 1 && left[2] < node.val && node.val < right[1]) {
            int total = left[3] + node.val + right[3];
            maxSum = Math.max(maxSum, total);
            return new int[]{1, Math.min(left[1], node.val), Math.max(right[2], node.val), total};
        }
        return new int[]{0, 0, 0, 0};
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 450: deleting the root with two children; deleting a node that doesn't exist (key not in tree) → recursion bottoms out without change
- LC 530: two-node tree → only one pair; all nodes have the same value difference → min_diff returns correctly from the single comparison
- LC 1373: all negative values → valid single-node BSTs have positive sums (node.val alone), but negative single nodes have sum < 0; max_sum initialised to 0, so an all-negative tree returns 0 correctly (the empty subtree sum is also valid)

---

### Interview Pattern Drill
| BST Operation | Traversal used | Key step |
|--------------|---------------|---------|
| Search | Iterative down tree | Compare val, go left/right |
| Insert | Iterative down tree | Create new node at null |
| Delete | Recursive | 3-case; two-child → inorder successor |
| Minimum difference | Inorder | Track prev, compute gap |
| Max BST sum in BT | Postorder | Return (valid, min, max, sum) 4-tuple |

---

## System Design (1 hour)
### Topic: Distributed Caching — Consistent Hashing & Cache Topology

**The sharding problem:**
A single Redis instance maxes out at ~100 GB RAM and ~100K ops/sec. For larger workloads, you need multiple cache nodes. How do you decide which node stores which key?

**Naive modular hashing:**
`node = hash(key) % N` — N = number of nodes.
Problem: if N changes (add or remove a node), `hash(key) % N` changes for EVERY key. All cached data becomes unreachable — a thundering herd of misses hits the DB simultaneously.

**Consistent hashing:**
Map both nodes and keys onto a virtual ring (hash space 0 to 2³²-1). Each key is stored on the first node clockwise from its hash position.

```
Ring:   0 ──── A ──── B ──── C ──── 2³²
Key k1: hash(k1) falls between A and B → stored on B
```

When a node is added (D between B and C): only keys between B and D migrate from C to D. All other keys are unaffected. On average, only `1/N` of keys migrate instead of all of them.

**Virtual nodes (vnodes):**
Each physical node is represented by multiple points on the ring (e.g., 150 virtual nodes per physical node). Benefits:
- Smoother key distribution (avoids hot spots when physical nodes' hash values cluster)
- Allows weighted distribution (a node with double RAM gets double the virtual nodes)

**Redis Cluster (built-in sharding):**
- Uses 16 384 hash slots (not a ring)
- `slot = CRC16(key) % 16384`
- Each master node owns a range of slots (e.g., node A: 0–5460, node B: 5461–10922, node C: 10923–16383)
- Adding a node: Redis migrates slots (and their keys) from existing nodes — online, no downtime
- Clients must be cluster-aware (use `redis-py` cluster client)
- Cross-slot transactions not supported — use hash tags `{user_id}:field` to force co-location on same slot

**Cache node failure handling:**
- Redis Cluster: each master has one or more replicas; on master failure, replica is promoted automatically
- Consistent hashing client library: on node failure, keys fall to the next node clockwise — the failed node's keys are temporarily computed against the next healthy node

**Interview talking point:** "If asked to design a distributed cache for 1 TB of hot data, answer: 10 nodes × 128 GB Redis each, using Redis Cluster with 16 384 slots distributed across nodes. For HA: each master has one replica in a different AZ. Key routing: CRC16 hash slot, handled transparently by the cluster client. Adding capacity: add new nodes and rebalance slots online — Redis will migrate keys automatically with no downtime."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to distribute work across multiple systems and design a routing mechanism to decide which system handles each piece — analogous to consistent hashing for distributed cache key assignment.
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| What is the inorder successor of a node in a BST and how do you find it? | The smallest node in the right subtree — traverse to `root.right` then follow `.left` until null |
| In Delete Node BST, why do you delete the successor from the right subtree after copying its value? | You've placed the successor's value in the current node; the original successor node must be removed from the right subtree to maintain BST structure |
| How does Maximum Sum BST return range information for parent validity checks? | Returns `(min_val, max_val)` of the subtree; null nodes return `(+inf, -inf)` so the parent's `l_max < node.val < r_min` check always passes for absent children |
| What is the key property of consistent hashing that naive modular hashing lacks? | On node addition/removal, only `~1/N` keys migrate — not all keys. Naive `hash(key) % N` remaps every key when N changes |
| What are Redis Cluster hash slots and why 16 384? | Each key maps to a slot via `CRC16(key) % 16384`; 16 384 slots are distributed among master nodes; 16 384 was chosen as a balance between granularity and the size of the slot→node mapping table (2 KB) |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
