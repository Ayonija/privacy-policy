# Week 12 Review — Days 78–84
**Phase 1: DSA Mastery | Month 3 | Slots 8–9 (Days 71–90)**

## Week Span
Days 78–80: Slot 8 close-out — Tries (Map Sum, Search Suggestions, Stream of Characters, XOR Trie, Palindrome Pairs) + Skyline Problem
Days 81–84: Slot 9 start — Greedy & Backtracking (Intervals, Jump Game, Gas Station, Subsets)
System Design: Message Queues (Kafka internals, Pub/Sub, RabbitMQ vs Kafka)

---

## Patterns This Week

### DSA Patterns

**Map Sum Trie with delta updates (Day 78 — LC 677):**
Each node stores accumulated sum. On re-insert: `delta = new_val - old_val`; propagate delta along key path.

**Sort + binary search autocomplete (Day 78 — LC 1268):**
Sort products; `bisect_left` for prefix start; return next 3 matching products. O(n log n + L log n).

**Reversed-word Trie streaming (Day 78 — LC 1032):**
Insert reversed words; maintain active searches; advance all on each character; is_end = match.

**Binary XOR Trie (Day 79 — LC 421):**
30-bit Trie; for each number, traverse choosing OPPOSITE bit to maximise XOR greedily from MSB.

**Suffix deduplication (Day 79 — LC 820):**
Put words in set; remove all non-empty suffixes of each word; remaining = independent; sum len+1.

**Palindrome pairs split enumeration (Day 79 — LC 336):**
HashMap of reversed words. For each word, split at every position: palindrome prefix/suffix + reversed complement in HashMap = valid pair. O(n × L²).

**Greedy sorted counter (Day 80 — LC 846):**
Sort keys; consume groupSize consecutive cards from smallest; fail if any count goes negative.

**Two-pointer camelcase matching (Day 80 — LC 1023):**
Pointer p on pattern; scan query; uppercase mismatch = False; lowercase allowed; match if p == len(pattern).

**Max-heap lazy-deletion event sweep (Day 80 — LC 218):**
Sort events; push on start, lazy-delete on end; record critical point when max height changes.

**Interval merge sort-by-start (Day 81 — LC 56):**
Sort by start; iterate: `curr.start <= last.end` → merge (update last.end); else append.

**Interval non-overlapping sort-by-end (Day 81 — LC 435):**
Sort by end; `curr.start < last_end` → count removal; else update last_end.

**Course Schedule III max-heap swap (Day 81 — LC 630):**
Sort by deadline; push duration to max-heap; if `total_time > deadline` → evict longest; return heap size.

**Jump Game greedy max_reach (Day 82 — LC 55):**
Track `max_reach`; if `i > max_reach` → False; `max_reach = max(max_reach, i + nums[i])`.

**Jump Game II window expansion (Day 82 — LC 45):**
Track `current_end`, `farthest`; when `i == current_end`: jump++, `current_end = farthest`.

**Min Taps reduction to Jump Game II (Day 82 — LC 1326):**
Build `max_right[left] = max(min(n, i+r))` for each tap; apply Jump Game II on `max_right`.

**Gas Station single-pass reset (Day 83 — LC 134):**
Running `tank`; when `tank < 0`: start = i+1, reset tank=0. Return start if total_tank ≥ 0.

**Queue Reconstruction sort-and-insert (Day 83 — LC 406):**
Sort (height desc, k asc); insert each at index k. Taller-first invariant ensures k-count correctness.

**Max Profit Job Scheduling DP + binary search (Day 83 — LC 1235):**
Sort by end time; `dp[i] = max(dp[i-1], profit + dp[last_compatible])`; binary search for last compatible.

**Subsets backtracking (Day 84 — LC 78):**
Collect path at every node; loop from start to end; recurse with start = i+1.

**Subsets II duplicate skip (Day 84 — LC 90):**
Sort; skip if `i > start AND nums[i] == nums[i-1]`.

**Partition K bucket backtracking (Day 84 — LC 698):**
Sort desc; assign each number to a bucket; prune with `seen` set for symmetric empty buckets.

---

## Flashcard Deck — 35 Cards

### Day 78 — Map Sum, Search Suggestions, Stream of Characters

**1.** How does Map Sum handle re-inserting an existing key?
Compute `delta = new_val - old_val`. Traverse the key's path; add `delta` to `#sum` at every node. Only the changed amount propagates — no need to re-sum all values.

