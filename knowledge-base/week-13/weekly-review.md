# Week 13 Review — Days 85–91
**Phase 1 → Phase 2 Transition | Months 3–4 | Slots 9–10**

---

## Week Span

**Days 85–90 (Slot 9 — Phase 1: DSA Mastery, Month 3):**  
Backtracking synthesis — Combinations, Combination Sum, Sudoku, Permutations, Palindrome Partitioning, N-Queens, Word Search, Letter Tiles, Expression Add Operators, Partition Labels, Remove K Digits, Candy, Max Events.  
System Design: Message queue patterns (at-least/exactly-once, Pub/Sub, CQRS, Outbox, DLQ).

**Day 91 (Slot 10 — Phase 2: Exposure & Assessment, Month 4):**  
1D DP introduction — Fibonacci variants, Decode Ways II, Student Attendance Record II.  
System Design: Chat Service high-level architecture.

---

## Phase Transition Notes

**Phase 2 daily structure (from Day 91 onward):**  
- 2h DSA (Medium–Hard)  
- 30 min System Design  
- 1h Contest / OA assessment  
- 30 min Behavioral

**Problem difficulty (Days 91–120):** 2 Hard + 1 revision per day.

---

## Patterns This Week

### DSA Patterns — Days 85–90 (Backtracking / Greedy)

**Combinations backtracking (Day 85 — LC 77):**  
Collect at depth k; loop from `start`; prune with `range(start, n - (k - len(path)) + 2)`.

**Combination Sum reuse (Day 85 — LC 39):**  
Sort candidates; recurse with same `i` (not `i+1`); break when `c > remaining`; base case `remaining == 0`.

**Sudoku constraint sets (Day 85 — LC 37):**  
`rows[r]`, `cols[c]`, `boxes[(r//3)*3 + c//3]`; try 1–9 per empty cell; backtrack on failure.

**Permutations visited array (Day 86 — LC 46):**  
Loop 0 to n; skip if `visited[i]`; collect at `len(path) == n`.

**Permutations II duplicate condition (Day 86 — LC 47):**  
Sort; skip if `i > 0 AND nums[i] == nums[i-1] AND NOT visited[i-1]`.

**Permutation Sequence factorial (Day 86 — LC 60):**  
0-index k; at each position: `idx = k // (n-1)!`; pick `digits[idx]`; remove; `k %= (n-1)!`.

**Palindrome Partitioning DP + backtrack (Day 87 — LC 131):**  
Precompute `is_pal[i][j]`; backtrack trying all palindrome prefixes from `start`.

**Generate Parentheses count-constrained (Day 87 — LC 22):**  
Two branches: add `(` if `open < n`; add `)` if `close < open`; collect at `len == 2n`.

**N-Queens row-by-row with sets (Day 87 — LC 51):**  
`cols`, `diag1 = r-c`, `diag2 = r+c`; place per row; backtrack column by column.

**Word Search grid DFS in-place (Day 88 — LC 79):**  
Mark `board[r][c] = '#'`; restore on backtrack; DFS from every cell matching `word[0]`.

**Letter Tile frequency counter (Day 88 — LC 1079):**  
`count` dict; for each char with count > 0: `result += 1`; recurse; restore count.

**Expression Add Operators prev_mult (Day 88 — LC 282):**  
Track `prev_mult`; on `*`: `eval = eval - prev_mult + prev_mult * curr`; `prev_mult = prev_mult * curr`.

**Partition Labels last-occurrence (Day 89 — LC 763):**  
Precompute `last[c]`; scan: `end = max(end, last[c])`; partition when `i == end`.

**Remove K Digits monotone stack (Day 89 — LC 402):**  
Increasing stack; pop when `stack[-1] > digit AND k > 0`; trim right if k > 0; strip leading zeros.

**Candy two-pass greedy (Day 89 — LC 135):**  
Left pass (ascending) + right pass (descending); `candies[i] = max(left[i], right[i])`.

**Monotone increasing digits (Day 90 — LC 738):**  
Right-to-left: when `digits[i] < digits[i-1]` → `digits[i-1]--`; mark = i. Fill 9s from mark.

**Min deletions unique freq (Day 90 — LC 1647):**  
Sort desc; `seen` set; reduce each freq while in seen; count reductions.

**Max events min-heap (Day 90 — LC 1353):**  
Sort by start; min-heap by end day; each day: add new events, remove expired, pop earliest-ending.

### DSA Patterns — Day 91 (1D DP Introduction)

**Fibonacci DP rolling vars (Day 91 — LC 70):**  
`a, b = 1, 2; a, b = b, a+b`. O(1) space; generalize to k-step with deque.

**Decode Ways II state streams (Day 91 — LC 639):**  
Track `e1` (ways ending in literal digit) and `s1` (ways ending in `*`) simultaneously. Transitions depend on the current and prior character type.

