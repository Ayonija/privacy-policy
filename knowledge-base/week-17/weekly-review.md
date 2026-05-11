# Week 17 Weekly Review — Days 113–119
**Phase 2: Exposure & Assessment | Month 4**

## Week Summary

| Day | Topic | DSA Patterns | System Design |
|-----|-------|-------------|---------------|
| 113 | Bitmask DP: TSP Variants | Shortest Path Visiting All Nodes (BFS+mask), Shortest Superstring (TSP DP + overlap), Distribute Integers (0/1 bitmask + subset enum) | Consistent Hashing advanced: Redis Cluster vs. consistent hash ring, gossip protocol |
| 114 | Digit DP | Count Unique Digits (combinatorics + DP), Non-consecutive Ones (binary digit DP), Numbers At Most N (digit-by-digit counting) | LFU Cache: 3 hashmaps, min_freq, OrderedDict tiebreaking |
| 115 | Game Theory DP | Predict the Winner (interval, score-based), Stone Game III (linear, score-based), Stone Game IV (boolean win/lose, P/N positions) | Cache Invalidation: write-through, write-back, write-around, cache-aside, event-driven CDC |
| 116 | XOR + Binary Trie | Single Number (XOR cancel), Maximum XOR (Trie greedy), Maximum XOR With Element (offline + Trie) | Redis Sentinel vs Cluster: quorum, replication lag, WAIT command |
| 117 | Bit Counting + AND Subarrays | Counting Bits (2 recurrences), Mysterious Function (monotone AND set), Count XOR Pairs in Range (Trie countLessThan) | Cache warming strategies; thundering herd problem + solutions |
| 118 | Subset Enumeration + XOR Parity | Max Product Word Lengths (26-bit mask), Longest Awesome Substring (parity prefix XOR), Strong Pair XOR (sliding window Trie) | Cache sizing, eviction policy selection (LRU/LFU/ARC/CLOCK), hotspot key |
| 119 | Probability DP | Knight Probability (forward probability DP), New 21 Game (sliding window prob), Two Boxes Same Distinct Balls (combinatorics + DP) | LRU + LFU integration: Redis approximated LRU, allkeys-lru vs allkeys-lfu |

---

## 35-Card Flashcard Deck (5 per day × 7 days)

### Day 113 — Bitmask DP: TSP Variants

| # | Q | A |
|---|---|---|
| 113.1 | In Find Shortest Superstring (LC 943), what does overlap[i][j] mean and how is dp[mask][last] defined? | overlap[i][j] = length of longest suffix of words[i] matching a prefix of words[j]. dp[mask][last] = minimum total superstring length using exactly the words in mask, ending with word last. Transition: dp[mask\|(1<<nxt)][nxt] = dp[mask][last] + len(words[nxt]) - overlap[last][nxt]. |
| 113.2 | In Shortest Path Visiting All Nodes (LC 847), why use BFS instead of DP? | The graph is unweighted — BFS guarantees shortest path by exploring states layer by layer. State = (node, visited_mask); first time full mask is reached gives the minimum edges. DP would need Dijkstra for weighted graphs. |
| 113.3 | In Distribute Repeating Integers (LC 1655), why iterate masks backwards when processing each supply count? | To prevent using the same supply count more than once. Backwards iteration ensures dp[mask] still reflects a state from PREVIOUS supplies when computing dp[mask \| sub] — same principle as 0/1 knapsack backwards pass. |
| 113.4 | What is the time complexity of TSP-style bitmask DP with n items? | O(2^n × n²): 2^n mask states × n last-node choices × n transitions from each state. Space: O(2^n × n) for dp table plus parent table for reconstruction. |
| 113.5 | In Redis Cluster, how does data distribution differ from consistent hashing? | Redis Cluster uses 16384 fixed hash slots assigned in contiguous ranges. Key → slot = CRC16(key) % 16384. Unlike consistent hashing, slot assignment is explicit and reconfigured manually during resharding — not automatic ring traversal. |

### Day 114 — Digit DP

