# Day 49 — BST: Serialize, Duplicate Subtrees & Preorder Recovery
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
BST serialization requires far fewer tokens than general tree serialization — structure is implied by values alone. Duplicate subtrees use postorder canonical string hashing. Preorder recovery with depth encoding uses a stack with depth tracking.

---

## DSA (2 hours)
### Pattern: BST Serialization Without Null Markers + Canonical Subtree Hashing + Stack-Based Preorder Reconstruction

**Serialize and Deserialize BST (LC 449):**
Unlike a general binary tree (which needs null markers), a BST can be reconstructed from its preorder traversal alone — the values uniquely determine the structure. Serialize = preorder values joined by a delimiter. Deserialize = rebuild by BST insertion using a queue/iterator of values.

Optimised deserialize: use bounds (like LC 98 validate BST) instead of insertion. Process tokens from the preorder stream; include a node if its value falls within `(low, high)` bounds.

**Find Duplicate Subtrees (LC 652):**
Postorder DFS. For each node, build a canonical string: `f"{left_str},{right_str},{node.val}"`. Store all seen strings in a HashMap. If a string appears for the second time, the current node is the root of a duplicate subtree — add it to the result list.

**Recover a Tree from Preorder Traversal (LC 1028):**
Input: a string like `"1-2--3--4-5--6--7"` where the number of leading dashes = depth. Parse into `(depth, value)` pairs. Use a stack to reconstruct: the stack holds the path from root to the current node. For a new node at depth `d`, pop the stack until `len(stack) == d`. The top of the stack is the parent — assign as left or right child.

**Trigger condition:**
- "serialize/deserialize BST with minimum data" → preorder only (no nulls); deserialize with bounds-based reconstruction
- "find all duplicate subtree roots" → postorder canonical string + HashMap frequency count
- "reconstruct tree from depth-encoded preorder string" → stack tracking depth, assign children based on depth

**Time complexity:** LC 449: O(n) | LC 652: O(n²) naive (string concat), O(n) with hashing | LC 1028: O(n)
**Space complexity:** O(n) for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Serialize and Deserialize BST | 449 | Medium | Preorder values only; bounds-based deserialize | BST structure is implied by values in preorder — no null markers needed |
| 2 | Find Duplicate Subtrees | 652 | Medium | Postorder canonical string + HashMap | Canonical string uniquely identifies a subtree shape+values; duplicates have the same string |
| 3 | Recover a Tree from Preorder Traversal | 1028 | Hard | Stack with depth tracking | Parse depth + value; maintain stack as path from root; parent = stack top when `len(stack) == depth` |

---

### Code Skeleton
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# Serialize and Deserialize BST (LC 449)
class Codec449:
    def serialize(self, root):
        result = []
        def preorder(node):
            if not node: return
            result.append(str(node.val))
            preorder(node.left)
            preorder(node.right)
        preorder(root)
        return ",".join(result)

    def deserialize(self, data):
        if not data: return None
        vals = iter(int(x) for x in data.split(","))
        # Bounds-based reconstruction: include node if low < val < high
        def build(low, high):
            val = next(vals, None)
            if val is None or not (low < val < high):
                # put val back — we peeked too far
                # use a sentinel approach: track separately
                return None
            node = TreeNode(val)
            node.left  = build(low, val)
            node.right = build(val, high)
            return node
        # Itertools-free approach with a list pointer
        vals_list = [int(x) for x in data.split(",")]
        idx = [0]
        def build2(low, high):
            if idx[0] == len(vals_list): return None
            val = vals_list[idx[0]]
            if not (low < val < high): return None
            idx[0] += 1
            node = TreeNode(val)
            node.left  = build2(low, val)
            node.right = build2(val, high)
            return node
        return build2(float('-inf'), float('inf'))

# Find Duplicate Subtrees (LC 652)
from collections import defaultdict
def findDuplicateSubtrees(root):
    seen = defaultdict(int)
    result = []
    def postorder(node):
        if not node: return "#"
        left  = postorder(node.left)
        right = postorder(node.right)
        key = f"{left},{right},{node.val}"
        seen[key] += 1
        if seen[key] == 2:   # exactly on second occurrence
            result.append(node)
        return key
    postorder(root)
    return result

# Recover Tree from Preorder Traversal (LC 1028)
def recoverFromPreorder(traversal):
    stack = []
    i = 0
    while i < len(traversal):
        depth = 0
        while i < len(traversal) and traversal[i] == '-':
            depth += 1; i += 1
        val = 0
        while i < len(traversal) and traversal[i].isdigit():
            val = val * 10 + int(traversal[i]); i += 1
        node = TreeNode(val)
        # pop stack to depth — stack holds path from root to parent
        while len(stack) > depth:
            stack.pop()
        if stack:
            parent = stack[-1]
            if parent.left is None:
                parent.left = node
            else:
                parent.right = node
        stack.append(node)
    return stack[0]  # root is the first element
