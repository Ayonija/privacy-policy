# Day 13 — Linked Lists: Merge & Sort
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Merge and sort linked lists — patterns that directly mirror merge sort and appear in both DSA and systems (merge of sorted streams).

## DSA (2 hours)
### Pattern: Merge Two Sorted Lists + Merge Sort on a List
- Use a dummy head to avoid special-casing the first node; compare heads, attach the smaller, advance that pointer.
- Sort List (LC 148) applies merge sort: find midpoint with fast/slow, split, recurse, merge — achieves O(n log n) with O(log n) stack space.
- Trigger condition: "merge two or more sorted sequences" OR "sort a linked list in O(n log n)" (ruling out O(n²) bubble sort on lists).
- Time complexity: O(n) for merge, O(n log n) for sort | Space complexity: O(1) for iterative merge, O(log n) for recursive sort

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Merge Two Sorted Lists | 21 | Easy | Dummy head + merge | Compare heads, attach smaller; when one list exhausts, attach the rest of the other |
| 2 | Add Two Numbers | 2 | Medium | Dummy head + carry | Simulate column addition: sum = l1.val + l2.val + carry; handle unequal lengths and final carry |
| 3 | Sort List | 148 | Medium | Merge Sort on list | Fast/slow to find mid; cut at mid; recurse on both halves; merge results |

