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
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Partition List (LC 86)
    public static ListNode partition(ListNode head, int x) {
        ListNode lessDummy = new ListNode(0);
        ListNode greaterDummy = new ListNode(0);
        ListNode less = lessDummy, greater = greaterDummy;
        ListNode curr = head;
        while (curr != null) {
            if (curr.val < x) {
                less.next = curr;
                less = less.next;
            } else {
                greater.next = curr;
                greater = greater.next;
            }
            curr = curr.next;
        }
        greater.next = null;        // important: cut off old tail
        less.next = greaterDummy.next;
        return lessDummy.next;
    }

    // Rotate List (LC 61) — skeleton
    public static ListNode rotateRight(ListNode head, int k) {
        if (head == null || head.next == null || k == 0) {
            return head;
        }
        // 1. compute length, make list circular (tail.next = head)
        // 2. effective = k % length
        // 3. advance (length - effective) steps to find new tail
        // 4. new_head = new_tail.next; new_tail.next = null
        return head; // TODO: implement
    }
}
```

### Interview Tips

- **Partition List — state both dummy heads upfront:** "I use two dummy heads — `lessDummy` and `greaterDummy` — and build two separate chains simultaneously. At the end, I join them: `less.next = greaterDummy.next`. The critical step is `greater.next = null` to cut off the old tail, which may still point back into the original list."
- **Rotation — compute effective rotation first:** "`k % length` handles the case where k is a multiple of length (no rotation needed) and when k > length (equivalent to k % length rotations). Failing to take the modulo leads to unnecessary traversal."
- **Brute force for rotation:** rotating one step at a time would be O(n×k) — for k = 10⁹ and n = 500, that's 5×10¹¹ operations. The stitch-and-reconnect approach is O(n) regardless of k.
- **Common mistake for Partition List:** forgetting `greater.next = null` — the last node in the greater list still has its original `.next` pointer, which may point back into the less list or create a cycle.

### STAR Interview Framework

> **How to use the STAR method when explaining Linked List Partition & Rotation in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list and a value x, and asked to partition it so all nodes with value < x come before nodes with value ≥ x, preserving relative order within each partition. A naive approach — collect all nodes, sort into two lists, re-link — is O(n) but creates a new list. The two-dummy-head pattern achieves O(n) time and O(1) extra space in-place."

**Task:** "My goal was to partition in a single scan by simultaneously building two chains (less-than and greater-or-equal) using dummy heads, then joining them at the end — no sorting, no extra storage beyond two dummy nodes."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Rearrange nodes by a condition without extra storage → two-dummy-head partition. Trigger: 'separate elements into groups while preserving relative order.'"
2. *Initialize:* "I create `lessDummy` and `greaterDummy` as sentinel nodes. I maintain `less = lessDummy` and `greater = greaterDummy` as the tails of their respective chains."
3. *Core loop logic:* "For each `curr` node: if `curr.val < x`, attach to the less chain (`less.next = curr; less = less.next`); otherwise attach to the greater chain. At the end, `greater.next = null` (critical — cuts off the old tail), then `less.next = greaterDummy.next`."
4. *Convergence guarantee:* "Each node is visited exactly once and attached to exactly one chain — O(n) single pass, O(1) space (two dummy nodes only)."
5. *Duplicate handling / edge case proactivity:* "`greater.next = null` is the most commonly forgotten step. Without it, the last node in the greater chain still points to wherever it pointed in the original list — creating a cycle or linking back into the less chain."

**Result:** "O(n) time, O(1) extra space in a single pass. The two-dummy-head pattern eliminates all head-insertion edge cases and is the canonical approach for any 'partition a list into k groups' problem."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Two Dummy Heads here |
|-------------|---------------------------|-------------------------------|
| Collect into two arrays, rebuild | When list structure can be discarded | O(n) extra space; two dummy heads achieves the same in O(1) extra space |
| Sort the list | When partition order doesn't need to be stable | O(n log n) time; partition is O(n); also partition preserves relative order which sort doesn't |

**Why NOT array collection:** O(n) extra space — two dummy heads is strictly better.
**Why NOT sorting:** Sorting is O(n log n) and doesn't preserve relative order within each partition, which is a requirement.

### Edge Cases to Trace Before Coding
- LC 83: all elements same → `curr.next` always equals `curr`; `slow` skips every duplicate; one node remains ✓
- LC 86: all nodes < x → entire list attaches to less chain; greaterDummy.next = null; result is original list ✓
- LC 61: k = 0 or k = length → effective rotation = 0; return head unchanged
- LC 61: single node → rotation is a no-op; return head immediately

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

**Leadership Principle:** Ownership

**STAR Story: Partitioning a Support Queue into Two Priority Chains for Faster Resolution**

**Situation (20%):** "At my previous company, our customer support queue was a single FIFO queue of all incoming tickets — high-severity incidents (service outages) and low-severity requests (billing questions) competed for the same agents. During high-traffic periods, critical outage tickets could sit unresolved for 45+ minutes behind a queue of billing inquiries, causing SLA violations and customer escalations."

**Task (part of S/T):** "I was the support engineering lead. My goal was to redesign the queue routing so critical tickets always reached senior on-call engineers within 5 minutes, without increasing headcount or reducing throughput for low-severity tickets."

**Action (60-70% — be specific about what YOU did):**
"First, I designed a two-queue partition: tickets were classified as P0 (critical) or P1/P2 (standard) by an automated triage rule I wrote — analogous to the partition list's `curr.val < x` condition. Each ticket attached to exactly one queue, preserving arrival order within each priority tier.
Then, I implemented priority-aware routing: on-call senior engineers pulled from the P0 queue first and only dipped into the P1/P2 queue when P0 was empty. Standard agents only worked the P1/P2 queue.
Next, I ran a 2-week shadow test where I logged what the new routing would have done against the old routing, confirmed P0 ticket response times would have dropped without degrading P1/P2 throughput.
Finally, I deployed the new routing with a rollback switch and monitored for 3 days before removing the old single-queue fallback."

**Result (10-20%):** "P0 ticket mean time to first response dropped from 43 minutes to 4 minutes — an 91% improvement. P1/P2 mean response time was unchanged at 18 minutes. SLA violation rate for critical tickets dropped from 34% to 2%. The partition logic was adopted by two other support tiers within the quarter."

**Interview tip:** Ownership means you spotted the problem, proposed the solution, built it, and monitored it — without waiting to be asked. The partition analogy is direct and makes the technical decision concrete. Prepare for: ownership, customer obsession, bias for action, or delivering results.

## Flashcards

| Q | A |
|---|---|
| How do you null-terminate the greater partition in LC 86, and why is it critical? | `greater.next = null` after the scan loop — the last node in the greater list may still point to an earlier node in the original list, creating a cycle |
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
