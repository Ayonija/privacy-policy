# Day 12 — Linked Lists: Fast/Slow Pointers
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Master the fast/slow pointer (Floyd's algorithm) — detects cycles, finds midpoints, and enables in-place palindrome checks.

## DSA (2 hours)
### Pattern: Fast / Slow Pointers (Floyd's Tortoise and Hare)
- Move slow one step and fast two steps; if they meet, a cycle exists. To find the cycle entry, reset one pointer to head and advance both one step at a time.
- Trigger condition: cycle detection, finding the middle of a list, or splitting a list in half without knowing its length.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Linked List Cycle | 141 | Easy | Fast/Slow | If fast ever equals slow (both non-null), a cycle exists |
| 2 | Linked List Cycle II | 142 | Medium | Fast/Slow + Math | After meeting point, reset slow to head; advance both one step — they meet at cycle entry |
| 3 | Palindrome Linked List | 234 | Medium | Fast/Slow + Reversal | Find mid with slow/fast; reverse second half in-place; compare two halves; restore (optional) |

### Code Skeleton
```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Cycle detection (LC 141)
    public static boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {   // fast.next check prevents crash on fast.next.next
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {        // same node reference (pointer equality, not value)
                return true;
            }
        }
        return false;
    }

    // Cycle entry (LC 142)
    public static ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                slow = head;         // reset one pointer to head
                while (slow != fast) {
                    slow = slow.next;
                    fast = fast.next;  // both advance one step
                }
                return slow;         // meet at cycle entry node
            }
        }
        return null;
    }
}
```

### Interview Tips

- **State both guard conditions:** "`while fast != null && fast.next != null` — we check `fast` (could be null at list end) AND `fast.next` (would crash on `fast.next.next`). Both guards matter." Saying this proves you think about null-pointer safety.
- **Pointer equality, not value equality:** `slow == fast` compares object references — they must point to the exact same node, not just nodes with equal values. Mentioning this separates candidates who understand references from those who don't.
- **LC 142 — state the math:** "After the meeting point, the mathematical property gives: distance(head → cycle_entry) == distance(meeting_point → cycle_entry, going forward). So resetting one pointer to head and advancing both one step places them at the cycle entry." Interviewers won't ask you to prove it, but showing awareness of the property scores points.
- **LC 234 three-step blueprint:** (1) find middle — fast/slow, (2) reverse second half in-place — three-pointer reversal, (3) compare half by half until one hits null. State all three before coding.
- **Common mistake:** in LC 234, after finding mid with fast/slow, `slow.next` is the start of the second half — but for even-length lists the "second half" must start at `slow.next` (not `slow`). Trace a 4-node example to confirm.

### STAR Interview Framework

> **How to use the STAR method when explaining Fast/Slow Pointers (Floyd's Algorithm) in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list and asked to determine if it contains a cycle — and if so, find where the cycle begins. A brute-force approach using a HashSet of visited nodes is O(n) time but O(n) space. Floyd's algorithm achieves O(n) time and O(1) space."

**Task:** "My goal was to detect a cycle and find its entry point using only two pointers — no extra storage — by exploiting the mathematical property that a fast pointer moving at 2x speed will meet a slow pointer inside any cycle."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Cycle detection in a linked structure with O(1) space — fast/slow pointer. The trigger: 'does this traversal loop?' or 'find the midpoint without knowing the length.'"
2. *Initialize:* "I set `slow = head, fast = head`. The guard is `while fast != null && fast.next != null` — both conditions required to prevent null pointer dereference on `fast.next.next`."
3. *Core loop logic:* "Advance `slow` one step and `fast` two steps. If `slow == fast` (pointer equality, not value equality), a cycle exists."
4. *Convergence guarantee:* "Once inside the cycle, `fast` gains exactly 1 position on `slow` per step. In at most `C` steps (cycle length), fast wraps around and meets slow. Outside a cycle, `fast` hits null in at most `n/2` steps."
5. *Duplicate handling / edge case proactivity:* "For finding the cycle entry (LC 142): after the meeting point, reset `slow = head` and advance both one step at a time. They meet at the cycle entry. This relies on the mathematical property that `distance(head → entry) == distance(meeting_point → entry)`."

**Result:** "Floyd's algorithm achieves O(n) time and O(1) space. The HashSet alternative is O(n) time and O(n) space — a 1000× space difference for a 10⁶-node list. The mathematical proof for cycle entry is worth stating: `F = nC - k` where F is the distance to the cycle entry and k is the meeting offset inside the cycle."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Fast/Slow Pointers here |
|-------------|---------------------------|-------------------------------|
| HashSet of visited nodes O(n) space | When space is not constrained and readability matters | Fast/slow is O(1) space; HashSet is O(n); for large lists O(n) space is a meaningful cost |
| Converting list to array and checking indices | When the list supports random access | Linked lists don't support random access; array conversion is O(n) space |

**Why NOT HashSet:** O(n) space. For a 10⁶-node list, that's ~8MB just for the HashSet vs ~16 bytes for two pointers. Also, problem often explicitly requires O(1) space.
**Why NOT array conversion:** Linked lists don't have O(1) random access; converting to array is O(n) space and doesn't solve the problem more simply than HashSet.

### Edge Cases to Trace Before Coding
- LC 141: no cycle, single node → `fast` becomes null after 0 iterations; return false ✓
- LC 141: tail points to head (full cycle) → fast catches slow after a few iterations ✓
- LC 234: `[1]` → single node is a palindrome; fast immediately hits end, slow at head, second half is empty ✓
- LC 234: `[1, 2]` → not palindrome; after fast/slow, slow is at node 1, reverse node 2, compare 1 ≠ 2 ✓
- LC 142: no cycle → return null (while loop exits normally)

## System Design (1 hour)
### Topic: Horizontal Scaling — Stateless Architecture Patterns
- **Stateless services:** every request carries all needed context (JWT token, request parameters); server holds no per-user state between requests.
- **Shared session store:** if session state is unavoidable, externalise it to Redis or Memcached; all server instances read from the same store.
- **Sticky sessions (session affinity):** load balancer routes a client to the same server for every request; avoids shared state but reintroduces single-server dependence.
- Sticky sessions are an anti-pattern for true horizontal scale — prefer stateless + shared cache.
- Interview talking point: "If asked how to make a stateful monolith horizontally scalable, answer: extract session state to Redis, move file uploads to object storage (S3), and ensure the service reads configuration from environment variables or a config service — then you can run N identical replicas behind a load balancer."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Dive Deep

**STAR Story: Finding a Hidden Cyclic Dependency That Only Manifested Under Load**

**Situation (20%):** "At my previous company, we had a microservices architecture where Service A called Service B, which under certain high-traffic conditions called Service C, which then called back to Service A — a circular dependency that our static dependency graph didn't capture because Service C's call to Service A was conditional. This only appeared under sustained load exceeding 5,000 RPS and caused cascading timeouts that took the entire checkout flow down for 22 minutes before we manually intervened."

**Task (part of S/T):** "I was the on-call engineer who mitigated the incident and was then assigned to find the root cause and eliminate the circular call path. My goal was to identify all latent circular dependencies in our service graph — not just the one that caused the incident."

**Action (60-70% — be specific about what YOU did):**
"First, I instrumented our distributed tracing system to capture full call graphs, not just parent-child relationships — this required adding one line of context propagation middleware to 15 services over 2 days.
Then, I wrote a cycle detection script that ran Floyd's algorithm on the captured call graphs — it treated each service as a node and each observed inter-service call as a directed edge. The algorithm ran over 30 days of sampled trace data.
Next, the script identified 3 latent circular paths — only one of which had triggered the incident. I documented all three with call frequency and risk level, then presented to the service owners.
Finally, I coordinated with the 3 affected service owners to add circuit breakers on the back-edges of each cycle, deployed within 1 week."

**Result (10-20%):** "The checkout cascade failure has not recurred in 8 months since the circuit breakers were added. The cycle detection script found 2 additional circular paths that had not yet triggered incidents — preventing at least 2 future outages. The tracing instrumentation also reduced our average incident investigation time from 45 minutes to 12 minutes."

**Interview tip:** Dive Deep rewards going beyond the immediate fix to find systemic root cause. "I wrote a script to detect all latent circular dependencies" is the deep action that earns credit. Prepare for: dive deep, ownership, problem-solving, or preventing future failures.

## Flashcards

| Q | A |
|---|---|
| Why does Floyd's algorithm work — what is the mathematical proof sketch? | Let F = distance to cycle start, C = cycle length, k = meeting point offset inside cycle; when they meet: 2*(F+k) = F+k+n*C → F = n*C − k; resetting slow to head and advancing both one step places them both at cycle entry after F more steps |
| How do you find the middle of a linked list using fast/slow pointers? | Start both at head; advance slow one step and fast two steps; when fast is null or fast.next is null, slow is at the middle |
| In Palindrome Linked List, why do you reverse the second half instead of using a stack? | Reversing in-place uses O(1) extra space; a stack uses O(n) |
| What invariant guarantees Floyd's algorithm terminates? | Once inside a cycle, slow and fast are in the same cycle; fast gains one position per step on slow, so they must meet within C steps (cycle length) |
| When is sticky sessions preferable to a shared session store? | When latency to the external session store is unacceptably high, or the service cannot be refactored to be stateless — but it trades scale for simplicity |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
