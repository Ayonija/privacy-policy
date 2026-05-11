# Day 12 — Linked Lists: Fast/Slow Pointers
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Master the fast/slow pointer (Floyd's algorithm) — detects cycles, finds midpoints, and enables in-place palindrome checks.

## DSA (2 hours)
### Pattern: Fast / Slow Pointers (Floyd's Tortoise and Hare)
- Move slow one step and fast two steps; if they meet, a cycle exists. To find the cycle entry, reset one pointer to head and advance both one step at a time.
- Trigger condition: cycle detection, finding the middle of a list, or splitting a list in half without knowing its length.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Linked List Cycle | 141 | Easy | Fast/Slow | If fast ever equals slow (both non-None), a cycle exists |
| 2 | Linked List Cycle II | 142 | Medium | Fast/Slow + Math | After meeting point, reset slow to head; advance both one step — they meet at cycle entry |
| 3 | Palindrome Linked List | 234 | Medium | Fast/Slow + Reversal | Find mid with slow/fast; reverse second half in-place; compare two halves; restore (optional) |

### Code Skeleton
```python
# Cycle detection (LC 141)
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:   # fast.next check prevents crash on fast.next.next
        slow = slow.next
        fast = fast.next.next
        if slow == fast:        # same node reference (pointer equality, not value)
            return True
    return False

# Cycle entry (LC 142)
def detect_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow, fast = slow.next, fast.next.next
        if slow == fast:
            slow = head         # reset one pointer to head
            while slow != fast:
                slow, fast = slow.next, fast.next  # both advance one step
            return slow         # meet at cycle entry node
    return None
```

### Interview Tips

- **State both guard conditions:** "`while fast and fast.next` — we check `fast` (could be None at list end) AND `fast.next` (would crash on `fast.next.next`). Both guards matter." Saying this proves you think about null-pointer safety.
- **Pointer equality, not value equality:** `slow == fast` compares object references — they must point to the exact same node, not just nodes with equal values. Mentioning this separates candidates who understand references from those who don't.
- **LC 142 — state the math:** "After the meeting point, the mathematical property gives: distance(head → cycle_entry) == distance(meeting_point → cycle_entry, going forward). So resetting one pointer to head and advancing both one step places them at the cycle entry." Interviewers won't ask you to prove it, but showing awareness of the property scores points.
- **LC 234 three-step blueprint:** (1) find middle — fast/slow, (2) reverse second half in-place — three-pointer reversal, (3) compare half by half until one hits None. State all three before coding.
- **Common mistake:** in LC 234, after finding mid with fast/slow, `slow.next` is the start of the second half — but for even-length lists the "second half" must start at `slow.next` (not `slow`). Trace a 4-node example to confirm.

### Edge Cases to Trace Before Coding
- LC 141: no cycle, single node → `fast` becomes None after 0 iterations; return False ✓
- LC 141: tail points to head (full cycle) → fast catches slow after a few iterations ✓
- LC 234: `[1]` → single node is a palindrome; fast immediately hits end, slow at head, second half is empty ✓
- LC 234: `[1, 2]` → not palindrome; after fast/slow, slow is at node 1, reverse node 2, compare 1 ≠ 2 ✓
- LC 142: no cycle → return None (while loop exits normally)

## System Design (1 hour)
### Topic: Horizontal Scaling — Stateless Architecture Patterns
- **Stateless services:** every request carries all needed context (JWT token, request parameters); server holds no per-user state between requests.
- **Shared session store:** if session state is unavoidable, externalise it to Redis or Memcached; all server instances read from the same store.
- **Sticky sessions (session affinity):** load balancer routes a client to the same server for every request; avoids shared state but reintroduces single-server dependence.
- Sticky sessions are an anti-pattern for true horizontal scale — prefer stateless + shared cache.
- Interview talking point: "If asked how to make a stateful monolith horizontally scalable, answer: extract session state to Redis, move file uploads to object storage (S3), and ensure the service reads configuration from environment variables or a config service — then you can run N identical replicas behind a load balancer."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Tell me about a time you found a hidden issue in a system that only revealed itself under cyclic or repeated conditions — analogous to Floyd's cycle detection finding a loop that a single linear scan misses.
- Leadership principle: Dive Deep

## Flashcards

| Q | A |
|---|---|
| Why does Floyd's algorithm work — what is the mathematical proof sketch? | Let F = distance to cycle start, C = cycle length, k = meeting point offset inside cycle; when they meet: 2*(F+k) = F+k+n*C → F = n*C − k; resetting slow to head and advancing both one step places them both at cycle entry after F more steps |
| How do you find the middle of a linked list using fast/slow pointers? | Start both at head; advance slow one step and fast two steps; when fast is None or fast.next is None, slow is at the middle |
| In Palindrome Linked List, why do you reverse the second half instead of using a stack? | Reversing in-place uses O(1) extra space; a stack uses O(n) |
| What invariant guarantees Floyd's algorithm terminates? | Once inside a cycle, slow and fast are in the same cycle; fast gains one position per step on slow, so they must meet within C steps (cycle length) |
| When is sticky sessions preferable to a shared session store? | When latency to the external session store is unacceptably high, or the service cannot be refactored to be stateless — but it trades scale for simplicity |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
