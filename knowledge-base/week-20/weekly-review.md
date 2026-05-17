# Week 20 Review — Days 134–140
**Phase 3 — Mock Interviews | Month 5**

> **Note:** This is the second week of Slot 14 (Trees Hard closeout + Graphs Hard + Uber/Lyft design closeout). Day 134 finishes Trees Hard; Days 135–140 cover Graphs Hard and the Uber/Lyft capstone mock.

---

## Phase
Phase 3 — Mock Interviews, Month 5. Full interview simulation fluency week — every session includes at least one cold-solve under timer with narration, system design from memory, and behavioral STAR response.

---

## Patterns Covered This Week

- Post-Order BST Validation with Sum Tracking (Max Sum BST — Day 134)
- Stack-Based Tree Reconstruction from Depth Encoding (Recover Tree from Preorder — Day 134)
- Topological Sort from Pairwise Word Constraints (Alien Dictionary — Day 135)
- DFS + Memoization on Grid with Strict Monotone Constraint (Longest Increasing Path — Day 135)
- Union-Find with Directed Two-Candidate Edge Analysis (Redundant Connection II — Day 136)
- Union-Find Island Labeling + Neighbor Size Aggregation (Making A Large Island — Day 136)
- Two-Level Topological Sort (Items Within Groups — Day 137)
- Topological Sort + DP Critical Path (Parallel Courses III — Day 137)
- Topological Sort + Color Frequency DP on DAG (Largest Color Value — Day 138)
- BFS Min-Heap Boundary Expansion (Trapping Rain Water II — Day 138)
- Post-Order Tree DP Cold Solve × 2 (Days 139–140)
- Topological Sort from Constraints Cold Solve (Day 140)

---

## System Design Topics Covered

**Uber / Lyft — Days 134–140:**
- **Day 134 — Dynamic Pricing & Surge:** Supply/demand per H3 zone in Redis; surge multiplier formula; ML inference overlay; fare lock with `fare_lock_id` in Redis (2-min TTL); rule-based floor + ML ceiling.
- **Day 135 — Trip Service & State Machine:** 5-state machine (REQUESTED → DRIVER_ACCEPTED → DRIVER_ARRIVED → IN_PROGRESS → COMPLETED/CANCELLED); PostgreSQL optimistic locking (version column); Kafka event per transition; idempotent state transitions.
- **Day 136 — ETA & Route Calculation:** Contraction Hierarchies for sub-millisecond routing; zone-pair route cache in Redis (15-min time bucket, 5-min TTL); traffic weight updates from GPS traces; Google Maps API fallback.
- **Day 137 — Notification Service:** Kafka `trip-events` → channel workers (push/SMS/in-app); dedup via Redis NX key `notif:{trip_id}:{event_type}` EX 60; APNs/FCM with delivery tracking; retry with exponential backoff.
- **Day 138 — Payment Service:** Idempotency key = `trip_id + attempt_seq`; tokenized card storage (no PAN); async charge with "processing" UI; driver payout via ACH batch (daily/weekly options); ML fraud scoring.
- **Day 139 — Driver App Backend:** WebSocket Driver Gateway (sticky sessions via consistent hash); adaptive GPS polling (4s in-trip, 8s idle); buffered batch on reconnect; geofencing push for surge/restricted zones.
- **Day 140 — Full End-to-End Integration:** Complete Uber/Lyft architecture diagram; critical path (request → match → notify); location update pipeline; all key numbers; two deep dives practiced.

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Post-Order BST Validation (Max Sum BST) | | |
| Stack-Based Tree Reconstruction | | |
| Topo Sort from Word Constraints (Alien Dict) | | |
| DFS + Memo on Grid (Longest Increasing Path) | | |
| Union-Find with Two-Candidate Directed Edge | | |
| Union-Find Island Labeling + Expansion | | |
| Two-Level Topological Sort | | |
| Topo Sort + DP Critical Path | | |
| Topo Sort + Color DP on DAG | | |
| BFS Min-Heap Boundary Expansion (3D Water) | | |

---

## Weekly Flashcard Deck
*(Days 134–140 = 35 cards — 5 per day)*

| Q | A |
|---|---|
| In LC 1373, why does a null node return MAX_VALUE for min and MIN_VALUE for max? | A null node imposes no BST constraint; any real node value satisfies comparison with MAX/MIN sentinels |
| In LC 1373, what four values does the post-order helper return? | (isBST: 1/0, minVal, maxVal, subtreeSum) |
| In LC 1028, what does the number of dashes before a value represent? | The depth of that node in the reconstructed tree |
| In LC 1028, when does a new node become the right child vs left child? | Right child if the parent already has a left child; otherwise left child |
| In Uber's surge pricing, what triggers the multiplier to increase? | Demand (open requests per zone) exceeds supply (available drivers per zone) by a threshold ratio |
| In LC 269, what is the edge case that invalidates the input before graph construction? | A word that is a proper prefix of the next word but appears before it (e.g., "abc" before "ab") |
| In LC 269, why break after the first differing character in adjacent words? | Only the first difference gives a definitive ordering constraint; later characters have unknown relative order |
| In LC 329, why is no visited array needed? | Strictly increasing values prevent cycles — each DFS step increases the grid value, making revisit impossible |
| In LC 329, what does `memo[r][c]` represent? | Length of the longest increasing path starting at cell (r, c) |
| In Uber's Trip Service, how does optimistic locking prevent race conditions? | Each trip row has a version column; update only succeeds WHERE version = expected_version; concurrent update fails (stale version) → 409 Conflict |
| In LC 685, what are the two failure modes for the extra directed edge? | (1) Node has two parents (two candidate edges); (2) cycle with all nodes having single parents |
| In LC 685, why try removing candidate2 first? | Candidate2 is the last edge pointing to the double-parent node; if graph is valid without it, that's the answer |
| In LC 827, why use a HashSet of roots when summing neighbor islands? | Deduplicates — same island may have multiple cells bordering the 0 cell; root ID is canonical representative |
| In LC 827, what is returned when there are no zero cells? | `n * n` — the entire grid is already one island |
| In Uber's ETA service, what is Contraction Hierarchies? | Preprocessing step adding shortcut edges; bidirectional Dijkstra visits O(s) important nodes (100-1000) vs millions in plain Dijkstra |
| In LC 1203, why assign unique group IDs to items with group == -1? | Allows group-level topo sort to handle their ordering without special-casing ungrouped items |
| In LC 1203, how is a group-level edge derived? | If item `pre` (group A) must come before item `next` (group B) and A ≠ B, add edge A → B in the group graph |
| In LC 2050, what does `earliest[course]` represent? | Earliest completion time for `course` — accounts for the longest prerequisite chain |
| In LC 2050, why is the answer `max(earliest[i])` rather than `sum`? | Courses run in parallel; bottleneck is the longest (critical path), not the total of all durations |
| In Uber's notification service, what does Redis NX key `notif:{trip_id}:{event_type}` prevent? | Duplicate notifications for the same event within 60 seconds |
| In LC 1857, how do you detect a cycle? | If `processed < n` after Kahn's BFS — some nodes never reached in-degree 0 due to cycle |
| In LC 1857, why initialize `dp[node][color] = 1` only when entering the queue? | The node's own color contributes 1 to paths ending at it; initializing before queue entry would be premature |
| In LC 407, why push `max(boundary_height, neighbor_height)` to the heap? | Water cannot escape below the current boundary; max ensures the inward boundary never drops below the current water level |
| In LC 407, why start with boundary cells rather than interior? | Water flows outward; computing from boundary inward captures all cells sealed by higher surrounding walls |
| In Uber's Payment Service, how does idempotency prevent double charges? | Each charge uses `payment_intent_id = trip_id + attempt_seq`; gateway deduplicates by this key and returns original result on retry |
| In Uber's critical match path, what is the order of operations after a rider requests a ride? | (1) Fare estimate + fare_lock; (2) Redis Geo search for nearby drivers; (3) Redis NX lock on first candidate; (4) Notify driver; (5) Driver accepts → Trip record created; (6) Notify rider |
| In LC 297 deserialization, why use a Queue of tokens rather than an index? | Queue naturally advances on each token consumption; index tracking is error-prone for recursive calls with shared mutable state |
| What Uber component prevents a driver from being matched to two riders at once? | Redis NX lock: `SET driver:{id}:status MATCHED NX PX 30000`; atomic; only one Matching Service instance wins |
| Name the 5 states in Uber's trip state machine. | REQUESTED → DRIVER_ACCEPTED → DRIVER_ARRIVED → IN_PROGRESS → COMPLETED (or CANCELLED at any state) |
| In Uber's driver app, what happens when connectivity is lost mid-trip? | GPS pings are buffered locally (up to ~60); sent as batch on reconnect; server processes in sequence; trip state mutations retry with exponential backoff until acknowledged |
| What is the GPS ping interval in Uber's driver app and when does it change? | 4 seconds when in a trip or near a ride request; 8 seconds when idle (adaptive polling to conserve battery) |
| In LC 560 (revision), what is the map initialized with and why? | `{0: 1}` — represents the empty prefix with sum 0; handles subarrays starting from index 0 whose sum equals k |
| In LC 543 (warm-up), how is `depth` different from `diameter` at each node? | `depth` = 1 + max(left, right) — returned to parent; `diameter` = left + right — updated globally, never returned |
| What are the two deep dives practiced in the Uber capstone mock (Day 140)? | (1) Ride request to match critical path (Redis Geo + NX lock + Kafka notify); (2) Location update pipeline (WebSocket → Kafka → Redis Geo → Cassandra) |
| What is the fare lock mechanism in Uber's pricing and why does it exist? | A `fare_lock_id` in Redis with 2-min TTL stores the estimated fare; on trip completion, Payment Service retrieves it to charge the locked price; prevents bait-and-switch after rider commits |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

---

## Week 20 → Slot 15 Transition

**Consolidate before moving on:**
- Uber/Lyft: draw the full architecture from memory in 5 minutes — API Gateway, 8 backend services, their data stores, and the two key Kafka topics (`trip-events`, `driver-location`).
- Trees Hard patterns (Days 131–134): post-order DP, serialization, inorder violation, greedy state DP, coordinate BFS, re-rooting — all should trigger in ≤ 60 seconds on sight.
- Graphs Hard patterns (Days 135–140): topo sort variants (constraint-derived, two-level, color DP), DFS+memo grid, Union-Find directed/island — identify and code without warm-up.
- Slot 15 begins: DP Hard + Backtracking Hard (Days 141–150) + YouTube/Netflix full design. DP Hard includes interval DP, bitmask DP, digit DP, and tree DP at the hardest level.