**2.** Why is sort + binary search simpler than Trie for Search Suggestions?
Sorted products with the same prefix form a contiguous block. `bisect_left` finds the start in O(log n); check next 3 for prefix match. No Trie node overhead; O(n log n) build, O(L log n) query.

**3.** How does Stream of Characters use a reversed-word Trie?
Insert reversed words. On each new character: start new search from root; advance all active searches. A search reaching `is_end` means a word ending at the current stream position matches. Dead searches (no matching child) are dropped.

**4.** How does Redis pipelining reduce fanout latency?
Pipeline batches 1000 ZADD commands in one TCP round-trip instead of 1000 separate RTTs. At 0.1 ms/RTT, 1000 unpipelined commands = 100 ms; pipelined = ~0.1 ms.

**5.** What is the trade-off of eventual consistency in the newsfeed write path?
Post Service returns success immediately after DB write; fanout is async (Kafka workers, 1–10 s). Users see posts within seconds, not immediately. Faster write latency; brief feed staleness.

### Day 79 — XOR Trie, Short Encoding, Palindrome Pairs

**6.** How does the XOR Trie greedily maximise XOR?
For each number, traverse the 30-bit Trie from MSB. At each level: prefer the OPPOSITE bit child (gives a 1-bit in XOR). If opposite doesn't exist, take same bit (0 in XOR). Result = all bits chosen.

**7.** How does the set-based Short Encoding work?
Add all words to a set. For each word: remove all non-empty suffixes from the set. Remaining words are not suffixes of any other → each needs its own `word#` slot. Return `sum(len(w) + 1)`.

**8.** What is the Palindrome Pairs split enumeration logic?
For each word[i], split at every position j: if `prefix` is palindrome and `reverse(suffix)` in HashMap → pair `(rev_idx, i)`. If `suffix` is palindrome and `reverse(prefix)` in HashMap → pair `(i, rev_idx)`. j > 0 guard on second case avoids duplicates.

**9.** How does the newsfeed compute trending topics?
Flink sliding window counts hashtag mentions (1-hour window). Min-heap of size K tracks top-K. Every 5 minutes: write top-K to Redis sorted set. Feed Service reads this for Explore/Discover tabs.

**10.** What is collaborative filtering for "Who to Follow"?
2-hop graph: find A's followees' followees (exclude already-followed). Rank by mutual follow count. Computed offline (Spark nightly batch); stored in Redis as pre-ranked suggestions.

### Day 80 — Hand of Straights, Camelcase, Skyline

**11.** How does Hand of Straights consume consecutive groups greedily?
Sort keys. For each smallest key with count > 0: take `count` groups starting there. Decrement `count[card], count[card+1], ..., count[card+groupSize-1]` by `count`. If any goes negative → return False.

**12.** How does Camelcase Matching's two-pointer work?
Pointer `p` on pattern; scan query character by character. `query[q] == pattern[p]` → advance both. `query[q]` is lowercase → allowed insertion, advance q only. `query[q]` is uppercase ≠ `pattern[p]` → return False. Match = `p == len(pattern)`.

**13.** How does The Skyline Problem handle lazy deletion on building ends?
Building ends push height to `removed` counter (not the heap). Before reading current max: clean heap top while `removed[-heap[0]] > 0` (pop and decrement). Current max = `-heap[0]`.

**14.** State the 5 questions for newsfeed system design.
(1) Fanout model? Hybrid (write for normal, read for celebrities). (2) Feed store? Redis sorted set. (3) Ranking? Chronological or ML score. (4) Notifications? Kafka + dedup + per-channel workers. (5) Scaling? Fanout workers + Redis cluster + Cassandra graph.

**15.** What is the trade-off of algorithmic vs chronological feed ranking?
Algorithmic: higher engagement, longer sessions, personalised. Risks: filter bubbles, ML complexity, opacity. Chronological: simple, transparent, no filter bubble — but favours high-frequency posters over quality.

### Day 81 — Merge Intervals, Non-Overlapping, Course Schedule III

**16.** How do you merge overlapping intervals?
Sort by start time. Maintain `merged` list. For each interval: if `curr.start <= merged[-1][1]` → overlap → `merged[-1][1] = max(merged[-1][1], curr.end)`. Else append as new interval.