| # | Q | A |
|---|---|---|
| 114.1 | What are the 3 key state variables in digit DP and what does each control? | (1) pos: current digit position. (2) tight: True if all previous digits match the limit — current digit ≤ limit[pos]; False means current digit is free (0..9). (3) extra_state: problem-specific (prev_bit for no-consecutive-ones, used_mask for unique digits). |
| 114.2 | In Non-negative Integers without Consecutive Ones (LC 600), write the transition rule. | For bit in {0,1}: if prev_one and bit==1, skip. Otherwise recurse: dp(pos+1, bit==1, tight and bit==int(bits[pos])). Sum over valid bit choices. |
| 114.3 | In Numbers At Most N Given Digit Set (LC 902), what does the final `else` clause of the for-loop count? | The number N itself. If every digit of N was found in the digit set (loop completed without breaking), then N is constructable — add 1. Breaking means N has a digit not in the set → N is invalid. |
| 114.4 | In LFU Cache, why do you need min_freq and when does it update? | min_freq tracks the least-frequent bucket for O(1) eviction. Updates: (1) Resets to 1 on every new key insert. (2) Increments only if the vacated bucket equals min_freq AND becomes empty. |
| 114.5 | Compare LRU and LFU eviction: when does each perform better? | LRU: better for temporal locality — recent items stay (hot-then-cold patterns). LFU: better for stable frequency distributions — popular items survive even if briefly idle (Pareto access patterns). LFU has higher implementation complexity (3 hashmaps vs 1 hashmap + DLL). |

### Day 115 — Game Theory DP

| # | Q | A |
|---|---|---|
| 115.1 | In game theory DP, what does dp[i][j] represent and what is the recurrence for Predict the Winner? | dp[i][j] = max score advantage (current player minus opponent) with subarray nums[i..j]. Recurrence: max(nums[i] - dp[i+1][j], nums[j] - dp[i][j-1]). First player wins if dp[0][n-1] >= 0. |
| 115.2 | In Stone Game III (LC 1406), write dp[i] and explain the subtraction. | dp[i] = max advantage starting at index i. dp[i] = max over k=1,2,3 of (Σ stones[i..i+k-1]) - dp[i+k]. Subtraction: current player takes the sum, opponent gets dp[i+k] advantage from remaining — net = sum - dp[i+k]. |
| 115.3 | In Stone Game IV (LC 1510), what characterizes a losing position (P-position)? | dp[i] = False. ALL moves from i (subtract any perfect square j²) lead to dp[i-j²] = True (opponent wins). An N-position has at least one move to a P-position. Terminal dp[0]=False (no moves, current player loses). |
| 115.4 | Name 4 cache invalidation strategies and when to use each. | (1) Write-through: strong consistency (financial). (2) Write-back: write-heavy, brief inconsistency tolerable. (3) Cache-aside: read-heavy, simple. (4) Event-driven CDC (Debezium+Kafka): DB is source of truth, near-real-time freshness needed. |
| 115.5 | What is the Sprague-Grundy theorem and when is it applicable? | Every impartial game position has a Grundy number. Position is losing iff Grundy=0. Applicable when: both players have identical moves, last-to-move wins, game always terminates. Enables combining independent sub-games by XOR of Grundy numbers. |

### Day 116 — XOR + Binary Trie

| # | Q | A |
|---|---|---|
| 116.1 | Explain how XOR helps find the single non-duplicate in an array of pairs. | XOR all elements. a⊕a=0 for each pair; 0⊕x=x. All paired elements cancel to 0. Only the lone element remains: result = 0⊕(all elements). Works because XOR is commutative and associative. |
| 116.2 | In Maximum XOR (LC 421), why process bits from MSB to LSB in the Trie? | XOR magnitude is dominated by the highest differing bit. Processing MSB→LSB lets us greedily choose the highest bit that can differ (take opposite bit) before deciding lower bits. Greedy choice is locally and globally optimal for maximizing XOR. |
| 116.3 | In Maximum XOR With Element From Array (LC 1707), why is offline processing needed? | Each query has limit mi: only use nums ≤ mi. Sorting both nums and queries by mi lets us incrementally insert valid nums into the Trie (each inserted at most once). Without sorting, each query requires rebuilding the Trie. |
| 116.4 | Name the 3 Redis deployment modes and when to choose Cluster over Sentinel. | Standalone (dev/small), Sentinel (HA, single-shard, automatic failover), Cluster (HA + horizontal sharding). Choose Cluster when data exceeds single-node RAM or write throughput requires multiple primaries. |
| 116.5 | What does prefix XOR enable and give a concrete example. | prefix[i] = nums[0]⊕...⊕nums[i]. XOR of subarray [L,R] = prefix[R] ⊕ prefix[L-1]. Example: count subarrays with XOR = k — for each R, count L where prefix[L-1] = prefix[R] ⊕ k (hashmap lookup, O(n) total). |

