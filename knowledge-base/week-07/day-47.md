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

### STAR Interview Framework

> **How to use the STAR method when explaining BST Delete (three-case) in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to delete a node from a BST while maintaining the BST property. The complication is the three-child-state cases — a node might have zero, one, or two children, and each requires a different structural response. The hardest case — two children — requires replacing the deleted node's value with its inorder successor and then deleting that successor."

**Task:** "My goal was to implement the delete operation in O(h) time — O(log n) for balanced BSTs, O(n) for skewed — with a clean recursive structure that handles all three cases without special-casing the recursion direction."

**Action:** Walk the interviewer through these steps:
1. *Search down the tree:* "Recurse left if `key < root.val`, right if `key > root.val`. Set `root.left` or `root.right` to the result of the recursive call — this handles structure repair automatically as the recursion unwinds."
2. *Case 1 — no children:* "If both children are null, return null. The parent's `root.left = deleteNode(...)` call receives null and detaches this node."
3. *Case 2 — one child:* "If only left is null, return right. If only right is null, return left. The single child replaces the deleted node."
4. *Case 3 — two children:* "Find the inorder successor: the smallest node in the right subtree (traverse right once, then follow left until null). Copy the successor's value into the current node. Then recursively delete the successor from the right subtree. The right subtree is guaranteed to be a valid BST after this."
5. *Why inorder successor, not predecessor:* "Either works. Inorder successor (smallest in right subtree) is the most common interview answer and guarantees the replacement value maintains the BST property: it's larger than the entire left subtree and smaller than everything in the right subtree."

**Result:** "O(h) time — O(log n) for balanced BSTs. The recursive structure automatically handles node relinking as the stack unwinds — no manual pointer updates needed outside the recursion."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer recursive inorder successor |
|-------------|---------------------------|----------------------------------------|
| Lazy deletion (mark nodes as deleted, don't restructure) | When deletions are rare and reads dominate | Avoids restructuring but inflates tree size and degrades BST operations over time |
| Replace with inorder predecessor | Equivalent correctness | Both work; successor is more commonly expected in interviews |
| Iterative delete | Avoid recursion stack for very deep trees | Significantly more complex code; recursion is cleaner |

**Why NOT lazy deletion:** Lazy deletion works for occasional deletes, but in a BST that experiences frequent deletions it causes the tree to grow in memory without bound and degrades search performance as deleted markers accumulate.

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

**Leadership principle: Think Big**

**STAR Story — Designing a routing mechanism to distribute work across multiple systems**

**Situation:** Our data processing platform was growing from handling 2 million events per day to a projected 50 million events per day within six months. All events were being written to a single PostgreSQL instance and processed by a single queue worker. The single node was already running at 85% CPU during peak hours, and early modelling showed it would be completely saturated at 3× current load — well before the projected scale. We needed a sharding strategy before we hit the wall.

**Task:** I was the infrastructure lead and was responsible for designing a sharding scheme that would distribute events across multiple nodes, with a routing mechanism that could be extended as we added capacity — without a complete rewrite of our event processing code.

**Action:** I evaluated two routing strategies. Naive modulo sharding (`node = event.customer_id % N`) was simple but had a fatal flaw: when we added a node (N changing from 4 to 5), every customer's events would route to a different node — corrupting any per-customer state that assumed events for a customer always hit the same node. I proposed consistent hashing instead: I mapped customer IDs onto a ring with 1,000 virtual nodes per physical node (4,000 virtual nodes total for our initial 4 nodes). Each event for a customer routed to the first virtual node clockwise from `hash(customer_id)`. Adding a fifth physical node required migrating only ~20% of virtual node assignments — not all customers. I implemented this in a shared routing library used by both the ingestion API and the queue workers, so both layers made identical routing decisions without coordination. I tested node addition in a staging environment by adding a fifth node while load was running at 10,000 events/sec and verified that only 19.2% of customer routing changed (expected ~20%) and that no events were lost or duplicated during the transition.

**Result:** We deployed 4-node consistent hashing within 6 weeks. Processing capacity scaled linearly — 4× the original throughput with 4× the nodes. When we added a fifth node three months later, the migration took 15 minutes with zero downtime and affected exactly the expected ~20% of customers. The routing library became a shared internal dependency used by four other services over the following year.

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
