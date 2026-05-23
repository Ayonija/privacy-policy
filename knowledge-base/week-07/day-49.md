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
```java
import java.util.*;

class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Serialize and Deserialize BST (LC 449)
    public static String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        preorderSerialize(root, sb);
        return sb.length() > 0 ? sb.substring(0, sb.length() - 1) : "";
    }
    private static void preorderSerialize(TreeNode node, StringBuilder sb) {
        if (node == null) return;
        sb.append(node.val).append(",");
        preorderSerialize(node.left, sb);
        preorderSerialize(node.right, sb);
    }
    public static TreeNode deserialize(String data) {
        if (data == null || data.isEmpty()) return null;
        String[] parts = data.split(",");
        int[] valsList = new int[parts.length];
        for (int i = 0; i < parts.length; i++) valsList[i] = Integer.parseInt(parts[i]);
        int[] idx = {0};
        return build2(valsList, idx, Long.MIN_VALUE, Long.MAX_VALUE);
    }
    private static TreeNode build2(int[] valsList, int[] idx, long low, long high) {
        if (idx[0] == valsList.length) return null;
        long val = valsList[idx[0]];
        if (!(low < val && val < high)) return null;
        idx[0]++;
        TreeNode node = new TreeNode((int) val);
        node.left  = build2(valsList, idx, low, val);
        node.right = build2(valsList, idx, val, high);
        return node;
    }

    // Find Duplicate Subtrees (LC 652)
    public static List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        Map<String, Integer> seen = new HashMap<>();
        List<TreeNode> result = new ArrayList<>();
        postorderDup(root, seen, result);
        return result;
    }
    private static String postorderDup(TreeNode node, Map<String, Integer> seen, List<TreeNode> result) {
        if (node == null) return "#";
        String left  = postorderDup(node.left,  seen, result);
        String right = postorderDup(node.right, seen, result);
        String key = left + "," + right + "," + node.val;
        seen.put(key, seen.getOrDefault(key, 0) + 1);
        if (seen.get(key) == 2) {   // exactly on second occurrence
            result.add(node);
        }
        return key;
    }

    // Recover Tree from Preorder Traversal (LC 1028)
    public static TreeNode recoverFromPreorder(String traversal) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        int i = 0;
        int n = traversal.length();
        while (i < n) {
            int depth = 0;
            while (i < n && traversal.charAt(i) == '-') {
                depth++;
                i++;
            }
            int val = 0;
            while (i < n && Character.isDigit(traversal.charAt(i))) {
                val = val * 10 + (int)(traversal.charAt(i));
                i++;
            }
            TreeNode node = new TreeNode(val);
            // pop stack to depth — stack holds path from root to parent
            while (stack.size() > depth) {
                stack.pollLast();
            }
            if (!stack.isEmpty()) {
                TreeNode parent = stack.peekLast();
                if (parent.left == null) {
                    parent.left = node;
                } else {
                    parent.right = node;
                }
            }
            stack.addLast(node);
        }
        return stack.peekFirst();  // root is the first element
    }
}
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining BST Serialization Without Null Markers in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to serialize a BST to a string and deserialize it back — using as few bytes as possible. A general binary tree requires null markers (e.g., 'null') to reconstruct structure unambiguously. But a BST has an additional property that makes null markers unnecessary — the sort order uniquely determines the structure from preorder values alone."

**Task:** "My goal was to serialize in O(n) and deserialize in O(n) without any null markers, exploiting the BST invariant to reconstruct structure purely from value ordering."

**Action:** Walk the interviewer through these steps:
1. *Serialize — preorder only:* "I do a standard preorder DFS and write `node.val` for each node, joined by commas. No null markers. A 3-node BST with root 2, left 1, right 3 serializes as `'2,1,3'` — 5 characters vs `'2,1,#,#,3,#,#'` with nulls."
2. *Why preorder works for BST:* "In preorder, we always write the root first. When deserializing, the first value must be the root. The next values can be placed left or right based on BST ordering — no null markers needed to distinguish them."
3. *Deserialize — bounds-based reconstruction:* "I process tokens left-to-right from the preorder sequence. At each recursion level, I accept the next token into the current subtree only if it falls within `(low, high)` bounds. If it doesn't fit, I don't consume it and return null — this triggers the parent to try placing it in the right subtree instead."
4. *Why bounds work:* "The BST property guarantees that all nodes in the left subtree of a node with value v are < v, and all nodes in the right subtree are > v. These bounds propagate exactly like the Validate BST bounds (LC 98), and they partition the preorder stream into left/right subtrees without needing null markers."
5. *Shared index:* "I use a shared mutable index `idx[0]` to advance through the token array across all recursive calls. When a call returns null (token out of bounds), the index is NOT advanced — the token is left for the parent's right subtree."

**Result:** "O(n) serialize, O(n) deserialize. BST serialization produces roughly half the bytes of general binary tree serialization (no null markers). The bounds-based deserialize is the key insight — it's the same invariant as BST validation."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer bounds-based BST preorder |
|-------------|---------------------------|--------------------------------------|
| General BFS with null markers (LC 297) | For general binary trees (not BSTs) | Correct but produces more bytes; doesn't exploit BST property |
| BST insertion deserialization | Simpler to implement | O(n log n) for balanced BST, O(n²) for skewed; bounds-based is O(n) |
| Preorder + bounds | BST serialization | O(n) time; minimal output; exploits BST property |

**Why NOT BST insertion deserialization:** Inserting n values one by one into a BST is O(n log n) average but O(n²) for a sorted or nearly-sorted sequence (skewed tree). Bounds-based reconstruction is always O(n).

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

**Leadership principle: Earn Trust**

**STAR Story — Designing reliable message delivery where loss was unacceptable**

**Situation:** Our platform sent transactional emails — order confirmations, shipping notifications, password resets — via a third-party email provider. The original implementation called the provider's API synchronously in the checkout HTTP request. When the provider had a 15-minute outage (which happened twice in the preceding quarter), all checkout requests failed with a 500 error because the email API call was timing out. Customers couldn't complete orders during these windows. Additionally, post-incident analysis revealed that in the previous 90 days we had silently dropped 340 transactional emails because of transient 429 (rate limit) errors — the synchronous call had no retry logic.

**Task:** I was asked to redesign the transactional email delivery system to guarantee zero message loss for checkout-critical emails, decouple email sending from the checkout request path, and handle provider failures without affecting checkout completion.

**Action:** I designed an outbox-pattern solution using Redis Streams. On order completion: the checkout service wrote an email job to a Redis Stream (`XADD emails:outbox * ...`) within the same logical commit as the order — this was the "at-least-once" guarantee since Redis Streams persisted the message. A separate email worker service used consumer groups (`XREADGROUP`) to consume jobs with at-least-once delivery semantics: it called the email provider API, and only on a 2xx response did it acknowledge the message (`XACK`). On failure (provider error or 429), the message remained in the pending list and was retried with exponential backoff up to 7 attempts over 2 hours. I added a dead-letter stream for messages that exceeded 7 attempts, with an alert that paged the on-call engineer. I verified exactly-once delivery by adding idempotency keys in the email API call — the provider ignored duplicate requests within 24 hours, so retries were safe.

**Result:** Over the following six months: zero emails dropped due to transient failures (down from 340/quarter). During one 22-minute provider outage, checkout completed normally for all customers and all 1,840 pending emails were delivered within 3 minutes of provider recovery. Email delivery reliability improved from 97.6% to 99.98%. Customer trust scores for order communication improved by 14 points in the quarterly survey.

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
