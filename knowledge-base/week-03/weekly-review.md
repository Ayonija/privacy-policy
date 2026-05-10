# Week 03 Review — Days 15–21

## Phase
Phase 1: DSA Mastery | Month 1

## Patterns Covered This Week
**Days 15–20 (from Slot 2):**
- Two dummy heads + partition (Partition List)
- Rotation via circular stitch (Rotate List)
- Remove duplicates — keep one copy (LC 83)
- Two pointers on two lists + equidistance trick (Intersection)
- Two-chain weave (Odd Even Linked List)
- Fast/Slow + reverse + interleave (Reorder List)
- Delete without head reference (copy-next trick)
- Stack-based column arithmetic (Add Two Numbers II)
- Remainder distribution (Split Linked List in Parts)
- k-Group reversal + recursive stitching
- Three-pass deep copy with interleaving (Copy List with Random Pointer)
- Fast/Slow with offset to find pre-middle node
- Monotonic stack applied to linked list (Next Greater Node)
- Fast/Slow + reverse second half + paired sum (Maximum Twin Sum)
- Find boundaries + splice (Merge In Between)
- Gap pointer value swap (Swapping Nodes)

**Day 21 (Slot 3):**
- Stack — LIFO matching (Valid Parentheses)
- Stack — arithmetic evaluation (Reverse Polish Notation)
- Stack — nested state (Decode String)

## System Design Topics Covered
**Scaling & Load Balancers (Days 15–20):** Covered health checks and failover, Layer 4 vs. Layer 7 LBs, DB read replicas and sharding, message queues as traffic spike buffers, auto-scaling with hysteresis, blue-green deployment, and the full four-tier web architecture.

**CAP Theorem intro (Day 21):** Introduced Brewer's theorem — Consistency, Availability, Partition Tolerance. In practice, P is mandatory, so systems choose CP (refuse stale reads) or AP (serve stale data) during a network partition.

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Partition List (two dummy heads) | | |
| Rotate List (circular stitch) | | |
| Intersection (equidistance trick) | | |
| Reorder List (fast/slow + reverse + interleave) | | |
| Add Two Numbers II (stack arithmetic) | | |
| k-Group reversal | | |
| Deep copy with random pointer | | |
| Maximum Twin Sum | | |
| Stack — bracket matching | | |
| Stack — expression evaluation | | |
| Stack — nested state decode | | |
| CAP Theorem (conceptual) | | |

## Weekly Flashcard Deck
All 35 cards for Days 15–21:

| Q | A |
|---|---|
| How do you null-terminate the greater partition in LC 86, and why is it critical? | `greater.next = None` — the last node may still point back into the original list, creating a cycle |
| What is the formula for the effective rotation amount in Rotate List? | `effective = k % length`; if k is a multiple of length the list is unchanged |
| How do you find the new tail in Rotate List? | Advance `length − effective − 1` steps from head |
| What is the difference between active and passive LB health checks? | Active: LB probes each backend on a schedule. Passive: observes real traffic errors — no extra requests but slower to detect |
| What problem does the circuit breaker pattern solve that health checks do not? | Cascading failures — stops callers hammering a failing downstream service and exhausting their own thread pool |
| Why do both pointers in LC 160 (Intersection) travel the same total distance? | Each traverses both lists: total = len(A) + len(B); they meet at the intersection on the same step |
| How do you interleave two halves in Reorder List without losing nodes? | Save `first.next` and `second.next` before relinking; set `first.next = second`, `second.next = saved_first` |
| What is the loop invariant for Odd Even Linked List? | `odd` always points to the last odd-indexed node; `even` to the last even-indexed node |
| What is replication lag and when does it matter? | Delay between primary write and replica update; matters when reading immediately after a write requiring the latest data |
| When does sharding become necessary vs. adding read replicas? | Sharding is needed when write throughput saturates the primary; read replicas only help with read load |
| How do you delete a linked list node when only that node is given? | Copy `node.next.val` into `node.val`; set `node.next = node.next.next` |
| Why can't the copy-next deletion work on the tail node? | The tail has no next node to copy from |
| How do you add two numbers stored most-significant-digit-first in a linked list? | Push both lists onto stacks; pop and add with carry; prepend result nodes to build the answer in correct order |
| How does a message queue decouple producers and consumers during a spike? | Producers enqueue at any rate; workers drain at capacity; the queue absorbs the burst |
| What is N+1 redundancy and why is it the minimum production standard? | N+1 instances so a single failure leaves N — enough to meet required capacity without degradation |
| In the three-pass deep copy algorithm, what does Pass 2 compute? | `curr.next.random = curr.random.next` — the clone's random points to the clone of the original's random target |
| Why does the interleaving approach for Copy List avoid a hash map? | The clone of node X is always `X.next` while interleaved — O(1) lookup without storing a mapping |
| What is the base case for Reverse Nodes in k-Group? | Fewer than k nodes remain — return head unchanged (do not reverse partial groups) |
| How does auto-scaling hysteresis prevent thrashing? | Different thresholds for scale-out and scale-in — e.g., add at CPU > 70%, remove at CPU < 30% |
| What is the operational advantage of blue-green over rolling deployment? | Instant rollback — redirect LB back to old environment; rolling requires re-deploying old version per instance |
| How do you stop slow at the node *before* the middle in Delete Middle Node? | Init `slow = head`, `fast = head.next`; when fast stops, slow is at pre-middle node |
| Why does the monotonic stack for Next Greater Node store indices, not values? | You need the index to fill the result array at the correct position |
| How does the twin sum algorithm handle even-length lists? | Fast/slow gives slow at the second middle; reversing from there gives the correct second half |
| In the four-tier architecture, what is the primary role of the CDN? | Serve static assets from nearest edge node, reducing origin load and latency |
| Why is Redis placed in front of the DB for hot-path reads? | Redis is in-memory and fast but not ACID-durable; it caches hot data, not the source of truth |
| How do you check that parentheses are balanced using a stack? | Push open brackets; on closing bracket check stack top for match; return `stack == []` at end |
| How do you evaluate a Reverse Polish Notation expression? | Push numbers; on operator pop two values, apply, push result; single remaining element is the answer |
| How do you decode a nested encoded string like `3[a2[bc]]`? | Push `(curr_string, curr_num)` on `[`; on `]` pop and append `curr_string * count` to restored string |
| What are the three properties of the CAP theorem? | Consistency (all nodes see same data), Availability (every request gets a response), Partition Tolerance (works during network splits) |
| Why can't a distributed system guarantee all three of CAP simultaneously? | During a partition you must choose: refuse requests (CP) or return stale data (AP) — not both |
| How do you find the kth node from the end without knowing list length? | Two pointers with gap k: advance fast k steps, then advance both until fast is None — slow is at kth from end |
| What is the recommended order of scaling steps before DB sharding? | Add caching (Redis) → read replicas → tune slow queries → connection pooling → only then sharding |
| Why scale in conservatively in auto-scaling? | Aggressive scale-in causes oscillation — a brief dip triggers deprovision then immediate re-provision |
| What is the one-sentence rule for L4 vs. L7 load balancing? | L4 for pure TCP/UDP low-latency; L7 for HTTP needing content-aware routing or deployment strategies |
| How does IP Hash provide session affinity? | `server_index = hash(client_ip) % num_servers` — same IP always maps to same server |

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| — | — | — | — | — |
