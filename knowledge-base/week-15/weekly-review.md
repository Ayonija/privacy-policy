# Week 15 Review — Days 99–105
**Phase 2: Exposure & Assessment | Month 4 | Slots 10–11 (Days 91–110)**

---

## Week Span

| Day | Topic | Key Pattern | System Design |
|-----|-------|-------------|---------------|
| 99 | Odd Even Jump & Misc Hard DP | Dual-state DP + monotonic stack; Interval DP (Strange Printer) | Chat + Notification end-to-end flow |
| 100 | Slot 10 Review & Assessment | Full 1D DP recall; K Inverse Pairs (prefix sum DP); LC 1269 Stay in Place | Chat Service full mock design |
| 101 | 0/1 Knapsack Classic + 2D | dp[w] backwards; 2D (LC 474); profit counting (LC 879) | REST vs GraphQL intro |
| 102 | Knapsack Partition + Counting | Difference DP (LC 956); complement counting (LC 2518); Target Sum | REST CRUD, idempotency, pagination |
| 103 | Grid DP / Two-Agent | Cherry Pickup I (c2 derived); Cherry Pickup II (row-by-row 9-way) | GraphQL N+1 and DataLoader |
| 104 | LCS Foundation | LCS recurrence; Distinct Subsequences (count); Wildcard Matching | Rate Limiting: Token Bucket, Leaky Bucket |
| 105 | LCS Advanced | Regex Matching (`*` transitions); Edit Distance (3 ops); Scramble String | Rate Limiting: Sliding Window, Distributed |

---

## Patterns Covered This Week

### 1. Dual-State 1D DP with Ordered Jump Targets (Day 99)
- `next_odd[i]` / `next_even[i]` precomputed via sorted order + monotonic stack
- `odd[i] = even[next_odd[i]]`; `even[i] = odd[next_even[i]]` — alternating state propagation
- Count `sum(odd)` — indices where an odd-jump first move can reach the end

### 2. Interval DP — Strange Printer (Day 99)
- `dp[i][j] = dp[i][j-1] + 1` (baseline)
- Optimize: for each `k` where `s[k] == s[j]`: `dp[i][j] = min(dp[i][k] + dp[k+1][j-1])`
- Fill by increasing interval length; O(n³)

### 3. Sliding Window Prefix Sum DP (Day 100)
- K Inverse Pairs (LC 629): `dp[n][k] = sum(dp[n-1][k-i] for i in 0..min(n-1,k))`
- Accelerated by prefix sums: `dp_new[j] = prefix[j+1] - prefix[max(0, j-n+1)]` → O(nk)
- Boundary DP (LC 1269): `max_pos = min(steps//2, arrLen-1)` bounds reachable positions

### 4. 0/1 Knapsack (Days 101–103)
- **Classic:** `dp[j] = max(dp[j], dp[j-w]+v)` iterating j from W down to w
- **2D (LC 474):** `dp[i][j] = max(dp[i][j], dp[i-zeros][j-ones]+1)` both dimensions backwards
- **Counting (LC 879):** `dp[members][prof] += dp[members-g][prof]` cap profit at minProfit
- **Difference minimization (LC 956):** `dp[diff] = max shorter rod when |h1-h2|=diff`
- **Complement counting (LC 2518):** `answer = 2^n - 2 × cnt_bad`; cnt_bad = subsets with sum < k
- **Two-agent grid (LC 741, 1463):** O(n³) memoized; c2 derived from step count in 741

### 5. LCS — String Alignment DP (Days 104–105)
- **Classic (LC 1143):** `dp[i][j] = dp[i-1][j-1]+1` if match, else `max(dp[i-1][j], dp[i][j-1])`
- **Distinct Subsequences (LC 115):** `dp[i][j] = dp[i-1][j] + (dp[i-1][j-1] if match)`  — ADDS not max
- **Wildcard Matching (LC 44):** `*` matches any sequence; `dp[i][j] = dp[i][j-1] OR dp[i-1][j]`
- **Regex Matching (LC 10):** `a*` = zero-or-more-of-a; zero: `dp[i][j-2]`; more: `dp[i-1][j]` if char matches
- **Edit Distance (LC 72):** `1 + min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])` on mismatch
- **Scramble String (LC 87):** Memoized recursion; 2 cases (no-swap, swap) at each split; anagram pruning