**17.** How does the non-overlapping intervals greedy count minimum removals?
Sort by end time. Scan: if `curr.start < last_end` → overlaps → count as removed. Else update `last_end = curr.end` (keep this interval). Return count.

**18.** How does Course Schedule III use a max-heap for greedy course swapping?
Sort by deadline. For each course: add duration to total_time; push to max-heap. If `total_time > deadline`: evict the largest-duration course (heappop, add back negated value). Return `len(heap)`.

**19.** What is the role of a Kafka consumer group?
A consumer group is a set of consumers that collectively read a topic. Each partition is assigned to exactly one consumer in the group (load balancing). Different consumer groups read independently — enabling pub/sub fan-out.

**20.** What is a Kafka offset?
The sequential position of a message within a partition. Consumers commit their current offset after processing. On restart, they resume from the last committed offset — enabling at-least-once delivery without reading all historical messages.

### Day 82 — Jump Game, Jump Game II, Min Taps

**21.** How does Jump Game I determine reachability in O(n)?
Maintain `max_reach`. For each index i: if `i > max_reach` → False. Update `max_reach = max(max_reach, i + nums[i])`. Return True if loop completes.

**22.** How does Jump Game II minimise jumps using window expansion?
Maintain `current_end` and `farthest`. For each i < n-1: `farthest = max(farthest, i + nums[i])`. When `i == current_end`: jump++, `current_end = farthest`. Count = number of window boundary crossings.

**23.** How do you reduce Min Taps to Jump Game II?
Build `max_right[left]`: for tap i with range r, `max_right[max(0,i-r)] = max(..., min(n,i+r))`. Run Jump Game II on `max_right`. Return -1 if farthest == current_end at window boundary (stuck).

**24.** What does `acks=all` mean in Kafka, and when should you use it?
Producer waits for ACK from all ISR replicas before considering the write committed. No data loss even if the leader crashes (a replica takes over with the full data). Use when data loss is unacceptable (payments, audit logs). Trade-off: higher write latency.

**25.** What is the Kafka ISR (In-Sync Replicas)?
Set of follower replicas caught up within `replica.lag.time.max.ms`. Only ISR replicas are eligible to become leader on failover. `acks=all` waits for all ISR — guarantees no committed data loss.

### Day 83 — Gas Station, Queue Reconstruction, Max Profit Job Scheduling

**26.** How does Gas Station's single-pass find the start station?
Running `tank += gas[i] - cost[i]`. When `tank < 0`: set `start = i+1`, reset `tank = 0`. After loop: if `total_tank ≥ 0` → return start; else -1. No station between the previous start and i can be valid.

**27.** Why does Queue Reconstruction sort descending by height?
When inserting [h, k], all previously inserted people are ≥ h → they all count toward k. Later-inserted (shorter) people don't affect the count. Descending sort ensures this invariant holds at every insert.

**28.** What is the DP recurrence for Max Profit Job Scheduling?
Sort by end time. Build end_times array. `dp[i] = max(dp[i-1], profit[i] + dp[j])` where j = `bisect_right(end_times, start[i]) - 1` (last job ending ≤ start[i]). O(n log n).

**29.** What is the difference between queue and pub/sub messaging models?
Queue: one consumer receives each message (competing consumers for load balancing). Pub/Sub: ALL subscribers receive each message (fan-out). Kafka supports both: within a consumer group = queue; across consumer groups = pub/sub.

**30.** What is a Dead Letter Queue (DLQ) and why is it needed?
DLQ receives messages that fail processing after max retries. Without DLQ: a poison-pill message blocks the queue indefinitely. With DLQ: bad messages are quarantined; workers continue processing; engineering team investigates and replays.

### Day 84 — Subsets, Subsets II, Partition K Equal Subsets

**31.** How does the Subsets backtracking generate all 2^n subsets?
Collect `path[:]` at every recursion node (before any choice). Loop from `start` to end: append `nums[i]`, recurse with `start = i+1`, pop. Each path length from 0 to n is generated.

**32.** How does Subsets II skip duplicates at the same recursion level?
Sort `nums`. In the loop: `if i > start and nums[i] == nums[i-1]: continue`. `i > start` (not `i > 0`) limits the skip to the current depth level, allowing the same value at deeper levels (different element positions).

