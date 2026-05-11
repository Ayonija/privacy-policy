# Week 16 Weekly Review — Days 106–112
**Phase 2: Exposure & Assessment | Month 4**

## Week Summary

| Day | Topic | DSA Patterns | System Design |
|-----|-------|-------------|---------------|
| 106 | LCS Variants | Interleaving String, Remove Boxes (interval DP), Max Dot Product of LCS | API Gateway — rate limiting, auth, routing |
| 107 | LIS Advanced | Number of LIS (segment tree counts), Mountain Array Removals, Delete Columns to Make Sorted III | API Versioning — URL path, header, query param |
| 108 | LIS Complex | Largest Divisible Subset, Max Height Cuboids (multi-dim LIS), LIS II with segment tree | GraphQL Federation + Automatic Persisted Queries |
| 109 | Hybrid DP | Longest String Chain, Jump Game V (memo), Make Array Strictly Increasing (BinarySearch+Knapsack) | Redis Lua rate limiting (token bucket) |
| 110 | Slot 11 Review | Full timed assessment + pattern recall table | Full API Design mock (REST + GraphQL + Rate Limiting) |
| 111 | Tree DP | House Robber III (2-state), Binary Tree Cameras (3-state), Sum of Distances in Tree (rerooting) | Consistent Hashing — ring, virtual nodes |
| 112 | Bitmask DP | Beautiful Arrangement, Partition to K Subsets, Min Work Sessions + LRU Cache | LRU Cache — O(1) HashMap + doubly-linked-list |

---

## 35-Card Flashcard Deck (5 per day × 7 days)

### Day 106 — LCS Variants

| # | Q | A |
|---|---|---|
| 106.1 | In Interleaving String (LC 97), what does dp[i][j] represent and what is the recurrence? | dp[i][j] = True if s1[0..i-1] and s2[0..j-1] can interleave to form s3[0..i+j-1]. Recurrence: dp[i][j] = (dp[i-1][j] and s1[i-1]==s3[i+j-1]) or (dp[i][j-1] and s2[j-1]==s3[i+j-1]). |
| 106.2 | In Remove Boxes (LC 546), why is the state dp[l][r][k] and what does k represent? | k = number of boxes to the LEFT of l that have the same color as boxes[l]. They can be grouped with boxes[l] for a combined score. dp[l][r][k] = max points removable from boxes[l-k..l..r]. |
| 106.3 | In Remove Boxes (LC 546), write the two key transitions. | (1) Remove boxes[l] alone with its k duplicates: (k+1)^2 + dp[l+1][r][0]. (2) Find m in (l,r] where boxes[m]==boxes[l]: dp[l+1][m-1][0] + dp[m][r][k+1] — defer removal to merge groups. |
| 106.4 | What is the time complexity of Remove Boxes (LC 546) and why? | O(n^4): O(n^3) states (l, r, k each up to n) × O(n) transitions per state to find matching box m. Space O(n^3) for memoization. |
| 106.5 | In an API Gateway, name 4 cross-cutting concerns it handles. | Authentication/Authorization, Rate Limiting (token bucket / sliding window), Request Routing (load balancing, service discovery), Observability (access logs, distributed tracing headers like X-Trace-ID). |

### Day 107 — LIS Advanced

| # | Q | A |
|---|---|---|
| 107.1 | In Number of LIS (LC 673), what two arrays do you maintain and what is the recurrence? | dp[i] = length of LIS ending at i. cnt[i] = number of LIS of that length ending at i. If dp[j]+1 > dp[i]: dp[i]=dp[j]+1, cnt[i]=cnt[j]. If dp[j]+1 == dp[i]: cnt[i]+=cnt[j]. Answer: sum(cnt[i]) for all i where dp[i]==max_len. |
| 107.2 | In Minimum Mountain Removals (LC 1671), how do you combine LIS from both ends? | Compute lis_left[i] = LIS length ending at i (left to right). Compute lis_right[i] = LIS length ending at i (right to left, i.e., LDS from right). For each valid peak i (lis_left[i]>=2 and lis_right[i]>=2): kept = lis_left[i]+lis_right[i]-1. Removals = n - max(kept). |
| 107.3 | In Delete Columns to Make Sorted III (LC 960), what does dp[j] represent? | dp[j] = length of the longest chain of columns ending at column j such that all rows are non-decreasing across the selected columns. Answer = n_cols - max(dp[j]). Recurrence: if rows[row][i] <= rows[row][j] for all rows, then dp[j] = max(dp[j], dp[i]+1). |
| 107.4 | What is the difference between URL path versioning, header versioning, and query param versioning in APIs? | URL path (/v1/resource): most visible, easiest to cache and route. Header (Accept: application/vnd.api.v2+json): cleaner URLs, harder to test in browser. Query param (?version=2): easy to add, can pollute URLs. URL path is preferred for public APIs; header for internal/enterprise. |
| 107.5 | In API Gateway rate limiting, what is the difference between token bucket and sliding window counter? | Token bucket: refills at rate r tokens/sec, allows bursts up to bucket capacity. Sliding window: counts requests in a rolling time window; smoother but more memory. Token bucket is preferred when short bursts are acceptable; sliding window log gives precise counts. |