---

## System Design Topics Covered

### REST vs GraphQL (Days 101–102)
- REST: resource-oriented, fixed endpoints, HTTP verb semantics, native HTTP caching
- GraphQL: single endpoint, client-driven queries, eliminates over/under-fetching, additive schema evolution
- REST idempotency: GET/PUT/DELETE idempotent; POST uses Idempotency-Key header
- Pagination: offset (simple, drifts), cursor (stable, real-time feeds), keyset (performant, sortable)

### GraphQL Deep Dive (Day 103)
- N+1 problem: N resolver calls for N parent objects → N+1 DB queries
- DataLoader: batches all `.load(key)` calls in one event loop tick → 2 total queries regardless of N
- DataLoader must be per-request to prevent cross-user data leakage
- Schema evolution: deprecate fields, never break — additive changes only

### Rate Limiting (Days 104–105)
- Token bucket: allows burst up to capacity, refills at rate r/sec; O(1) check; best for API rate limiting
- Leaky bucket: constant output rate, no burst; use for traffic shaping downstream
- Fixed window: simple but boundary burst (2× limit exploit)
- Sliding window log: exact, O(log n) per request, memory-intensive (sorted set)
- Sliding window counter: approximate (±1%), O(1); two-window interpolation
- Distributed: Redis for shared state; Lua scripts for atomic check-and-decrement

---

## Weekly Flashcard Deck — 35 Cards

### Day 99 — Odd Even Jump & Strange Printer

**1.** Describe the two-step approach to Odd Even Jump (LC 975).
Step 1: Precompute `next_odd[i]` and `next_even[i]` — sort by (value ASC, idx ASC) and (value DESC, idx ASC) respectively; use monotonic stack to find next index in sorted order to the right. Step 2: DP from right to left — `odd[i] = even[next_odd[i]]`; `even[i] = odd[next_even[i]]`. Count `sum(odd)`.

**2.** In Strange Printer (LC 664), when does `s[k]==s[j]` save a print turn?
The turn that prints s[k] can extend to cover s[j] in one stroke. Cost: `dp[i][k] + dp[k+1][j-1]` — handle s[i..k] and overprint the middle s[k+1..j-1].

**3.** What optimization makes LC 1269 (Stay in Same Place) tractable?
`max_pos = min(steps//2, arrLen-1)`. You can't be more than steps//2 from index 0 and still return in remaining steps.

**4.** How does prefix sum accelerate K Inverse Pairs Array (LC 629)?
`dp_new[j] = prefix[j+1] - prefix[max(0, j-length+1)]` replaces O(n) summation with O(1) lookup. Without: O(n²k). With: O(nk).

**5.** In Odd Even Jump, why is `make_next` called with different sort orders?
Odd jump goes to smallest value ≥ current → sort by (value ASC, idx ASC). Even jump goes to largest value ≤ current → sort by (value DESC, idx ASC). In each order, a monotonic stack finds the next index to the right.

### Day 100 — Slot 10 Assessment

**6.** What is the 3-pointer trick in Ugly Number II (LC 264)?
Maintain i2, i3, i5 into the ugly[] array. Each step: `min(ugly[i2]*2, ugly[i3]*3, ugly[i5]*5)`. Advance ALL tied pointers to prevent duplicates (6 = 2×3 = 3×2).

**7.** State the two critical storage systems in Chat Service and why separated.
Cassandra: durable message store (conversation history, survives crashes). Redis: ephemeral routing + presence (user-to-server map, online status, dedup keys). Separated because ephemeral data needs different TTL, eviction, and failure semantics.

