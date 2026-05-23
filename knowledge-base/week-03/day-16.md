# Day 16 — Linked Lists: Intersection & Reorder
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Solve intersection detection and in-place reordering — problems that demand both pointer intuition and careful stitching.

## DSA (2 hours)
### Pattern: Two Pointers on Two Lists + In-Place Reorder
- Intersection: advance both pointers; when one reaches null, redirect it to the other list's head; they meet at the intersection node (or both hit null if no intersection) — total traversal = m + n for both.
- Reorder List: find mid, reverse second half, then interleave the two halves node by node.
- Trigger condition: "find where two lists join" OR "interleave nodes from two halves of a list."
- Time complexity: O(m+n) for intersection, O(n) for reorder | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Intersection of Two Linked Lists | 160 | Easy | Two pointers on two lists | Both pointers traverse m+n total; they sync up at the intersection node |
| 2 | Odd Even Linked List | 328 | Medium | Two-chain weave | Separate odd-indexed and even-indexed nodes into two chains; append even chain to odd chain |
| 3 | Reorder List | 143 | Medium | Fast/Slow + Reverse + Merge | Find mid; reverse second half; interleave first half with reversed second half |

### Code Skeleton
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Intersection (LC 160)
    public static ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode a = headA, b = headB;
        while (a != b) {
            a = (a != null) ? a.next : headB;
            b = (b != null) ? b.next : headA;
        }
        return a;  // null if no intersection
    }

    // Reorder List (LC 143) — skeleton
    public static void reorderList(ListNode head) {
        // 1. find mid (fast/slow)
        // 2. reverse second half in-place
        // 3. interleave: first, second, first.next, second.next, ...
        // TODO: implement
    }
}
```

### Interview Tips

- **Intersection — state the equidistance trick upfront:** "Both pointers traverse len(A) + len(B) total steps — they land on the intersection node simultaneously because they've covered equal distance. If there's no intersection, both hit null at the same time and the while loop exits."
- **Reorder List — announce all three steps before coding:** "(1) Find mid with fast/slow. (2) Reverse second half in-place. (3) Interleave: save both `next` pointers before relinking." Forgetting step 3's save-before-relink causes a pointer loss.
- **Brute force for intersection:** store all nodes of A in a set, scan B looking for membership — O(m+n) time, O(m) space. The two-pointer trick achieves O(m+n) time and O(1) space.
- **Common mistake for Reorder List:** calling the interleave step before reversing the second half — the second half must be reversed first so `left` and `right` converge toward each other, not away.

### STAR Interview Framework

> **How to use the STAR method when explaining Two Pointers on Two Lists + In-Place Reorder in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given two linked lists that may share a common tail, and asked to find the node where they intersect — or return null. A HashSet approach stores all nodes of one list and scans the other — O(m+n) time but O(m) space. The two-pointer equidistance trick achieves O(m+n) time and O(1) space."

**Task:** "My goal was to find the intersection node in O(1) extra space by recognising that if both pointers traverse both lists — A then B, and B then A — they cover the same total distance and meet at the intersection node."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Find where two structures converge without extra storage → two-pointer equidistance trick. Trigger: 'intersection of two linked lists' or 'where two paths join.'"
2. *Initialize:* "Set `a = headA, b = headB`. The loop condition is `while a != b` — when both are null and there's no intersection, both reach null simultaneously and the loop exits."
3. *Core loop logic:* "`a = a != null ? a.next : headB; b = b != null ? b.next : headA`. When `a` reaches null, redirect to `headB`. When `b` reaches null, redirect to `headA`. Both pointers now traverse exactly `len(A) + len(B)` total steps."
4. *Convergence guarantee:* "If an intersection exists at distance d from each list's head, both pointers reach it after `len(A) + len(B) - d` steps. If no intersection, both reach null simultaneously after exactly `len(A) + len(B)` steps."
5. *Duplicate handling / edge case proactivity:* "The equality check is pointer equality (`a == b`), not value equality. Two nodes with the same value but at different memory addresses are NOT the same intersection node."

**Result:** "O(m+n) time, O(1) space vs O(m) for a HashSet. The equidistance property eliminates the need for length computation. For lists of length m = n = 10⁵, this saves 800KB of HashSet memory for a 10⁵-node set."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Two Pointers here |
|-------------|---------------------------|-------------------------------|
| HashSet of nodes from list A | When O(m) space is acceptable | O(1) space vs O(m); equidistance trick is the canonical O(1)-space solution |
| Compute lengths, align, scan together | When you want an easier-to-explain approach | Also O(m+n) time and O(1) space — but two-pointer equidistance is more elegant and tests pattern knowledge |

**Why NOT HashSet:** O(m) extra space. For m = 10⁵ nodes, that's ~800KB for the set vs two pointer variables.
**Why NOT length-align:** Both approaches are O(m+n), O(1) — equidistance is slightly more elegant and the pattern interviewers specifically test.

### Edge Cases to Trace Before Coding
- LC 160: no intersection → both pointers reach null simultaneously; `while a != b` exits when both are null ✓
- LC 160: both lists are the same list → both pointers immediately equal; answer is headA ✓
- LC 328 (Odd Even): single node → nothing to rearrange; return head ✓
- LC 143: two-node list → first half is node 1, second half (reversed) is node 2; interleave produces [1,2] ✓

## System Design (1 hour)
### Topic: Horizontal Scaling — Database Bottleneck & Read Replicas
- Web servers scale horizontally easily (stateless); the database is the usual bottleneck because it holds state.
- **Read replicas:** primary DB handles writes; replicas serve reads; typical web apps are 80–90% reads, so replicas absorb most load.
- **Replication lag:** replicas may be milliseconds behind the primary — stale reads are possible; use primary for reads that require strong consistency (e.g., immediately after a write).
- **Sharding:** split data across multiple primary DBs by a shard key; allows write scaling but introduces cross-shard query complexity.
- Interview talking point: "If asked how to scale a relational DB that is becoming a write bottleneck, answer: first add read replicas for read relief; for write scaling, consider sharding by user_id or region, or moving to an append-only event log (CQRS) — but warn that sharding adds operational complexity and cross-shard joins become expensive."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Are Right, A Lot

**STAR Story: Interleaving Two Teams' Feature Work Into a Unified API**

**Situation (20%):** "Two teams at my company were building complementary features that both needed to be surfaced through a single unified API endpoint: Team A was building real-time pricing data and Team B was building inventory availability. Both were 6 weeks into development when product decided the features should be interleaved in the API response rather than returned as two separate sections — each pricing item paired with its corresponding inventory item in alternating order."

**Task (part of S/T):** "As the platform engineer responsible for the API layer, my goal was to merge both teams' outputs into the interleaved response format without requiring either team to change their backend logic — preserving their independent development timelines."

**Action (60-70% — be specific about what YOU did):**
"First, I analyzed both response schemas and confirmed they shared a common product ID key — I could join them on that key, analogous to the Reorder List's ability to pair nodes by position once the second half is reversed.
Then, I designed an adapter layer in the API gateway that fetched both response streams in parallel (reducing latency vs sequential), then interleaved the results: for each position i, the response contained `pricing[i]` followed by `inventory[i]`.
Next, I built unit tests covering equal-length responses, pricing-longer responses, and inventory-longer responses — covering all three length relationships.
Finally, I ran an integration test with both teams' staging environments and confirmed the interleaved output matched the product spec exactly."

**Result (10-20%):** "The interleaved API went live on schedule — no delay to either team's timeline. API response time was 45ms vs the ~110ms that sequential fetching would have produced. The adapter pattern I built was reused for 4 subsequent feature combinations over the next quarter, each time avoiding a redesign of either contributing service."

**Interview tip:** Are Right, A Lot rewards technical judgment calls with good outcomes. The key is "I analyzed the schema, confirmed the join key existed, designed the adapter, and validated correctness" — showing methodical reasoning, not just intuition. Prepare for: are right a lot, invent and simplify, earn trust, or technical decision-making.

## Flashcards

| Q | A |
|---|---|
| Why do both pointers in LC 160 (Intersection) travel the same total distance? | Each pointer traverses list A then list B (or B then A) — total = len(A) + len(B); they arrive at the intersection node on the same step |
| How do you interleave two halves in Reorder List without losing nodes? | Save `first.next` and `second.next` before relinking; then set `first.next = second` and `second.next = saved_first_next` |
| What is the loop invariant for Odd Even Linked List? | `odd` pointer always points to the last node in the odd chain; `even` always points to the last node in the even chain |
| What is replication lag and when does it matter? | The delay between a write on the primary DB and its appearance on replicas; matters when you read immediately after a write and require seeing the latest data |
| When does sharding become necessary vs. adding more read replicas? | Sharding is needed when *write* throughput saturates the primary; read replicas only help with read load |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
