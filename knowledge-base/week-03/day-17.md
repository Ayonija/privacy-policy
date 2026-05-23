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
| 3 | Split Linked List in Parts | 725 | Medium | Length + remainder distribution | `base = length / k; extra = length % k`; first `extra` parts get `base+1` nodes; remaining get `base` |

### Code Skeleton
```java
import java.util.Deque;
import java.util.ArrayDeque;

class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

class Solution {
    // Add Two Numbers II (LC 445) — stack approach
    public static ListNode addTwoNumbersII(ListNode l1, ListNode l2) {
        Deque<Integer> s1 = new ArrayDeque<>(), s2 = new ArrayDeque<>();
        while (l1 != null) { s1.push(l1.val); l1 = l1.next; }
        while (l2 != null) { s2.push(l2.val); l2 = l2.next; }
        int carry = 0;
        ListNode dummy = new ListNode(0);
        while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {
            int val = carry;
            if (!s1.isEmpty()) val += s1.pop();
            if (!s2.isEmpty()) val += s2.pop();
            carry = val / 10;
            val = val % 10;
            ListNode node = new ListNode(val);
            node.next = dummy.next;
            dummy.next = node;       // prepend to build result in correct order
        }
        return dummy.next;
    }

    // Split in Parts (LC 725) — skeleton
    public static ListNode[] splitListToParts(ListNode head, int k) {
        // 1. compute length
        // 2. base = length / k; extra = length % k
        // 3. for each part: take base nodes (+ 1 if extra > 0); cut; store head
        return new ListNode[k]; // TODO: implement
    }
}
```

### Interview Tips

- **Add Two Numbers II — justify the stack before coding:** "Since digits are stored most-significant first, I need to add from the least-significant end. I push both lists onto stacks to get O(1) access to digits in reverse order. The alternative — reverting the lists — modifies the input which is usually undesirable."
- **Split in Parts — compute `base` and `extra` first:** "State `base = length / k; extra = length % k` before touching any code. The first `extra` parts get `base + 1` nodes; the remaining `k - extra` parts get `base` nodes. If `length < k`, some parts are empty (null head)."
- **Brute force for Split in Parts:** there is no brute force — this is an O(n + k) algorithm by nature. The value is in computing the distribution correctly, not in optimizing complexity.
- **Common mistake in Add Two Numbers II:** using a queue instead of a stack — the deque's `push()` adds to the front in most implementations; confirm the data structure gives LIFO access, not FIFO.

### STAR Interview Framework

> **How to use the STAR method when explaining Delete Without Head + List Splitting in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a linked list and an integer k, and asked to split it into k parts where each part is as equal-length as possible, with earlier parts getting the extra node if the length isn't divisible. A brute-force approach — collect all nodes into an array, then slice — is O(n + k) time but O(n) space. The in-place approach achieves O(n + k) time and O(k) space (just the output array of heads)."

**Task:** "My goal was to split the list in-place by computing the exact size of each part upfront: `base = length / k; extra = length % k`. The first `extra` parts get `base + 1` nodes; the rest get `base` nodes."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Divide a list into k parts in-place → length computation + remainder distribution. No sorting, no extra storage."
2. *Initialize:* "Count the length in one pass. Compute `base = length / k; extra = length % k`. Allocate `result = new ListNode[k]`."
3. *Core loop logic:* "For each part `i` from 0 to k-1: the part size is `base + (i < extra ? 1 : 0)`. Store `curr` as `result[i]`. Advance `curr` by `partSize - 1` steps. Cut: `next = curr.next; curr.next = null; curr = next`."
4. *Convergence guarantee:* "Total nodes distributed = `base * k + extra = length`. Each node is visited at most twice — once for length counting, once for distribution. O(n + k)."
5. *Duplicate handling / edge case proactivity:* "If `length < k`, some parts are empty — their result entry stays null. The cut step `if curr != null: curr.next = null` must guard against null to avoid NullPointerException when the list runs out."

**Result:** "O(n + k) time, O(k) output space. The key insight is computing the distribution formula `base + (i < extra ? 1 : 0)` upfront — this eliminates all per-step conditional logic during splitting and makes the algorithm deterministic."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer In-Place Splitting here |
|-------------|---------------------------|-------------------------------|
| Collect all nodes to array, slice | When code clarity matters more than space | O(n) extra space vs O(k) for in-place; in-place is standard for linked list problems |
| Recursive splitting | When a functional style is preferred | Recursive uses O(k) call stack and is harder to reason about for unequal splits |

**Why NOT array collection:** O(n) extra space. The in-place approach allocates only `result[k]` — O(k) output.
**Why NOT recursive:** Recursive splitting is harder to reason about for the unequal-length case and doesn't gain any asymptotic advantage.

### Edge Cases to Trace Before Coding
- LC 445 (Add Two Numbers II): one list is empty → stack is empty; `val += 0` correctly handles the empty side
- LC 725: `k = 1` → one part containing the entire list; `base = length, extra = 0`
- LC 725: `k > length` → the first `length` parts are single-node lists; the remaining `k - length` parts are null
- LC 203 (Remove Elements): all nodes match `val` → every node is removed; return null

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

**Leadership Principle:** Hire and Develop the Best

**STAR Story: Distributing Unequal Work Loads to Match Engineer Skill and Capacity**

**Situation (20%):** "During a sprint planning session, our team had 7 high-priority tasks to distribute across 5 engineers with significantly different experience levels — from a senior engineer with 8 years of distributed systems experience to a new grad in their third month. Equal task distribution (1-2 tasks each, chosen arbitrarily) would have over-loaded the junior engineers and left the senior engineer under-utilized."

**Task (part of S/T):** "As the tech lead, my goal was to distribute the 7 tasks across 5 engineers in a way that maximized team throughput while simultaneously giving each engineer tasks just above their current level — the 'stretch' principle. No engineer should have tasks so far above their level that they'd stall, or so far below that they'd be unchallenged."

**Action (60-70% — be specific about what YOU did):**
"First, I scored each task on two dimensions: complexity (1-5) and impact (1-5). I scored each engineer on their current demonstrated skill level (1-5) based on their last 3 PRs.
Then, I distributed tasks using a 'base + remainder' approach: everyone got at least 1 task. The 2 remaining tasks went to the 2 most senior engineers who had bandwidth — analogous to the first `extra` parts in the Split in Parts algorithm getting the larger slice.
Next, I paired each junior engineer with a senior engineer on adjacent tasks, so the junior could ask questions without a formal mentoring overhead.
Finally, I held a daily 15-minute standup check on task status, and I explicitly re-assigned tasks mid-sprint when one engineer hit a blocker that another could absorb faster."

**Result (10-20%):** "All 7 tasks shipped in the sprint — the first time in 3 sprints that 100% of planned tasks were completed on time. The new grad engineer shipped their task independently for the first time. The senior engineer noted they felt more productive than in the previous sprint where they'd been paired on lower-complexity work. I formalized the skill-weighted distribution approach and used it in the next 4 sprints."

**Interview tip:** Hire and Develop answers should show you invested in people's growth, not just task completion. "I matched task complexity to just above each engineer's current level" is the growth investment. Prepare for: hire and develop, dive deep, delivering results, or building teams.

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
