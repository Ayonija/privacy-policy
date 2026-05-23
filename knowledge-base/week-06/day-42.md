# Day 42 — Trees: BFS Level Order & Serialize/Deserialize
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Master BFS level-order traversal variants and the hardest tree serialization problem — encoding a full binary tree as a BFS string with null sentinels and reconstructing it exactly.

---

## DSA (2 hours)
### Pattern: BFS Level-Order Processing & BFS Serialization

**Binary Tree Level Order Traversal (LC 102):**
Use a deque. At the start of each level, capture `level_size = len(queue)`. Process exactly `level_size` nodes, collecting their values into a level list. Enqueue non-null children. Append the level list to result.

**Binary Tree Zigzag Level Order Traversal (LC 103):**
Same BFS structure. Add a boolean `left_to_right`. After building each level list, reverse it when `left_to_right` is False. Toggle the flag after every level.

**Serialize and Deserialize Binary Tree (LC 297):**
*Serialize (BFS):* Process nodes level by level. Write `node.val` for real nodes and `"null"` for None children. Result: `"1,2,3,null,null,4,5"`.
*Deserialize:* Split the string into tokens. The first token is the root. Use a queue of nodes whose children have not yet been assigned. For each node in the queue, pop two consecutive tokens from the token list — the left and right children (or null).

**Trigger condition:**
- "collect nodes level by level" → BFS with `level_size` capture
- "alternate direction per level" → BFS + toggle flag + reverse
- "encode tree to string and rebuild exactly" → BFS serialization with null sentinels; queue-based deserialization

**Time complexity:** O(n) for all | **Space complexity:** O(n) — queue holds at most one full level (up to n/2 nodes at the bottom of a balanced tree)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Binary Tree Level Order Traversal | 102 | Medium | BFS with `level_size` snapshot | Capture queue length before processing each level; enqueue children only for non-null nodes |
| 2 | Binary Tree Zigzag Level Order Traversal | 103 | Medium | BFS + direction toggle | Same as level order; reverse each odd-numbered level (or use deque append direction) |
| 3 | Serialize and Deserialize Binary Tree | 297 | Hard | BFS serialize + queue-based deserialize | Null sentinels preserve structure; deserialize by assigning tokens as children in queue order |

---

