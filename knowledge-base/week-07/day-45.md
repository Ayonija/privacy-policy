# Day 45 — Trees: Path Problems & Greedy Camera Placement
**Week 07 | Phase 1: DSA Mastery | Month 2**

## Focus
Root-to-leaf path problems share a DFS backtracking spine. Binary Tree Cameras introduces greedy DFS — the hardest tree DP/greedy problem in this slot — where placing cameras at the parent of an uncovered leaf is provably optimal.

---

## DSA (2 hours)
### Pattern: DFS Backtracking (Path Collection) + Greedy DFS (State Machine)

**Path Sum II (LC 113):**
DFS backtracking. Push the current node's value onto a running path, recurse left and right with `remaining = target - node.val`. When a leaf is reached with `remaining == 0`, copy the current path to result. Pop the node after both children return (backtrack).

**Sum Root to Leaf Numbers (LC 129):**
DFS accumulating a decimal number: `current_num = prev_num * 10 + node.val`. At each leaf, add `current_num` to total. No backtracking needed — the number is passed by value.

**Binary Tree Cameras (LC 968):**
Greedy: place cameras as high as possible (at the parent of an uncovered leaf). DFS postorder. Each node returns one of three states:
- `0` — node is NOT covered (needs a camera on its parent)
- `1` — node IS covered but has no camera
- `2` — node HAS a camera

Rules (for each node, based on children states):
- If either child returns `0` (uncovered) → place camera here → return `2`, increment count
- If either child returns `2` (has camera) → this node is covered → return `1`
- Otherwise (both children covered, no camera) → this node is not covered → return `0`
- After DFS: if root returns `0` → it is uncovered → add 1 more camera

**Trigger condition:**
- "collect all root-to-leaf paths with sum = target" → DFS backtracking, push/pop path
- "sum of all root-to-leaf numbers" → DFS passing accumulated value, add at leaf
- "minimum cameras to monitor all nodes" → greedy DFS postorder, 3-state return

**Time complexity:** O(n) for all | **Space complexity:** O(h) for DFS stack + O(h) for path in LC 113

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Path Sum II | 113 | Medium | DFS backtracking + path collection | Push node on entry, pop on exit (backtrack); add copy of path to result at leaf with remaining=0 |
| 2 | Sum Root to Leaf Numbers | 129 | Medium | DFS with accumulated decimal | `num = prev * 10 + node.val`; add to total at leaf; no backtracking needed (value passed by int) |
| 3 | Binary Tree Cameras | 968 | Hard | Greedy postorder DFS, 3-state return | Return 0/1/2; place camera only when a child is uncovered (state 0); check root after DFS |

---

