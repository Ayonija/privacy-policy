# Day 14 — Review Day: Linked Lists Week 2
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Solidify all linked list patterns from Days 11–13; identify the remove-nth and dedup variants before moving to Week 03.

## DSA (2 hours)
### Pattern: Review — Remove, Dedup, and Two-Pointer on Lists
- Remove Nth From End: use two pointers with a gap of n; when the fast pointer hits the end, slow is at the node before the target.
- Remove duplicates from sorted list II: delete ALL nodes with duplicate values (not just duplicates), requiring a prev pointer that stops before the duplicate run.
- These patterns extend fast/slow and dummy-head thinking into deletion-focused problems.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Middle of the Linked List | 876 | Easy | Fast/Slow | When fast reaches end, slow is at mid; for even-length lists slow is the second middle |
| 2 | Remove Nth Node From End of List | 19 | Medium | Two Pointers (gap n) | Dummy head; advance fast by n+1 steps first; then move both until fast is None |
| 3 | Remove Duplicates from Sorted List II | 82 | Medium | Dummy head + skip run | When a duplicate run is detected, skip all nodes with that value; prev.next jumps over entire run |

### Code Skeleton
```python
# Remove Nth from end (LC 19)
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)
    fast = slow = dummy
    for _ in range(n + 1):   # advance fast n+1 steps — leaves 1-node gap between slow and fast
        fast = fast.next
    while fast:               # advance both until fast is None
        fast, slow = fast.next, slow.next
    slow.next = slow.next.next  # slow is now at the node BEFORE the target
    return dummy.next

# Remove duplicates II (LC 82)
def delete_duplicates(head):
    dummy = ListNode(0, head)
    prev = dummy   # prev stops before any duplicate run
    curr = head
    while curr:
        if curr.next and curr.val == curr.next.val:
            val = curr.val
            while curr and curr.val == val:  # skip ALL nodes with this value
                curr = curr.next
            prev.next = curr   # prev jumps over the entire duplicate run
        else:
            prev = curr        # non-duplicate: advance prev normally
            curr = curr.next
    return dummy.next
```

### Interview Tips

- **The n+1 gap trick:** "I advance `fast` by `n+1` steps from the dummy head (not n) so that when `fast` is None, `slow` is at the node *before* the one I want to delete — allowing `slow.next = slow.next.next`." Draw this on the whiteboard if possible.
- **Why dummy head is essential for LC 19:** if `n` equals the list length, the node to remove is the head — without a dummy, you'd crash accessing `slow.next.next`. Dummy makes the head removal identical to any other node removal.
- **LC 82 vs LC 83 (Remove Duplicates I):** LC 83 keeps one copy of each duplicate; LC 82 removes ALL copies. The key difference: LC 82's `prev` pointer *doesn't advance* when it encounters a duplicate run; it only advances on non-duplicate nodes.
- **Common mistake in LC 82:** checking `curr.val == curr.next.val` without guarding `curr.next is not None` — accessing `curr.next.val` when `curr.next` is None throws an error. Always check `curr.next` first.
- **Brute force for LC 876 (Middle):** count nodes, then traverse again to the midpoint — O(2n). Fast/slow finds the middle in one pass O(n).

### Edge Cases to Trace Before Coding
- LC 19: `n` equals list length → remove head; dummy prevents crash; `slow` stays at dummy after n+1 advance ✓
- LC 19: single-node list, `n = 1` → remove head; dummy.next becomes None ✓
- LC 82: list with all same values `[1,1,1]` → entire list is a duplicate run; `prev` stays at dummy; `prev.next = None`; return `dummy.next = None` ✓
- LC 82: no duplicates → `curr.next` is always different from `curr`; `prev` advances every step; list returned unchanged ✓
- LC 876: even-length list → slow ends at the second middle node (as per problem definition)

## System Design (1 hour)
### Topic: Load Balancers — Layer 4 vs. Layer 7
- **Layer 4 (Transport):** routes based on IP + TCP/UDP port; no knowledge of content; very fast, low overhead; cannot make routing decisions based on URL path or headers.
- **Layer 7 (Application):** inspects HTTP headers, URL, cookies; can route `/api` to one server farm and `/static` to another; supports A/B testing and canary deployments.
- Layer 4 example: AWS Network Load Balancer (NLB); Layer 7 example: AWS Application Load Balancer (ALB) / Nginx.
- Trade-offs: L4 is faster and simpler; L7 is more flexible but adds latency (must fully parse the HTTP request).
- Interview talking point: "If asked which load balancer layer to use for a microservices API gateway, answer: Layer 7 — you need path-based routing (`/users` → User Service, `/orders` → Order Service) and header inspection for auth, which Layer 4 cannot provide."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Tell me about a time you had to remove or skip certain inputs from a process because they didn't meet a quality bar — analogous to skipping entire duplicate runs in LC 82.
- Leadership principle: Insist on the Highest Standards

## Flashcards

| Q | A |
|---|---|
| How do you set the gap between two pointers to find the nth node from the end? | Advance fast pointer by n+1 steps from the dummy head; then advance both until fast is None; slow is now just before the target |
| Why advance n+1 steps (not n) in Remove Nth From End? | You want slow to stop at the node *before* the target so you can do `slow.next = slow.next.next`; n+1 leaves that one-step buffer |
| How do you skip an entire duplicate run in LC 82? | Detect `curr.val == curr.next.val`; record the value; advance curr while `curr.val == recorded_val`; then set `prev.next = curr` (bypassing all duplicate nodes) |
| What routing capability does a Layer 7 load balancer have that Layer 4 cannot? | Path-based and header-based routing (e.g., route `/api` vs `/static` to different backends, inspect auth headers, enable canary deployments) |
| Name a real AWS service for each load balancer layer. | Layer 4: Network Load Balancer (NLB) — ultra-low latency, TCP/UDP. Layer 7: Application Load Balancer (ALB) — HTTP/HTTPS, path/host routing |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
