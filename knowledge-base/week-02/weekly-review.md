# Week 02 Review — Days 8–14

## Phase
Phase 1: DSA Mastery | Month 1

## Patterns Covered This Week
**Days 8–10 (from Slot 1):**
- Two Pointers — 4Sum, k-Sum generalisation
- Two Pointers — Interval List Intersection
- Sliding Window — gain maximisation (Grumpy Bookstore)
- Sliding Window — complement (Maximum Points from Cards)
- Sliding Window — Monotonic Deque (Longest Continuous Subarray, abs diff ≤ limit)

**Days 11–14 (Slot 2):**
- Linked List Reversal — iterative (full and partial)
- Linked List Reversal — pair/group swap
- Fast/Slow Pointers — cycle detection (Floyd's algorithm)
- Fast/Slow Pointers — cycle entry, list middle, palindrome check
- Merge — two sorted lists, dummy head pattern
- Add Two Numbers — column arithmetic with carry
- Merge Sort on a linked list
- Two Pointers (gap n) — remove nth from end
- Dummy head + skip run — remove all duplicates II

## System Design Topics Covered
**Big-O Deep Dives (Days 8–10):** Covered when O(n³) is acceptable (k-Sum), interval intersection at scale (calendar systems, IP range lookups), monotonic deque amortised O(n) proof, and sliding window rate limiters.

**Vertical vs. Horizontal Scaling (Days 11–14):** Vertical scale-up has a hard ceiling and creates a SPOF; horizontal scale-out requires stateless services and a coordination layer. Covered stateless architecture, sticky sessions, shared session store in Redis, and load balancing algorithms (Round Robin, Least Connections, IP Hash, Weighted). Concluded with Layer 4 vs. Layer 7 load balancers and AWS NLB vs. ALB trade-offs.

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| 4Sum / k-Sum generalisation | | |
| Interval List Intersection | | |
| Sliding Window — gain maximisation | | |
| Sliding Window — complement | | |
| Monotonic Deque window | | |
| Linked List Reversal (full) | | |
| Linked List Reversal (partial / LC 92) | | |
| Fast/Slow — cycle detection | | |
| Fast/Slow — cycle entry (Floyd's proof) | | |
| Merge two sorted lists | | |
| Merge Sort on list | | |
| Remove nth from end (gap pointer) | | |
| Remove duplicates II (skip run) | | |
| Scaling concepts (H vs V, LB algorithms) | | |

## Weekly Flashcard Deck
All 35 cards for Days 8–14:

| Q | A |
|---|---|
| How do you generalise two-pointer pair-sum to k-sum? | Fix (k−2) elements with nested loops, then run one two-pointer pass for the remaining pair; time = O(n^(k-1)) |
| What is the duplicate-skip rule for the fixed indices in 4Sum? | `if i > 0 and nums[i] == nums[i-1]: continue` and similarly for j with `j > i+1` guard |
| In interval intersection, which pointer advances after checking overlap? | The pointer whose interval has the smaller end value |
| How do you determine if two intervals [a,b] and [c,d] overlap? | They overlap if and only if `max(a,c) ≤ min(b,d)` |
| In Move Zeroes, how do you preserve relative order of non-zero elements? | Slow pointer as write position; overwrite left-to-right with non-zero values; fill zeros at end |
| How does a monotonic decreasing deque find the window maximum in O(1)? | Front of deque always holds the index of the current window's max; pop from back any element ≤ new element before appending |
| When do you pop from the front of the deque? | When the front index is < left (it has slid out of the current window) |
| What two deques are needed for LC 1438 and what invariant does each maintain? | max_deque (decreasing, front = window max) and min_deque (increasing, front = window min) |
| How do you form the output for Summary Ranges? | When fast+1 is not consecutive or end is reached: output "slow" or "slow→fast"; then set slow = fast+1 |
| Why prefer a monotonic deque over a sorted structure for sliding window max? | Deque is O(n) total; sorted structure costs O(n log n) |
| Write the three-pointer iterative linked list reversal loop body. | `next_node = curr.next; curr.next = prev; prev = curr; curr = next_node` |
| What does the dummy node buy you in linked list problems? | Eliminates head edge case — you can always attach nodes via `curr.next` uniformly |
| What is the time and space complexity of reversing a linked list in-place? | O(n) time, O(1) space |
| What is the key difference between vertical and horizontal scaling? | Vertical: bigger machine, single node, hard ceiling. Horizontal: more machines, stateless required, no theoretical ceiling |
| Why must services be stateless to scale horizontally? | Any server may handle any request; local state would be invisible to other servers |
| Why does Floyd's algorithm work — what is the mathematical proof sketch? | After meeting: F = n*C − k; resetting slow to head and advancing both one step puts them at cycle entry after F more steps |
| How do you find the middle of a linked list using fast/slow pointers? | Both start at head; slow moves one step, fast moves two; when fast is None or fast.next is None, slow is at mid |
| In Palindrome Linked List, why do you reverse the second half instead of using a stack? | In-place reversal uses O(1) space; a stack uses O(n) |
| What invariant guarantees Floyd's algorithm terminates? | Fast gains one position per step on slow inside the cycle; they must meet within C steps |
| When is sticky sessions preferable to a shared session store? | When latency to the external store is unacceptably high or service cannot be refactored to be stateless |
| How do you cut a linked list in half for merge sort? | Find mid with fast/slow; save `mid.next` as right-half head; set `mid.next = None` |
| Why does Add Two Numbers need a dummy head? | First result node is computed inside the loop — dummy lets you attach uniformly |
| What is the space complexity of recursive merge sort on a linked list? | O(log n) for the call stack |
| When does Round Robin load balancing produce imbalance? | When servers have different capacities or request durations vary significantly |
| How does IP Hash load balancing provide session affinity? | `server_index = hash(client_ip) % num_servers` — same IP always maps to same server |
| How do you set the gap between two pointers to find the nth node from the end? | Advance fast pointer n+1 steps from dummy head; then move both until fast is None |
| Why advance n+1 steps (not n) in Remove Nth From End? | You want slow to stop at the node *before* the target to perform the deletion |
| How do you skip an entire duplicate run in LC 82? | Record the duplicate value; advance curr while `curr.val == val`; then set `prev.next = curr` |
| What routing capability does a Layer 7 load balancer have that Layer 4 cannot? | Path-based and header-based routing, canary deployments, auth header inspection |
| Name a real AWS service for each load balancer layer. | Layer 4: NLB (Network Load Balancer). Layer 7: ALB (Application Load Balancer) |
| How does the Grumpy Bookstore gain window technique work? | Compute baseline (always-satisfied customers); slide a window of size `minutes` to find where suppressing grumpiness gains the most extra customers |
| What is the complement technique in Maximum Points from Cards? | Taking k cards from ends = leaving n-k in middle; minimise middle window sum to maximise card score |
| What is the boundary burst problem in fixed-window rate limiters? | Client sends k requests just before and just after a window boundary, achieving 2k in a short span |
| How does Weighted Round Robin differ from regular Round Robin? | Weighted assigns proportional requests to servers based on declared capacity — e.g., a 4-core server gets 2x requests of a 2-core server |
| How does Least Connections load balancing decide which server to use? | Routes to the server with the fewest currently active connections — better for variable-duration requests |

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| — | — | — | — | — |
