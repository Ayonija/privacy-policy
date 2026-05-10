# Day 20 — Linked Lists: Slot 2 Synthesis
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Slot 2 close-out: drill the three hardest linked list patterns from scratch, and consolidate the full scaling + load balancer system design unit.

## DSA (2 hours)
### Pattern: Review — Merge Between, Delete Node, Copy with Random
- Merge In Between (LC 1669): find nodes at positions a−1 and b+1; splice in a second list; reconnect.
- All three problems today require *precise pointer manipulation* — re-solve each from memory without looking at previous code; note where you hesitate.
- Trigger condition: "splice a sublist into another" → find boundary nodes + stitch; this generalises the partition and rotation patterns from earlier this week.
- Time complexity: O(n + m) for merge-in-between | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Delete Node in a Linked List | 237 | Easy | Copy-next + skip | Copy next node's val into current; `node.next = node.next.next` |
| 2 | Merge In Between Linked Lists | 1669 | Medium | Find boundaries + splice | Walk to node a−1 (call it `left`) and node b+1 (call it `right`); `left.next = list2_head`; walk to list2 tail; `tail.next = right` |
| 3 | Swapping Nodes in a Linked List | 1721 | Medium | Fast/Slow + value swap or pointer swap | Find kth from front and kth from end using gap pointer; swap values (easier) or swap nodes (harder, O(1)) |

### Code Skeleton
```python
# Merge In Between (LC 1669)
def merge_in_between(list1, a, b, list2):
    # find node at position a-1
    left = list1
    for _ in range(a - 1):
        left = left.next
    # find node at position b+1
    right = left
    for _ in range(b - a + 2):
        right = right.next
    # splice list2 in
    left.next = list2
    # walk to list2 tail
    tail = list2
    while tail.next:
        tail = tail.next
    tail.next = right
    return list1

# Swapping Nodes (LC 1721) — value swap skeleton
def swap_nodes(head, k):
    # 1. find kth node from front (call it front)
    # 2. use gap pointer to find kth from end (call it back)
    # 3. swap front.val, back.val
    pass
```

## System Design (1 hour)
### Topic: Scaling & Load Balancers — Slot 2 Synthesis
- Consolidate the full scaling picture: single server → vertical scale-up → stateless + horizontal scale-out → LB + auto-scaling → DB read replicas → sharding + message queues.
- Common interview failure mode: jumping straight to sharding without first exhausting simpler steps (add caching, add replicas, tune queries).
- Load balancer decision tree: L4 for TCP/UDP low-latency (gaming, VoIP); L7 for HTTP services needing path routing, auth headers, or canary deploys.
- Auto-scaling best practice: scale out aggressively (add fast), scale in conservatively (remove slowly) — cost of an extra server is lower than cost of dropped requests.
- Interview talking point: "If asked to describe the complete journey from a single server to a globally distributed system, answer: 1→ vertical scale; 2→ move state to DB; 3→ add LB + horizontal app servers; 4→ add CDN for static assets; 5→ DB read replicas; 6→ cache layer (Redis); 7→ DB sharding or migration to distributed DB; 8→ multi-region with geo-routing."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Walk me through a time you incrementally scaled a solution — starting with the simplest fix and only adding complexity when the simpler approach proved insufficient.
- Leadership principle: Frugality

## Flashcards

| Q | A |
|---|---|
| How do you find the node at position b+1 starting from node a−1 in LC 1669? | From the node at position a−1, advance `b − a + 2` more steps |
| How do you find the kth node from the end without knowing list length? | Use two pointers with a gap of k; advance fast k steps, then advance both until fast reaches None — slow is at the kth from end |
| What is the recommended order of scaling steps before resorting to DB sharding? | Add caching (Redis) → add read replicas → tune slow queries → evaluate connection pooling → only then consider sharding |
| Why should auto-scaling scale in conservatively? | Scaling in too aggressively can cause oscillation (add/remove repeatedly); a brief traffic dip shouldn't trigger deprovisioning that forces immediate re-provisioning |
| What is the one-sentence summary of when to use L4 vs. L7 load balancing? | L4 for pure TCP/UDP throughput with minimal latency overhead; L7 for HTTP services needing content-aware routing, header inspection, or deployment strategies |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
