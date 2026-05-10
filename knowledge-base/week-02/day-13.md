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
```python
# Merge two sorted lists (LC 21)
def merge_two_lists(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next, l1 = l1, l1.next
        else:
            curr.next, l2 = l2, l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next

# Sort list skeleton (LC 148)
def sort_list(head):
    if not head or not head.next:
        return head
    # 1. find midpoint, cut list in half
    # 2. sort_list(left_half), sort_list(right_half)
    # 3. merge results
    pass
```

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
- STAR prompt: Describe a time you had to combine outputs from multiple independent workstreams into a single coherent result — analogous to merging sorted lists from separate sorted sources.
- Leadership principle: Deliver Results

## Flashcards

| Q | A |
|---|---|
| How do you cut a linked list in half for merge sort? | Find mid with fast/slow pointers; save `mid.next` as the right half head, then set `mid.next = None` to sever the list |
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
