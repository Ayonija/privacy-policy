# Day 92 — 1D DP: House Robber Pattern
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master the House Robber recurrence — the canonical "skip-or-take" 1D DP — and extend it to circular arrays and binary trees so you can handle all three variants that appear in FAANG interviews.

---

## DSA (2 hours)

### Pattern: 1D DP — Skip-or-Take (House Robber Family)

**Core idea:**  
At each position you decide: include this element (and skip the previous) or skip it (and keep the prior best). `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. The key constraint is that two adjacent elements cannot both be taken.

**Trigger condition:**  
- Problem says "cannot pick two adjacent/adjacent-by-definition elements"
- Circular array? Run the recurrence twice: once excluding last element, once excluding first; take the max.
- Tree? Post-order traversal: each node returns `(rob_this, skip_this)` pair up to its parent.

**Complexity:**  
Time: O(n) | Space: O(1) for array variants, O(h) recursion stack for tree variant

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | House Robber IV | 2560 | Hard | Binary search + greedy DP | Binary search on the minimum capability; check feasibility with greedy skip-or-take |
| 2 | House Robber III | 337 | Medium | Tree DP (post-order) | Each node returns (rob_root, skip_root); parent aggregates children's pairs |
| 3 | House Robber (revision) | 198 | Medium | Classic 1D DP | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`; compress to two variables |

---

### Full Solution Walkthrough — House Robber (LC 198) — Revision

```python
def rob(nums: list[int]) -> int:
    prev2, prev1 = 0, 0
    for n in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + n)
    return prev1
```

**Trace for `[2,7,9,3,1]`:**  
`0,0 → 0,2 → 2,7 → 7,11 → 11,11 → 11,12` → return 12.

---

### Full Solution Walkthrough — House Robber II (LC 213) — Circular Extension

**Insight:** Houses form a circle → first and last are adjacent. Run rob() twice:
- Subarray `[0, n-2]` (exclude last)
- Subarray `[1, n-1]` (exclude first)
Take the max.

```python
def rob(nums: list[int]) -> int:
    if len(nums) == 1: return nums[0]

    def rob_range(lo, hi):
        prev2, prev1 = 0, 0
        for i in range(lo, hi + 1):
            prev2, prev1 = prev1, max(prev1, prev2 + nums[i])
        return prev1

    return max(rob_range(0, len(nums)-2), rob_range(1, len(nums)-1))
```

---

### Full Solution Walkthrough — House Robber III (LC 337) — Tree DP

**Insight:** Each node independently decides rob/skip. Post-order: leaf returns `(leaf.val, 0)`. Parent combines children.

```python
def rob(root) -> int:
    def dfs(node):
        if not node:
            return (0, 0)  # (rob_this_node, skip_this_node)
        left_rob, left_skip = dfs(node.left)
        right_rob, right_skip = dfs(node.right)

        rob_curr = node.val + left_skip + right_skip
        skip_curr = max(left_rob, left_skip) + max(right_rob, right_skip)
        return (rob_curr, skip_curr)

    return max(dfs(root))
```

**Why this works:** If you rob a node, children must be skipped (`left_skip + right_skip`). If you skip a node, each child independently takes its best option.

---

### Full Solution Walkthrough — House Robber IV (LC 2560) — Hard

**Problem:** Given `nums` (money in each house) and `k` (must rob exactly k houses, no two adjacent), find the minimum possible maximum amount stolen ("capability").

**Key insight:** Binary search on the answer (minimum capability `cap`). For a given `cap`, greedily count how many houses with `nums[i] <= cap` can be robbed without adjacency.

```python
def minCapability(nums: list[int], k: int) -> int:
    def can_rob_k(cap: int) -> bool:
        count, prev_robbed = 0, False
        for n in nums:
            if n <= cap and not prev_robbed:
                count += 1
                prev_robbed = True
            else:
                prev_robbed = False
        return count >= k

    lo, hi = min(nums), max(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if can_rob_k(mid):
            hi = mid       # can achieve with lower capability
        else:
            lo = mid + 1   # need higher capability
    return lo
```

**Complexity:** O(n log(max - min)) — binary search × linear feasibility check.

**Why greedy works in `can_rob_k`:** We want maximum count of valid houses ≤ cap. Greedy: take every eligible house as soon as possible (no benefit to waiting).

---

## System Design (30 min)

### Topic: Chat Service — Message Storage Schema

**Core components:**
1. **Message table** (Cassandra): `(conversation_id PK, message_id clustering, sender_id, content, created_at)`
2. **Conversation table**: `(user_id PK, conversation_id, last_message_at, unread_count)`
3. **Inbox fan-out**: On send, write to message table once + update each participant's conversation row
4. **Message ID**: Use Snowflake ID (64-bit: timestamp + machine ID + sequence) — sortable + globally unique
5. **Media storage**: S3 for attachments; store only URL in message row

**Key trade-offs:**
- **Single table vs. inbox per user:** Single message table is simpler; inbox per user duplicates data but makes "unread messages" queries trivial (no fan-in needed at read time).
- **Consistency model:** Eventual consistency acceptable for message delivery (users tolerate ~100ms delay); strong consistency needed for "message sent" acknowledgment.
- **Pagination:** Use `message_id` (not `OFFSET`) for cursor-based pagination — stable under concurrent inserts.