### Code Skeleton
```java
import java.util.*;

class TreeNode { int val; TreeNode left, right; TreeNode(int val) { this.val = val; } }

class Solution {
    // Path Sum II (LC 113)
    public static List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        List<List<Integer>> result = new ArrayList<>();
        dfsPath(root, targetSum, new ArrayList<>(), result);
        return result;
    }
    private static void dfsPath(TreeNode node, int remaining, List<Integer> path, List<List<Integer>> result) {
        if (node == null) return;
        path.add(node.val);
        if (node.left == null && node.right == null && remaining == node.val) {
            result.add(new ArrayList<>(path));   // copy — not a reference
        } else {
            dfsPath(node.left,  remaining - node.val, path, result);
            dfsPath(node.right, remaining - node.val, path, result);
        }
        path.remove(path.size() - 1);   // backtrack
    }

    // Sum Root to Leaf Numbers (LC 129)
    public static int sumNumbers(TreeNode root) {
        return dfsSum(root, 0);
    }
    private static int dfsSum(TreeNode node, int num) {
        if (node == null) return 0;
        num = num * 10 + node.val;
        if (node.left == null && node.right == null) {
            return num;   // leaf: full number formed
        }
        return dfsSum(node.left, num) + dfsSum(node.right, num);
    }

    // Binary Tree Cameras (LC 968)
    private int cameras = 0;
    // States: 0 = not covered, 1 = covered (no camera), 2 = has camera
    public int minCameraCover(TreeNode root) {
        if (dfsCam(root) == 0) {   // root itself is uncovered
            cameras++;
        }
        return cameras;
    }
    private int dfsCam(TreeNode node) {
        if (node == null) return 1;   // null nodes are "covered" (don't need coverage)
        int left  = dfsCam(node.left);
        int right = dfsCam(node.right);
        if (left == 0 || right == 0) {   // a child is uncovered → place camera here
            cameras++;
            return 2;
        }
        if (left == 2 || right == 2) {   // a child has a camera → this node is covered
            return 1;
        }
        return 0;   // both children covered but no adjacent camera → this node is uncovered
    }
}
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining DFS Backtracking (Path Collection) in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a binary tree and a target sum, and asked to return all root-to-leaf paths whose node values sum to the target. A common bug is appending the current path object by reference — you get a list of lists where all entries point to the same mutated list, producing garbage output."

**Task:** "My goal was to collect all valid paths in O(n) time and O(h) space using DFS backtracking, with correct path copying at the leaf."

**Action:** Walk the interviewer through these steps:
1. *DFS preorder — push before recursing:* "On entering a node, I add `node.val` to the `path` list and subtract it from `remaining`. This happens before recursing to children."
2. *Leaf check:* "When I reach a leaf (both children null) and `remaining == node.val` at the moment of entry (equivalently, after subtraction `remaining - node.val == 0`), I copy the current path — `result.add(new ArrayList<>(path))` — and add the copy to result. **Copy, not reference** — this is the most common bug."
3. *Recurse:* "After the leaf check, I recurse on left and right children. Both calls share the same `path` list but each DFS frame sees the correct path because we backtrack after each returns."
4. *Backtrack — pop on exit:* "After recursing both children, I pop the current node from `path`. This restores the path to exactly the state it was in before we entered this node, so that the parent's sibling subtree sees the correct path."
5. *Why backtrack at the leaf too:* "Even at a leaf where we added to result, we still pop. The leaf's value was pushed on entry — it must be popped on exit like every other node."

**Result:** "O(n) time — each node visited once. O(h) space for the recursion stack and path (at most O(h) entries in `path` at any time). Zero risk of aliasing bugs because we copy the path at collection time."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer backtracking DFS |
|-------------|---------------------------|------------------------------|
| Build a new path list at each node (passing by value) | Simpler code | O(n × h) total space — creates a new list at every node call; backtracking uses O(h) |
| BFS with path tracking | When iterative is required | Requires storing a path per queue entry — O(n × h) space |
| DFS backtracking | Most cases | O(h) space; single shared path list; standard interview pattern |

**Why NOT pass-by-value path:** Creating `new ArrayList<>(path)` at every recursive call duplicates the path O(n) times — one per node — using O(n × h) total space. Backtracking with one shared list is O(h).

---

### Edge Cases to Trace Before Coding
- LC 113: no valid path exists → return `[]`; root itself is a leaf and equals targetSum → `[[root.val]]`; negative values — path sums can overshoot and come back, so you must reach a leaf before checking
- LC 129: single node → return `node.val`; tree with all zeros → returns 0 (leaf adds 0)
- LC 968: null children return `1` (covered) — this prevents placing unnecessary cameras at leaves; single node → DFS returns 0 for root → add 1 camera manually

---

### Interview Pattern Drill
| Pattern | DFS direction | Path/state passed | Answer collected |
|---------|-------------|-------------------|-----------------|
| Path collection | Pre-order (push before recurse) | Mutable list (backtrack on exit) | At leaf matching condition |
| Accumulated value | Pre-order | Immutable int (no backtrack) | At leaf |
| Greedy 3-state | Post-order (children before parent) | State int returned | Nonlocal camera count |

---

## System Design (1 hour)
### Topic: Cache Write Strategies — Write-Through, Write-Back, Write-Around

**Write-Through:**
Write to cache AND database in the same operation. Cache is always consistent with the DB.
```
write(key, val):
    db.write(key, val)        # synchronous
    cache.set(key, val, ttl)  # synchronous
```
- Pros: no stale cache reads; if cache crashes, DB is up-to-date
- Cons: write latency = DB write latency (cache write is fast but adds a step); cache stores writes that may never be re-read (wasted space for write-only data)
- Best for: read-heavy workloads where reads must always see the latest write (user profile updates)

**Write-Back (Write-Behind):**
Write to cache only; flush to the DB asynchronously after a delay.
```
write(key, val):
    cache.set(key, val, dirty=True)   # immediate
    # background job: flush dirty entries to DB every N seconds
