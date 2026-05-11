# Month 4 Review — Phase 2: Exposure & Assessment (Days 91–120)
**Slots 10, 11, 12 | DSA: DP + Bit Manipulation | System Design: API Design, Caching, Consistent Hashing**

---

## Month Overview

| Slot | Days | Weeks | DSA Focus | System Design Focus |
|------|------|-------|-----------|---------------------|
| Slot 10 | 91–100 | 13–14 | DP Fundamentals | Chat Service, Notification System |
| Slot 11 | 101–110 | 15–16 | DP Intermediate (Knapsack, LCS, LIS) | API Design: REST vs GraphQL, Rate Limiting |
| Slot 12 | 111–120 | 16–18 | DP Advanced + Bit Manipulation | Consistent Hashing, LRU/LFU Cache |

**Daily structure maintained throughout:**
- 2h DSA (2 Hard + 1 revision)
- 30min System Design
- 1h Contest/OA mock
- 30min Behavioral

---

## All Patterns Mastered — Month 4

### Slot 10: DP Fundamentals (Days 91–100)

| Pattern | Representative Problem | Key Recurrence |
|---------|----------------------|----------------|
| 1D DP (Fibonacci variant) | Climbing Stairs (LC 70) | dp[i] = dp[i-1] + dp[i-2] |
| 1D DP (coin change) | Coin Change (LC 322) | dp[i] = min over coins c: dp[i-c] + 1 |
| 1D DP (path counting) | Unique Paths (LC 62) | dp[i][j] = dp[i-1][j] + dp[i][j-1] |
| 1D DP (jump) | Jump Game II (LC 45) | Greedy / dp[i] = min jumps to reach i |
| Decode ways | Decode Ways (LC 91) | dp[i] = dp[i-1] (1-digit) + dp[i-2] (2-digit if valid) |
| House Robber (linear) | House Robber (LC 198) | dp[i] = max(dp[i-2]+nums[i], dp[i-1]) |
| Interval DP (burst) | Burst Balloons (LC 312) | dp[l][r] = max over k: left[l][k]+right[k][r]+val |
| Interval DP (strange) | Strange Printer (LC 664) | dp[i][j] = min turns; try last printed char matching s[i] |
| Grid DP with constraint | Odd Even Jump (LC 975) | Build jump targets with sorted maps; dp[i][parity] |

### Slot 11: DP Intermediate (Days 101–110)

| Pattern | Representative Problem | Key Recurrence |
|---------|----------------------|----------------|
| 0/1 Knapsack | Partition Equal Subset Sum (LC 416) | dp[j] \|= dp[j-num], iterate j backwards |
| 2D Knapsack | Ones and Zeroes (LC 474) | dp[i][j] = max: iterate both dims backwards |
| Counting Knapsack | Profitable Schemes (LC 879) | dp[m][p] with capped minimum profit |
| Difference DP | Tallest Billboard (LC 956) | dp[diff] = max shorter rod at this difference |
| Grid 2-agent DP | Cherry Pickup II (LC 1463) | dp(row, c1, c2); 9 transitions per step |
| LCS classic | Longest Common Subsequence (LC 1143) | dp[i][j] = dp[i-1][j-1]+1 or max(skip) |
| Counting subsequences | Distinct Subsequences (LC 115) | dp[i][j] = dp[i-1][j] + (dp[i-1][j-1] if match) |
| Edit distance | Edit Distance (LC 72) | dp[i][j] = min(insert, delete, replace) |
| LIS O(n log n) | Longest Increasing Subsequence (LC 300) | Binary search on tails array |
| Number of LIS | Number of LIS (LC 673) | Maintain (dp[i], cnt[i]); careful accumulation |
| Multi-dim LIS | Max Height Cuboids (LC 1691) | Sort normalized; 3D containment check for dp |
| Hybrid DP | Longest String Chain (LC 1048) | dp[word] = max chain ending at word; check predecessors |
| Hybrid DP + binary search | Make Array Strictly Increasing (LC 1187) | State (index, replaced_with); bisect for next valid |

### Slot 12: DP Advanced + Bit Manipulation (Days 111–120)