**33.** What are the two key optimisations for Partition K Equal Subsets backtracking?
(1) Sort `nums` descending: large numbers cause fast failures (bucket exceeds target early). (2) Symmetric bucket pruning: `seen` set of bucket sums — skip assigning to a bucket with the same sum as a previously-tried bucket at this level, avoiding identical subtrees.

**34.** When should you choose Kafka over RabbitMQ?
Choose Kafka when you need: (1) multiple independent consumer groups (pub/sub fan-out), (2) message replay / log retention for historical reprocessing, (3) very high throughput (millions/sec). Choose RabbitMQ for complex routing, traditional task queues, or request/reply (RPC) patterns.

**35.** State the general backtracking template.
`def backtrack(state)`: if complete → record. For each valid choice: apply → recurse → undo. Subsets/Combinations: loop from `start` (no reuse, ordered). Permutations: loop from 0, use `visited` set (all orderings).

---

## STAR Framework Consolidated Review — Week 12

### How to Use Your Week 12 STAR Stories in Interviews

This week covered advanced Tries and Heap synthesis (Days 78–80), Greedy patterns (Days 81–83), and Backtracking (Day 84). Each day built a full STAR story tied to the specific algorithm. Here is a consolidated guide to adapting them across LP question types.

**Pattern → LP Question Matrix**

| LP Question Type | Which STAR story to use | Key adaptation |
|-----------------|------------------------|----------------|
| "Tell me about a time you obsessed over the customer experience" | Day 79 (Recommendation system with interpretable 'why' labels — 62% pages/session increase) | Emphasise that the interpretability labels were as impactful as the algorithm — developer trust was the user need |
| "Tell me about a time you thought bigger than your immediate assignment" | Day 80 (Unified personalisation platform across 3 product lines — 136% documentation CTR improvement) | Lead with: "The ask was fix one product; I built a platform that served every product we'd ever build" |
| "Tell me about a time you simplified a process" | Day 81 (CI/release pipeline decoupling — 90% reduction in blocked deployments) | Frame the invention as decoupling CI test outcomes from release decisions; simplicity was the invention |
| "Tell me about a time you acted quickly under pressure" | Day 82 (At-least-once fix for payment event loss — 3 days vs 6 weeks, 0 event drops) | Emphasise: shipped targeted fix in 3 days, started proper migration in parallel — both things at once |
| "Tell me about a time you owned a problem outside your scope" | Day 83 (Kafka incident ownership across 3 teams — email/fulfillment restored in 10 min) | Be explicit: "This was not my system. I acted anyway." Leads naturally into the cross-team coordination story |
| "Tell me about a time you disagreed with your team and were right" | Day 84 (Kafka vs RabbitMQ decision — 6 engineer-weeks saved, 4 successful replays in 18 months) | Frame the decision matrix and 2-week empirical spike as the methodology; data won the debate |
| "Tell me about a data-driven decision" | Day 79 (A/B test: recommendations with vs without labels — 34% CTR lift from labels alone) | Lead with the split-test design; unexpected finding (labels as important as algorithm) was the key insight |

**5 Versatile STAR Stories from Week 12**

1. **Story 1 — Interpretable Recommendation System (Day 79):** "Built a hybrid content recommendation engine (co-occurrence + TF-IDF) for a 40,000-article developer documentation platform. Added interpretability labels ('developers also read this after...') that increased recommendation CTR by 34% independently of algorithm quality. Overall: pages-per-session from 1.3 to 2.1 (62% improvement), NPS for content discovery from 22 to 41."

2. **Story 2 — Unified Personalisation Platform (Day 80):** "Designed a three-layer personalisation service (Feature Store + FAISS retrieval + XGBoost ranking) shared across three product lines. Cross-product signal sharing drove 15% additional lift through cross-domain embeddings. Documentation recommendations: 11% → 26% CTR (136%). Decommissioned two independent codebases, saving 8 engineer-weeks/quarter."

3. **Story 3 — CI/Release Pipeline Decoupling (Day 81):** "Decoupled CI test outcomes from deployment release decisions using a Redis sorted-set release queue. Reduced blocked deployments from 4–6/day to 0.4/day (90% reduction), deployment frequency from 8/day to 14/day (75% improvement), pipeline debugging time from 45 min/day to under 5 min. Zero new infrastructure required."