### Code Skeleton
```python
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right

# Binary Tree Level Order Traversal (LC 102)
def levelOrder(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        level_size = len(queue)
        level = []
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result

# Binary Tree Zigzag Level Order Traversal (LC 103)
def zigzagLevelOrder(root):
    if not root: return []
    result, queue, left_to_right = [], deque([root]), True
    while queue:
        level = [queue.popleft().val for _ in range(len(queue))
                 if not (queue[-1].left and queue.append(queue[-1].left))  # placeholder
                 ]
        # cleaner version:
    result, queue, left_to_right = [], deque([root]), True
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level if left_to_right else level[::-1])
        left_to_right = not left_to_right
    return result

# Serialize and Deserialize Binary Tree (LC 297)
class Codec:
    def serialize(self, root):
        if not root: return ""
        result, queue = [], deque([root])
        while queue:
            node = queue.popleft()
            if node:
                result.append(str(node.val))
                queue.append(node.left)    # enqueue even if None
                queue.append(node.right)
            else:
                result.append("null")
        return ",".join(result)

    def deserialize(self, data):
        if not data: return None
        tokens = iter(data.split(","))
        root = TreeNode(int(next(tokens)))
        queue = deque([root])
        while queue:
            node = queue.popleft()
            left_val = next(tokens)
            if left_val != "null":
                node.left = TreeNode(int(left_val))
                queue.append(node.left)
            right_val = next(tokens)
            if right_val != "null":
                node.right = TreeNode(int(right_val))
                queue.append(node.right)
        return root
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining BFS Serialization / Deserialization in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to serialize a binary tree to a string and deserialize it back to the identical tree — with no loss of structural information (nulls matter: a node with no left child is different from a node whose left child is a leaf). A naive preorder approach without null sentinels is ambiguous — the same string could represent multiple different trees."

**Task:** "My goal was to design a serialization scheme that encodes structure unambiguously so that the deserialized tree is a byte-for-byte replica of the original, in O(n) time for both directions."

**Action:** Walk the interviewer through these steps:
1. *Choose BFS serialization:* "I use BFS because level-order with null sentinels uniquely encodes the tree. Every node's position in the output string corresponds directly to its position in the queue — no ambiguity."
2. *Serialize — enqueue even null children:* "I enqueue `node.left` and `node.right` even when they are None. For a None node, I write `'null'` and do not enqueue its (non-existent) children. For a real node, I write `str(node.val)` and enqueue both children."
3. *Serialize output:* "The result is `','.join(result)`. A 3-node balanced tree with root 1, left 2, right 3 serializes as `'1,2,3,null,null,null,null'`."
4. *Deserialize — queue of parents:* "I split on `','` and build an iterator over tokens. I create the root from the first token and put it in a deque. For each node dequeued, I pop two tokens — its left child value and right child value. If not `'null'`, I create the child node and enqueue it. This naturally reconstructs the tree in the same level-order the serializer produced."
5. *Why queue-based deserialization:* "The queue of 'parents awaiting children' mirrors the BFS queue of the serializer. Each real node in the queue consumes exactly two tokens in the correct order — left before right. No index arithmetic needed."

**Result:** "O(n) time, O(n) space for both serialize and deserialize. The scheme handles arbitrarily deep trees, trees with nulls in the middle (not just at leaves), and empty trees. It is the exact design used in LeetCode's built-in tree serialization format."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer BFS serialization |
|-------------|---------------------------|-------------------------------|
| Preorder (root, left, right) with null markers | DFS-based encoding, slightly shorter strings | Deserialization is recursive — stack overflow for trees with depth > 10,000 |
| Preorder + Inorder (two sequences) | Reconstruct from two traversals | Only works for trees with unique values; fails on duplicates |
| BFS with null sentinels | Any binary tree, any values | Unambiguous, iterative, handles duplicates and deep trees |

**Why NOT preorder + inorder:** This classic reconstruction requires unique values. The BFS serialization approach has no such restriction and works for any tree including duplicates.

---

### Edge Cases to Trace Before Coding
- LC 102: empty tree → `[]`; single node → `[[root.val]]`
- LC 103: even-indexed levels (0-based) go left-to-right, odd go right-to-left; trace a 3-level tree manually
- LC 297: serializing `[]` (empty) returns `""` or `"null"` — make sure deserialize handles both; serializing a tree with nulls inside (not just leaves): `"1,null,2"` — the null at position 1 means no children are enqueued for that null, so node 2 is correctly assigned as the right child of root

---

### Interview Pattern Drill
| Traversal | Queue contents | Level boundary detection | Direction |
|-----------|---------------|--------------------------|-----------|
| Level order | non-null nodes | `level_size = len(queue)` before loop | left→right |
| Zigzag | non-null nodes | same | toggle each level |
| BFS serialize | ALL nodes including None | none needed (null marker in output) | left→right |

---

## System Design (1 hour)
### Topic: Redis Data Structures & Use Cases

**Redis is a single-threaded in-memory data structure server.** Its value is not just key-value storage — it's that the *value* can be one of several rich data structures, each with O(1) or O(log n) operations.

**String** — most basic; stores bytes (up to 512 MB)
- `SET key value [EX seconds]` / `GET key` / `INCR key`
- Use cases: session tokens, rate-limit counters, distributed locks (`SET key value NX EX 30` — atomic acquire-or-fail), feature flags

**List** — doubly linked list; O(1) head/tail push/pop
- `LPUSH` / `RPUSH` / `LPOP` / `RPOP` / `LRANGE`
- Use cases: message queues (producer RPUSH, consumer LPOP), activity feeds (newest N events), task queues

**Set** — unordered collection of unique strings; O(1) add/remove/member
- `SADD` / `SMEMBERS` / `SINTER` / `SUNION` / `SCARD`
- Use cases: unique visitor sets per day, social graph (SINTER for mutual friends), tagging systems

**Sorted Set (ZSet)** — set where each member has a floating-point score; members ordered by score; O(log n) updates
- `ZADD key score member` / `ZREVRANGE key 0 9` / `ZRANGEBYSCORE` / `ZRANK`
- Use cases: leaderboards (score = points), rate limiting (score = timestamp, sliding window), priority queues

**Hash** — map of field→value within a single key; O(1) field access
- `HSET key field value` / `HGET key field` / `HGETALL`
- Use cases: user profile objects (`HSET user:123 name "Alice" email "a@b.com"`), caching DB rows, session data with multiple attributes

**HyperLogLog** — probabilistic cardinality estimator; fixed 12 KB regardless of set size; ~0.81 % error
- `PFADD key element` / `PFCOUNT key`
- Use case: unique visitor counts at massive scale (100 M+ users) without storing all user IDs

**Key expiry:** any key type can have a TTL set via `EXPIRE key seconds` or `SET key val EX seconds`. Redis uses lazy expiry + active sampling (every 100 ms checks 20 random keys with TTLs).

**Redis data structure decision table:**
| Need | Data structure | Key command |
|------|---------------|-------------|
| Session token | String | `SET`/`GET` with TTL |
| Leaderboard top-10 | Sorted Set | `ZADD`, `ZREVRANGE 0 9` |
| Mutual friends | Set | `SINTER user:A:friends user:B:friends` |
| User profile (multi-field) | Hash | `HSET`, `HGETALL` |
| Task queue | List | `RPUSH` (produce), `BLPOP` (consume, blocking) |
| Unique visitors at scale | HyperLogLog | `PFADD`, `PFCOUNT` |
| Sliding-window rate limiter | Sorted Set | score = timestamp, `ZREMRANGEBYSCORE` + `ZCARD` |

**Interview talking point:** "If asked to design a leaderboard for a game with 10 M players updated in real-time, answer: Redis Sorted Set — `ZADD leaderboard score player_id` on each game completion (O(log n)). Top-100 = `ZREVRANGE leaderboard 0 99 WITHSCORES` (O(100)). Player rank = `ZREVRANK leaderboard player_id` (O(log n)). This pattern handles millions of writes/sec and serves reads in microseconds with zero SQL joins."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Are Right, A Lot**

**STAR Story — Designing a system that faithfully records and reproduces structured state**

**Situation:** Our platform ran multi-step onboarding workflows that could span several days — users might complete step 1, return the next day for step 2, and finish on day 3. Each workflow had a tree-like decision structure: the path a user followed depended on their answers to earlier questions. We stored only the final completed state in the database, not the intermediate tree of decisions taken. When users abandoned a session mid-flow, the system had no way to resume them — they had to restart from scratch. About 23% of users who started onboarding abandoned it and never returned, and user interviews revealed the forced restart was the primary reason.

**Task:** I was asked to design a session-resume feature: when a user returned, they should be dropped back at exactly the step where they left off, with all previous answers preserved. My goal was to design the serialization and storage of the in-progress workflow state so it could be fully reconstructed across sessions.

**Action:** I analysed the workflow data structure: each active session was a tree of visited nodes (steps), with each node storing the question answered, the chosen option, and references to child nodes (subsequent steps unlocked by that choice). I designed a BFS serialization of the active session tree: I serialized the tree node-by-node in level order, writing null sentinels for branches the user had not yet taken. This produced a compact JSON string for any partial or complete session. I stored this string in a `session_state` column in our Redis session store with a 30-day TTL. On resume, I deserialized the string by reconstructing the tree in BFS order — parsing the token list and assigning children in queue order, exactly analogous to LC 297. I verified round-trip fidelity with property-based tests: I generated 10,000 random partial workflows, serialized and deserialized each, and asserted the resulting tree matched the original at every node and edge.

**Result:** Session resume launched to 100% of users within two sprints. The 23% abandonment rate dropped to 9% within 30 days — a 61% reduction. Completed onboarding increased by 18% in the same period, which translated to a measurable lift in 30-day retention. Serialization and deserialization each ran in under 2 ms even for the deepest workflows (15 levels), well within our 20 ms budget for session restoration.

---

## Flashcards

| Q | A |
|---|---|
| In BFS level-order traversal, how do you know when one level ends and the next begins? | Snapshot `level_size = len(queue)` before the inner loop; process exactly `level_size` nodes, then the remaining queue items belong to the next level |
| How does Zigzag Level Order differ from standard BFS level order? | Identical BFS structure; maintain a `left_to_right` boolean; reverse each level list when the flag is False; toggle after every level |
| How does BFS serialization represent missing children in LC 297? | Write `"null"` for None nodes; during deserialization, only non-null nodes are enqueued as parents — nulls consume no children tokens |
| What Redis data structure implements a real-time leaderboard and why? | Sorted Set — `ZADD` updates score in O(log n); `ZREVRANGE` retrieves top-K in O(K); `ZREVRANK` returns a player's rank in O(log n) |
| What Redis structure gives unique visitor counts for 100 M users at fixed memory? | HyperLogLog — `PFADD`/`PFCOUNT`; uses fixed 12 KB per key regardless of cardinality; ~0.81 % error rate is acceptable for analytics |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