### Day 108 — LIS Complex

| # | Q | A |
|---|---|---|
| 108.1 | In Largest Divisible Subset (LC 368), why sort the array first? | After sorting, if nums[i] % nums[j] == 0 and nums[j] % nums[k] == 0 for i>j>k, then nums[i] % nums[k] == 0 (transitivity). Sorting ensures we only check pairs in one direction, reducing O(n^2) to a clean forward pass. |
| 108.2 | In Max Height by Stacking Cuboids (LC 1691), what transformation enables LIS? | Sort each cuboid's dimensions within itself (normalize to w≤h≤d). Sort all cuboids lexicographically. Now cuboid a can be placed on b iff all 3 dimensions of a ≤ b. dp[i] = max height of stack ending with cuboid i: dp[i] = max(dp[j] + height[i]) for all j where a[j] fits under a[i]. |
| 108.3 | In Russian Doll Envelopes (LC 354), why sort by width ascending and height DESCENDING? | When widths are equal, you cannot stack them (strict containment required). Sorting heights descending ensures that for equal widths, LIS on heights can only pick one envelope per width group — preventing incorrect stacking of same-width envelopes. |
| 108.4 | What is Automatic Persisted Queries (APQ) in GraphQL and why use it? | Client sends query hash instead of full query text. If server has it cached: executes immediately (saves bandwidth). If not: client resends full query, server caches and executes. Reduces payload for frequently repeated large queries (e.g., mobile clients on slow networks). |
| 108.5 | In GraphQL Federation, what does the @key directive do? | @key marks a type's primary key fields across services. The gateway uses it to stitch responses: Service A returns {id} for User, Service B uses @key(fields: "id") to extend User with profile data. Enables cross-service type references without coupling services. |

### Day 109 — Hybrid DP

| # | Q | A |
|---|---|---|
| 109.1 | In Longest String Chain (LC 1048), how do you check if word A is a predecessor of word B? | len(B) == len(A)+1 and B can be formed by inserting exactly one character into A. Implementation: for each position i in B (0..len(B)), check if B[:i]+B[i+1:] == A. O(len^2) per pair, O(n*len^2) total. |
| 109.2 | In Jump Game V (LC 1340), why does memo/top-down DP work cleanly here? | dp[i] = max jump count starting from index i. Transitions go to j where |i-j|<=d and arr[j] < arr[i] (can only jump to strictly lower). Since you can only jump to lower heights, there are no cycles — memoization is safe. |
| 109.3 | In Make Array Strictly Increasing (LC 1187), what are the state dimensions? | dp[i][j] = min operations to make arr1[0..i] strictly increasing where arr1[i] was replaced with sorted_arr2[j] (or j=len(arr2) means arr1[i] was kept). Transition: from (i,j) find the largest value in arr2 strictly less than current value at position i+1. |
| 109.4 | In Redis Lua scripting for rate limiting, why does Lua guarantee atomicity? | Redis executes Lua scripts atomically — no other command runs between script instructions. This eliminates the TOCTOU race condition in check-then-act patterns (e.g., read count, increment, set expiry). MULTI/EXEC transactions also provide atomicity but Lua is more flexible. |
| 109.5 | Write the token bucket rate limiter logic in pseudocode using Redis Lua. | `local tokens = tonumber(redis.call('GET', key) or capacity)`. `local now = tonumber(ARGV[1])`. `local last = tonumber(redis.call('GET', key..':ts') or now)`. `tokens = min(capacity, tokens + rate*(now-last))`. `if tokens >= 1: tokens-=1; allow; else deny`. `redis.call('SET', key, tokens); redis.call('SET', key..':ts', now)`. |

### Day 110 — Slot 11 Review

| # | Q | A |
|---|---|---|
| 110.1 | In 0/1 Knapsack (LC 416 Partition Equal Subset Sum), why iterate j backwards? | Forward iteration would allow the same item to be used multiple times (unbounded knapsack). Backward iteration ensures dp[j-num] still reflects the state BEFORE adding the current item, so each item is used at most once. |
| 110.2 | In Ones and Zeroes (LC 474), what is the 2D knapsack recurrence? | dp[i][j] = max strings using at most i zeros and j ones. For each string with z zeros, o ones: iterate i from maxZeros→z, j from maxOnes→o: dp[i][j] = max(dp[i][j], dp[i-z][j-o]+1). Both dimensions iterated backward for 0/1 property. |
| 110.3 | In Cherry Pickup II (LC 1463), why is state dp(row, c1, c2) sufficient (not c1_prev, c2_prev)? | Each row is independent given the column positions of both robots. The problem asks for max cherries, and both robots move one row at a time — so the state (row, robot1_col, robot2_col) fully captures all future decisions without needing history. |
| 110.4 | In the REST vs GraphQL comparison, name two scenarios where GraphQL is better and two where REST is better. | GraphQL better: (1) mobile clients needing to minimize over-fetching (flexible queries). (2) Rapidly evolving schemas — add fields without versioning. REST better: (1) Simple CRUD with well-defined resources (tooling, caching easier). (2) File upload/streaming (GraphQL lacks native support). |
| 110.5 | In Tallest Billboard (LC 956), what does dp[diff] store and why? | dp[diff] = max height of the SHORTER rod when the height difference between the two rods is diff. Tracking difference (not absolute heights) compresses state from O(sum^2) to O(sum). For each rod, 3 transitions: add to taller, add to shorter, ignore. |

