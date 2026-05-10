# Day 11 — Linked Lists: Reversal Pattern
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Build the iterative linked list reversal — the fundamental building block for nearly every hard linked list problem.

## DSA (2 hours)
### Pattern: Linked List Reversal
- Maintain three pointers: `prev = None`, `curr = head`, `next_node`; on each step save `curr.next`, repoint `curr.next = prev`, then advance both pointers.
- Trigger condition: "reverse a list or a sublist", "check for palindrome in a list", or any problem that requires traversing in reverse order without extra space.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Reverse Linked List | 206 | Easy | Iterative Reversal | Three-pointer dance: prev, curr, next; return prev when curr is None |
| 2 | Reverse Linked List II | 92 | Medium | Partial Reversal | Walk to position left−1; reverse exactly right−left+1 nodes; stitch back both ends |
| 3 | Swap Nodes in Pairs | 24 | Medium | Group Reversal (k=2) | Use a dummy node; for each pair swap then advance by 2; handle odd-length lists |

### Code Skeleton
```python
# Full reversal (LC 206)
def reverse_list(head):
    prev, curr = None, head
    while curr:
        next_node = curr.next
        curr.next = prev
        prev = curr
        curr = next_node
    return prev

# Partial reversal (LC 92) — skeleton only
def reverse_between(head, left, right):
    dummy = ListNode(0, head)
    prev = dummy
    # 1. advance prev to node before position left
    # 2. reverse right-left+1 nodes
    # 3. stitch: pre.next = ... tail.next = ...
    return dummy.next
```

## System Design (1 hour)
### Topic: Vertical vs. Horizontal Scaling — Core Concepts
- **Vertical scaling (scale up):** add more CPU/RAM to a single machine; simple, no code changes, but has a hard ceiling and creates a single point of failure.
- **Horizontal scaling (scale out):** add more machines and distribute load; theoretically unlimited, enables redundancy, but requires stateless services and a coordination layer.
- Key trade-off: vertical is faster to deploy and cheaper at small scale; horizontal is more resilient and cost-effective at large scale.
- Stateless requirement: to scale horizontally, servers must not store session state locally — move state to a shared cache (Redis) or database.
- Interview talking point: "If asked when to choose vertical over horizontal scaling, answer: vertical is appropriate when you need to scale quickly, the bottleneck is a single process that cannot be parallelised (e.g., a legacy monolith), and you are still within the machine's capacity ceiling — otherwise horizontal."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a situation where you had to change the direction of a project mid-way through — analogous to reversing a linked list by rethreading pointers rather than creating a new one.
- Leadership principle: Bias for Action

## Flashcards

| Q | A |
|---|---|
| Write the three-pointer iterative linked list reversal loop body. | `next_node = curr.next; curr.next = prev; prev = curr; curr = next_node` |
| What does the dummy node buy you in linked list problems? | Eliminates the edge case of operating on the head node — you can always access `dummy.next` as the new head without special-casing |
| What is the time and space complexity of reversing a linked list in-place? | O(n) time, O(1) space |
| What is the key difference between vertical and horizontal scaling? | Vertical: bigger machine, single node, hard ceiling. Horizontal: more machines, requires stateless architecture, no theoretical ceiling |
| Why must services be stateless to scale horizontally? | Any server in the pool may handle any request; if state is stored locally, a different server won't have it — move state to a shared store |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