**Interview talking point:**  
*"If asked how you paginate chat history efficiently, answer: use a cursor on `message_id` (the Snowflake timestamp-prefixed ID). The query is `WHERE conversation_id = ? AND message_id < cursor LIMIT 20`. This is O(log n) on the clustering index regardless of history length, unlike `OFFSET` which scans from row 1."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode — House Robber Variant Drill

**Goal:** Solve House Robber II (LC 213) from scratch in under 10 min; then attempt a new timed problem from the DP tag.

**Session structure:**
- 0:00 — House Robber II: write recurrence in comments first, then code
- 10:00 — Submit; if wrong, debug before peeking at solutions
- 15:00 — Pick one new Medium/Hard DP problem; solve under contest conditions (30 min cap)
- 45:00 — Debrief: what pattern did problem 2 use? Where did you lose time?

**Debrief prompt:** Did you recognize that House Robber II reduces to two calls of House Robber I? If it took you more than 2 minutes to see that reduction, practice identifying "sub-problem decomposition" before coding.

---

## Behavioral (30 min)

**STAR prompt:**  
Describe a time you had to solve a problem that had no clear "right answer" — you had to weigh trade-offs and make a judgment call. How did you frame the decision?

**Target LP:** *Bias for Action* (Amazon) — making a reversible decision quickly under uncertainty.

**Full STAR Story — "Read vs. Write Fan-out Trade-off for Group Chat":**
**S (20%):** "At ChatCo, we were designing group messaging for groups up to 10,000 members. Write fan-out (copy to each inbox) vs. read fan-out (fetch on read) had no clear winner — each had merit depending on assumptions about group size distribution."
**T:** "I needed to make and document a defensible architectural decision within one sprint, without a perfect data set on group size distribution."
**A (60% — 'I' not 'we'):** "(1) I analyzed our closest existing data — average groups had 42 members, 95th percentile was 380 members, 99th percentile was 1,200 members. (2) I modeled write amplification: at 42 members and 10M messages/day, write fan-out cost was 420M Redis writes/day — affordable. At 1,200 members it became 12B writes/day — unsustainable. (3) I proposed a hybrid: write fan-out for groups ≤ 200 members, read fan-out with cursor-based polling for larger groups. (4) I made this a reversible decision by flagging the threshold in a feature flag rather than hardcoding it."
**R (20%):** "Shipped the hybrid on schedule. The 200-member threshold handled 97% of groups with write fan-out (fast reads). Large groups gracefully used polling. Feature flag allowed threshold adjustment without a deploy 3 months later when data changed."
*Works for: Bias for Action, Are Right A Lot, Think Big.*

### STAR Interview Framework

> **House Robber skip-or-take DP:** naive recursion O(2^n) → rolling variables O(n) time, O(1) space

**S:** "Given houses with money, can't rob adjacent. Naive O(2^n) recursion recomputes every pair of adjacent skip/take decisions."
**T:** "Need O(n) time, O(1) space by keeping only two rolling variables for the previous two best profits."
**A (60%):**
1. *Classify:* "Can't pick adjacent elements, maximize sum → House Robber skip-or-take DP."
2. *Init:* "prev2=0, prev1=0 (empty prefix has 0 profit)."
3. *Loop/Recurrence:* "For each num: prev2, prev1 = prev1, max(prev1, prev2 + num)."
4. *Termination:* "Return prev1."
5. *Gotcha:* "Circular variant (LC 213): run the recurrence twice — once excluding the first house, once excluding the last; take max. Forgetting either subarray gives wrong answer."
**R:** "O(n) time, O(1) space. Tree variant: post-order DFS returning (rob_node, skip_node) pair — O(n) time, O(h) space."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Full O(n) DP array | Need to reconstruct the actual chosen houses | If only the max value is needed, rolling vars save space |
| Greedy (take every other) | Uniform values | Non-uniform values break simple alternation greedy |

---

## Flashcards

| Q | A |
|---|---|
| Write the House Robber recurrence from memory. | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Compress: `prev2, prev1 = prev1, max(prev1, prev2 + nums[i])`. |
| How does House Robber II (circular) reduce to House Robber I? | Run House Robber I on `nums[0..n-2]` (skip last) and on `nums[1..n-1]` (skip first). Return the max. The two subarrays cover all cases where first and last are never both picked. |
| What does each tree node return in House Robber III? | A tuple `(rob_this, skip_this)`. `rob_this = node.val + left_skip + right_skip`. `skip_this = max(left_rob, left_skip) + max(right_rob, right_skip)`. |
| How does House Robber IV use binary search? | Binary search on `cap` (the minimum capability). Feasibility check: greedy scan — count how many houses with `value ≤ cap` can be picked without adjacency. If count ≥ k, `cap` is achievable. |
| Why is a Snowflake ID better than UUID for chat message ordering? | Snowflake IDs embed a millisecond timestamp in the high bits, making them sortable by insertion order. UUIDs are random → no temporal ordering. Sorting Snowflake IDs by ID value approximates sort by time without a separate `created_at` index scan. |

---

## Checklist

- [ ] Wrote House Robber recurrence from memory (no reference)
- [ ] Solved House Robber II — stated "two sub-arrays" insight before coding
- [ ] Solved House Robber III — drew the `(rob, skip)` pair tree before coding
- [ ] Traced House Robber IV binary search on `[2,3,5,9]`, k=2 by hand
- [ ] Sketched Chat Service message schema (table columns, partition/clustering keys)
- [ ] Completed 1-hour assessment session with debrief written
- [ ] Delivered STAR behavioral answer aloud (timed ≤ 2 min)
- [ ] Reviewed all 5 flashcards from memory
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
