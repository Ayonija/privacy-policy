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

### Interview Tips

- **Maximum Twin Sum — announce all three steps before coding:** "(1) Find mid with fast/slow. (2) Reverse second half from `slow`. (3) Walk two pointers from both ends, summing pairs, tracking maximum." State all three before writing any code.
- **Delete Middle Node — use `fast = head.next` offset:** "Starting `fast` one step ahead of the usual position makes `slow` stop at the node BEFORE the middle, enabling deletion via `slow.next = slow.next.next`."
- **Next Greater Node — store indices, not values:** "The result array needs to be filled at index `i`, so I store the index in the stack. If I stored values, I'd lose the mapping from element to position."
- **Brute force for Next Greater Node:** O(n²) — for each element, scan right until a greater value is found. Monotonic stack reduces to O(n).

### STAR Interview Framework

> **How to use the STAR method when explaining Fast/Slow + Stack on Linked Lists in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list of even length and asked to find the maximum twin sum — where twin pairs are the first and last nodes, second and second-to-last nodes, etc. Naively, I could convert the list to an array and use two pointers — O(n) time but O(n) space. The in-place approach achieves O(n) time and O(1) space by reversing the second half."

**Task:** "My goal was to pair nodes from both ends of the list without extra storage, using fast/slow to find the midpoint, reversing the second half in-place, then summing pairs with two inward-moving pointers."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Pair nodes from both ends of a list → find midpoint with fast/slow, reverse second half, two-pointer inward traversal. Trigger: 'symmetric pairs from a list.'"
2. *Initialize:* "Fast/slow to find mid: `while fast and fast.next: slow = slow.next; fast = fast.next.next`. After the loop, `slow` is at the midpoint — start of the second half."
3. *Core loop logic:* "Reverse from `slow` using the three-pointer template. Then set `left = head, right = prev` (head of reversed second half). While `right != null`: `maxSum = max(maxSum, left.val + right.val); left = left.next; right = right.next`."
4. *Convergence guarantee:* "The loop runs exactly `n/2` times — one pass over each half. Total O(n)."
5. *Duplicate handling / edge case proactivity:* "For even-length lists (guaranteed by the problem), fast/slow places `slow` at exactly the second of the two middle nodes — the correct start of the second half to reverse."

**Result:** "O(n) time, O(1) space vs O(n) space for an array conversion. The three-step blueprint — find mid, reverse, two-pointer — is reusable for any 'symmetric traversal of a list' problem. Announcing all three steps before coding is the key communication move that earns credit."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer In-Place Reversal here |
|-------------|---------------------------|-------------------------------|
| Convert list to array, two pointers from ends | When space is not constrained | O(n) extra space; in-place is O(1) and is the pattern being tested |
| Stack to reverse second half | When implementing reversal from scratch is error-prone | O(n/2) extra space for the stack; in-place reversal is O(1) |

**Why NOT array:** O(n) extra space — problem almost always expects O(1) for this class of linked list problem.
**Why NOT stack for reversal:** O(n/2) extra space — in-place three-pointer reversal uses O(1).

### Edge Cases to Trace Before Coding
- LC 2095 (Delete Middle): single node → no middle to delete; return null. Two nodes → delete second node
- LC 1019 (Next Greater): all elements same → no element has a greater neighbor; result is all zeros
- LC 2130 (Twin Sum): two-node list → single pair; answer is their sum ✓
- LC 1019: monotonically decreasing → stack fills completely, nothing gets popped; result is all zeros

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

**Leadership Principle:** Think Big

**STAR Story: Architecting for a Viral Traffic Event That Exceeded 100x Normal Load**

**Situation (20%):** "At my previous company, we ran a product launch event that was expected to generate 5× our normal peak traffic. Instead, a high-profile influencer mentioned us 20 minutes before the launch, and traffic spiked to 130× normal in 8 minutes — from 500 RPS to 65,000 RPS. Our original architecture — a single auto-scaling group of app servers hitting a single primary PostgreSQL database — began dropping requests at 12,000 RPS. The product was effectively down for 22 minutes."

**Task (part of S/T):** "I was the lead engineer on the post-mortem. My goal was to redesign the architecture so it could handle 200× normal traffic within 15 minutes of a spike, without pre-provisioning infrastructure."

**Action (60-70% — be specific about what YOU did):**
"First, I identified the bottleneck chain: CDN handled static assets fine, app servers scaled but overloaded the DB connection pool, and the DB became the single point of contention.
Then, I designed a three-layer protection: (1) I added a Redis cache in front of the DB for all read-heavy product catalog queries — these accounted for 85% of DB traffic. (2) I configured read replicas and updated the app to route reads to replicas. (3) I moved the order-write path to an async queue (SQS), so writes were acknowledged immediately and processed at the DB's max throughput.
Next, I load-tested the new architecture to 300× normal traffic in a staging environment and confirmed the DB query rate stayed within limits even under spike conditions.
Finally, I implemented a chaos engineering exercise — intentionally spiked traffic at 3× expected capacity monthly to validate the system held."

**Result (10-20%):** "In the next high-traffic event 3 months later (85× spike from a press mention), the system handled 95% of requests without degradation. The remaining 5% were gracefully degraded — the queue backed up but no requests were dropped. The post-mortem became a company reference document for scalability planning."

**Interview tip:** Think Big rewards architectural thinking and proactive failure mode analysis. "I identified the bottleneck chain at each tier" and "I load-tested to 300×" show you thought beyond the immediate fix. Prepare for: think big, ownership, bias for action, or failure prevention.

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
