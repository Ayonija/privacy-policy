# Day 18 — Linked Lists: Advanced Patterns
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Tackle the two hardest linked list patterns — k-group reversal and deep copy — that appear at senior-level interviews.

## DSA (2 hours)
### Pattern: k-Group Reversal & Deep Copy with Random Pointer
- k-Group: count k nodes ahead to confirm enough remain; reverse that group; the group's original head becomes the new tail connecting to the recursive result.
- Deep copy with random: weave cloned nodes between originals (`A → A' → B → B'`); copy random pointers using `node.next.random = node.random.next`; then separate the two lists.
- Trigger condition: k-group is a direct interview ask; deep copy appears whenever a graph/list structure has non-trivial pointer topology that prevents simple traversal cloning.
- Time complexity: O(n) for both | Space complexity: O(1) for k-group; O(n) for deep copy (the cloned nodes themselves)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Convert Binary Number in a Linked List to Integer | 1290 | Easy | Single pass | Maintain running value: `val = val * 2 + node.val` as you traverse |
| 2 | Reverse Nodes in k-Group | 25 | Hard | Group reversal + recursion | Count k ahead; if fewer than k remain, stop; reverse k nodes; tail.next = recurse(next_group_head) |
| 3 | Copy List with Random Pointer | 138 | Medium | Interleave + separate | Three passes: interleave clones, copy random pointers, separate the two lists |

### Code Skeleton
```python
# Copy List with Random Pointer (LC 138) — O(1) space approach
def copy_random_list(head):
    if not head: return None
    # Pass 1: weave — insert clone after each original
    curr = head
    while curr:
        clone = Node(curr.val, curr.next, None)
        curr.next = clone
        curr = clone.next
    # Pass 2: copy random pointers
    curr = head
    while curr:
        if curr.random:
            curr.next.random = curr.random.next
        curr = curr.next.next
    # Pass 3: separate
    dummy = Node(0)
    clone_curr = dummy
    curr = head
    while curr:
        clone_curr.next = curr.next
        curr.next = curr.next.next
        clone_curr = clone_curr.next
        curr = curr.next
    return dummy.next

# Reverse Nodes in k-Group (LC 25) — skeleton
def reverse_k_group(head, k):
    # 1. check if k nodes remain; if not, return head
    # 2. reverse k nodes (reuse reversal template)
    # 3. connect: original_head.next = reverse_k_group(next_group, k)
    pass
```

### Interview Tips

- **k-Group — count k nodes ahead before reversing:** "I count k nodes ahead first. If fewer than k remain, return head unchanged. This is the base case — interviewers often miss it." State this check before touching the reversal logic.
- **Deep copy — state all three passes before coding:** "Pass 1: interleave clones (A → A' → B → B'). Pass 2: copy random pointers using `curr.next.random = curr.random.next`. Pass 3: separate the two lists." Naming the three passes demonstrates structured thinking.
- **Brute force for deep copy:** HashMap(original_node → clone_node) — O(n) time and O(n) space (the map). The interleaving approach is also O(n) space (the cloned nodes themselves) but avoids the explicit HashMap.
- **Common mistake in k-Group:** connecting the recursive result incorrectly. After reversing k nodes, `original_head` becomes the tail — `original_head.next = reverse_k_group(next_group, k)`.

### STAR Interview Framework

> **How to use the STAR method when explaining k-Group Reversal & Deep Copy with Random Pointer in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to copy a linked list where each node has a `next` pointer and a `random` pointer that can point to any node in the list or null. A HashMap approach maps each original node to its clone, then sets random pointers using the map — O(n) time and O(n) space (for the map). The interleaving approach achieves O(n) time and O(1) extra space (beyond the cloned nodes themselves)."

**Task:** "My goal was to deep-copy the list without a HashMap, using the interleaving technique where each clone is inserted immediately after its original — making `node.next` the clone of `node` during the copy phase."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Non-trivial pointer topology that prevents simple traversal → interleave-and-separate. Trigger: 'clone a structure with cross-links.'"
2. *Initialize:* "I describe three passes before touching any code: (1) weave clones between originals, (2) copy random pointers, (3) separate. This is the entire algorithm."
3. *Core loop logic for Pass 2:* "`curr.next.random = curr.random.next` — the clone of `curr` is `curr.next`; the clone of `curr.random` is `curr.random.next`. This is O(1) per node and requires no HashMap."
4. *Convergence guarantee:* "All three passes are O(n) single-scan loops. Total: O(n) with O(1) extra space beyond the n cloned nodes."
5. *Duplicate handling / edge case proactivity:* "In Pass 2, `if curr.random:` guard is required — if `curr.random == null`, `curr.next.random` should remain null. Without the guard, `null.next` throws a NullPointerException."