### Day 101 — 0/1 Knapsack Classic

**8.** Write the 0/1 Knapsack recurrence and why weights iterate backwards.
`dp[j] = max(dp[j], dp[j-w]+v)` for j from W down to w. Backwards: dp[j-w] still reflects pre-item state → each item used at most once.

**9.** How does Partition Equal Subset Sum reduce to 0/1 Knapsack?
Target = total//2. `dp[j] |= dp[j-num]` backwards. Answer = dp[target]. Reject if total is odd.

**10.** How does LC 474 (Ones and Zeroes) add a second knapsack dimension?
Each string costs (zeros, ones). Capacity = (m, n). `dp[i][j] = max(dp[i][j], dp[i-zeros][j-ones]+1)`. Both dimensions iterate backwards.

**11.** In Profitable Schemes (LC 879), why cap profit at minProfit?
Any profit ≥ minProfit is equivalent — we don't distinguish levels above the threshold. Capping collapses 3D DP to 2D, keeping array size at (n+1)×(minProfit+1).

**12.** In REST, which HTTP methods are idempotent? Which are safe?
Idempotent: GET, PUT, DELETE, HEAD. Safe (no side effects): GET, HEAD. POST is neither; PATCH is idempotent in theory but often not in practice.

### Day 102 — Knapsack Partition

**13.** In Tallest Billboard (LC 956), what does dp[d] represent?
Maximum height of the shorter support when the two supports differ in height by exactly d. Answer = dp[0] — when diff=0, the shorter height equals the common height.

**14.** Describe the three dp[d] transitions for a rod of height h.
Skip (unchanged). Add to taller: dp[d+h] = max(dp[d+h], dp[d]). Add to shorter: if h≤d: dp[d-h] = max(..., dp[d]+h); else: dp[h-d] = max(..., dp[d]+d).

**15.** In Number of Great Partitions (LC 2518), why is the formula `2^n − 2×cnt_bad`?
cnt_bad = subsets with sum < k. Each is a "bad" group 1. By symmetry, an equal count are bad as group 2. When total ≥ 2k they're disjoint → subtract 2×cnt_bad.

**16.** How does Target Sum reduce to subset-sum counting?
P−N=target, P+N=total → P=(total+target)//2. Count subsets summing to P using counting knapsack dp[j]+=dp[j-num]. Guard: odd sum or |target|>total → return 0.

**17.** What are 3 REST pagination strategies and when to use cursor-based?
Offset (simple, drifts), cursor (stable, encodes last-seen ID), keyset (fast, WHERE id>last). Use cursor for time-ordered feeds where new inserts cause offset drift.

### Day 103 — Grid DP / Two-Agent

**18.** In Cherry Pickup (LC 741), how does two-agent forward DP equal a round trip?
Return trip reversed = second forward path. Both go (0,0)→(n-1,n-1) simultaneously; don't double-count shared cells.

**19.** How do you derive c2 in Cherry Pickup (LC 741)?
Both agents take same steps: r1+c1=r2+c2. So c2=r1+c1-r2. Reduces 4D state to 3D (r1,c1,r2).

**20.** What are the 9 transitions in Cherry Pickup II?
Each robot moves left/stay/right independently: 3×3=9 combinations. Filter out-of-bounds before recursing.

**21.** How does Last Stone Weight II reduce to knapsack?
Minimize |S1-S2|=total-2j. Find max j ≤ total//2 achievable by a subset. Standard 0/1 boolean knapsack.

**22.** What is the DataLoader pattern and what problem does it solve?
DataLoader batches all `.load(key)` calls within one event loop tick into one batch function call. Solves GraphQL N+1: instead of N queries for N parent objects, makes 1 batch query.

### Day 104 — LCS Foundation

**23.** Write the LCS recurrence.
Match: dp[i][j]=dp[i-1][j-1]+1. Mismatch: dp[i][j]=max(dp[i-1][j], dp[i][j-1]). Base: dp[i][0]=dp[0][j]=0.