### Code Skeleton
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Merge two sorted lists (LC 21)
    public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                curr.next = l1;
                l1 = l1.next;
            } else {
                curr.next = l2;
                l2 = l2.next;
            }
            curr = curr.next;
        }
        curr.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }

    // Sort list (LC 148) — merge sort approach
    public static ListNode sortList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        // Step 1: find midpoint and sever list
        ListNode slow = head, fast = head.next;   // fast starts at head.next to bias mid left
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode second = slow.next;
        slow.next = null;   // sever: left half is head..slow, right half is second..
        // Step 2 + 3: sort halves and merge
        return mergeTwoLists(sortList(head), sortList(second));
    }
}
```

### Interview Tips

- **`curr.next = (l1 != null) ? l1 : l2` — explain it:** "This attaches whichever list still has remaining nodes in one line — if l1 is non-null we attach it, otherwise we attach l2." Interviewers appreciate concise, intentional idioms over verbose if-else.
- **Why dummy head is always safe:** without dummy, you'd need special-casing to set the initial result head before the loop. With dummy, the first node attachment is identical to all subsequent ones — zero edge cases.
- **Sort List — announce the approach first:** "Merge sort: (1) find middle with fast/slow, (2) sever at mid, (3) recursively sort both halves, (4) merge. Time O(n log n), space O(log n) for call stack — optimal for a linked list since we can't random-access for quicksort."
- **Carry in Add Two Numbers — most common bug:** after the main while loop, if `carry == 1` append one final node. Trace `999 + 1` to verify.
- **Common mistake:** in Sort List, initialising `fast = head` instead of `fast = head.next` — with `fast = head` on a 2-node list, `fast.next` is non-null so the loop runs an extra step and `slow` ends up past the midpoint.

### STAR Interview Framework

> **How to use the STAR method when explaining Merge Two Sorted Lists & Merge Sort on a Linked List in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to sort a linked list in O(n log n) time and O(1) extra space (excluding recursion stack). Quicksort on a linked list requires O(n²) worst-case without random access for pivot selection. Merge sort is the optimal choice — O(n log n) guaranteed and naturally suited to linked lists because splitting and merging don't require random access."

**Task:** "My goal was to implement merge sort on a linked list: find the midpoint with fast/slow pointers, recursively sort both halves, then merge in a single pass using the dummy-head merge pattern."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Sort a linked list without random access → merge sort. Two key sub-problems: (1) find midpoint — fast/slow pointer; (2) merge two sorted lists — dummy head + compare-and-attach."
2. *Initialize:* "Base case: if `head == null || head.next == null`, return head. For the midpoint, I initialize `fast = head.next` (not `head`) so that fast is one step ahead — ensuring `slow` lands at the LEFT middle node for even-length lists."
3. *Core loop logic:* "Advance `slow` one step, `fast` two steps until `fast == null || fast.next == null`. Then `second = slow.next; slow.next = null` — this severs the list into two halves. Recurse on each half, then merge."
4. *Convergence guarantee:* "At each level, we split the list in half — log n levels. Each merge at a level does O(n) work. Total: O(n log n). Stack depth is O(log n)."
5. *Duplicate handling / edge case proactivity:* "Initialising `fast = head.next` (not `fast = head`) is the critical detail. On a 2-node list with `fast = head`, the while condition `fast != null && fast.next != null` is true, so the loop runs and `slow` advances past the midpoint — corrupting the split."

**Result:** "Merge sort on a linked list is O(n log n) time and O(log n) space (call stack). For n = 10⁵, this is ~1.7M operations vs O(n²) bubble sort which would be 10¹⁰. The merge step is O(n) and uses the dummy-head trick to eliminate all head-insertion edge cases."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Merge Sort here |
|-------------|---------------------------|-------------------------------|
| Insertion sort O(n²) | When list is nearly sorted (O(n k) for k inversions) | O(n²) worst case vs O(n log n) guaranteed; only better for nearly-sorted input |
| Convert to array, sort, rebuild O(n) space | When space is not constrained | O(n) extra space vs O(log n) for merge sort; problem often requires in-place |

**Why NOT insertion sort:** O(n²) worst case — for a reverse-sorted 10⁵-element list, that's 10¹⁰ operations vs 1.7M for merge sort.
**Why NOT array conversion:** O(n) extra space for the array; merge sort uses only O(log n) stack space. Also, re-linking nodes from an array requires O(n) additional work.

### Edge Cases to Trace Before Coding
- LC 21: one or both inputs are empty → `(l1 != null) ? l1 : l2` handles correctly; `while l1 != null && l2 != null` skips immediately ✓
- LC 2: both lists represent 0 → result is `[0]`; carry stays 0 throughout ✓
- LC 2: `[9,9,9] + [9,9,9,9]` → most digits plus carry; ensure the longer list is still processed after shorter runs out
- LC 148: single node → base case returns head immediately
- LC 148: even vs odd length — trace `[4,2,1,3]` vs `[4,2,1,3,5]` to confirm mid is correct

## System Design (1 hour)
### Topic: Load Balancers — Types & Algorithms
- **Round Robin:** distribute requests sequentially across servers; simple, assumes equal server capacity.
- **Least Connections:** route to the server with fewest active connections; better for variable request durations.
- **IP Hash:** hash client IP to determine server; provides natural session affinity without sticky session config.
- **Weighted Round Robin:** assign weights proportional to server capacity; used when servers are heterogeneous.
- Trade-offs: Round Robin is O(1) and stateless; Least Connections requires tracking active connections (small overhead but better load distribution); IP Hash breaks balance when a few clients generate most traffic.
- Interview talking point: "If asked which load balancing algorithm suits a video streaming service, answer: Least Connections — streaming requests hold connections for minutes; round robin would pile many streams on one server while others idle."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Deliver Results

**STAR Story: Merging Outputs from Three Independent Data Teams into a Single Coherent Report**

**Situation (20%):** "At my previous company, we needed to deliver a monthly executive dashboard that combined outputs from three independent data teams: the product analytics team, the finance team, and the customer success team. Each team delivered their segment in a different format, on different schedules, with different column naming conventions. Every month, someone spent 2–3 days manually reconciling the three outputs into one coherent report — a process that was error-prone and delayed the dashboard by up to a week."

**Task (part of S/T):** "I was asked to own the dashboard automation project. My goal was to automate the merge process so the monthly dashboard published within 4 hours of the last team's data delivery, with zero manual reconciliation."

**Action (60-70% — be specific about what YOU did):**
"First, I met with each team to document their output schema, delivery time, and update frequency — analogous to understanding each sorted list's structure before merging.
Then, I designed a merge pipeline that treated each team's output as a sorted input stream: each dataset was normalized to a canonical schema, then joined on a shared entity ID. I chose a merge-join rather than a hash join because all three datasets were already sorted by entity ID after normalization — O(n) merge vs O(n log n) sort-then-hash.
Next, I implemented the pipeline in dbt with automated schema validation, so if any team changed their column names the pipeline would fail loudly rather than silently produce wrong data.
Finally, I scheduled the pipeline to trigger automatically when all three sources were available, with a Slack notification to stakeholders when the dashboard published."

**Result (10-20%):** "The monthly dashboard now publishes within 2 hours of the last data delivery, down from 2–3 days. Manual reconciliation errors — which had caused 2 dashboard recalls in the prior year — dropped to zero. The 3 data teams reduced their coordination overhead by 80% because the canonical schema contract handled alignment automatically."

**Interview tip:** Deliver Results answers should connect your technical solution to a business metric. "Dashboard publish time: 3 days → 2 hours" and "reconciliation errors: 2/year → 0" are the numbers that make the story credible. Prepare for: delivering results, ownership, simplification, or cross-team collaboration.

## Flashcards

| Q | A |
|---|---|
| How do you cut a linked list in half for merge sort? | Find mid with fast/slow pointers; save `mid.next` as the right half head, then set `mid.next = null` to sever the list |
| Why does Add Two Numbers need a dummy head? | The first result node is computed inside the loop — dummy lets you attach it with `curr.next` without special-casing the head |
| What is the space complexity of recursive merge sort on a linked list? | O(log n) for the recursive call stack; each level halves the list, so there are log n levels |
| When does Round Robin load balancing produce imbalance? | When servers have different capacities or when request durations vary significantly — a slow request on one server ties it up while round robin still routes to it |
| How does IP Hash load balancing provide session affinity? | `server_index = hash(client_ip) % num_servers`; same client IP always maps to the same server as long as server count doesn't change |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
