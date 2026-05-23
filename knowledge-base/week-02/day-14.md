# Day 14 — Review Day: Linked Lists Week 2
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Solidify all linked list patterns from Days 11–13; identify the remove-nth and dedup variants before moving to Week 03.

## DSA (2 hours)
### Pattern: Review — Remove, Dedup, and Two-Pointer on Lists
- Remove Nth From End: use two pointers with a gap of n; when the fast pointer hits the end, slow is at the node before the target.
- Remove duplicates from sorted list II: delete ALL nodes with duplicate values (not just duplicates), requiring a prev pointer that stops before the duplicate run.
- These patterns extend fast/slow and dummy-head thinking into deletion-focused problems.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Middle of the Linked List | 876 | Easy | Fast/Slow | When fast reaches end, slow is at mid; for even-length lists slow is the second middle |
| 2 | Remove Nth Node From End of List | 19 | Medium | Two Pointers (gap n) | Dummy head; advance fast by n+1 steps first; then move both until fast is null |
| 3 | Remove Duplicates from Sorted List II | 82 | Medium | Dummy head + skip run | When a duplicate run is detected, skip all nodes with that value; prev.next jumps over entire run |

### Code Skeleton
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Remove Nth from end (LC 19)
    public static ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode fast = dummy, slow = dummy;
        for (int i = 0; i < n + 1; i++) {   // advance fast n+1 steps — leaves 1-node gap between slow and fast
            fast = fast.next;
        }
        while (fast != null) {               // advance both until fast is null
            fast = fast.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;  // slow is now at the node BEFORE the target
        return dummy.next;
    }

    // Remove duplicates II (LC 82)
    public static ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prev = dummy;   // prev stops before any duplicate run
        ListNode curr = head;
        while (curr != null) {
            if (curr.next != null && curr.val == curr.next.val) {
                int val = curr.val;
                while (curr != null && curr.val == val) {  // skip ALL nodes with this value
                    curr = curr.next;
                }
                prev.next = curr;   // prev jumps over the entire duplicate run
            } else {
                prev = curr;        // non-duplicate: advance prev normally
                curr = curr.next;
            }
        }
        return dummy.next;
    }
}
```

### Interview Tips

- **The n+1 gap trick:** "I advance `fast` by `n+1` steps from the dummy head (not n) so that when `fast` is null, `slow` is at the node *before* the one I want to delete — allowing `slow.next = slow.next.next`." Draw this on the whiteboard if possible.
- **Why dummy head is essential for LC 19:** if `n` equals the list length, the node to remove is the head — without a dummy, you'd crash accessing `slow.next.next`. Dummy makes the head removal identical to any other node removal.
- **LC 82 vs LC 83 (Remove Duplicates I):** LC 83 keeps one copy of each duplicate; LC 82 removes ALL copies. The key difference: LC 82's `prev` pointer *doesn't advance* when it encounters a duplicate run; it only advances on non-duplicate nodes.
- **Common mistake in LC 82:** checking `curr.val == curr.next.val` without guarding `curr.next != null` — accessing `curr.next.val` when `curr.next` is null throws a NullPointerException. Always check `curr.next` first.
- **Brute force for LC 876 (Middle):** count nodes, then traverse again to the midpoint — O(2n). Fast/slow finds the middle in one pass O(n).

### STAR Interview Framework

> **How to use the STAR method when explaining Remove, Dedup, and Gap-Pointer patterns on Linked Lists in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list and asked to remove the nth node from the end in a single pass, without knowing the list length. A two-pass approach — first count length, then walk to position `length - n` — is O(n) time but requires two traversals. The gap-pointer approach achieves the same in a single pass."

**Task:** "My goal was to find and remove the nth-from-last node in one pass by maintaining a gap of exactly n+1 between two pointers — so when the fast pointer reaches null, the slow pointer is precisely at the node before the target."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Find position relative to the end of a list in one pass → gap pointer (two pointers with fixed distance). The trigger: 'nth from end' without knowing length."
2. *Initialize:* "I use a dummy head: `dummy.next = head`. Both `fast` and `slow` start at `dummy`. I advance `fast` exactly `n+1` steps first."
3. *Core loop logic:* "Then advance both `fast` and `slow` one step at a time until `fast == null`. At this point, `slow` is at the node just BEFORE the target (the one to delete). Execute `slow.next = slow.next.next`."
4. *Convergence guarantee:* "The gap is exactly n+1 (fast starts n+1 ahead of slow). When fast hits null (at position `length + 1` from dummy), slow is at position `length + 1 - (n+1) = length - n` from dummy, which is the node before the nth-from-last."
5. *Duplicate handling / edge case proactivity:* "Advancing `n+1` steps (not n) is the critical detail — this ensures slow stops at the node BEFORE the target, enabling `slow.next = slow.next.next`. Using the dummy head ensures this works even when `n == list_length` (removing the head)."

**Result:** "Single-pass O(n) solution vs O(2n) two-pass. The gap-pointer technique is generalisable: any 'find position k from the end' problem uses a fast pointer k+1 steps ahead. The dummy head is non-optional here — without it, removing the head node requires special-casing that is easy to get wrong under interview pressure."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Gap Pointer here |
|-------------|---------------------------|-------------------------------|
| Two-pass: count then walk O(2n) | When readability matters more than elegance | Single-pass is expected at FAANG; two-pass is also O(n) but shows less pattern fluency |
| Store all nodes in an array then index from end | When memory is not constrained | O(n) extra space vs O(1) for gap pointer |

**Why NOT two-pass:** O(2n) operations — technically still O(n), but single-pass is the pattern being tested. Two-pass signals you don't know the gap-pointer technique.
**Why NOT array:** O(n) extra space. The interview almost always tests the O(1)-space single-pass approach.

### Edge Cases to Trace Before Coding
- LC 19: `n` equals list length → remove head; dummy prevents crash; `slow` stays at dummy after n+1 advance ✓
- LC 19: single-node list, `n = 1` → remove head; dummy.next becomes null ✓
- LC 82: list with all same values `[1,1,1]` → entire list is a duplicate run; `prev` stays at dummy; `prev.next = null`; return `dummy.next = null` ✓
- LC 82: no duplicates → `curr.next` is always different from `curr`; `prev` advances every step; list returned unchanged ✓
- LC 876: even-length list → slow ends at the second middle node (as per problem definition)

## System Design (1 hour)
### Topic: Load Balancers — Layer 4 vs. Layer 7
- **Layer 4 (Transport):** routes based on IP + TCP/UDP port; no knowledge of content; very fast, low overhead; cannot make routing decisions based on URL path or headers.
- **Layer 7 (Application):** inspects HTTP headers, URL, cookies; can route `/api` to one server farm and `/static` to another; supports A/B testing and canary deployments.
- Layer 4 example: AWS Network Load Balancer (NLB); Layer 7 example: AWS Application Load Balancer (ALB) / Nginx.
- Trade-offs: L4 is faster and simpler; L7 is more flexible but adds latency (must fully parse the HTTP request).
- Interview talking point: "If asked which load balancer layer to use for a microservices API gateway, answer: Layer 7 — you need path-based routing (`/users` → User Service, `/orders` → Order Service) and header inspection for auth, which Layer 4 cannot provide."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Insist on the Highest Standards

**STAR Story: Removing Low-Quality Data from a Training Pipeline Before It Reached the Model**

**Situation (20%):** "At my previous company, our machine learning team was training a recommendation model on user interaction data. After an audit, I discovered that 18% of the training data was coming from internal employees and automated test accounts — interactions that didn't represent real user behavior. The model was overfitting to internal usage patterns, which caused it to recommend internal-use-only content to real users."

**Task (part of S/T):** "I was the data engineering lead. My goal was to identify and remove all contaminated records — not just filter the obvious test accounts, but eliminate entire data runs that were corrupted by them — analogous to LC 82's 'remove the entire duplicate run, not just the extra copies.'"

**Action (60-70% — be specific about what YOU did):**
"First, I built an audit query that traced each training record back to its originating user session. I found that contaminated records weren't evenly distributed — they came in clusters (entire sessions from test accounts), and sessions that overlapped with test activity were also corrupted even if the individual records looked legitimate.
Then, I implemented a session-level filter: if any record in a session was flagged as non-user (test account or employee), I excluded the entire session — not just the flagged records. This is the 'skip the entire run' principle.
Next, I ran the filtered dataset through a data validation pipeline I wrote, which checked statistical distributions against a holdout of known-clean sessions. All metrics fell within 2 standard deviations.
Finally, I retrained the model on the clean dataset and ran a 2-week A/B test before promoting it to production."

**Result (10-20%):** "The retrained model improved recommendation click-through rate by 14% — the previous model had been dragged down by the internal-use bias. The data contamination rate dropped from 18% to under 0.3%. I also implemented ongoing session-level monitoring that flags any session with a test-account record, preventing future contamination from accumulating silently."

**Interview tip:** Insist on the Highest Standards answers should show you identified a systemic quality issue (not just a symptom) and implemented a structural fix — not a workaround. "I filtered the entire corrupted session, not just the flagged records" is the key action that shows the standard you held. Prepare for: highest standards, ownership, dive deep, or quality without compromise.

## Flashcards

| Q | A |
|---|---|
| How do you set the gap between two pointers to find the nth node from the end? | Advance fast pointer by n+1 steps from the dummy head; then advance both until fast is null; slow is now just before the target |
| Why advance n+1 steps (not n) in Remove Nth From End? | You want slow to stop at the node *before* the target so you can do `slow.next = slow.next.next`; n+1 leaves that one-step buffer |
| How do you skip an entire duplicate run in LC 82? | Detect `curr.val == curr.next.val`; record the value; advance curr while `curr.val == recorded_val`; then set `prev.next = curr` (bypassing all duplicate nodes) |
| What routing capability does a Layer 7 load balancer have that Layer 4 cannot? | Path-based and header-based routing (e.g., route `/api` vs `/static` to different backends, inspect auth headers, enable canary deployments) |
| Name a real AWS service for each load balancer layer. | Layer 4: Network Load Balancer (NLB) — ultra-low latency, TCP/UDP. Layer 7: Application Load Balancer (ALB) — HTTP/HTTPS, path/host routing |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
