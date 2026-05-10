# Day 19 — Linked Lists: Pattern Mix & Twin Sum
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Apply fast/slow and monotonic stack to linked lists — problems that combine this week's patterns in non-obvious ways.

## DSA (2 hours)
### Pattern: Fast/Slow + Stack on Linked Lists
- Next Greater Node: convert list to array then apply monotonic stack (decreasing), or traverse list pushing values onto a stack to find next greater.
- Maximum Twin Sum: fast/slow to find mid; reverse second half; walk both halves simultaneously summing pairs; track maximum.
- Delete Middle Node: fast/slow to find the node just *before* the middle; then delete middle with `prev.next = prev.next.next`.
- Trigger condition: "next greater in a list" → monotonic stack; "symmetric pairs from both ends" → fast/slow + reverse.
- Time complexity: O(n) | Space complexity: O(n) for next greater, O(1) for twin sum

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Delete the Middle Node of a Linked List | 2095 | Medium | Fast/Slow (offset) | Advance slow by 1 and fast by 2, but start fast one step ahead so slow stops at node *before* middle |
| 2 | Next Greater Node In Linked List | 1019 | Medium | Monotonic Stack | Convert list to array; apply decreasing monotonic stack; stack stores indices, not values |
| 3 | Maximum Twin Sum of a Linked List | 2130 | Medium | Fast/Slow + Reverse + Two Pointers | Find mid, reverse second half, walk inward with two pointers summing node pairs |

### Code Skeleton
```python
# Maximum Twin Sum (LC 2130)
def pair_sum(head):
    # Step 1: find mid
    slow, fast = head, head
    while fast and fast.next:
        slow, fast = slow.next, fast.next.next
    # Step 2: reverse second half starting at slow
    prev, curr = None, slow
    while curr:
        curr.next, prev, curr = prev, curr, curr.next
    # Step 3: two pointers from both ends
    left, right = head, prev
    max_sum = 0
    while right:
        max_sum = max(max_sum, left.val + right.val)
        left, right = left.next, right.next
    return max_sum

# Next Greater Node (LC 1019) — skeleton
def next_larger_nodes(head):
    # 1. collect values into a list
    # 2. apply monotonic decreasing stack (stores indices)
    # 3. result[i] = value of next greater element, 0 if none
    pass
```

## System Design (1 hour)
### Topic: Scaling a Web Application — End-to-End Architecture
- **Tier 1 — DNS + CDN:** route users to the nearest edge; static assets (JS, CSS, images) served from CDN, not the origin — reduces origin load by 80–90%.
- **Tier 2 — Load Balancer:** Layer 7 ALB routes traffic across stateless web servers; auto-scaling group maintains the right server count.
- **Tier 3 — App servers:** stateless; read from primary DB or replica; write to primary DB; heavy async tasks pushed to a message queue.
- **Tier 4 — Data:** primary DB + read replicas for relational data; Redis cache in front for hot-path reads; S3 for blob storage.
- Interview talking point: "If asked to sketch a scalable web app from scratch, start with this four-tier diagram: DNS → CDN → LB + auto-scaling app servers → DB primary/replica + Redis cache — then add details per the interviewer's constraints."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe how you would architect a system that could handle a 100x traffic increase in 30 minutes — and what the bottlenecks would be at each stage.
- Leadership principle: Think Big

## Flashcards

| Q | A |
|---|---|
| How do you stop slow at the node *before* the middle in Delete Middle Node? | Initialise `slow = head`, `fast = head.next` (one step ahead); advance both — when fast stops, slow is at the pre-middle node |
| Why does the monotonic stack for Next Greater Node store indices, not values? | You need the index to fill the result array at the correct position; storing only the value loses the mapping |
| How does the twin sum algorithm handle even-length lists? | Fast/slow gives slow at the exact midpoint (second of the two middle nodes for even-length); reversing from there gives the correct second half |
| In the four-tier web architecture, what is the primary role of the CDN? | Serve static assets from the nearest edge node, reducing origin server load and decreasing latency for geographically distributed users |
| Why is Redis placed in front of the DB for the hot-path reads rather than replacing the DB? | Redis is in-memory and fast but does not offer ACID transactions or durable storage; it serves as a read cache for frequently accessed data, not the source of truth |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
