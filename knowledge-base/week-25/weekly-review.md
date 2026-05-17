# Week 25 Review — Days 169–175

## Phase
Phase 4 — Final Sprint & Real Applications | Month 6

---

## Patterns Covered This Week

### DSA Patterns
1. **Memoized Game/Combinatorial DFS (State Enumeration)** — Day 169: State (n, first, second); normalize with mirror symmetry; enumerate all bracket outcomes; O(N^4) worst case.
2. **Backtracking on Character Frequency** — Day 169 (revision): Frequency array; backtrack over distinct characters; O(7!) worst case; duplicates handled naturally.
3. **Backtracking with used[] Boolean** — Day 169 (OA): Position i; try all unused j where j%i==0 or i%j==0; O(n!) pruned.
4. **Patience Sort / Binary Search on Tails (LIS/LNS Variants)** — Day 170: tails[] array; upper_bound for non-decreasing; lower_bound for strictly increasing; O(N log N).
5. **Interval Merge (Linear Sweep)** — Day 170 (revision): Three-phase sweep — before, overlap, after; O(n).
6. **Interval Coverage (Sort + Greedy)** — Day 170 (OA): Sort by start ASC then end DESC; track maxRight; O(n log n).
7. **Binary Search on Partition** — Day 171: Shorter array partition; cross-partition max/min invariant; O(log(min(m,n))).
8. **Binary Search — Rotated Array** — Day 171 (revision/OA): One half always sorted; check target in sorted half; O(log n).
9. **Dijkstra on Grid / Multi-Source BFS** — Day 172: Min-heap replaces FIFO queue for non-uniform edge weights; Math.max for path cost; O(m*n log(m*n)).
10. **DFS + Multi-Source BFS (Shortest Bridge)** — Day 172 (revision): DFS flood-fill island 1; multi-source BFS outward; first contact = shortest bridge.
11. **BFS on Grid** — Day 172 (OA): 8-direction BFS; level = path length; O(n^2).
12. **Binary Lifting (Sparse Table on Trees)** — Day 173: up[node][j] = 2^j-th ancestor; O(n log n) precompute; O(log n) per query.
13. **BFS Level-Order / DFS with Depth Tracking** — Day 173 (revision): Reset sum per BFS level; return final level sum.
14. **DFS + Bitmask** — Day 173 (OA): XOR-toggle digit bit; at leaf check bitmask & (bitmask-1) == 0.
15. **DP on Strings (Parsing + Sequence)** — Day 174: dp[i] = ways to restore s[0..i-1]; prune on leading zero or value > k; O(n log k).
16. **DP on Sorted Word Sequences** — Day 174 (revision): Sort by length; dp[word] = max chain; try all single-character deletions; O(n * L^2).
17. **2D DP (Count Square Submatrices)** — Day 174 (OA): dp[i][j] = min(up, left, diag) + 1; answer += dp[i][j]; O(m*n).
18. **Backtracking with Aggressive Pruning** — Day 175: Topmost-leftmost cell first; try largest square first; prune when count >= result; exponential worst case, fast in practice.
19. **Backtracking + Pruning (K Equal Sum Subsets)** — Day 175 (revision): Sort descending; skip duplicate bucket sums; O(k * 2^n) pruned.
20. **Backtracking / DP Subset Sum (Target Sum)** — Day 175 (OA): ±nums[i] DFS counting; or subset sum transform: sum(P) = (total+target)/2.

---

## System Design Topics Covered This Week

| Day | Topic |
|-----|-------|
| 169 | Elevator System Class Diagram (LOOK Scheduling, State Machine) |
| 170 | E-commerce Platform DB Schema (Capstone LLD) |
| 171 | Instagram — End-to-End Design |
| 172 | WhatsApp — End-to-End Design |
| 173 | Google Drive / Dropbox — End-to-End Design |
| 174 | Amazon E-commerce — End-to-End Design |
| 175 | Airbnb — End-to-End Design |

---

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in after each session) | | | |
| | | | |
| | | | |