**Student Attendance II 6-state machine (Day 91 — LC 552):**  
State `dp[a][l]`: absences ∈ {0,1}, trailing lates ∈ {0,1,2}. At each step, branch on P (reset l), L (l < 2 → l+1), A (a == 0 → a+1, reset l).

---

## System Design Topics

**Message Queues (Days 85–90):**  
At-least-once vs. exactly-once delivery; Outbox Pattern (transactional outbox + CDC); CQRS for event sourcing; DLQ structure and replay; consumer group mechanics; backpressure via pull model; Kafka vs. SQS vs. RabbitMQ decision framework.

**Chat Service Architecture (Day 91):**  
WebSocket servers + Redis Pub/Sub for routing; Cassandra for message persistence (partition by conversation_id); fan-out on write vs. read; Notification Service for offline users; Snowflake IDs for sortable message ordering.

---

## Flashcard Deck — 35 Cards

### Day 85 — Combinations, Combination Sum, Sudoku Solver

**1.** How do Combinations and Subsets differ in backtracking?
Both loop from `start` and recurse with `start = i+1`. Subsets collects at every node. Combinations collects only when `len(path) == k`. Combinations also prunes: loop upper bound = `n - (k - len(path)) + 1` (0-indexed).

**2.** How does Combination Sum allow unlimited reuse of elements?
Recurse with `i` (same index) instead of `i+1`. Sort candidates to enable early break: `if c > remaining: break`. Base case: `remaining == 0` → collect.

**3.** How do you pre-initialise Sudoku constraint sets?
Scan the board once: for each filled cell `board[r][c] = d`, add `d` to `rows[r]`, `cols[c]`, `boxes[(r//3)*3 + c//3]`. Then backtrack with O(1) validation per digit.

**4.** What is the delivery model difference between at-least-once and exactly-once in Kafka?
At-least-once: commit offset after processing; crashes cause re-delivery. Exactly-once: Kafka transactions atomically commit offset + output; no duplicates. In practice: use at-least-once + idempotent consumer (UUID dedup in DB).

**5.** How do you implement idempotent message processing without Kafka transactions?
Each message has a unique ID. Before processing: `INSERT INTO processed_ids ON CONFLICT DO NOTHING`. If already in table → skip. If not → process + insert atomically. Effective exactly-once without transaction overhead.

### Day 86 — Permutations, Permutations II, Permutation Sequence

**6.** How does Permutations I backtracking differ from Combinations?
Permutations loops from 0 (any position) with `visited` array. Combinations loop from `start` (monotone). Permutations collects at `len(path) == n`; Combinations at `len == k`.

**7.** State the Permutations II duplicate skip condition and explain why it works.
Skip if `i > 0 AND nums[i] == nums[i-1] AND NOT visited[i-1]`. This means: same value appeared before us AND it's not in the current path → we already explored that subtree. Prevents duplicate permutations.

**8.** How does Permutation Sequence avoid generating all n! permutations?
0-index k. At each of n positions: `idx = k // (n-1)!`; pick `digits[idx]` and remove it; `k %= (n-1)!`. O(n²) — direct computation using the factorial number system.

**9.** What triggers a Kafka consumer group rebalance?
Consumer joins, leaves, or fails to send heartbeat within `session.timeout.ms`. During rebalance, consumption pauses. Minimise with: cooperative/incremental rebalancing or static membership.

**10.** What is consumer lag and how do you act on it?
Lag = `log_end_offset - committed_offset` per partition. Growing lag: consumer falling behind producer. Response: scale out consumer pods (bounded by partition count), tune `max.poll.records`, or investigate slow processing.

### Day 87 — Palindrome Partitioning, Generate Parentheses, N-Queens

**11.** How does the Palindrome Partitioning DP table enable O(1) palindrome checks?
`is_pal[i][j] = (s[i] == s[j]) AND (j-i ≤ 2 OR is_pal[i+1][j-1])`. Precomputed in O(n²); each check is O(1).

**12.** What are the two constraints in Generate Parentheses backtracking?
`open < n` → can add `(`. `close < open` → can add `)`. Together: every prefix is valid; every valid sequence of length 2n is reachable.

**13.** How do you track attacked diagonals in N-Queens without a 2D board scan?
`diag1` set for `r - c` (↘); `diag2` set for `r + c` (↗). Two queens on same diagonal iff they share `r - c` or `r + c`. O(1) lookup.

**14.** What is the Outbox Pattern and what dual-write problem does it solve?
Writes the event to an `outbox` table in the SAME DB transaction. A worker reads outbox and publishes to Kafka. Solves: DB write succeeds but Kafka publish fails → inconsistency.

**15.** What is CQRS and how does it relate to event-driven systems?
Separates write model (commands → event store) from read model (queries → materialized view). Events update projections; projections serve reads. Enables independent scaling of reads and writes.

### Day 88 — Word Search, Letter Tiles, Expression Add Operators