**Result:** "O(n) time, O(1) extra space vs O(n) space for a HashMap. The key insight: while interleaved, `node.next` IS the clone — O(1) lookup without any mapping data structure. This is the same philosophy as a hash function: embed the mapping in the structure itself."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Interleaving here |
|-------------|---------------------------|-------------------------------|
| HashMap(original → clone) O(n) extra space | When code clarity matters more than space | Interleaving is O(1) extra space; HashMap is O(n); both are O(n) time |
| Recursive clone | When the list is short and stack depth is not a concern | Recursive uses O(n) call stack — equivalent to HashMap approach but harder to reason about |

**Why NOT HashMap:** O(n) extra space for the mapping. Interleaving embeds the clone pointer in the structure itself, eliminating the map.
**Why NOT recursive:** O(n) call stack space — same asymptotic cost as HashMap with no advantage.

### Edge Cases to Trace Before Coding
- LC 1290 (Binary to Integer): empty list → return 0; single node → return its value (0 or 1)
- LC 25 (Reverse k-Group): fewer than k nodes remain → return head unchanged (do not reverse partial groups)
- LC 138 (Copy List): all random pointers are null → Pass 2 loop body skips every node; result is correct ✓
- LC 138: single node with random pointing to itself → `curr.next.random = curr.random.next = curr.next` ✓

## System Design (1 hour)
### Topic: Auto-Scaling & Cloud Load Balancing Patterns
- **Horizontal auto-scaling:** cloud provider adds/removes instances based on metrics (CPU %, request latency, queue depth); AWS Auto Scaling Groups + ALB do this natively.
- **Scale-out trigger:** CPU > 70% for 5 minutes → add instance; CPU < 30% for 15 minutes → remove instance (hysteresis prevents thrashing).
- **Warm-up period:** new instances need time to initialise (load caches, JVM warm-up); LB should delay sending full traffic until health check passes.
- **Blue-green deployment:** run two identical environments; switch the load balancer to point to green after verification; instant rollback by switching back.
- Interview talking point: "If asked how to deploy a new version with zero downtime, answer: blue-green — provision a green environment with the new version, run smoke tests, then switch the load balancer to green; if issues arise, switch back to blue in seconds — no rolling restart needed."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Bias for Action

**STAR Story: Blue-Green Deployment of a Critical Database Schema Migration**

**Situation (20%):** "We needed to migrate a core PostgreSQL table's schema — adding a new column and backfilling 50 million rows — in a live production system handling 800 orders per minute. A traditional in-place migration would require locking the table for an estimated 40 minutes, causing a total service outage during peak hours. We had a 2-week deadline from the product team."

**Task (part of S/T):** "I was the tech lead for the migration. My goal was to complete the schema change with zero downtime and a rollback window of under 30 seconds if anything went wrong during cutover."

**Action (60-70% — be specific about what YOU did):**
"First, I designed a blue-green approach for the migration: I created the new table schema in a separate 'green' table, ran the backfill job asynchronously over 3 days without touching the live 'blue' table, and used a dual-write pattern where new writes went to both tables simultaneously.
Then, I verified green table correctness by running a row-by-row checksum comparison over a 10K-row random sample. Checksums matched 100%.
Next, I flipped the application's read pointer from blue to green using a feature flag — a 30-second configuration change with instant rollback capability. I timed this for 3 AM on a Sunday (traffic trough).
Finally, I monitored the system for 48 hours before dropping the blue table, keeping the rollback path open."

**Result (10-20%):** "Zero downtime during the migration — 800 orders per minute continued uninterrupted. The read pointer flip took 22 seconds. No rollback was needed. We kept the blue table available as a fallback for 5 more days before dropping it. The blue-green database migration pattern I documented became the company standard for all subsequent large schema changes."

**Interview tip:** Bias for Action with complex technical changes means "I moved quickly but safely." The blue-green analogy is literal and direct. Prepare for: bias for action, ownership, delivering results, or technical risk management.

## Flashcards

| Q | A |
|---|---|
| In the three-pass deep copy algorithm, what does Pass 2 compute? | `curr.next.random = curr.random.next` — the clone's random pointer equals the clone of the original's random target |
| Why does the interleaving approach for Copy List avoid a hash map? | The clone of any node X is always `X.next` (while interleaved), so you can find any clone in O(1) without storing a mapping |
| What is the base case for Reverse Nodes in k-Group? | If fewer than k nodes remain ahead, return the head unchanged (do not reverse partial groups) |
| How does auto-scaling hysteresis prevent thrashing? | Use different thresholds for scale-out and scale-in (e.g., add at CPU > 70%, remove at CPU < 30%) so a brief spike doesn't trigger constant add/remove cycles |
| What is the key operational advantage of blue-green deployment over rolling deployment? | Instant rollback — just redirect the load balancer back to the old environment; rolling deployments require re-deploying the old version to each instance |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