---

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Memoized Game/Combinatorial DFS | | |
| Backtracking on Character Frequency | | |
| Backtracking with used[] Boolean | | |
| Patience Sort / Binary Search on Tails | | |
| Interval Merge (Linear Sweep) | | |
| Interval Coverage (Sort + Greedy) | | |
| Binary Search on Partition | | |
| Binary Search — Rotated Array | | |
| Dijkstra on Grid / Multi-Source BFS | | |
| DFS + Multi-Source BFS (Shortest Bridge) | | |
| BFS on Grid | | |
| Binary Lifting (Sparse Table on Trees) | | |
| BFS Level-Order / DFS with Depth Tracking | | |
| DFS + Bitmask | | |
| DP on Strings (Parsing + Sequence) | | |
| DP on Sorted Word Sequences | | |
| 2D DP (Count Square Submatrices) | | |
| Backtracking with Aggressive Pruning | | |
| Backtracking + Pruning (K Equal Sum Subsets) | | |
| Backtracking / DP Subset Sum (Target Sum) | | |
| Elevator System LLD (LOOK Scheduling) | | |
| E-commerce Platform DB Schema | | |
| Instagram System Design | | |
| WhatsApp System Design | | |
| Google Drive / Dropbox System Design | | |
| Amazon E-commerce System Design | | |
| Airbnb System Design | | |

**Scoring guide:** 5 = can implement cold in 20 min; 4 = minor prompts needed; 3 = need to look up recurrence; 2 = partial understanding; 1 = blocked.

---

## Weekly Flashcard Deck

All 35 cards aggregated from Days 169–175, 5 per day.