```
- Pros: write latency is cache latency (sub-millisecond); batches DB writes (efficient for write-heavy workloads)
- Cons: data loss risk if cache crashes before flush; eventual consistency (DB may lag)
- Best for: write-heavy workloads where small data loss is acceptable (analytics counters, like/dislike counts)

**Write-Around:**
Write directly to the database, bypassing the cache entirely.
```
write(key, val):
    db.write(key, val)     # only to DB
    # cache is NOT updated — it will be populated on next read (cache-aside)
```
- Pros: prevents cache pollution for write-once/read-never data; good for bulk loads or batch writes
- Cons: next read after a write will always miss (must go to DB to refresh cache)
- Best for: data that is written rarely and may never be re-read (log files, audit records)

**Read-Through:**
Cache sits in front of DB. On miss, cache (not application code) fetches from DB and stores the result.
- Differs from cache-aside: the cache library/proxy handles the miss logic, not application code
- Common in managed caching systems (AWS ElastiCache with lazy loading, Varnish)

**Consistency matrix:**
| Strategy | Read consistency | Write latency | Data loss risk | Best use |
|---------|-----------------|---------------|---------------|----------|
| Write-through | Strong | DB latency | None | User data, profiles |
| Write-back | Eventual (short lag) | Cache latency | Yes (on crash) | Counters, analytics |
| Write-around | Stale on next read | DB latency | None | Batch writes, audit logs |
| Cache-aside (read) | Eventual (TTL-bounded) | n/a | None | General-purpose reads |

**Interview talking point:** "For a social network's like counter, write-back is ideal — every like writes to Redis (sub-millisecond), and a background worker flushes to Postgres every 5 seconds. We tolerate losing at most 5 seconds of like counts if Redis crashes (acceptable). For user account settings, write-through is required — a user who changes their email address must see the change reflected immediately on their next request."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Customer Obsession**

**STAR Story — Choosing between synchronous and asynchronous consistency for a write path**

**Situation:** We were building a checkout flow for an e-commerce platform. When a customer placed an order, we needed to: (1) write the order to the primary orders database, (2) update the inventory service to decrement stock counts, and (3) send a confirmation email. The first instinct from the team was to do all three synchronously in the same HTTP request — write order, call inventory API, trigger email — and return success only when all three succeeded. This guaranteed consistency but added up to 800 ms to checkout latency (inventory API: 200 ms, email service: 400 ms) and made checkout fail if either service was down.

**Task:** I was the backend lead responsible for the checkout service's architecture. My goal was to provide customers with fast, reliable checkout confirmation while maintaining correct inventory counts and reliable email delivery.

**Action:** I analysed each step by its consistency requirement. Order write: must be synchronous — without it, checkout fails. Inventory decrement: must be consistent but not necessarily synchronous — we could use an outbox pattern. Email: eventually consistent is fine — a 2-second delay is imperceptible. I designed a hybrid write path: the order write and an inventory reservation (a database row marking stock as "reserved") happened in a single local database transaction — fast (< 50 ms), atomic, no external calls. A background worker consumed the outbox and called the inventory service asynchronously, with retries and a dead-letter queue. The email trigger was a Kafka event published after commit, consumed by the email service independently. I verified that the inventory reservation model correctly prevented overselling even during the async window between reservation and final decrement.

**Result:** Checkout latency dropped from 950 ms to 120 ms — a 7.9× improvement. Checkout success rate improved from 94.3% (impacted by inventory and email service downtime) to 99.8% — errors now only from our own database. Email delivery reliability actually improved from 97% to 99.6% because the retry logic in the Kafka consumer was more robust than the original synchronous attempt. Zero overselling incidents in the following six months.

---

## Flashcards

| Q | A |
|---|---|
| What must you do after recursing both children in Path Sum II? | Pop the current node from the path (backtrack) — this restores the path to its state before entering this node so sibling branches see the correct path |
| Why does Sum Root to Leaf Numbers not need backtracking? | The accumulated number is an integer passed by value — each recursive call gets its own copy; no shared mutable state |
| In Binary Tree Cameras, what does a null child return and why? | Returns 1 (covered) — null nodes do not need to be monitored, so treating them as "already covered" prevents placing unnecessary cameras at every leaf |
| What is the difference between write-through and write-back? | Write-through writes to DB and cache synchronously (no data loss, higher latency). Write-back writes to cache only and flushes DB asynchronously (low latency, risk of data loss if cache crashes) |
| When should you use write-around caching? | When data is written once and may never be read again — e.g., bulk batch imports, audit logs, media uploads; avoids polluting cache with data that won't be re-read |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
