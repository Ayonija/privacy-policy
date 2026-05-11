# Day 11 — Linked Lists: Reversal Pattern
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Build the iterative linked list reversal — the fundamental building block for nearly every hard linked list problem.

## DSA (2 hours)
### Pattern: Linked List Reversal
- Maintain three pointers: `prev = null`, `curr = head`, `nextNode`; on each step save `curr.next`, repoint `curr.next = prev`, then advance both pointers.
- Trigger condition: "reverse a list or a sublist", "check for palindrome in a list", or any problem that requires traversing in reverse order without extra space.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Reverse Linked List | 206 | Easy | Iterative Reversal | Three-pointer dance: prev, curr, next; return prev when curr is null |
| 2 | Reverse Linked List II | 92 | Medium | Partial Reversal | Walk to position left−1; reverse exactly right−left+1 nodes; stitch back both ends |
| 3 | Swap Nodes in Pairs | 24 | Medium | Group Reversal (k=2) | Use a dummy node; for each pair swap then advance by 2; handle odd-length lists |

### Code Skeleton
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Full reversal (LC 206)
    public static ListNode reverseList(ListNode head) {
        ListNode prev = null, curr = head;
        while (curr != null) {
            ListNode nextNode = curr.next;   // SAVE next before overwriting
            curr.next = prev;               // reverse the pointer
            prev = curr;                    // advance prev
            curr = nextNode;                // advance curr
        }
        return prev;   // prev is the new head when curr is null
    }

    // Partial reversal (LC 92) — skeleton only
    public static ListNode reverseBetween(ListNode head, int left, int right) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prev = dummy;
        // Step 1: walk prev to node at position (left - 1)
        for (int i = 0; i < left - 1; i++) {
            prev = prev.next;
        }
        // Step 2: reverse (right - left + 1) nodes starting at prev.next
        // Step 3: stitch — prev.next = reversed_head; reversed_tail.next = remainder
        return dummy.next;
    }
}
```

### Interview Tips

- **Draw before coding:** draw 3–4 nodes as boxes with arrows, then draw what you want after reversal. Show this to the interviewer. Linked list problems have high visual debugging value.
- **The three-pointer dance — say it out loud:** "1. Save next. 2. Reverse pointer. 3. Advance prev. 4. Advance curr." Verbalise all four steps; missing step 1 (saving next) is the most common reversal bug.
- **Dummy head rule:** any problem that might modify the head node should start with `dummy = new ListNode(0); dummy.next = head` and return `dummy.next`. This eliminates all special-casing of the head.
- **Brute force alternative:** collect all values in a list, reverse the sublist, rewrite values into nodes — O(n) time, O(n) space. The in-place approach is O(n) time, O(1) space. State both and say you'll implement the O(1) space version.
- **Common mistake:** in LC 92, forgetting to save the node at position `left` as the "tail" of the reversed sublist before reversal — you need it to stitch `tail.next = node_after_right` at the end.

### Edge Cases to Trace Before Coding
- LC 206: empty list (`head = null`) → return `null`; `prev` starts at `null`, returns `null` ✓
- LC 206: single node → loop runs once, `prev = head`, `curr = null`, return `head` ✓
- LC 92: `left == right` → nothing to reverse; ensure your loop handles 0 reversals gracefully
- LC 92: `left = 1` → the dummy head handles this; `prev = dummy`, so `prev.next` is the new reversed head ✓
- LC 24 (Swap Pairs): odd-length list → last node has no pair; leave it unchanged

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
| Write the three-pointer iterative linked list reversal loop body. | `nextNode = curr.next; curr.next = prev; prev = curr; curr = nextNode` |
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
