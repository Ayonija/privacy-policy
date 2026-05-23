# Month 1 Review — Days 1–30
**Phase 1: DSA Mastery | Months 1–3**

## Phase
Phase 1 — DSA Foundations. Goal: build fluent pattern recognition across arrays, linked lists, stacks and queues; establish a system design vocabulary from Big-O through CAP Theorem.

---

## Patterns Mastered (comfort ≥ 4/5)
Fill in after self-assessment — mark any pattern you can write from memory in under 2 minutes:

- [ ] Two Pointers — pair/triplet sum (sorted input)
- [ ] Two Pointers — greedy shrink (Container With Most Water)
- [ ] Two Pointers — slow/fast (remove duplicates, is subsequence)
- [ ] Three Pointers — Dutch National Flag
- [ ] Sliding Window — fixed size (character frequency match)
- [ ] Sliding Window — variable size (sum/product/budget constraint)
- [ ] Sliding Window — frequency map (longest repeating replacement)
- [ ] Sliding Window — at-most-k-distinct
- [ ] Sliding Window — complement (Maximum Points from Cards)
- [ ] Sliding Window — monotonic deque (max/min window)
- [ ] Linked List — iterative reversal (full + partial)
- [ ] Linked List — fast/slow (cycle detection + entry, middle, palindrome)
- [ ] Linked List — merge two sorted lists (dummy head)
- [ ] Linked List — merge sort
- [ ] Linked List — remove nth from end (gap pointer)
- [ ] Linked List — partition (two dummy heads)
- [ ] Linked List — deep copy with random pointer (three-pass interleave)
- [ ] Linked List — k-group reversal
- [ ] Stack — LIFO matching (parentheses, brackets)
- [ ] Stack — expression evaluation (RPN)
- [ ] Stack — nested state decode (Decode String)
- [ ] Stack — adjacent cancel (Remove Duplicates, Make String Great)
- [ ] Stack — path simplification
- [ ] Stack — parallel min tracking (Min Stack)
- [ ] Stack — balance counting (O(1) space)
- [ ] Monotonic Stack — next greater element (basic + circular)
- [ ] Monotonic Stack — span accumulation (Stock Span)
- [ ] Monotonic Stack — collision simulation (Asteroid Collision)
- [ ] Monotonic Stack — fleet merging (Car Fleet)
- [ ] Monotonic Stack — contribution technique (Sum of Subarray Mins/Ranges)
- [ ] Monotonic Stack — greedy removal (Remove K Digits, Remove Duplicate Letters)
- [ ] Monotonic Stack — pattern detection (132 Pattern)
- [ ] Queue — two-stack implementation
- [ ] Queue — sliding window (Recent Calls)
- [ ] Queue — greedy simulation (Dota2 Senate)
- [ ] Deque — simulation (Reveal Cards)

---

## Patterns Needing Drill
Self-assessment: list any pattern where you could not write the skeleton from memory in 2 minutes:

- (fill in)

---

## System Design Topics Mastered
- [ ] Big-O notation — all complexity classes, amortised analysis, output vs. algorithm complexity
- [ ] RAM model — memory hierarchy (registers → L1 → L2 → RAM → disk), cache locality
- [ ] Vertical vs. horizontal scaling — stateless architecture, sticky sessions vs. shared session store
- [ ] Load balancers — Round Robin, Least Connections, IP Hash, Weighted Round Robin
- [ ] Load balancers — Layer 4 (NLB) vs. Layer 7 (ALB), path-based routing
- [ ] Load balancer health checks — active vs. passive, circuit breaker pattern
- [ ] DB scaling — read replicas, replication lag, sharding (when to shard)
- [ ] Message queues — decoupling producers/consumers, traffic spike buffering
- [ ] Auto-scaling — scale-out/scale-in hysteresis, warm-up periods, blue-green deployment
- [ ] Four-tier web architecture — DNS/CDN → LB → App servers → DB/Cache
- [ ] CAP Theorem — CP vs. AP during a network partition
- [ ] Consistency models — strong (linearisability), sequential, causal, eventual
- [ ] Eventual consistency in practice — DNS, shopping cart, social counts, CRDT, LWW
- [ ] PACELC theorem — latency vs. consistency in normal operation
- [ ] Real systems — Zookeeper (CP), Cassandra (AP/tunable), DynamoDB (AP + optional strong), etcd (CP), Spanner (PC/EL)

