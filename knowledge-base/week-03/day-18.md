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
- STAR prompt: Tell me about a time you deployed a significant change in a way that allowed instant rollback — analogous to blue-green deployment where the old environment stays live until the new one is verified.
- Leadership principle: Bias for Action

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
