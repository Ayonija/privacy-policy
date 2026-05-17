# Week 19 Review — Days 127–133
**Phase 3 — Mock Interviews | Month 5**

> **Note:** Days 127–130 are in Slot 13 (advanced sliding window + two-pointer hard); Days 131–133 begin Slot 14 (Trees Hard). This review covers all 7 days.

---

## Phase
Phase 3 — Mock Interviews, Month 5. Each day includes 2 hard problems cold-solved under time pressure plus a system design module — shifting from pattern learning to full interview simulation fluency.

---

## Patterns Covered This Week

- Sliding Window DP + Monotonic Deque (Constrained Subsequence Sum — Day 130)
- Two Pointer with Running Product Score (Count Subarrays Score < K — Day 130)
- Two Pointer Suffix Comparison (Last Substring Lexicographic — Day 128)
- Sort + Dedup + Sliding Window (Min Operations Array Continuous — Day 128)
- Merge Sort + Cross-Partition Counting (Reverse Pairs — Day 129)
- Binary Search on Length + Rolling Hash (Longest Duplicate Substring — Day 129)
- Post-Order Tree DP (Maximum Path Through Node — Day 131)
- Preorder Serialization with Null Sentinel (Serialize/Deserialize Tree — Day 131)
- Inorder Traversal Violation Detection (Recover BST — Day 132)
- Greedy Post-Order State DP (Minimum Camera Cover — Day 132)
- BFS + Column/Row Coordinate Tracking (Vertical Order Traversal — Day 133)
- Two-Pass Re-rooting DP (Sum of Distances in Tree — Day 133)

---

## System Design Topics Covered

**Twitter/X (Slot 13 closeout):**
- **Day 130 — Full End-to-End:** Synthesized all Twitter/X components; architecture diagram with six services; deep dives on write path + rate limiting; key metrics table (400M DAU, 500M tweets/day, 100ms p99 timeline).

**Uber / Lyft (Slot 14 start):**
- **Day 131 — Overview & Scale:** 500M trips/year, 5M drivers, 15M trips/day; six core services; geospatial index choice (Redis Geo / H3); driver state machine; matching latency target ≤ 5s.
- **Day 132 — Ride Matching & Geospatial:** Redis Geo `GEOSEARCH`; H3 hexagonal cells; Redis NX atomic lock for driver assignment (prevents double-booking); driver location update pipeline.
- **Day 133 — Real-Time Location Tracking:** WebSocket driver connections; 1.25M location events/sec; Kafka `driver-location` topic; Location Worker updating Redis Geo; Cassandra for location history; staleness detection.

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Sliding Window DP + Monotonic Deque | | |
| Rolling Hash + Binary Search | | |
| Merge Sort + Cross-Partition Count | | |
| Post-Order Tree DP (path sum style) | | |
| Tree Serialization / Deserialization | | |
| Inorder Violation Detection (Recover BST) | | |
| Greedy Camera Placement (State DP) | | |
| BFS + Coordinate Tracking | | |
| Re-rooting DP (Sum of Distances) | | |

---

## Weekly Flashcard Deck
*(Days 127–133 = 35 cards — 5 per day)*

| Q | A |
|---|---|
| What is the DP recurrence for LC 1425 (Constrained Subsequence Sum)? | `dp[i] = nums[i] + max(0, max(dp[i-k], ..., dp[i-1]))`; must include `nums[i]`; only add window max if positive |
| How does the monotonic deque reduce LC 1425 from O(nk) to O(n)? | Deque stores indices in decreasing dp-value order; max of last k dp values is at front in O(1); each index pushed/popped at most once |
| In LC 2302, why is `score = sum × length` monotone as the window grows? | Adding a positive element increases both sum and length; score strictly increases; fixed-right boundary for valid left |
| What is LC 2302's counting logic at each right pointer? | After shrinking left so score < k, all subarrays from left to right are valid → add `right - left + 1` |
| Name the six core services in the Twitter/X design. | Tweet Service, Timeline Service, User Graph Service, Search Service, Notification Service, Media Service |
| In LC 1163, why is the answer always a suffix of `s`? | Any non-suffix substring has a suffix starting at the same character that is lexicographically ≥ it |
| In LC 1163, why skip `k` positions when advancing the losing pointer? | All suffixes starting between the old position and `i + k + 1` are already proven smaller than the winning candidate |
| What preprocessing step is critical in LC 2009 before the sliding window? | Sort and deduplicate — duplicates can be freely replaced and should not count as valid elements |
| In LC 2009, what does the sliding window on the deduplicated array represent? | Distinct values that already fit in `[nums[left], nums[left] + n - 1]`; replacements needed = `n - window_size` |
| In LC 493, why count reverse pairs before the merge step? | After merging, you can't distinguish which elements came from left vs right; the `i < j` constraint is lost |
| In LC 493's counting step, why does a two-pointer across sorted halves work? | Both halves are sorted; if `left[i] > 2 * right[j]`, then it's > all `right[0..j]` — monotonicity allows j to advance without going back |
| What is the Mersenne prime used in Rabin-Karp rolling hash and why? | `(1L << 61) - 1`; allows fast modular reduction via bit ops; low collision probability |
| In LC 1044, why verify string equality on a hash match? | Hash collisions cause false positives; string comparison confirms a genuine duplicate |
| What is the "boundary burst" vulnerability in fixed-window rate limiting? | 2N requests in a short period — N at end of window W and N at start of W+1; both windows count separately |
| In LC 124, why does `maxGain` return `node.val + max(leftGain, rightGain)` not the sum of both? | The path going up can only use one branch; using both creates a fork that can't extend to the parent |
| In LC 124, why take `max(0, maxGain(child))` rather than using the raw value? | Negative subtree gains reduce the path sum; clamping to 0 means you simply don't include that subtree |
| In LC 297, what does the "#" sentinel in serialization represent? | A null child pointer — without it you can't reconstruct the tree structure |
| Why is preorder preferred over inorder for tree serialization? | Preorder gives the root first; you can create the node immediately and recursively build left then right |
| In Uber's architecture, why store driver locations in Redis Geo rather than PostgreSQL? | 1.25M writes/second; Redis Geo handles sub-millisecond latency; PostgreSQL would bottleneck |
| In LC 99, when does inorder traversal produce exactly one inversion rather than two? | When the two swapped nodes are adjacent in inorder order — one `prev.val > curr.val` pair |
| In LC 99, why update `second = curr` at every inversion rather than just the first? | The actual second displaced node may be far away; there can be two inversions; the later one's `curr` is correct |
| In LC 968, why do null nodes return state 1 (covered)? | Null nodes don't exist; returning 1 prevents unnecessary cameras at leaf parents |
| What is the greedy argument for LC 968's camera placement at parent of uncovered leaf? | Camera at parent covers leaf + parent + grandparent — strictly better than camera at leaf |
| In Uber's matching, what does the Redis NX lock prevent? | Two matching workers assigning the same driver to different riders simultaneously |
| In LC 987, what sorts nodes within the same (col, row) position? | Node value ascending — min-heap (PriorityQueue) per coordinate cell |
| In LC 834, what does `count[v]` represent? | Subtree size rooted at v, including v itself |
| In LC 834's re-rooting formula, why add `n - count[child]`? | When root moves from parent to child, the `n - count[child]` nodes outside child's subtree each get 1 farther |
| In LC 987, why use TreeMap for column grouping? | Keeps columns in sorted order automatically; O(log n) insert and in-order traversal |
| Why partition Kafka by driver_id in Uber's location pipeline? | Preserves ordering of events for each driver; same consumer handles all events for a given driver |
| How does Uber detect stale driver locations? | Background job checks `driver:{id}:updated_at` in Redis; if older than 30s, marks STALE and removes from geospatial index |
| What is Twitter's celebrity threshold for switching from push to pull fan-out? | ~50,000 followers — above this, tweets are not pushed to follower timelines; fetched at read time instead |
| What Twitter component computes trending hashtags in real time? | Flink streaming with time-decayed sliding window; results cached in Redis |
| In Uber's trip state machine, name the four main driver states. | OFFLINE → AVAILABLE → MATCHED → IN_TRIP → AVAILABLE |
| What is the target match latency for Uber? | ≤ 5 seconds from ride request to driver assignment |
| What data structure does Uber use for driver location queries in a city? | Redis Geo (sorted set with geohash scores); `GEOSEARCH` for radius queries |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

---

## Week 19 → Week 20 Transition

**Consolidate before moving on:**
- Slot 13 closeout: can you draw Twitter/X end-to-end (write path + read path + caching + rate limiting) from memory in 5 minutes?
- Slot 14 opening: Trees hard patterns reviewed — post-order DP, serialization, inorder violation, greedy camera DP, BFS coordinates, re-rooting DP. All should feel triggerable on sight.
- Week 20 continues with more Trees Hard (Days 134) and then Graphs Hard (Days 135–140) plus Uber/Lyft deep dives (pricing, trip service, ETA, payments, driver app, full integration).