| Pattern | Representative Problem | Key Recurrence |
|---------|----------------------|----------------|
| Tree DP 2-state | House Robber III (LC 337) | dp(node) = (rob_this, skip_this) tuple |
| Tree DP 3-state | Binary Tree Cameras (LC 968) | UNCOVERED/HAS_CAMERA/COVERED; greedy placement |
| Rerooting DP | Sum of Distances in Tree (LC 834) | 2-pass: dfs1 builds ans[0]; dfs2 reroots |
| Bitmask DP (position) | Beautiful Arrangement (LC 526) | popcount(mask) = current position |
| Bitmask partition | Partition to K Subsets (LC 698) | dp[mask] = partial sum mod target |
| Bitmask + subset enum | Min Work Sessions (LC 1986) | dp[mask\|sub] = dp[mask]+1; O(3^n) |
| TSP bitmask DP | Shortest Superstring (LC 943) | dp[mask][last] = min length; overlap precomputed |
| BFS + bitmask | Shortest Path Visiting All Nodes (LC 847) | BFS state (node, mask) |
| Digit DP | Non-consecutive Ones (LC 600) | dp(pos, prev_one, tight) |
| Digit counting | Numbers At Most N (LC 902) | Digit-by-digit; for-else pattern |
| Score-based game DP | Stone Game III (LC 1406) | dp[i] = max(Σstones[i..i+k-1] - dp[i+k]) |
| Boolean game DP | Stone Game IV (LC 1510) | dp[i]=True if any move to dp[i-j²]=False |
| XOR + Trie | Maximum XOR (LC 421) | Greedy: take opposite bit MSB→LSB |
| Offline XOR Trie | Max XOR With Element (LC 1707) | Sort by limit; incrementally insert; query |
| AND subarray monotone | Mysterious Function (LC 1521) | O(log V) distinct AND values per right endpoint |
| XOR range counting | Count XOR Pairs (LC 1803) | Trie countLessThan(high+1) - countLessThan(low) |
| XOR parity bitmask | Longest Awesome Substring (LC 1542) | Prefix XOR parity; palindrome if mask is 0 or power of 2 |
| Forward probability DP | Knight Probability (LC 688) | dp[r][c] distribute 1/8 to valid neighbors |
| Sliding window probability | New 21 Game (LC 837) | dp[i] = window_sum/maxPts; slide window |

---

## System Design Topics Covered — Month 4

### Chat Service (Slot 10)
- WebSocket vs. Long Polling vs. SSE
- Message ordering with logical clocks
- Fanout: write-time vs. read-time for group messages
- Message persistence: hot path (Redis pub/sub) vs. cold storage (Cassandra)

### Notification System (Slot 10)
- Push vs. pull delivery models
- APNs / FCM integration
- Notification deduplication with idempotency keys
- Rate limiting per user per channel

### API Design: REST vs. GraphQL (Slot 11)
- REST: resource-oriented, HTTP semantics, easy caching, versioning strategies
- GraphQL: flexible queries, single endpoint, no over-fetching, schema stitching
- Rate limiting: token bucket (Redis Lua), sliding window, leaky bucket
- APQ (Automatic Persisted Queries) for bandwidth optimization
- GraphQL Federation with @key directive

### Consistent Hashing (Slot 12)
- Hash ring [0, 2^32) with virtual nodes (V=150 for <5% variance)
- O(log nV) get_server with bisect; O(K/n) keys remapped on node change
- Redis Cluster: 16384 hash slots, gossip protocol, MOVED redirect
- Sentinel vs. Cluster: HA modes, replication lag, WAIT command

### LRU/LFU Cache Design (Slot 12)
- LRU: HashMap + doubly-linked-list (dummy sentinels); O(1) get/put
- LFU: 3 hashmaps (key→val, key→freq, freq→OrderedDict); O(1) get/put
- ARC: adaptive — T1/T2/B1/B2 lists; balances recency vs. frequency
- Cache invalidation: write-through, write-back, write-around, cache-aside, CDC
- Thundering herd: mutex lock, probabilistic early expiration (XFetch), stale-while-revalidate
- Redis eviction policies: allkeys-lru, allkeys-lfu, volatile-lru; approximated sampling
- Hotspot keys: local in-process cache (Caffeine) or key replication with N suffixes

---

## OA / Mock Statistics

Fill in after completing each session:

| Date | Type | Platform / Partner | Problems | Solved | Time Used | Notes |
|------|------|--------------------|----------|--------|-----------|-------|
| | LeetCode Weekly Contest | leetcode.com | 4 | | 90 min | |
| | LeetCode Biweekly Contest | leetcode.com | 4 | | 90 min | |
| | FAANG OA Simulation | pramp.com | 2 | | 60 min | |
| | System Design Mock | study partner | 1 | | 45 min | |
| | Behavioral Mock | study partner | STAR stories | | 30 min | |

---

## Month Flashcard Deck Links

All flashcard decks are included in the weekly review files:

| Week | File | Cards |
|------|------|-------|
| Week 13 | `/knowledge-base/week-13/weekly-review.md` | 35 cards (Days 91–97) |
| Week 14 | `/knowledge-base/week-14/weekly-review.md` | 35 cards (Days 98–104) |
| Week 15 | `/knowledge-base/week-15/weekly-review.md` | 35 cards (Days 98–104 continued / 105) |
| Week 16 | `/knowledge-base/week-16/weekly-review.md` | 35 cards (Days 106–112) |
| Week 17 | `/knowledge-base/week-17/weekly-review.md` | 35 cards (Days 113–119) |

**Quick-access: Month 4 highest-priority cards** (restudy these before Phase 3):