```

---

### Edge Cases to Trace Before Coding
- LC 449: single-node tree → serialize `"val"`, deserialize correctly with bounds `(-inf, +inf)` — one value returned, then None on next call
- LC 652: subtree of a single leaf node (e.g., null-null-5) — canonical string is `"#,#,5"`; if leaf 5 appears twice → result contains one of those leaf nodes (added on second occurrence)
- LC 1028: multi-digit values (e.g., "12") — parse digits until non-digit; depth-0 node is the root (no leading dashes); verify that right child assignment only happens when left is already set

---

### Interview Pattern Drill
| Problem | Traversal direction | Key data structure | What makes it unique |
|---------|-------------------|-------------------|---------------------|
| Serialize BST | Preorder | Iterator + bounds | No null markers — BST structure implied by values |
| Duplicate Subtrees | Postorder | HashMap of canonical strings | Canonical string = unique identifier for subtree shape |
| Recover from Preorder | Preorder | Stack as depth-path | Depth (dash count) encodes parent-child relationship |

---

## System Design (1 hour)
### Topic: Redis at Scale — Sentinel, Cluster, and Pub/Sub

**Redis Standalone:**
Single master node. Simple to operate. Single point of failure — if it crashes, all cache lookups fail (and potentially all traffic falls through to the DB).

**Redis Sentinel (High Availability):**
Sentinel is a separate process (run at least 3) that monitors a master + its replicas.
- Monitors: each Sentinel pings master every second
- Quorum: if `>= quorum` Sentinels agree the master is down → it is considered SDOWN (subjectively down). If a majority of all Sentinels confirm → ODOWN (objectively down)
- Failover: a Sentinel is elected to promote the most up-to-date replica to master; other replicas reconfigure to follow the new master
- Client connection: clients ask Sentinel for the current master address before connecting (`redis-py` supports `sentinel` connection pool automatically)

**Redis Cluster (Horizontal Sharding):**
- 16 384 hash slots; each master owns a range
- Minimum deployment: 3 masters + 3 replicas (6 nodes total)
- Automatic failover (no Sentinel needed)
- Clients must be cluster-aware
- No cross-slot multi-key operations (unless hash tags `{same_prefix}` force co-location)
- `CLUSTER INFO`, `CLUSTER NODES` for inspecting state

**Choosing Sentinel vs. Cluster:**
| Dimension | Sentinel | Cluster |
|-----------|----------|---------|
| Sharding | No | Yes (16 384 slots) |
| HA | Yes | Yes (built-in) |
| Dataset fits in one node? | Yes (must) | No (data is sharded) |
| Cross-key operations | Yes | No (unless same slot) |
| Client complexity | Low | Medium (cluster-aware client) |
| Minimum nodes | 1 master + 2 Sentinels | 6 nodes (3M + 3R) |

**Redis Pub/Sub:**
```
# Publisher:
redis.publish("notifications:user:42", json.dumps(event))

# Subscriber (blocking):
p = redis.pubsub()
p.subscribe("notifications:user:42")
for message in p.listen():
    process(message)
```
- Messages are not persisted — if no subscriber is listening, the message is lost
- For durability: use Redis Streams (`XADD`, `XREAD`) — messages are persisted and consumer groups enable reliable at-least-once delivery

**Redis Streams vs Pub/Sub:**
| Dimension | Pub/Sub | Streams |
|-----------|---------|---------|
| Persistence | No | Yes |
| Consumer groups | No | Yes |
| Message replay | No | Yes (read from any offset) |
| At-least-once | No | Yes |

**Interview talking point:** "If asked how to build a real-time notification system using Redis, answer: use Redis Streams with consumer groups — not Pub/Sub. Pub/Sub is fire-and-forget (missed if the consumer is offline). Streams persist messages and consumer groups let multiple service instances share the load, with each message delivered to exactly one consumer in the group. Use `XACK` to mark messages processed."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you designed a system for reliable message delivery where loss was unacceptable — what guarantees did you need, and how did you achieve them?
- Leadership principle: Earn Trust

---

## Flashcards

| Q | A |
|---|---|
| Why can a BST be serialized without null markers, unlike a general binary tree? | The BST property (left < node < right) means preorder values alone uniquely determine the tree structure — the deserializer uses value bounds to decide left vs. right placement |
| How does Find Duplicate Subtrees uniquely identify a subtree? | Postorder canonical string: `"left_str,right_str,node.val"`; two subtrees with the same shape and values produce identical strings; a HashMap tracks first vs. second occurrence |
| In Recover Tree from Preorder, how do you determine a node's parent? | Count leading dashes for depth `d`; pop the stack until `len(stack) == d`; the top is the parent; assign as left child if left is None, else right child |
| What is the minimum safe Redis Sentinel deployment and why? | 3 Sentinel processes — quorum requires a majority to declare a master down; with 2 Sentinels, a network split (each Sentinel isolated) could cause two simultaneous promotions (split-brain) |
| What is the key difference between Redis Pub/Sub and Redis Streams? | Pub/Sub: fire-and-forget, no persistence, message lost if no subscriber. Streams: messages are persisted, consumer groups enable reliable at-least-once delivery with acknowledgement |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
