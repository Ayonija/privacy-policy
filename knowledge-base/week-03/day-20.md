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

### Interview Tips

- **Merge In Between — find both boundary nodes before stitching:** "I walk to node `a-1` and keep walking to node `b+1` in a single traversal. The formula: from node `a-1`, advance `b - a + 2` more steps to reach `b+1`. Missing the formula and recounting from the head is O(n) extra work."
- **Swapping Nodes — clarify value swap vs. node swap:** "Value swap is O(1) code complexity and sufficient for most interviews; node swap requires O(1) additional pointer work but is O(n) code complexity. Ask the interviewer which is expected, then default to value swap unless told otherwise."
- **Review day strategy:** re-solve each problem from scratch in 15 minutes without looking at previous solutions. If you can't, add it to `revision-log.md` and schedule a retry in 3 days.
- **Common mistake in Merge In Between:** forgetting to walk to the tail of `list2` before stitching `tail.next = right`. If you stitch `list2_head.next = right`, you lose all of list2 after the head.

### STAR Interview Framework

> **How to use the STAR method when explaining Merge In Between + Pointer Manipulation in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given two linked lists and asked to remove nodes at positions a through b in list1 and replace them with list2. A naive approach collects values from both lists into arrays, splices, and rebuilds — O(n + m) time but O(n + m) space. The in-place approach achieves the same time with O(1) extra space."

**Task:** "My goal was to splice list2 into list1 by finding exactly two boundary nodes in list1 — the node at position a-1 and the node at position b+1 — then stitching list2 between them."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Splice a sublist into another → find boundary nodes + stitch. Trigger: 'replace a range in a list with another list.'"
2. *Initialize:* "I walk `left = list1` forward `a-1` steps. Then from `left`, I walk `right` forward `b - a + 2` more steps to reach position `b+1`. The formula `b - a + 2` = `(b - (a-1)) + 1` — distance from `a-1` to `b+1`."
3. *Core loop logic:* "Stitch: `left.next = list2`. Walk list2 to its tail: `while tail.next: tail = tail.next`. Then `tail.next = right`."
4. *Convergence guarantee:* "Single traversal of list1 to find boundaries: O(b) ≤ O(n). Single traversal of list2 to find its tail: O(m). Total: O(n + m)."
5. *Duplicate handling / edge case proactivity:* "Must walk to the TAIL of list2 before stitching — not just `list2.next = right`. If you stitch `list2.next = right` on list2's head, you lose the rest of list2. This is the most common mistake."

**Result:** "O(n + m) time, O(1) extra space. The boundary formula `b - a + 2` steps from position `a-1` eliminates the need for a second traversal from the head. State this formula before writing any code."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer In-Place Splice here |
|-------------|---------------------------|-------------------------------|
| Collect all values to array, splice, rebuild | When code clarity matters more than space | O(n + m) extra space; in-place is O(1) extra space |
| Two-pass: first count positions, then walk | When index arithmetic is error-prone | Single pass with `b - a + 2` formula is O(n) vs O(2n) for two passes |

**Why NOT array:** O(n + m) extra space — the in-place splice uses O(1).
**Why NOT two passes:** O(2n) — single-pass boundary computation with the formula is simpler and faster.

### Edge Cases to Trace Before Coding
- LC 237 (Delete Node): tail node → the copy-next trick cannot work on the tail; problem guarantees it won't be the tail
- LC 1669 (Merge In Between): `a = 0` → left is list1 head itself; right is at position `b+1` from head
- LC 1721 (Swap Nodes): `k` is in the middle → kth from front and kth from end are different nodes; ensure both are found before swapping
- LC 1721: `k = 1` → swap head and tail

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

**Leadership Principle:** Frugality

**STAR Story: Scaling a Search Service Incrementally Without Over-Engineering**

**Situation (20%):** "At my previous company, our product search service was handling 200 QPS on a single PostgreSQL database with a LIKE query. As the catalog grew to 2 million products and search volume grew to 2,000 QPS, P95 latency climbed to 8 seconds and we started seeing timeouts. The initial engineering proposal was to immediately migrate to Elasticsearch — a 3-month project."

**Task (part of S/T):** "I was responsible for search infrastructure. My goal was to restore P95 latency to under 500ms as quickly as possible, spending the minimum in engineering effort and infrastructure cost, and only escalating to Elasticsearch if simpler solutions proved insufficient."

**Action (60-70% — be specific about what YOU did):**
"First, I added a GIN trigram index to the `products.name` column — a 2-hour change. Latency dropped from 8 seconds to 1.2 seconds at 2,000 QPS. That was a significant improvement but not enough.
Then, I added a Redis cache for the top 500 search queries (covering 80% of search volume). Cache hit rate reached 78% within 24 hours. P95 latency for cached queries fell to 12ms; overall P95 dropped to 350ms.
Next, I monitored for 2 weeks. At 3,500 QPS — 2 months later — latency began climbing again. Only then did I begin the Elasticsearch migration, with a much clearer requirements picture than we would have had if we'd migrated at 2,000 QPS.
Finally, I ran both search backends in parallel for 3 weeks during the Elasticsearch migration, with traffic gradually shifting from PostgreSQL to Elasticsearch."

**Result (10-20%):** "The trigram index and Redis cache cost 1 week of engineering time and $200/month in Redis infrastructure. The Elasticsearch migration, when it happened, cost 8 weeks rather than the originally estimated 3 months — because we had 2 months of production usage data to inform the index configuration. Total savings: ~4 months of premature engineering effort and the clarity that came from incremental validation."

**Interview tip:** Frugality means "I solved the problem with the minimum necessary complexity and cost." The key is showing you knew WHEN each level of complexity was justified — not that you always chose the cheapest path. Prepare for: frugality, invent and simplify, bias for action, or pragmatic decision-making.

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