| Q | A |
|---|---|
| **Day 169** | |
| In LC 1900, what is the base case and what does it mean? | `first + second == n + 1` — the two target players are matched against each other in this round (seats 1..n are paired as 1 vs n, 2 vs n-1, …). Record the current round number in both earliest and latest. |
| Why use `TreeSet<Integer>` for elevator requests instead of a `PriorityQueue`? | `TreeSet` supports `ceiling(x)` (next floor at or above x) and `floor(x)` (next floor at or below x) in O(log N) — exactly what LOOK algorithm needs to find the next stop in the current direction. `PriorityQueue` only gives the global minimum. |
| In LC 1079 (Letter Tile Possibilities), why does the frequency array approach automatically avoid counting duplicate sequences? | Because we iterate over distinct characters (freq[0..25]), not individual tile positions. Choosing the same character twice in a row is naturally handled by decrementing and restoring frequency — identical characters are treated as interchangeable. |
| What is the LOOK algorithm and how does it differ from SCAN? | LOOK reverses direction at the last pending request in the current direction (not at the physical floor boundary). SCAN always goes to floor 0 or floor N before reversing. LOOK eliminates empty travel past the last request. |
| In LC 526 (Beautiful Arrangement), what is the pruning condition that makes backtracking feasible? | At position i, only place values j where `j % i == 0 || i % j == 0`. This eliminates most candidates early, making the effective branching factor much lower than N — the actual count is at most a few hundred for N≤15. |
| **Day 170** | |
| What is the difference between lower_bound and upper_bound in patience sort, and which does each LIS variant use? | `lower_bound` finds the first index where `tails[i] >= x` → used for STRICTLY INCREASING LIS (replace at the first equal-or-greater tail). `upper_bound` finds the first index where `tails[i] > x` → used for NON-DECREASING LNS (allow equal, so only replace when strictly exceeded). |
| In LC 57 (Insert Interval), what are the three phases of the sweep? | Phase 1: copy all intervals that end before newInterval starts (`end < newInterval.start`). Phase 2: merge all overlapping intervals into newInterval (`start <= newInterval.end`). Phase 3: copy all remaining intervals. |
| Why sort by `end DESC` for the same `start` value in LC 1288 (Remove Covered Intervals)? | So that among intervals with the same start, the longest (largest end) is processed first — it will set `maxRight` to the maximum, causing all shorter intervals with the same start to be detected as covered in the same pass. |
| What SQL idiom prevents overselling in a flash sale at the database level? | `UPDATE Products SET inventory_count = inventory_count - 1 WHERE product_id = ? AND inventory_count > 0` — if `rowsAffected == 0`, the item is sold out. DB row-level locking serializes concurrent transactions on the same product row. |
| Why store `unit_price` and `delivery_address` as snapshots in Orders/OrderItems rather than FKs to the current values? | Prices and addresses change over time. A snapshot captures the exact state at transaction time, making historical order records accurate regardless of future updates to the product price or the user's address. |
| **Day 171** | |
| How do you set up the binary search partition for LC 4 Median of Two Sorted Arrays? | Ensure nums1 is shorter; binary search partitionX in [0,m]; partitionY = (m+n+1)/2 - partitionX; check maxLeftX≤minRightY && maxLeftY≤minRightX; adjust lo/hi on violation; time O(log(min(m,n))) |
| In a rotated sorted array, how do you determine which half is sorted? | Compare nums[mid] with nums[lo]: if nums[lo]≤nums[mid], left half [lo..mid] is sorted; else right half [mid..hi] is sorted. Then check if target falls in the sorted half. |
| What is Instagram's feed generation strategy for celebrity accounts vs regular accounts? | Regular users (<1M followers): fan-out on write — post is pushed to all follower feeds asynchronously. Celebrity accounts (>1M followers): fan-out on read — followers pull celebrity posts at query time and merge with pre-computed feed. |
| How does Instagram handle story expiry at 24 hours at scale? | Stories stored in Cassandra with TTL=86400s on the row; Cassandra natively deletes expired rows via tombstones and compaction — no cron job needed. |
| What is the time complexity of binary search on partition for two sorted arrays of size m and n? | O(log(min(m,n))) — binary search only on the shorter array; each iteration eliminates half the shorter array. |
| **Day 172** | |
| How does Dijkstra apply to LC 778 Swim in Rising Water? | dist[r][c] = minimum possible max-elevation on any path from (0,0) to (r,c); min-heap processes cells in order of current path cost; update neighbor if max(curr_cost, neighbor_elevation) < dist[neighbor]. |
| In LC 934 Shortest Bridge, how do you find and connect the two islands? | Step 1: DFS from any 1-cell to flood-fill island 1 (mark as 2, add all to BFS queue). Step 2: Multi-source BFS expands in 4 directions; first time a 0-cell at distance k reaches another 1-cell, return k. |
| What is WhatsApp's strategy for delivering messages to offline users? | Message is stored in Cassandra with status PENDING; when recipient reconnects, their Chat Server queries Cassandra for undelivered messages and pushes them, then marks DELIVERED. |
| How does WhatsApp's online presence system work at 2B user scale? | Redis key user:{id}:online with 30s TTL; client sends heartbeat every 15s to refresh TTL; presence subscribers use Redis pub/sub on user-specific channels; presence is eventually consistent (up to 30s stale). |
| What makes Dijkstra on a grid different from standard BFS? | Standard BFS assumes unit edge weights — each step costs 1. Dijkstra handles non-uniform weights — in LC 778, the "cost" to enter a cell is its elevation (not 1), so a min-heap replaces the FIFO queue to always process the cheapest unvisited cell first. |
| **Day 173** | |
| How do you precompute binary lifting ancestors for a tree of n nodes? | Create up[n][LOG] where LOG=ceil(log2(maxDepth)). up[node][0]=parent. For j from 1 to LOG-1: up[node][j]=up[up[node][j-1]][j-1]. Precomputation: O(n log n). Each query: O(log n) by decomposing k in binary. |
| How do you find the deepest leaves sum in a binary tree in one pass? | BFS level-order — reset sum at start of each level, accumulate all node values; return sum after processing the last level. O(n) time O(w) space where w = max width. |
| In the pseudo-palindromic paths problem, how does a bitmask detect palindrome feasibility? | XOR-toggle a bit for each digit encountered (bit i = 1 if digit i appears odd times). At a leaf, a path is pseudo-palindromic if at most one bit is set: check bitmask & (bitmask-1) == 0. |
| How does Google Drive achieve delta sync to minimize upload bandwidth? | The client splits the file into fixed-size chunks (4MB) and computes a hash per chunk. On save, only chunks whose hash has changed are uploaded. A 100MB file with a 1MB edit uploads only 1 chunk (1MB) instead of 100MB. |
| What is the trade-off between last-write-wins and Operational Transform for file conflict resolution? | LWW is simple and fast but loses concurrent edits (one user's work is overwritten). OT preserves all concurrent edits by transforming operations against each other but requires a central sequencer and is complex to implement correctly — Google Docs uses OT; Dropbox uses LWW with conflict copies. |
| **Day 174** | |
| How do you set up the DP recurrence for LC 1416 Restore The Array? | dp[i] = ways to restore s[0..i-1]. For each i, try all substrings s[j..i-1] parsing to a number in [1,k]; if leading zero or value > k, stop. dp[i] += dp[j] for each valid j. Base: dp[0]=1. Answer: dp[n] mod 1e9+7. |
| In Longest String Chain, why must you sort by word length before running DP? | A predecessor of a word is always shorter by exactly 1 character. Sorting by length guarantees that when you compute dp[word], all potential predecessors have already been computed — DP processes shorter words before longer ones. |
| How does the 2D DP for Count Square Submatrices work? | dp[i][j] = size of the largest all-ones square with bottom-right corner at (i,j). If matrix[i][j]=1: dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1. The answer += dp[i][j] because a square of size k contributes k² — 1 counts to the total (dp accumulates this automatically). |
| What is the Saga pattern and when does Amazon use it? | Saga = a sequence of local transactions, each publishing an event for the next step; on failure, compensating transactions undo previous steps. Amazon uses it for order processing: Place → Charge → Reserve → Fulfill; if Reserve fails, a compensating event triggers a Refund. |
| How does Amazon's search handle 300M products with sub-100ms latency? | Elasticsearch clusters with sharded inverted indexes; product data is indexed by category/brand/attributes for fast filtering; search first retrieves candidate set (recall phase via BM25), then ML ranking model re-ranks top candidates (precision phase) — two-stage retrieval in <100ms total. |
| **Day 175** | |
| What is the key pruning condition in backtracking for LC 1240 Tiling a Rectangle? | Prune when current square count >= best found so far (count >= result). Also: always fill the topmost-leftmost uncovered cell to canonicalize the search space and eliminate symmetrical paths. |
| In LC 698 Partition to K Equal Sum Subsets, what two pruning optimizations reduce the search space most? | (1) Sort nums descending — larger numbers constrain placement early, failing faster. (2) Skip duplicate bucket sums — if placing num in bucket A (sum=5) fails, placing it in bucket B (also sum=5) will also fail; skip all buckets with identical current sums. |
| How does LC 494 Target Sum transform into a subset sum problem? | Let P = set of + numbers, N = set of - numbers. sum(P) - sum(N) = target AND sum(P) + sum(N) = total → 2*sum(P) = total + target → sum(P) = (total+target)/2. Find number of subsets summing to (total+target)/2 using 0/1 knapsack. |
| How does Airbnb prevent double-booking using Cassandra lightweight transactions? | Cassandra CAS: INSERT INTO availability (listing_id, date, booking_id) VALUES (?, ?, ?) IF NOT EXISTS. Cassandra uses Paxos to ensure only one INSERT succeeds across all replicas; second INSERT returns applied=false and the application handles the conflict. |
| What is the difference between Airbnb's search showing 'available' and the actual availability state in Cassandra? | Propagation lag (~2 seconds) — Elasticsearch is updated via a Kafka consumer after a booking is confirmed in Cassandra. During those 2 seconds, Elasticsearch may show a listing as available that is actually just-booked. The final availability check happens at HOLD time against Cassandra (the source of truth), not Elasticsearch. |

---

## Mock / Contest Log
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | LeetCode Weekly | | | |
| | Mock Interview | | | |
| | | | | |
