# Week 22 Review — Days 148–154

## Phase
Phase 3 → Phase 4 Transition | Month 5 (Days 148–150) → Month 6 (Days 151–154)

---

## Patterns Covered This Week

### From Slot 15 (Days 148–150) — Backtracking Hard

| Day | Pattern |
|-----|---------|
| 148 | Grid Backtracking (Counting Paths with Obstacles) |
| 148 | Trie-Augmented Backtracking (Multi-Word Search) |
| 149 | Exhaustive Expression Evaluation Backtracking (24 Game) |
| 149 | Trie-Constrained Grid Partition (Word Squares) |
| 149 | Combination Backtracking with element reuse |
| 150 | 3D Interval DP on Groups — `dp[l][r][k]` (Remove Boxes) |
| 150 | Backtracking on Unknown Environment with virtual coordinate system (Robot Room Cleaner) |
| 150 | Palindrome Partitioning Backtracking + DP precomputation |

### From Slot 16 (Days 151–154) — Final Sprint Revision Begins

| Day | Pattern |
|-----|---------|
| 151 | Two Pointers (sort + shrink window) |
| 151 | Variable Sliding Window |
| 151 | Prefix Sum + Suffix Product (array-as-hash index placement) |
| 152 | Iterative Linked List Reversal in K-Chunks |
| 152 | Fast/Slow Pointer (find Nth from end) |
| 152 | Monotonic Decreasing Stack (next greater element) |
| 153 | Post-order DFS Arm Pattern (combine top-2 child arms) |
| 153 | BFS Level-Order (queue size snapshot per level) |
| 153 | LCA (post-order null propagation) |
| 154 | BFS with Two Distance Arrays (first + second minimum) |
| 154 | Union-Find (path compression + union by rank) |
| 154 | Kahn's BFS Topological Sort (cycle detection via order size) |
| 154 | Reverse Multi-Source BFS (ocean border flood-fill) |

---

## System Design Topics Covered

| Day | Topic |
|-----|-------|
| 148 | YouTube / Netflix — Full HLD Architecture Diagram with all components integrated |
| 149 | YouTube / Netflix — Full Mock Interview Round (45-minute design-from-scratch prompt) |
| 150 | YouTube / Netflix — Full Integration, Key Trade-offs & Numbers to Memorize |
| 151 | Load Balancing & API Gateway — L4 vs L7, algorithms, sticky sessions, health checks |
| 152 | Caching Strategies — Cache-aside, Write-through, Write-behind, TTL, thundering herd |
| 153 | Database Design — SQL vs NoSQL, sharding strategies, replication, leader election |
| 154 | Message Queues & Event-Driven Architecture (Kafka) — partitions, delivery semantics, DLQ, backpressure |

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in) | | | |

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Grid Backtracking | | |
| Trie-Augmented Backtracking | | |
| Exhaustive Expression Evaluation | | |
| Trie-Constrained Grid Partition | | |
| 3D Interval DP (`dp[l][r][k]`) | | |
| Backtracking on Unknown Environment | | |
| Two Pointers | | |
| Variable Sliding Window | | |
| Prefix Sum / Suffix Product | | |
| Iterative Linked List Reversal (K-Group) | | |
| Fast/Slow Pointer | | |
| Monotonic Stack | | |
| Post-order DFS Arm Pattern | | |
| BFS Level-Order (queue size snapshot) | | |
| LCA (post-order null propagation) | | |
| BFS with Two Distance Arrays | | |
| Union-Find (path compression + rank) | | |
| Kahn's Topological Sort | | |
| Reverse Multi-Source BFS | | |

---

## Weekly Flashcard Deck (35 cards — Days 148–154)

Cards 1–5 are from Day 148. Cards 6–10 are from Day 149. Cards 11–15 are from Day 150. Cards 16–20 are from Day 151. Cards 21–25 are from Day 152. Cards 26–30 are from Day 153. Cards 31–35 are from Day 154.