4. **Story 4 — Kafka Consumer Group Incident (Day 83):** "Took ownership of a cross-team Kafka incident where a crashing inventory worker's rebalances disrupted email and fulfillment consumers for 3 hours. Isolated the faulty consumer group in 15 minutes; restored notifications in 10 minutes. 3,200 customers received proactive apology emails — CX ticket volume 40% lower than comparable prior incident. Proposed and implemented consumer-group isolation across all four teams within one sprint."

5. **Story 5 — Kafka vs RabbitMQ Decision (Day 84):** "Advocated for Kafka over team consensus on RabbitMQ for a 5-consumer event pipeline with 30-day replay requirements. Built a decision matrix; ran a 2-week empirical spike (250K msgs/sec, sub-5ms P99). Kafka's native log retention replaced an entire separate event store, saving approximately 6 engineer-weeks across four schema migration replays in 18 months. Decision matrix framework became a team standard."

---

## Patterns Mastered This Week
Rate yourself 1–5 after this review.

- [ ] Reversed-word Trie streaming (Stream of Characters) — LC 1032
- [ ] Binary XOR Trie greedy traversal — LC 421
- [ ] Max-heap lazy-deletion event sweep (Skyline) — LC 218
- [ ] Interval merge (sort by start) — LC 56
- [ ] Non-overlapping intervals (sort by end, activity selection) — LC 435
- [ ] Course Schedule III max-heap swap — LC 630
- [ ] Jump Game I greedy max_reach — LC 55
- [ ] Jump Game II window expansion — LC 45
- [ ] Max Profit Job Scheduling DP + binary search — LC 1235
- [ ] Subsets backtracking template — LC 78
- [ ] Subsets II duplicate pruning — LC 90
- [ ] Partition K Equal Subsets bucket backtracking — LC 698

---

## Problems Needing Drill
| Pattern | Problem | Retry date |
|---------|---------|------------|
| (learner fills in) | | |

## STAR Quick Reference — Week 12

| Pattern | One-line STAR hook |
|---------|-------------------|
| Reversed-word Trie streaming | "Given a character stream needing real-time word detection, applied reversed-word Trie → O(L) per character vs O(n·L) re-scan of all words per character." |
| Binary XOR Trie | "Given N numbers needing maximum XOR pair, applied 30-bit Trie greedy traversal → O(30n) vs O(n²) brute-force pair check." |
| Max-heap lazy-deletion event sweep | "Given building events needing real-time skyline, applied lazy deletion on max-heap → O(n log n) vs O(n²) rebuilding heap on each event end." |
| Interval merge (sort by start) | "Given overlapping deployment windows needing consolidation, applied sort + single-pass merge → O(n log n) vs O(n²) pairwise comparison." |
| Non-overlapping intervals (sort by end) | "Given conflicting meetings needing minimum cancellations, applied activity selection greedy → O(n log n) vs O(2^n) brute-force subset search." |
| Course Schedule III max-heap swap | "Given courses with deadlines needing maximum enrollment, applied greedy heap swap → O(n log n) vs O(2^n) subset DP." |
| Jump Game I/II greedy | "Given jump lengths needing reachability or minimum jumps, applied greedy max_reach → O(n) vs O(n²) DP." |
| Max Profit Job Scheduling DP + binary search | "Given non-overlapping jobs needing maximum profit schedule, applied DP + binary search for last compatible job → O(n log n) vs O(n²) DP." |
| Subsets backtracking | "Given N items needing all 2^N subsets, applied pruned backtracking → generate exactly what is needed vs iterative bitmask with same complexity but less readable." |
| Partition K Equal Subsets backtracking | "Given N numbers needing K equal-sum buckets, applied sort-desc + symmetric pruning backtracking → dramatically reduced search tree vs naive backtracking." |

**Career story titles:**
1. "CI/Release Pipeline Decoupling — Decoupled interval-style CI test outcomes from release decisions using Redis sorted-set queue, reducing blocked deployments 90% and increasing deployment frequency 75% at a SaaS company."
2. "Kafka vs RabbitMQ Decision — Advocated for Kafka over team consensus using a decision matrix and 2-week empirical spike, saving ~6 engineer-weeks across four schema migration replays in 18 months."
3. "Kafka Consumer Group Incident — Took cross-team ownership of a crashing consumer group that disrupted email and fulfillment for 3 hours; isolated fault in 15 minutes, restored notifications in 10, reducing CX tickets 40% vs a comparable prior incident."
