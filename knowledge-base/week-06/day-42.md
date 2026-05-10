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
- STAR prompt: Describe a time you had to design a system that needed to faithfully record and reproduce structured state — analogous to serializing a tree and being able to deserialize it exactly.
- Leadership principle: Are Right, A Lot

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
