# Day 15 — Linked Lists: Partition & Rotation
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Apply dummy-head and pointer manipulation to partition and rotate problems — classic FAANG screening questions.

## DSA (2 hours)
### Pattern: Linked List Partition & Rotation
- Partition: maintain two dummy heads (one for each partition); append nodes to the correct list as you scan; join the two lists at the end.
- Rotation: find the new tail (length − k%length − 1 steps from head), the new head (new_tail.next), then stitch the old tail back to the original head.
- Trigger condition: "rearrange nodes by a condition" (partition) OR "rotate right by k" — both require finding exact positions without random access.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove Duplicates from Sorted List | 83 | Easy | Single pointer scan | Keep one node per value; `curr.next = curr.next.next` when duplicate; unlike LC 82, keep one copy |
| 2 | Partition List | 86 | Medium | Two dummy heads | Scan once: append to `less` list if val < x, else to `greater` list; join at end; null-terminate greater |
| 3 | Rotate List | 61 | Medium | Find new tail + stitch | Compute length; effective rotation = k % length; new tail is at position length − k − 1 |

### Code Skeleton
```python
# Partition List (LC 86)
def partition(head, x):
    less_dummy = ListNode(0)
    greater_dummy = ListNode(0)
    less, greater = less_dummy, greater_dummy
    curr = head
    while curr:
        if curr.val < x:
            less.next = curr
            less = less.next
        else:
            greater.next = curr
            greater = greater.next
        curr = curr.next
    greater.next = None        # important: cut off old tail
    less.next = greater_dummy.next
    return less_dummy.next

# Rotate List (LC 61) — skeleton
def rotate_right(head, k):
    if not head or not head.next or k == 0:
        return head
    # 1. compute length, make list circular (tail.next = head)
    # 2. effective = k % length
    # 3. advance (length - effective) steps to find new tail
    # 4. new_head = new_tail.next; new_tail.next = None
    pass
```

## System Design (1 hour)
### Topic: Load Balancer Health Checks & Failover
- **Active health checks:** LB sends periodic pings (HTTP GET `/health`) to each backend; marks server unhealthy after N consecutive failures.
- **Passive health checks:** LB observes real traffic; if error rate from a server exceeds a threshold, pulls it from rotation.
- **Failover:** when a server is marked unhealthy, LB redistributes its connections to remaining servers; stateless services recover seamlessly.
- **Circuit breaker pattern:** a client-side version of health checks — if downstream calls fail N times, stop sending requests for a cooldown period (prevents cascading failures).
- Interview talking point: "If asked how a load balancer detects and handles a crashed backend server, answer: active health check sends HTTP GET every 5s to `/health`; after 2 consecutive 5xx/timeouts the server is removed from rotation; when health checks recover it is re-added — zero human intervention."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you divided a group into two categories based on a criterion and processed each category separately — analogous to the partition list problem splitting nodes into two chains.
- Leadership principle: Ownership

## Flashcards

| Q | A |
|---|---|
| How do you null-terminate the greater partition in LC 86, and why is it critical? | `greater.next = None` after the scan loop — the last node in the greater list may still point to an earlier node in the original list, creating a cycle |
| What is the formula for the effective rotation amount in Rotate List? | `effective = k % length`; if k is a multiple of length, the list is unchanged |
| How do you find the new tail in Rotate List? | Advance `length − effective − 1` steps from the head; the node at that position is the new tail |
| What is the difference between active and passive load balancer health checks? | Active: LB probes each backend on a schedule. Passive: LB observes real traffic errors — no extra requests but slower to detect failures |
| What problem does the circuit breaker pattern solve that a load balancer health check does not? | Cascading failures — without a circuit breaker, a caller keeps hammering a slow/failing downstream service, exhausting its own thread pool; the circuit breaker short-circuits after N failures |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