| Q | A |
|---|---|
| Rerooting formula (LC 834) | ans[child] = ans[parent] - count[child] + (n - count[child]) |
| Bitmask subset enumeration | sub = mask; while sub > 0: ...; sub = (sub-1) & mask; O(3^n) total |
| TSP DP transition (LC 943) | dp[mask\|(1<<nxt)][nxt] = dp[mask][last] + len(nxt) - overlap[last][nxt] |
| Digit DP tight flag | tight=True → digit ≤ limit[pos]; tight=False → digit free 0..9 |
| Game Theory recurrence | dp[i] = max over moves k of: (gain) - dp[next_state] |
| XOR Trie query | At each bit MSB→LSB: want = 1-bit; if want in trie → xor_val \|= (1<<i); else take same bit |
| LFU _update() | remove from freq_keys[freq]; if empty and freq==min_freq: min_freq++; add to freq_keys[freq+1] |
| New 21 Game window | window_sum += dp[i] when i<k; window_sum -= dp[i-maxPts] when i>=maxPts |
| Consistent hashing keys remapped | K/n keys remapped when 1 server added/removed; only successor affected |
| Cache invalidation CDC | DB → Debezium (reads binlog) → Kafka → Consumer → Redis DEL key |

---

## Strength / Gap Summary — Month 4

After completing Day 120's pattern index recall test, fill in your assessment:

**Fully confident patterns (✓):**
(fill in from recall test)

**Partial recall patterns (~) — Phase 3 warm-up priority:**
(fill in from recall test)

**Blank patterns (✗) — Phase 3 remediation required:**
(fill in from recall test)

---

## Month 4 Contest / Problem Count

| Category | Target | Achieved | Notes |
|----------|--------|----------|-------|
| LeetCode Hard solved | 60 (2/day × 30 days) | | |
| LeetCode Medium revisions | 30 (1/day × 30 days) | | |
| System Design topics | 12 | | |
| Behavioral STAR stories prepared | 10 | | |
| Weekly contests participated | 4 | | |
| Mock interviews completed | 4 | | |

---

## Next Month Goals — Phase 3: Mock Interviews (Month 5, Days 121–150)

### Phase 3 Objectives

**Goal:** Transition from "can I solve this?" (Phase 2) to "can I solve this in an interview?" (Phase 3).

Phase 3 problems are the same patterns as Phase 2, but now the constraints are:
- 20–25 minutes per problem, not 2 hours
- Thinking out loud is required, not optional
- Communication of trade-offs is evaluated, not just correctness
- Follow-up questions will probe your understanding

### Month 5 Daily Structure

| Block | Duration | Activity |
|-------|----------|----------|
| Mock interview | 45 min | Timed problem (no hints); think out loud; full solution + complexity analysis |
| System design mock | 30 min | Full design with interruptions and pivots |
| Behavioral | 20 min | Timed STAR delivery (2 min max per story); refine delivery |
| Remediation | 25 min | Re-implement weak patterns from Phase 2 |

### Top 5 Habits to Build in Phase 3

1. **State the pattern out loud before coding.** "This looks like bitmask DP where the state is the visited set." Say it even when obvious — it signals pattern recognition to the interviewer.

2. **Write examples before writing code.** On any problem, take 2 minutes to write 2 small examples with expected output. This catches edge cases before you're 20 lines into a solution.

3. **Complexity upfront, not afterthought.** After stating your approach: "This will be O(2^n × n) time and O(2^n × n) space — is that acceptable given n ≤ 20?" Get alignment before implementing.

4. **Narrate bottlenecks, not syntax.** Don't say "I'm writing a for loop." Say "I'm iterating over all masks and for each mask, trying all unvisited nodes as the next step — this is the O(2^n × n²) inner loop."

5. **Land the edge cases explicitly.** Before running examples: "Edge cases I'm handling: empty input, single element, k=0." Saying it out loud shows thoroughness even if the code handles it automatically.

### Phase 3 Milestone Targets

| Week | Target |
|------|--------|
| Week 18–19 | 2 full mock interviews per week with a partner; debrief each one |
| Week 20–21 | Apply to target companies; time all practice sessions strictly |
| Week 22 | FAANG take-home OAs; simulate no-search-engine constraints |
| Week 22–23 | On-site interview simulation (4 rounds back to back) |

### Phase 3 Problem List (Remediation + New)

From Phase 2 weak spots (fill in your ✗ patterns):
- [ ] Re-implement every ✗ pattern 3 times in 3 days without looking at notes
- [ ] Add each ✗ problem to spaced repetition: day 1, day 3, day 7, day 14

New patterns to introduce in Phase 3:
- [ ] Segment tree with lazy propagation (range update, range query)
- [ ] Suffix array + LCP array
- [ ] Heavy-Light Decomposition
- [ ] Centroid Decomposition (intro)
- [ ] Network Flow (max flow, min cut)
- [ ] Monotonic stack / queue in advanced contexts

---

## Month 4 Final Reflection

Complete this before moving to Month 5:

**What went well:**
(your notes)

**What was harder than expected:**
(your notes)

**One habit I'm carrying into Phase 3:**
(your notes)

**One technical concept I want to deepen in Phase 3:**
(your notes)

**My three weakest patterns from Month 4 (be specific):**
1.
2.
3.
