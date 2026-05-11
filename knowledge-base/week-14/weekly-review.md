# Week 14 Review — Days 92–98
**Phase 2: Exposure & Assessment | Month 4 | Slot 10 (Days 91–100)**

---

## Week Span

| Day | Topic | Key Pattern | System Design |
|-----|-------|-------------|---------------|
| 92 | House Robber Pattern | Skip-or-Take DP; circular; tree DP; binary search + greedy | Chat: Message Storage Schema |
| 93 | Jump Game DP | Reachability DP; greedy boundary; monotonic deque DP | Chat: WebSocket Architecture & Routing |
| 94 | Stock Trading State Machines | Multi-state DP (k transactions, cooldown, fee) | Chat: Group Messaging & Fan-out |
| 95 | Word Break & String Segmentation | 1D segmentation DP; memoized DFS enumeration | Chat: Message Delivery Guarantees |
| 96 | Palindrome DP & Interval DP intro | `is_pal` precomputation; min-cuts; LPS via LCS | Notification: Architecture Overview |
| 97 | Coin Change & Interval DP | Unbounded knapsack; greedy heap; interval DP (stick) | Notification: APNs / FCM deep dive |
| 98 | LIS & Job Scheduling DP | Patience sort O(n log n); 2D LIS; DP + binary search | Notification: Fan-out & dedup |

---

## Patterns Covered This Week

### 1. Skip-or-Take (House Robber Family)
- `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — compress to two variables
- **Circular:** run twice on `[0..n-2]` and `[1..n-1]`; take max
- **Tree:** post-order DFS returning `(rob_node, skip_node)` pair
- **Binary search variant (LC 2560):** binary search on `cap`; greedy feasibility O(n)

### 2. Jump Game Family
- LC 55 (feasibility): `max_reach = max(max_reach, i + nums[i])`; return False if `i > max_reach`
- LC 45 (min jumps): track `(jumps, current_end, farthest)`; increment jumps when `i == current_end`
- LC 1696 (max score, jump 1–k): `dp[i] = nums[i] + deque_max`; monotonic deque maintains sliding window max

### 3. Stock Trading State Machines
- **1 transaction (LC 121):** track `min_price`, update `max_profit = max(max_profit, p - min_price)`
- **Unlimited (LC 122):** sum positive consecutive differences
- **k=2 (LC 123):** 4 hardcoded states `buy1, sell1, buy2, sell2`; update in-order each day
- **k general (LC 188):** arrays `buy[t], sell[t]`; shortcut: if `k ≥ n//2`, use unlimited
- **Cooldown (LC 309):** 3 states `held, sold, rest`; simultaneous update
- **Fee (LC 714):** 2 states `hold, cash`; `cash = max(cash, hold + p - fee)`

### 4. String Segmentation DP
- **Word Break (LC 139):** `dp[i] = True if ∃j: dp[j] AND s[j:i] ∈ word_set`
- **Word Break II (LC 140):** memoized DFS from each valid position; return list of sentences
- **Concatenated Words (LC 472):** run Word Break per word with "exclude self, need ≥ 2 parts" constraint

### 5. Palindrome DP
- **Build `is_pal`:** `is_pal[i][j] = (s[i]==s[j]) AND (j-i<2 OR is_pal[i+1][j-1])`; fill `i` from n-1 down
- **Palindrome Partitioning II (LC 132):** `dp[i] = min(dp[j]+1)` where `is_pal[j][i-1]`
- **Min Insertions (LC 1312):** `n - LCS(s, reverse(s))` = `n - LPS(s)`; or interval DP

### 6. Unbounded Knapsack
- **Coin Change (LC 322):** `dp[i] = min(dp[i-c]+1)` for each coin ≤ i
- **Coin Change II (LC 518):** coins in outer loop, amounts inner → counts combinations not permutations
- **Greedy heap variant (LC 871):** max-heap of past fuel stations; retroactively refuel when empty

### 7. Interval DP
- **Stick Cut (LC 1547):** `dp[i][j] = min(dp[i][k] + dp[k][j] + cuts[j]-cuts[i])`; fill by length
- **Palindrome insertions (LC 1312):** `dp[i][j] = 1 + min(dp[i+1][j], dp[i][j-1])` when `s[i]≠s[j]`

### 8. LIS & Patience Sort
- **O(n²):** `dp[i] = max(dp[j]+1)` for all `j<i` where `nums[j] < nums[i]`
- **O(n log n):** `tails` array; `bisect_left(tails, n)` → if at end: append; else replace. Length = LIS length.
- **Russian Doll (LC 354):** sort by (width ASC, height DESC for equal widths); LIS on heights
- **Job Scheduling (LC 1235):** sort by end time; build non-decreasing `dp` list of `(end, profit)`; `bisect_right` for latest compatible job

---

## System Design Topics Covered