---

## OA / Mock Interview Stats (Month 4+)
*Not applicable — Month 4 begins OA practice.*

---

## Month Flashcard Master Deck
Aggregate by loading each weekly review:

| Week | Review File | Cards |
|------|-------------|-------|
| 01 | [week-01/weekly-review.md](week-01/weekly-review.md) | 35 |
| 02 | [week-02/weekly-review.md](week-02/weekly-review.md) | 35 |
| 03 | [week-03/weekly-review.md](week-03/weekly-review.md) | 35 |
| 04 | [week-04/weekly-review.md](week-04/weekly-review.md) | 35 |
| **Total** | | **140 cards** |

*Week 05 flashcards (Days 29–30) are included in the month-2 weekly review cycle.*

---

## Problems Solved This Month
| Day Range | Topic | Problem Count |
|-----------|-------|---------------|
| 1–10 | Arrays & Strings (Two Pointers, Sliding Window) | 30 |
| 11–20 | Linked Lists (Fast/Slow, Reversal, Merge) | 30 |
| 21–30 | Stacks & Queues (Monotonic Stack, Deque) | 30 |
| **Total** | | **90 problems** |

---

## Next Month Goals
1. **DSA:** Begin Hashing (HashMap patterns, frequency maps, advanced sliding window) on Day 31 — target 3 problems/day at Medium+Medium+Hard difficulty.
2. **System Design:** Master DB Indexing and B-Trees (Slot 4) then Caching — Redis, Memcached, eviction strategies (Slot 5).
3. **Weak pattern drill:** Re-solve any pattern rated < 3/5 in the Strength/Gap section above at least once per week in Month 2.

## STAR Mastery: Month 1 Review

### Pattern → LP Question Mapping
| LP Question | Best pattern story from this month | Key framing |
|-------------|-----------------------------------|-------------|
| "Tell me about optimizing something" | Sliding Window variable-size (O(n) vs O(n²) brute-force nested loops) | Lead with before/after: "reduced from O(n²) nested scan to O(n) single pass on a stream of 1M events" |
| "Tell me about ownership/initiative" | Monotonic Stack contribution technique (proactively identified subarray range bottleneck) | "I identified", "I proposed the monotone stack approach before being asked" |
| "Tell me about ambiguity" | Two Pointers (multiple valid orderings — left/right shrink vs fast/slow) | Highlight your decision framework: "I chose left/right for sorted input because fast/slow adds O(n) space" |
| "Tell me about working under pressure" | Sliding Window frequency map (shipped fix for a real-time alert system during an incident) | Add time constraint: "production was degrading, I had 90 minutes to ship" |
| "Tell me about a failure" | Monotonic Stack (initially missed contribution technique, used O(n²) first) | Focus on learning: "I caught it in code review — the pattern was right but the complexity analysis was wrong" |

### 5 Versatile Career Story Titles for Month 1
1. "Sliding Window — Replaced an O(n²) nested-loop substring scanner with a variable-size sliding window on a log analytics pipeline, reducing processing time from 8 minutes to 11 seconds on 10M daily events."
2. "Two Pointers — Applied Dutch National Flag three-pointer partitioning to a real-time packet classifier, eliminating an O(n²) sort for tri-state packet categorization and enabling 200K packets/second throughput."
3. "Monotonic Stack — Used the contribution technique to compute subarray range sums in O(n) for a financial risk aggregator, replacing an O(n²) double loop that caused 45-second timeouts on 100K-item portfolios."
4. "Linked List Merge Sort — Applied merge sort to a deduplication pipeline on a 500K-record linked structure, achieving O(n log n) guaranteed sort vs O(n²) insertion sort that had caused SLA breaches on large ingestions."
5. "LRU-style Deque — Designed a sliding-window recent-calls tracker using a monotonic deque, enabling O(1) amortized max/min queries for a rate-limiting dashboard that previously re-scanned O(k) entries per request."

*Tip: Each story can be adapted to 3+ LP questions by emphasizing different parts of the Action.*