### Day 111 — Tree DP

| # | Q | A |
|---|---|---|
| 111.1 | In House Robber III (LC 337), write dp(node) and explain what it returns. | dp(node) returns (rob_this, skip_this). rob_this = node.val + skip_left + skip_right. skip_this = max(rob_left, skip_left) + max(rob_right, skip_right). Returns optimal value for each choice at this node. |
| 111.2 | In Binary Tree Cameras (LC 968), what are the 3 states and what does each mean? | 0=UNCOVERED: node not covered, needs parent to place camera. 1=HAS_CAMERA: camera placed here, covers parent and self. 2=COVERED: covered by a child's camera, no camera placed here. |
| 111.3 | In Sum of Distances in Tree (LC 834), what is the rerooting formula and why? | ans[child] = ans[parent] - count[child] + (n - count[child]). Moving root from parent to child: count[child] subtree nodes get 1 step closer; n-count[child] nodes get 1 step farther. |
| 111.4 | In consistent hashing, why use virtual nodes? | Without virtual nodes, node addition/removal causes uneven key distribution. Virtual nodes spread each physical node's responsibility across the ring, achieving ~uniform distribution. V≈150 gives <5% standard deviation in load. |
| 111.5 | In consistent hashing, how do you find which node owns a key? | Hash the key to get a position in [0, 2^32). Binary search the sorted list of virtual node positions for the first position >= key hash. Wrap around to the first node if none found. O(log(nV)) with bisect. |

### Day 112 — Bitmask DP

| # | Q | A |
|---|---|---|
| 112.1 | In Bitmask DP, how do you enumerate all non-empty subsets of a bitmask `mask`? | `sub = mask; while sub > 0: process(sub); sub = (sub-1) & mask`. This runs in O(2^popcount(mask)) per mask and O(3^n) total across all masks. |
| 112.2 | In Beautiful Arrangement (LC 526), how do you determine the current position from the bitmask? | pos = bin(mask).count('1') = popcount(mask). Since positions 1..n are filled in order, when exactly k numbers are placed (popcount=k), we're filling position k. |
| 112.3 | In Partition to K Equal Sum Subsets (LC 698), what does dp[mask] store and what is the mod trick? | dp[mask] = current partial sum in the active (incomplete) subset. When sum reaches target, dp[mask] = sum % target = 0 (one subset complete, reset). dp[full]==0 means all subsets completed. |
| 112.4 | In LRU Cache, what data structures give O(1) get and put? | HashMap (key → doubly-linked-list node) for O(1) lookup + Doubly Linked List for O(1) move-to-head and remove-from-tail. Head = most recent (never evict), tail = least recent (evict from here). |
| 112.5 | In consistent hashing, when a node is added to the ring, how many keys need to be reassigned? | Only the keys previously owned by the new node's clockwise successor. On average K/n keys where K = total keys and n = number of nodes. All other nodes are completely unaffected. |

---

## Strength / Gap Assessment

Rate yourself: ✓ = confident, ~ = shaky, ✗ = needs work

| Pattern | Status | Notes |
|---------|--------|-------|
| LCS classic (LC 1143) | | |
| Interleaving String interval DP (LC 97) | | |
| Remove Boxes 3D DP (LC 546) | | |
| LIS with binary search O(n log n) | | |
| Number of LIS with counts (LC 673) | | |
| Multi-dim LIS / cuboid stacking (LC 1691) | | |
| Hybrid DP: String Chain (LC 1048) | | |
| Hybrid DP: Make Array Strictly Increasing (LC 1187) | | |
| Tree DP 2-state (LC 337) | | |
| Tree DP 3-state (LC 968) | | |
| Rerooting DP (LC 834) | | |
| Bitmask DP with popcount-position (LC 526) | | |
| Bitmask partition with mod-target (LC 698) | | |
| Bitmask DP + subset enumeration (LC 1986) | | |
| Consistent Hashing ring + virtual nodes | | |
| LRU Cache with doubly-linked list | | |

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

## Week 16 Goals — Self-Check

- [ ] Can code dp(node) returning a tuple for House Robber III from memory in < 10 min
- [ ] Can state all 3 transitions for Binary Tree Cameras without reference
- [ ] Can write the rerooting formula and explain it from first principles
- [ ] Can code LRU Cache (including dummy sentinels) from memory in < 15 min
- [ ] Can explain consistent hashing virtual nodes and their purpose to a non-technical interviewer
- [ ] Subset enumeration idiom memorized: `sub = (sub-1) & mask`
- [ ] Popcount-as-position trick memorized for Beautiful Arrangement variant problems