**24.** In Distinct Subsequences (LC 115), how does the recurrence differ from LCS?
Always: dp[i][j]=dp[i-1][j] (skip s[i-1]). On match: dp[i][j]+=dp[i-1][j-1] (also use s[i-1]). ADDS instead of max — counting all distinct ways, not finding best single choice.

**25.** In Wildcard Matching (LC 44), what are the two `*` transitions?
1) dp[i][j]=dp[i][j-1] — `*` matches zero chars (advance p only). 2) dp[i][j]|=dp[i-1][j] — `*` matches one more char (advance s).

**26.** How do you initialize dp[0][j] in Wildcard Matching?
dp[0][j]=dp[0][j-1] only if p[j-1]=='*'. Empty s matches only patterns that are all-*.

**27.** Explain the token bucket rate limiting algorithm.
Store (tokens, last_refill_time) per user. On request: compute new_tokens=(now-last)*rate, cap at capacity; if tokens≥1 → allow and decrement; else reject. Use Redis Lua for atomicity.

### Day 105 — LCS Advanced

**28.** Write the two transitions for `*` in Regular Expression Matching (LC 10).
Zero occurrences: dp[i][j]=dp[i][j-2]. One or more (only if p[j-2]=='.' or p[j-2]==s[i-1]): dp[i][j]|=dp[i-1][j].

**29.** How does Edit Distance (LC 72) differ from LCS in handling a mismatch?
LCS mismatch: max(dp[i-1][j], dp[i][j-1]) — choose best skip. Edit mismatch: 1+min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1]) — pay 1 for replace/delete/insert.

**30.** In Scramble String (LC 87), what are the two recursive cases at split i?
No-swap: dp(a[:i],b[:i]) AND dp(a[i:],b[i:]). Swap: dp(a[:i],b[n-i:]) AND dp(a[i:],b[:n-i]).

**31.** Why does Scramble String check `sorted(a)==sorted(b)` before recursing?
Necessary condition: scrambling preserves the multiset of characters (just reorders). If sorted forms differ, a can never become b — prune the branch immediately.

**32.** What is the boundary burst problem in fixed-window rate limiting?
User sends `limit` requests just before window end + `limit` just after window start = 2× limit in a short burst. Both windows reset independently, so neither triggers rejection.

**33.** Compare sliding window log vs. sliding window counter for rate limiting.
Log: exact, stores all request timestamps in sorted set, O(log n) per request, high memory. Counter: approximate (≈1% error), stores only 2 integers, O(1) per request. Counter is preferred at scale.

**34.** In LC 10 vs LC 44, how does `*` behavior differ?
LC 44 (`*` alone): matches any sequence of any character — no preceding char dependency. LC 10 (`a*`): `*` quantifies the preceding char p[j-2] specifically — zero or more of p[j-2].

**35.** State 3 knapsack variants and their core DP direction rule.
0/1 knapsack: iterate weights HIGH→LOW (each item once). Unbounded knapsack: iterate weights LOW→HIGH (reuse allowed). 2D knapsack: iterate both dimensions HIGH→LOW.

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Odd Even Jump (dual-state DP) | | |
| 0/1 Knapsack (classic 1D) | | |
| 0/1 Knapsack (2D, LC 474) | | |
| Counting knapsack (Target Sum, LC 879) | | |
| Difference DP (Tallest Billboard) | | |
| Two-agent grid DP (Cherry Pickup I & II) | | |
| LCS classic + space optimization | | |
| Distinct Subsequences (count variant) | | |
| Wildcard Matching (LC 44) | | |
| Regular Expression Matching (LC 10) | | |
| Edit Distance (LC 72) | | |
| Scramble String (memoized interval DP) | | |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | LeetCode (Day 101–102) | | | |
| | HackerRank OA sim (Day 102–103) | | | |
| | LeetCode mock (Day 104–105) | | | |

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in) | | | |