### Day 117 — Bit Counting + AND Subarrays

| # | Q | A |
|---|---|---|
| 117.1 | In Counting Bits (LC 338), write two O(n) recurrences for dp[i]. | (1) dp[i] = dp[i>>1] + (i&1): right-shift removes last bit, add 1 if that bit was 1. (2) dp[i] = dp[i&(i-1)] + 1: Brian Kernighan — i&(i-1) clears the lowest set bit; popcount = that count + 1. |
| 117.2 | In Mysterious Function (LC 1521), why does the set of distinct AND values have at most O(log V) elements per right endpoint? | Bitwise AND can only clear bits. As we extend a subarray left, each new AND can only produce fewer or equal set bits. At most 32 bit transitions per right endpoint → O(log V) = O(32) distinct AND values. |
| 117.3 | In Count Pairs With XOR in a Range (LC 1803), how do you count pairs with XOR < limit using a Trie? | For each num, traverse Trie MSB first. When limit bit=1: count subtree reachable via XOR-bit=0 (all < limit here); follow XOR-bit=1 path. When limit bit=0: only follow XOR-bit=0 path. Sum counts at limit-bit=1 decision points. |
| 117.4 | What is the thundering herd problem and name 2 solutions. | A popular key expires; simultaneous requests all miss and hammer the DB. Solutions: (1) Mutex/lock — only one request rebuilds, others wait or serve stale. (2) Probabilistic early expiration (XFetch) — proactively refresh before expiry with p=exp(-delta/beta), spreading load. |
| 117.5 | Brian Kernighan's algorithm counts set bits in O(k). What is the key operation and why? | n = n & (n-1) removes the lowest set bit each iteration. n-1 flips all bits from the lowest set bit downward; AND with n clears from lowest set bit down, preserves above. Loop until n=0 counts exactly k=popcount(n) iterations. |

### Day 118 — Subset Enumeration + XOR Parity

| # | Q | A |
|---|---|---|
| 118.1 | In Longest Awesome Substring (LC 1542), what does the parity bitmask represent and what two conditions make a valid palindrome substring? | mask = XOR of (1<<digit) for each digit, tracking parity of each digit's count. Substring [l+1..r] is palindrome-formable iff prefix[r] ⊕ prefix[l] == 0 (all even) OR is a power of 2 (exactly one odd). Store first-seen index per mask. |
| 118.2 | In Maximum Product of Word Lengths (LC 318), how do you check if two words share no letters in O(1)? | Precompute mask[i] = bitmask of letters in words[i] (bit k set iff letter 'a'+k appears). Two words share no letters iff mask[i] & mask[j] == 0. O(n²) pair comparison with O(1) per pair. |
| 118.3 | Why does Longest Awesome Substring store only the FIRST occurrence of each parity mask? | To maximize substring length. If mask appears at positions p1 < p2, substring from p1 has length i-p1 > i-p2 for the same right endpoint i. first_seen stores earliest occurrence; never updated once set. |
| 118.4 | What is the ARC eviction policy? | ARC maintains T1 (accessed once), T2 (accessed 2+), and ghost lists B1 (recently evicted from T1) and B2 (from T2). B1 hit → grow T1 (recency matters). B2 hit → grow T2 (frequency matters). Automatically balances recency vs. frequency. Used in ZFS. |
| 118.5 | How does the hotspot key problem manifest and name two solutions. | A single popular key routes all requests to one cache node → bottleneck. Solutions: (1) Local in-process cache per app server with short TTL — distributes reads. (2) Key replication with random suffix (hot_key_0..N-1) — spread reads across N nodes; write to all copies. |

### Day 119 — Probability DP