### Chat Service (Days 92–95)
- **Message schema (Day 92):** Cassandra `(conversation_id PK, message_id clustering)`; Snowflake IDs; inbox fan-out
- **WebSocket routing (Day 93):** WS server pool + Redis Pub/Sub for cross-server routing; heartbeat; ~50K connections per server
- **Group messaging (Day 94):** fan-out on write for small groups; fan-out on read for large groups (>1000 members); per-group sequence counter in Redis
- **Delivery receipts (Day 95):** SENT → DELIVERED → READ state machine; at-least-once delivery with UUID idempotency key; cursor-based pagination on `message_id`

### Notification System (Days 96–98)
- **Architecture (Day 96):** Kafka → Notification Worker → APNs/FCM adapter; user preference store; device token registry; delivery log
- **APNs/FCM (Day 97):** HTTP/2 batch sends; FCM multicast (500 tokens/call); token staleness handling (`Unregistered` → delete token); high vs. normal priority
- **Fan-out & dedup (Day 98):** Redis `SET NX EX` idempotency; in-app inbox fallback; thundering herd mitigation (rate limit + jitter)

---

## Weekly Flashcard Deck — 35 Cards

### Day 92 — House Robber Family

**1.** Write the House Robber recurrence from memory.
`dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Compress: `prev2, prev1 = prev1, max(prev1, prev2 + nums[i])`.

**2.** How does House Robber II (circular) reduce to House Robber I?
Run House Robber I on `nums[0..n-2]` and `nums[1..n-1]`. Return the max. First and last are never both picked in either subarray.

**3.** What does each tree node return in House Robber III?
`(rob_this, skip_this)`. `rob_this = node.val + left_skip + right_skip`. `skip_this = max(left_rob, left_skip) + max(right_rob, right_skip)`.

**4.** How does House Robber IV use binary search?
Binary search on `cap`. Feasibility: greedy scan counting houses with `value ≤ cap` that can be non-adjacently picked. If count ≥ k → achievable.

**5.** Why is a Snowflake ID better than UUID for chat message ordering?
Snowflake IDs embed a millisecond timestamp in the high bits → sortable by insertion order. UUIDs are random → no temporal ordering.

### Day 93 — Jump Game Family

**6.** How does Jump Game (LC 55) track reachability in O(n)?
Maintain `max_reach`. At each index `i`: if `i > max_reach` → False. Otherwise `max_reach = max(max_reach, i + nums[i])`. Return True.

**7.** What is the greedy insight in Jump Game II (LC 45)?
Track farthest reachable from all positions in current jump. When you exhaust the current jump's boundary, increment jump count and set new boundary = farthest. Always extend as far as possible per jump.

**8.** Write the recurrence for Jump Game VI.
`dp[i] = nums[i] + max(dp[i-k..i-1])`. Sliding window max maintained by monotonic deque of indices with decreasing dp values.

**9.** What is the monotonic deque invariant in Jump Game VI?
Front = index of maximum dp value in window `[i-k, i-1]`. Values non-increasing front to back. When adding `dp[i]`: pop from back all indices with smaller dp values.

**10.** In Chat Service, why use Redis Pub/Sub for WebSocket routing instead of a message queue?
Routing is ephemeral — we just need fast fan to the right server. Pub/Sub has microsecond latency and no storage overhead. Durability is handled separately by Cassandra.

### Day 94 — Stock Trading State Machines

**11.** What are the 4 DP states in Best Time to Buy and Sell Stock III?
`buy1` (best cost basis after 1st buy), `sell1` (max profit after 1st sell), `buy2` (max profit after 2nd buy = sell1 - price), `sell2` (final max profit). Updated in-order each day.

**12.** Why does Stock IV short-circuit when k ≥ n//2?
Maximum non-overlapping transactions in n days is n//2. k ≥ n//2 is effectively unlimited → use greedy (sum all positive consecutive differences).

**13.** Write the state transitions for Stock with Cooldown (LC 309).
`held = max(held, rest - p)` (buy if resting); `sold = held + p` (sell from held); `rest = max(rest, sold)` (enter rest or stay). All three update simultaneously.

**14.** What is the general state machine template for stock problems?
States: (holding/not, transactions_remaining). Transitions: buy (not→hold, transactions−1), sell (hold→not), rest (stay). `dp[t][h]` where t = transactions used, h = 0/1 holding.

**15.** In Chat group design, when do you switch from write fan-out to read fan-out?
At group size > ~1000 members. Write fan-out costs O(members) writes per message → write amplification at large scale. Read fan-out stores once; client polls with a cursor (last seen sequence ID).

### Day 95 — Word Break & String Segmentation

**16.** Write the Word Break DP recurrence and initialization.
`dp[0] = True`. For `i` in `1..n`: `dp[i] = True` if `∃j < i: dp[j] is True AND s[j:i] ∈ word_set`. O(n²) with set lookup.

**17.** What is the base case for Word Break II's memoized DFS?
`dfs(len(s))` returns `[""]` — one valid path (empty suffix). Callers prepend their word to each suffix.

**18.** How does Concatenated Words differ from Word Break?
The word being tested must be excluded from the dictionary and must use ≥ 2 sub-words. Run Word Break with those constraints for each word.

**19.** When would you use a Trie instead of a set for Word Break?
When words are long and you want to avoid slicing substrings O(L) per check. A Trie lets you scan character by character from position `i`, matching all words simultaneously.

**20.** In Chat, what is the difference between SENT, DELIVERED, and READ states?
SENT = server acknowledged receipt. DELIVERED = target device acknowledged download (double tick). READ = user opened conversation (blue tick). Each triggers a server write + WebSocket notification to sender.

### Day 96 — Palindrome DP

**21.** Write the is_pal DP recurrence (bottom-up).
`is_pal[i][j] = (s[i] == s[j]) AND (j - i < 2 OR is_pal[i+1][j-1])`. Fill `i` from `n-1` down to 0, `j` from `i` to `n-1`.

**22.** What is the recurrence for Palindrome Partitioning II?
`dp[i] = 0` if `is_pal[0][i-1]`. Otherwise `min(dp[j] + 1)` for all `j ∈ [1,i)` where `is_pal[j][i-1]`.

**23.** How does Minimum Insertion Steps reduce to LCS?
`min_insertions = n - LPS(s)` where LPS = LCS(s, reverse(s)). Every character not in LPS needs a mirrored insertion.

**24.** Write the interval DP recurrence for Minimum Insertions (LC 1312).
If `s[i] == s[j]`: `dp[i][j] = dp[i+1][j-1]`. Else: `dp[i][j] = 1 + min(dp[i+1][j], dp[i][j-1])`. Fill by increasing length. Base: `dp[i][i] = 0`.

**25.** In Notification System design, what happens when APNs returns an "Unregistered" error?
The push token is no longer valid. Delete this token from the Device Token Registry immediately to avoid wasting future sends.

### Day 97 — Coin Change & Interval DP

**26.** Write the Coin Change DP recurrence and initialization.
`dp[0] = 0`. For `i` in `1..amount`: `dp[i] = min(dp[i-c] + 1)` for each `coin c ≤ i`. Initialize all `dp[i] = inf`. Return `dp[amount]` if finite, else -1.

**27.** Why does Coin Change II put coins in the outer loop?
To count combinations: each coin considered once. If amounts were outer, you'd count [1,2] and [2,1] separately (permutations). Coins outer → each combination counted exactly once.

**28.** What is the greedy insight for Minimum Refueling Stops?
When you run out of fuel, retroactively refuel at the largest station you've passed. Max-heap of passed fuel stations gives optimal station in O(log n) per stop.

**29.** Write the interval DP recurrence for Minimum Cost to Cut a Stick.
Add sentinels 0 and n. `dp[i][j] = min(dp[i][k] + dp[k][j] + cuts[j] - cuts[i])` for each cut `k` in `(i,j)`. Fill by increasing interval length.

**30.** In APNs/FCM, when should you use a data message vs. a notification message?
Data: silent delivery; app processes and displays custom UI (for chat threading, dedup). Notification: OS displays automatically (for simple alerts). Use data for chat.

### Day 98 — LIS & Job Scheduling

**31.** Describe the patience sort (O(n log n) LIS) algorithm and its invariant.
For each number: `bisect_left` into `tails`; if it extends → append; else → replace. Invariant: `tails[k]` = smallest possible tail of any increasing subsequence of length `k+1`.

**32.** Why does Russian Doll Envelopes sort equal-width envelopes by height DESC?
Prevents selecting two same-width envelopes in the subsequence. With DESC heights, a later same-width element's height < earlier → it replaces in tails (doesn't extend the count).

**33.** Describe the Job Scheduling DP in two sentences.
Sort jobs by end time. `dp` = non-decreasing list of `(end, max_profit)`. For each job, `bisect_right` on end times to find the latest compatible job; append only when new profit exceeds current max.

**34.** What is the key invariant for `dp` in Job Scheduling binary search?
`dp` must be non-decreasing. If a new profit doesn't exceed `dp[-1]`, don't append — binary search relies on this monotonicity.

**35.** In Notification System design, how do you prevent duplicate sends after a worker crash?
Redis SET NX with TTL: `SET notif:{user_id}:{message_id} 1 NX EX 86400`. If key exists → already sent, skip. Atomic SET NX is race-condition-free.

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| House Robber (linear, circular, tree) | | |
| House Robber IV (binary search + greedy) | | |
| Jump Game family (greedy + deque) | | |
| Stock I–IV + cooldown + fee (all variants) | | |
| Word Break I + II + Concatenated Words | | |
| Palindrome DP (is_pal table + min cuts) | | |
| Min Insertions (LPS = LCS approach) | | |
| Coin Change (min + ways variants) | | |
| Refueling Stops (greedy heap) | | |
| Stick Cut (interval DP) | | |
| LIS (O(n²) DP + patience sort) | | |
| Russian Doll Envelopes | | |
| Job Scheduling (DP + binary search) | | |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | LeetCode warm-up (Day 92–93) | | | |
| | HackerRank OA sim (Day 94–97) | | | |
| | LeetCode mock (Day 98) | | | |

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in) | | | |
