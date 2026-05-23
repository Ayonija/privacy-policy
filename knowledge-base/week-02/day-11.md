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

### STAR Interview Framework

> **How to use the STAR method when explaining Linked List Reversal in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list and asked to reverse a sublist from position `left` to `right` in-place. The brute-force alternative — collect all values into an array, reverse the subarray, write back — is O(n) time but O(n) space. The in-place approach must achieve O(n) time and O(1) space."

**Task:** "My goal was to reverse exactly `right - left + 1` nodes in-place by rethreading pointers, without allocating any extra data structure and without modifying node values."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "In-place reversal with pointer rethreading — the three-pointer dance. The pattern trigger: 'reverse a list or sublist without extra space.'"
2. *Initialize:* "I use a dummy head: `dummy.next = head`. I walk `prev` to the node at position `left - 1` in `left - 1` steps. I save `tail = prev.next` (the future tail of the reversed sublist)."
3. *Core loop logic:* "I reverse `right - left + 1` nodes using the three-pointer template: save `nextNode = curr.next`, repoint `curr.next = prev`, advance both. After the loop, `prev` is the new head of the reversed sublist and `tail` is the new tail."
4. *Convergence guarantee:* "The reversal loop runs exactly `right - left + 1` times — bounded and deterministic. No backtracking, O(n) total."
5. *Duplicate handling / edge case proactivity:* "The two critical stitches after reversal: `prev_left_minus_one.next = reversed_head` and `tail.next = node_after_right`. Missing the second stitch cuts off the remainder of the list — trace this on a 5-node example to catch it."

**Result:** "This achieves O(n) time and O(1) space. The key invariant to state: the original head of the sublist becomes the tail of the reversed sublist — save it before reversal as `tail`, then stitch `tail.next = remainder`."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer In-Place Reversal here |
|-------------|---------------------------|-------------------------------|
| Collect values → reverse array → rewrite nodes | When space is not constrained and correctness matters more than elegance | O(n) space vs O(1); problem typically requires in-place; also more error-prone if list has complex node structure |
| Recursive reversal | For full list reversal when stack depth is acceptable | Recursive uses O(n) call stack (implicit space); iterative is O(1) space and preferred at L5+ |

**Why NOT array-based:** O(n) extra space; most interview problems explicitly require O(1) space for linked list reversal.
**Why NOT recursive:** O(n) stack space; can overflow for large lists; iterative three-pointer is the canonical O(1) solution expected in interviews.

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

**Leadership Principle:** Bias for Action

**STAR Story: Pivoting a Project Mid-Development Without Losing Existing Work**

**Situation (20%):** "Six weeks into a 12-week project to build a real-time user segmentation feature, stakeholder feedback revealed that the core assumption — that users would be segmented at login — was wrong. Marketing needed segments to update continuously throughout a session, not just at login. This meant our entire event ingestion architecture needed to be fundamentally changed, but we couldn't afford to discard the 6 weeks of UI and API work already done."

**Task (part of S/T):** "I was the tech lead. My goal was to pivot the ingestion architecture to support continuous segmentation while preserving all the UI and API work — in 2 weeks, so we could finish the project on the original timeline."

**Action (60-70% — be specific about what YOU did):**
"First, I did a rapid dependency analysis over 2 days to identify what we could keep vs. what needed to be rebuilt. The UI layer, the segment API, and the segment storage schema were all stable. The only layer that needed to change was the event stream processor — analogous to rethreading the links in the existing chain rather than building a new list.
Then, I proposed keeping all existing nodes (UI, API, storage) and replacing only the event processor link — I added a streaming Kafka consumer that updated segment membership in real time, replacing the batch login-time processor.
Next, I split the implementation: I took the Kafka consumer myself, assigned two engineers to integration testing, and assigned one to load testing the new high-frequency write pattern.
Finally, I presented the pivot plan to the PM and got approval within 24 hours, avoiding a 3-week delay that would have come from calling an all-hands design review."

**Result (10-20%):** "The pivot was complete in 11 days — within the 2-week target. The project shipped 1 week behind the original date instead of the 3-week delay the full rebuild would have caused. The real-time segmentation feature processed 850K segment updates per hour in production without performance issues. Acting quickly with a targeted plan preserved 6 weeks of existing work."

**Interview tip:** Bias for Action questions reward decisive pivots with a plan. Show you assessed what to keep, moved quickly on the change, and communicated proactively. Prepare for: bias for action, ownership, delivering results, or adapting to change.

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