| # | Q | A |
|---|---|---|
| 119.1 | In New 21 Game (LC 837), what does dp[i] represent and what is the sliding window optimization? | dp[i] = probability of reaching exactly score i. dp[i] = window_sum / maxPts where window_sum = Σ dp[j] for j in [i-maxPts, i-1]. Maintain: add dp[i] when i < k (player continues); subtract dp[i-maxPts] when i >= maxPts. O(n) instead of O(n×maxPts). |
| 119.2 | In Knight Probability (LC 688), why compute probability forward (from start)? | Forward DP is natural: dp[r][c] = probability of being at (r,c) after current steps. Each step distributes probability to 8 neighbors (1/8 each). Off-board probability is simply discarded. Backward DP is equivalent but less intuitive. |
| 119.3 | What are P-positions and N-positions in combinatorial game theory? | P-position (current player loses): ALL moves lead to N-positions. N-position (current player wins): at least ONE move leads to a P-position. Terminal positions (no moves) are P-positions. |
| 119.4 | How does Redis approximate LRU eviction at scale? | Redis samples maxmemory-samples (default 5, tunable to 10+) random keys per eviction event. Evicts the sampled key with the oldest last-access timestamp. Avoids O(1) overhead per access of maintaining a full LRU list. With 10 samples, within ~1-2% of true LRU quality. |
| 119.5 | In probability DP, when do you iterate forward vs. backward? | Forward: state = current position, propagate to future states — dp[next] += dp[curr] × prob. Backward: compute probability/expected value of success from current state — dp[curr] = Σ prob × dp[next]. Forward is natural for "what is the distribution after k steps"; backward for "what is the chance of success from here." |

---

## Strength / Gap Assessment

Rate yourself: ✓ = confident, ~ = shaky, ✗ = needs work

| Pattern | Status | Notes |
|---------|--------|-------|
| TSP bitmask DP: dp[mask][last] recurrence | | |
| Overlap precomputation for Shortest Superstring | | |
| BFS + bitmask state (LC 847) | | |
| Digit DP: tight flag, leading zero, extra state | | |
| Binary digit DP (consecutive ones, LC 600) | | |
| Digit-by-digit counting without DP (LC 902) | | |
| Score-based game theory DP (interval + linear) | | |
| Boolean win/lose DP, P/N positions | | |
| Binary Trie: insert, max XOR query (greedy) | | |
| Offline queries sorted by constraint + Trie | | |
| Monotone AND set O(n log V) technique | | |
| Trie with count field for range XOR counting | | |
| XOR parity prefix + palindrome condition | | |
| Per-word character bitmask intersection | | |
| Sliding-window Trie with deletion (LC 2935) | | |
| Forward probability DP with rolling array | | |
| Sliding window probability sum (LC 837) | | |
| LFU Cache: 3 hashmaps + min_freq | | |
| Cache invalidation strategies (5 types) | | |
| Redis Cluster vs Sentinel trade-offs | | |
| Thundering herd: 3 mitigation strategies | | |
| Eviction policy selection: LRU/LFU/ARC | | |

---

## Problems to Revisit

Fill in after review session:

| Problem | LC # | Issue | Retry Date |
|---------|------|-------|------------|
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |

---

## Mock / Contest Log

| Date | Activity | Problems Attempted | Solved | Time | Notes |
|------|----------|--------------------|--------|------|-------|
| | | | | | |
| | | | | | |

---

## Week 17 Goals — Self-Check

- [ ] Can write TSP dp[mask][last] recurrence and transition from memory in < 2 minutes
- [ ] Digit DP tight-flag logic memorized: when tight=True, digit ≤ limit[pos]; when False, digit free 0..9
- [ ] Can state P/N position definition and give an example from Stone Game IV
- [ ] XOR Trie greedy: explain why "take opposite bit" maximizes XOR from MSB
- [ ] Can implement LFU Cache _update() from memory including min_freq increment rule
- [ ] Can name all 5 cache invalidation strategies and give a one-line use case for each
- [ ] Sliding window sum pattern for LC 837 memorized: window_sum add/subtract conditions
- [ ] Can explain ARC policy (T1, T2, B1, B2) and when it outperforms pure LRU or LFU