**16.** How does Word Search mark visited cells without an extra boolean grid?
Temporarily replace `board[r][c]` with `'#'` before recursing; restore after. The sentinel won't match any letter.

**17.** How does Letter Tile Possibilities avoid double-counting?
Uses a frequency Counter, not position indices. Branch on distinct characters; count `+1` before each recursive call because any non-empty sequence is valid.

**18.** What is the multiply-tracking trick in Expression Add Operators?
Track `prev_mult`. When appending `* curr`: `new_eval = eval - prev_mult + prev_mult * curr`. New `prev_mult = prev_mult * curr`.

**19.** What determines the maximum consumer parallelism for a Kafka topic?
The number of partitions. Each partition → at most one consumer in a group. Consumers > partitions = idle consumers.

**20.** What are four backpressure mechanisms in production messaging?
(1) Rate-limit producers at API gateway. (2) Consumer prefetch limit (`basic.qos`). (3) Circuit breaker. (4) Adaptive batching (`fetch.min.bytes`).

### Day 89 — Partition Labels, Remove K Digits, Candy

**21.** How does Partition Labels greedily extend partitions?
Precompute `last[c]`. Scan: `end = max(end, last[c])`. When `i == end`: seal partition, set `start = i + 1`.

**22.** How does Remove K Digits use a monotone stack?
Maintain increasing stack. For each digit: pop while `k > 0 AND stack[-1] > digit`. After scan: trim last k; strip leading zeros.

**23.** Why does Candy require two greedy passes?
Left pass satisfies ascending constraint. Right pass satisfies descending constraint. Valley child must satisfy both: `max(left[i], right[i])`.

**24.** What is the difference between push and pull messaging?
Push: broker delivers to consumer (low latency; consumer can be overwhelmed). Pull: consumer polls at own pace (natural backpressure). Kafka = pull. RabbitMQ = push.

**25.** What information must a DLQ message preserve?
Original payload, error reason + stack trace, retry count, first/last failure timestamps, original queue name.

### Day 90 — Monotone Digits, Min Deletions, Max Events

**26.** How does Monotone Increasing Digits work?
Right-to-left: when `digits[i] < digits[i-1]` → `digits[i-1]--`; mark = i. Fill positions from `mark` to end with '9'.

**27.** How does Min Deletions Make Frequencies Unique work?
Sort frequencies descending. `seen` set. For each freq: while `freq > 0 AND freq in seen` → `freq--; deletions++`. Add final non-zero freq to seen.

**28.** How does Maximum Events use a min-heap to maximise attendance?
Sort by start day. Each day: add all events starting today to min-heap (keyed by end). Remove expired. Pop earliest-ending. Greedy: earliest-ending leaves most future days open.

**29.** State the 6 questions for a message queue system design interview.
(1) Delivery guarantee? (2) Single or multiple consumer groups? (3) Need replay? (4) Failure handling (DLQ + retry)? (5) Backpressure: push or pull? (6) Scaling: partition count?

**30.** When should you choose Kafka vs SQS vs RabbitMQ?
Kafka: fan-out + replay + high throughput. SQS: simple managed FIFO, no replay. RabbitMQ: complex routing, RPC. Choose simplest tool that meets requirements.

### Day 91 — 1D DP Introduction

**31.** How do you detect a 1D DP problem vs. a greedy problem?
DP: future decisions interact; a wrong early choice can invalidate a better later one. Greedy: local optimal equals global optimal (prove by exchange argument).

**32.** What are the two state streams in Decode Ways II and why both?
`e1` = ways ending in literal digit; `s1` = ways ending in `*`. Separate because `*` and literal form different two-char pairs with the next character.

**33.** How do you reduce Fibonacci DP from O(n) space to O(1)?
Two rolling variables: `a, b = dp[i-2], dp[i-1]`. Each step: `a, b = b, a + b`. Works when dp[i] depends only on dp[i-1] and dp[i-2].

**34.** What are the 6 states in Student Attendance Record II?
`(absences, trailing_lates)` where absences ∈ {0,1} and trailing_lates ∈ {0,1,2}. Total = 6. Each step: P (reset l), L (if l<2: l+1), A (if a=0: a+1, reset l).

**35.** In Chat Service design, what does the Redis user-to-server mapping solve?
When user A (on server 1) sends to user B (on server 2), the API needs to know which server B's WebSocket is on. Redis `user_id → server_id` gives O(1) lookup.

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Combination / Subset backtracking | | |
| Permutations (with/without duplicates) | | |
| N-Queens / constraint-set tracking | | |
| Word Search in-place DFS | | |
| Expression Add Operators (prev_mult) | | |
| Greedy interval partitioning | | |
| Monotone stack (digits, remove k) | | |
| Max events greedy + min-heap | | |
| Fibonacci 1D DP (rolling vars) | | |
| State machine DP (Decode Ways II) | | |
| Multi-state DP (Attendance Record II) | | |
| Chat Service architecture | | |

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in) | | | |
