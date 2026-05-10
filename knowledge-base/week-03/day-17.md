# Day 17 — Linked Lists: Delete, Split & Arithmetic II
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Handle node deletion, list splitting, and reverse-order arithmetic — less common but reliably tested linked list problems.

## DSA (2 hours)
### Pattern: Delete Without Head Reference + List Splitting
- Delete node without head (LC 237): copy next node's value into the current node, then skip the next node — works only when the node to delete is not the tail.
- Split in Parts: compute base size and remainder; give the first `remainder` parts one extra node; advance to the end of each part, cut, record head.
- Trigger condition: "delete a node given only that node" OR "divide a list into k equal-or-near-equal parts."
- Time complexity: O(n) | Space complexity: O(k) for split output

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove Linked List Elements | 203 | Easy | Dummy head + scan | Standard deletion scan; dummy prevents head-removal edge case |
| 2 | Add Two Numbers II | 445 | Medium | Stack (or reverse) | Digits are stored most-significant first; push both lists onto stacks; pop and add with carry |
| 3 | Split Linked List in Parts | 725 | Medium | Length + remainder distribution | `base, extra = divmod(length, k)`; first `extra` parts get `base+1` nodes; remaining get `base` |

### Code Skeleton
```python
# Add Two Numbers II (LC 445) — stack approach
def add_two_numbers_ii(l1, l2):
    s1, s2 = [], []
    while l1: s1.append(l1.val); l1 = l1.next
    while l2: s2.append(l2.val); l2 = l2.next
    carry = 0
    dummy = ListNode(0)
    while s1 or s2 or carry:
        val = carry
        if s1: val += s1.pop()
        if s2: val += s2.pop()
        carry, val = divmod(val, 10)
        node = ListNode(val)
        node.next = dummy.next
        dummy.next = node       # prepend to build result in correct order
    return dummy.next

# Split in Parts (LC 725) — skeleton
def split_list_to_parts(head, k):
    # 1. compute length
    # 2. base, extra = divmod(length, k)
    # 3. for each part: take base nodes (+ 1 if extra > 0); cut; store head
    pass
```

## System Design (1 hour)
### Topic: Scaling Bottlenecks — Beyond the DB
- **Application tier:** CPU-bound tasks (image processing, ML inference) bottleneck here; solution is horizontal scaling + queue-based async processing.
- **Message queue as buffer:** decouple producers and consumers; queue absorbs traffic spikes; workers drain at their own rate (e.g., SQS, RabbitMQ).
- **Single points of failure:** a single load balancer is itself a SPOF; production systems use an active-active or active-passive pair (e.g., AWS ELB is managed and multi-AZ by default).
- **N+1 redundancy:** always run at least N+1 instances of any critical component so one failure doesn't reduce capacity below the required threshold.
- Interview talking point: "If asked how to handle a sudden 10x traffic spike without pre-scaling, answer: put a message queue in front of the processing workers; incoming requests enqueue jobs immediately (fast); workers drain the queue at their max capacity — the queue absorbs the burst, trading latency for availability."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you distributed work unevenly across a team because some members had more capacity — analogous to the base+extra distribution in Split Linked List in Parts.
- Leadership principle: Hire and Develop the Best

## Flashcards

| Q | A |
|---|---|
| How do you delete a linked list node when only that node is given (not the head)? | Copy `node.next.val` into `node.val`, then set `node.next = node.next.next` — effectively delete the next node while preserving the current pointer |
| Why doesn't the "copy-next-and-skip" deletion work on the tail node? | The tail has no next node to copy from; you would need a reference to the previous node to remove the tail |
| How do you add two numbers where digits are stored most-significant first? | Push both lists onto stacks (O(n) space); pop digits and add with carry; prepend result nodes to build the answer in correct order |
| How does a message queue decouple producers and consumers during a traffic spike? | Producers enqueue jobs at any rate; consumers drain at their capacity; the queue absorbs the difference — trading increased latency for sustained throughput |
| What is N+1 redundancy and why is it the minimum production standard? | Running N+1 instances so any single failure leaves N instances — enough to meet required capacity without service degradation |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