| # | Q | A |
|---|---|---|
| 1 | How do you track visited cells in Unique Paths III without a separate visited array? | Modify the grid in-place: set `grid[r][c] = -1` (or another sentinel) when visiting, restore to original value when backtracking. |
| 2 | Why does Trie-augmented backtracking make Word Search II efficient? | All words share a single DFS pass. The Trie prunes paths early: if no word has the current prefix, that DFS branch is abandoned immediately without exploring further. |
| 3 | What does removing a found word from the Trie accomplish? | Prevents the same word from being added to the result multiple times if it appears at multiple starting positions in the grid. |
| 4 | In the full YouTube HLD, why is Cassandra chosen for video metadata over MySQL? | Video metadata is write-heavy (new uploads, view count updates, comment writes) and can tolerate eventual consistency. Cassandra scales writes horizontally by partitioning on video_id. |
| 5 | What are the two CDN caching TTL strategies for video content and why? | Encoded segments: 1-year TTL (immutable; URL includes content hash). Manifest files: 5–30 seconds TTL (must reflect current segment list after live/new uploads). |
| 6 | How do you handle the 24 Game recursion when two numbers remain? | Pick any 2 remaining numbers, apply all operators (a+b, a-b, b-a, a×b, a÷b, b÷a where divisor≠0); add result to list; recurse on list of 1. If result equals 24 (within 1e-6), return true. |
| 7 | What symmetry constraint makes Word Squares prunable? | `square[i][j] == square[j][i]` — the k-th character of the j-th word equals the j-th character of the k-th word. This means the prefix of each new word is determined by the column of already-placed words. |
| 8 | What is the Combination Sum backtracking rule to allow element reuse? | When recursing after picking `candidates[i]`, pass `i` (not `i+1`) as the start index. This allows picking the same element again. |
| 9 | In the YouTube mock interview, what three paths should you describe first? | Upload path (client → S3 → transcode → CDN origin), Streaming path (client → CDN edge → ABR), Discovery path (search via Elasticsearch, recommendations via two-tower). |
| 10 | Why use epsilon comparison (`< 1e-6`) instead of `== 24` in the 24 Game? | Floating-point arithmetic on doubles accumulates rounding errors. Division especially: `8 / (3 - 8/3) = 24.000000000000004`. Epsilon comparison handles this correctly. |
| 11 | What does the `k` dimension represent in Remove Boxes `dp[l][r][k]`? | `k` = the number of extra boxes identical to `boxes[l]` already attached to the left of box l (from previous merges). Removing the left group scores `(k+1)²` points. |
| 12 | How do you implement backtracking in Robot Room Cleaner without a map? | Virtual coordinate tracking: maintain `(row, col)` in a visited set. To go back: `turnRight x2`, `move`, `turnRight x2` (180° turn + move + 180° turn restores original position and direction). |
| 13 | In Palindrome Partitioning, how do you precompute `isPalin[i][j]`? | `isPalin[i][j] = (s[i] == s[j]) && (j-i <= 2 || isPalin[i+1][j-1])`. Fill from bottom-right to top-left (decreasing i, increasing j). |
| 14 | State three YouTube numbers that anchor CDN architecture decisions. | 1B hours/day watched → need 450 PB/day CDN egress. 95%+ CDN hit rate → origin traffic is < 5%. 17,000+ Netflix OCAs embedded in ISPs → sub-50ms latency globally. |
| 15 | What is the hybrid fan-out strategy for YouTube subscription feeds and when does it switch? | Push (fan-out on write) for channels with < 1M subscribers. Pull (fan-out on read) for channels > 1M subscribers — 100M write fan-outs per upload is too expensive synchronously. |
| 16 | What is the key invariant that makes LC 41 First Missing Positive O(n) despite the nested while loop? | Each element is swapped into its correct position at most once. Once `nums[i] == i+1`, that element is never moved again. So total swaps across all iterations of the outer loop is at most n. |
| 17 | In 3Sum, after finding a zero-sum triplet, how do you skip duplicates before moving left and right pointers? | Skip while `nums[left] == nums[left+1]` (increment left) and while `nums[right] == nums[right-1]` (decrement right), then do the final `left++; right--`. |
| 18 | How does LC 238 Product of Array Except Self achieve O(1) extra space? | Use the output array as the left-pass accumulator: `output[i] = product of all nums[0..i-1]`. Then do a right pass with a single `right` variable: `output[i] *= right; right *= nums[i]`. No extra arrays needed. |
| 19 | What is the difference between L4 and L7 load balancers? | L4 operates at TCP/UDP (sees IP + port only, cannot inspect HTTP). L7 operates at HTTP (sees headers, URL, cookies; can route by path or cookie; enables sticky sessions by cookie). |
| 20 | Why do sticky sessions via IP hash break when auto-scaling adds a new backend node? | Adding a node changes the hash ring mapping; the same client IP now hashes to a different backend that does not have the session. Fix: externalize session state to Redis so any node can serve any client. |
| 21 | In LC 25 Reverse Nodes in K-Group, after reversing k nodes, what does the original head node become and what should its `next` pointer be set to? | The original head becomes the tail of the reversed chunk. Its `next` should point to the result of recursively calling `reverseKGroup` on the remaining list starting at `curr`. |
| 22 | In Daily Temperatures (LC 739), what invariant does the monotonic stack maintain? | The stack always holds indices of days in decreasing order of temperature. When a warmer day is found at index i, all indices with temperature less than `temperatures[i]` are popped and their wait time is computed as `i - poppedIndex`. |
| 23 | Why do you move the fast pointer n+1 steps (not n) in LC 19 Remove Nth From End? | The slow pointer must stop at the node just before the one to delete. Moving fast n+1 steps creates a gap of n+1 between fast and slow, so when fast reaches null, slow is at the predecessor of the target node. |
| 24 | What is the key risk of write-behind (write-back) caching? | If the cache crashes before the async flush reaches the database, writes that were acknowledged to the client are permanently lost. Mitigation: use a durable write-ahead log or write-behind queue on persistent storage. |
| 25 | What is cache stampede (thundering herd) and what is the Redis-native solution? | When a hot cache key expires simultaneously for many clients, all miss and query the DB at once, overwhelming it. Redis solution: `SET lock:key 1 NX EX 5` — only one process wins the lock and regenerates the key; others wait and read the freshly populated value. |
| 26 | In the post-order DFS arm pattern, what is the difference between the value returned from `dfs(node)` and the value used to update `maxResult`? | `dfs(node)` returns `1 + max(left, right)` — the longest single arm from this node upward (for the parent to extend). `maxResult` is updated with `left + right` — the path that passes through this node as the peak, which cannot extend further upward. |
| 27 | In LC 2246, why do you skip a child arm if the child's character equals the parent's character? | The problem requires adjacent characters in the path to be different. A child with the same character as its parent cannot be part of the same path, so its arm length contribution is set to 0 (skipped entirely). |
| 28 | In BFS level-order traversal, why must you snapshot `int size = q.size()` at the start of each level loop? | Because you add new children to the queue inside the same loop. If you check `q.isEmpty()` or `q.size()` dynamically, you would process next-level nodes as part of the current level. Snapshotting size isolates exactly the current level. |
| 29 | When should you shard a database instead of just adding read replicas? | When write throughput exceeds what a single primary can handle (roughly > 10K writes/sec). Read replicas copy the primary's data and only help read load — they do not reduce write load on the primary. Sharding distributes writes across multiple primaries. |
| 30 | In Cassandra with replication factor 3 and QUORUM consistency, how many node failures can the cluster survive for reads and writes? | QUORUM requires (3/2)+1 = 2 responses. With 3 nodes and needing 2, the cluster can survive 1 node failure and still satisfy QUORUM reads and writes. |
| 31 | In LC 2045 Second Minimum Time, how do you compute the wait time at a node when you arrive at time `t` and the signal change period is `change`? | `int period = t / change; int waitTime = (period % 2 == 1) ? (period + 1) * change : t;` — if you arrive during a red phase (odd period), wait until the next green; if already green, proceed immediately. |
| 32 | In LC 417 Pacific Atlantic Water Flow, why do you do BFS from the ocean borders instead of BFS from each cell toward the ocean? | Forward flow from each cell would require checking all possible paths downhill. Reverse flow from the ocean border is simpler: BFS uphill (to cells with height >= current) naturally finds all cells that can drain to that ocean. Intersection of the two reachable sets gives the answer. |
| 33 | What is the Union-Find path compression optimization and what does it achieve? | During `find(x)`, set `parent[x] = find(parent[x])` recursively, making every node on the path point directly to the root. This flattens the tree so future finds are O(1). Combined with union by rank, amortized cost per operation is O(α(n)) ≈ constant. |
| 34 | In Kahn's topological sort, how do you detect a cycle in the graph? | After BFS completes, if `order.size() != n`, there is a cycle. Nodes in the cycle never reach in-degree 0, so they are never added to the queue and never appear in the result. |
| 35 | What is a Dead Letter Queue (DLQ) in Kafka and what problem does it solve? | A DLQ is a separate topic to which messages are routed after N failed processing retries. It solves the poison-pill problem: without a DLQ, a message that always fails blocks all subsequent messages in the same partition, halting the consumer indefinitely. |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |
