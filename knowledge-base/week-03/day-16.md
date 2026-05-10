# Day 16 — Linked Lists: Intersection & Reorder
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Solve intersection detection and in-place reordering — problems that demand both pointer intuition and careful stitching.

## DSA (2 hours)
### Pattern: Two Pointers on Two Lists + In-Place Reorder
- Intersection: advance both pointers; when one reaches None, redirect it to the other list's head; they meet at the intersection node (or both hit None if no intersection) — total traversal = m + n for both.
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
```python
# Intersection (LC 160)
def get_intersection_node(headA, headB):
    a, b = headA, headB
    while a != b:
        a = a.next if a else headB
        b = b.next if b else headA
    return a  # None if no intersection

# Reorder List (LC 143) — skeleton
def reorder_list(head):
    # 1. find mid (fast/slow)
    # 2. reverse second half in-place
    # 3. interleave: first, second, first.next, second.next, ...
    pass
```

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
- STAR prompt: Describe a time you had to interleave work from two parallel streams — such as merging two teams' outputs into a unified deliverable — analogous to the merge step in Reorder List.
- Leadership principle: Are Right, A Lot

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
