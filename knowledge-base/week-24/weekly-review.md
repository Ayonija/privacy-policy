# Week 24 Review — Days 162–168

## Phase
Phase 4 — Final Sprint | Month 6 | Timed OA Simulation + LLD Deep Dive

## Patterns Covered This Week

- **Day 162:** Sweep Line + Binary Search on Event Arrays
- **Day 163:** Min-Heap Enumeration of Ranked Sums / Combinations
- **Day 164:** Prefix/Suffix DP with Heap-Maintained Running Optimum
- **Day 165:** Binary Search on Answer Value (Feasibility Check)
- **Day 166:** Sliding Window on Positions (Collect → Prefix Sum → Median Cost)
- **Day 167:** Two-Pass DFS (Precompute + Propagate)
- **Day 168:** Offline Queries + Min-Heap (Sort + Sweep)

## System Design Topics Covered (LLD)

| Day | LLD Topic | Key Design Decision |
|-----|-----------|---------------------|
| 162 | Library Management System ER Diagram | Use `SELECT available_copies FROM Books WHERE isbn=? FOR UPDATE` inside a transaction to prevent two patrons checking out the last copy simultaneously; the row-level lock ensures atomicity |
| 163 | Movie Ticket Booking DB Schema (BookMyShow) | `PRIMARY KEY (seat_id, showtime_id)` on BookingSeats acts as a uniqueness constraint — the second concurrent INSERT fails with a duplicate key error, preventing double-booking without application-level locking |
| 164 | Hotel Booking DB Schema | For high-demand bookings, use a Redis distributed lock (`SET room:{hotel_id}:{room_type_id}:{check_in} LOCKED NX EX 30`) — NX ensures mutual exclusion, EX prevents deadlock if the lock holder crashes |
| 165 | ATM Machine State Machine Design | Use the State pattern so each state's behavior is fully encapsulated in its own class; adding new states (e.g., MaintenanceModeState) requires only a new class implementing ATMState — no changes to ATM or existing state classes (Open/Closed Principle) |
| 166 | LRU Cache + LFU Cache Class Design | LFU uses a `LinkedHashSet` per frequency bucket — insertion order within the set gives O(1) LRU tie-breaking among keys at the same frequency, enabling O(1) eviction without an extra linked list |
| 167 | Social Media Platform DB Schema | Hybrid fan-out strategy: push (fan-out on write) for users with < 1,000,000 followers; pull (fan-out on read) for celebrity accounts; at read time merge the pre-computed push feed with freshly pulled celebrity posts |
| 168 | REST API Design Patterns | Idempotency keys turn non-idempotent POST operations into effectively-idempotent ones — the server caches the response (Redis TTL 24h) so retries on network failure never double-process; cursor pagination uses a composite index seek (O(log N)) vs offset pagination which degrades at depth |

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Sweep Line + Binary Search on Event Arrays | | |
| Min-Heap Enumeration of Ranked Sums / Combinations | | |
| Prefix/Suffix DP with Heap-Maintained Running Optimum | | |
| Binary Search on Answer Value (Feasibility Check) | | |
| Sliding Window on Positions (Collect → Prefix Sum → Median Cost) | | |
| Two-Pass DFS (Precompute + Propagate) | | |
| Offline Queries + Min-Heap (Sort + Sweep) | | |

## Weekly Flashcard Deck (35 cards — Days 162–168)

### Day 162 Flashcards (5 cards)
| Q | A |
|---|---|
| In LC 2251, what is the difference between upperBound and lowerBound and why does each apply to starts vs ends? | upperBound(starts, t) counts elements ≤ t (intervals that have started by time t, inclusive). lowerBound(ends, t) counts elements < t (intervals whose end is strictly before t, meaning they finished before the person arrived). Active = started - finished_before. |
| What is the critical mistake to avoid in LC 1905 (Count Sub Islands)? | Do not return early from DFS when you discover a cell where grid2==1 but grid1==0. You must continue the DFS to mark all cells of the island as visited, otherwise future DFS calls will re-enter those cells and double-count islands. |
| How does multi-source BFS work in LC 994 (Rotting Oranges)? | Enqueue all rotten oranges (value 2) into the queue simultaneously before starting BFS. This ensures all rotten oranges spread in the same time step, simulating simultaneous rot. The level (time) counter increments after each full BFS level is processed. |
| What SQL construct prevents two concurrent checkouts of the last library book? | SELECT available_copies FROM Books WHERE isbn=? FOR UPDATE inside a transaction. The FOR UPDATE clause acquires a row-level exclusive lock; only one transaction proceeds; others wait; the second transaction reads the updated (decremented to 0) value after the first commits. |
| Name the two indexes that make active-loan queries fast for a library system. | idx_loans_patron on Loans(patron_id) — for "how many books does patron X have?"; idx_loans_active on Loans(isbn, return_date) WHERE return_date IS NULL — partial index covering only active loans for a given ISBN, avoiding full table scan. |

### Day 163 Flashcards (5 cards)
| Q | A |
|---|---|
| What are the two heap pushes in LC 2386's reduction enumeration and what does each represent? | Extend push: (reduction + sorted[i+1], i+1) — adds the next element to the current subset, increasing the reduction from maxSum. Swap push: (reduction - sorted[i] + sorted[i+1], i+1) — replaces sorted[i] with sorted[i+1] in the subset, keeping subset size the same but swapping to the next element. |
| In LC 373, why do you only seed the heap with (nums1[i]+nums2[0]) and not all (i, j) pairs? | nums2[0] is the smallest element in nums2, so (nums1[i], nums2[0]) gives the smallest possible sum for each fixed i. All other pairs (nums1[i], nums2[j]) with j>0 can be reached by expanding from (i, 0) via the heap, ensuring we enumerate pairs in ascending sum order without redundancy. |
| What is the invariant that makes the PRIMARY KEY on (seat_id, showtime_id) sufficient to prevent double-booking? | The database engine enforces uniqueness atomically — two concurrent INSERTs with the same (seat_id, showtime_id) cannot both succeed; exactly one will get a duplicate key error, causing the losing transaction to roll back. This requires no application-level locking. |
| What is the greedy invariant maintained in LC 1405 (Longest Happy String)? | At each step, always pick the most frequent available character unless the last two characters in the result are already that character. In that case, pick the second most frequent. This greedy choice maximizes string length because not using the most frequent character now would waste it — it must be used as soon as possible. |
| What does the expires_at column in the Bookings table accomplish in the BookMyShow schema? | It implements a time-boxed seat lock: a seat reserved (status=LOCKED) must be paid for before expires_at; if not, a background job releases the lock by deleting from BookingSeats and marking the booking CANCELLED. This prevents seats from being held indefinitely by users who abandon checkout. |

### Day 164 Flashcards (5 cards)
| Q | A |
|---|---|
| In LC 2163, why do you use a max-heap for prefix and a min-heap for suffix? | Prefix = min sum of n elements → keep the n smallest → evict the largest when size exceeds n, so use a max-heap (the root is the largest, easy to evict). Suffix = max sum of n elements → keep the n largest → evict the smallest when size exceeds n, so use a min-heap (the root is the smallest, easy to evict). |
| What is the valid split point range in LC 2163 and why? | i ranges from n-1 to 2n-1 (inclusive). At i=n-1: left half is nums[0..n-1] (exactly n elements, the minimum needed). At i=2n-1: right half is nums[2n..3n-1] (exactly n elements, the minimum needed). Outside this range, one half has fewer than n elements and cannot contribute a valid n-element selection. |
| In LC 416 (Partition Equal Subset Sum), why must the inner loop run from target down to num? | The inner loop processes the current num only once per knapsack iteration (0/1 knapsack — each element used at most once). Iterating from high to low ensures that when dp[j] is updated using dp[j-num], dp[j-num] still reflects the state before the current num was considered. Iterating low to high would allow num to be used multiple times (unbounded knapsack). |
| In LC 1027 (Longest Arithmetic Subsequence), what does dp[i][diff] store? | The length of the longest arithmetic subsequence ending at index i with common difference diff. The value includes both endpoints, so a two-element subsequence [nums[j], nums[i]] stores 2. When computing dp[i][diff]: dp[j].getOrDefault(diff, 1) + 1 — the 1 default represents nums[j] alone (length 1 before appending nums[i]). |
| What two Redis lock properties make SET NX EX safe for hotel booking concurrency? | NX (Not eXists): only sets the key if it does not already exist — ensures mutual exclusion, only one request acquires the lock. EX (EXpiry in seconds): automatically releases the lock after the TTL even if the lock holder crashes — prevents deadlock where a crashed process holds the lock forever. |

### Day 165 Flashcards (5 cards)
| Q | A |
|---|---|
| In LC 2040, how do you count pairs with product ≤ mid when nums1[i] is negative? | When nums1[i] < 0, dividing both sides of nums1[i]*nums2[j] ≤ mid by nums1[i] flips the inequality: nums2[j] ≥ ceil(mid / nums1[i]). Count = nums2.length - lowerBound(nums2, ceil(mid / nums1[i])). Use long arithmetic and handle Java's round-toward-zero division carefully for negative operands. |
| What are the lo and hi bounds for LC 875 (Koko Eating Bananas) and why? | lo=1 (Koko eats at least 1 banana/hour), hi=max(piles) (eating faster than the largest pile is wasteful — it provides no benefit and the answer cannot exceed max(piles)). The feasibility check is sum(ceil(pile/k)) ≤ h. |
| What are the lo and hi bounds for LC 1011 (Capacity To Ship) and why? | lo=max(weights): the ship must be able to carry the heaviest single package or shipping is impossible. hi=sum(weights): a ship with this capacity ships everything in exactly 1 day — always feasible. The feasibility check is the greedy day-counting simulation. |
| What is the Open/Closed Principle and how does the State pattern in ATM satisfy it? | Open/Closed: classes should be open for extension but closed for modification. State pattern satisfies it by encapsulating each state's behavior in its own class. Adding a new state (e.g., MaintenanceModeState) requires creating a new class implementing ATMState — no existing state classes or the ATM class itself need modification. |
| In the ATM State pattern, which state must be stateful and why? | CardInsertedState must be stateful — it stores cardId (needed for PIN validation with BankService) and pinAttempts (needed to decide whether to retry or capture/eject the card). All other states can be stateless singletons because they do not need to remember per-session data between method calls. |

### Day 166 Flashcards (5 cards)
| Q | A |
|---|---|
| What data structure enables O(1) LRU eviction? | A doubly linked list with sentinel head/tail; the node just before the tail sentinel is always the LRU item and can be removed in O(1). |
| Why use `LinkedHashSet` instead of `HashSet` in LFU frequency buckets? | `LinkedHashSet` preserves insertion order, making the first element the oldest at that frequency — enabling O(1) LRU tie-breaking without an extra linked list. |
| In "minimum adjacent swaps for K consecutive ones," why collect positions into array P instead of working on the raw array? | It reduces the problem from O(N) array swaps to O(K) median-cost on positions, and prefix sums make each window cost O(1). |
| What is the offset term in LC 1703 and why must it be subtracted? | The positions P[l..r] have gaps between them; adjacent swaps cannot skip indices. The offset `P[l]*K + K*(K-1)/2` accounts for the non-swap positional spread, leaving only the true swap count. |
| In the sliding-window frequency diff approach (LC 567), what does `nonZero` count and when is the answer found? | `nonZero` counts characters whose frequency differs between the current window and s1. The answer is found when `nonZero == 0` (all 26 character frequencies match). |

### Day 167 Flashcards (5 cards)
| Q | A |
|---|---|
| In LC 2458, what two things does DFS pass 1 compute and what does DFS pass 2 propagate? | Pass 1 computes `depth[node]` (root-to-node distance) and `height[node]` (node-to-deepest-leaf). Pass 2 propagates `maxHeightWithout` downward — the best tree height achievable if this node's subtree were removed. |
| Why track top-2 child heights in the two-pass DFS instead of just the best? | Each child's complement needs the best height from its siblings. If you only stored the best, the child with the best height would get an incorrect complement (it cannot use its own height). Top-2 lets you serve both children correctly in O(1). |
| What index is essential for paginated feed queries and why? | `INDEX (user_id, created_at DESC)` on the Feeds table — it lets the DB seek directly to the user's latest posts without a full table scan, making page N as fast as page 1. |
| What prevents a user from liking the same post twice at the DB level? | `PRIMARY KEY (user_id, post_id)` on the Likes table — inserting a duplicate throws a unique constraint violation, which the application catches and returns "already liked". |
| In LC 1443 (Collect All Apples), when do you add 2 to the child's cost? | When `childCost > 0` (the child's subtree has at least one apple) OR `hasApple[child]` is true — in both cases you must make the round trip (2 edges) to that child. If neither is true, you can skip the child entirely. |

### Day 168 Flashcards (5 cards)
| Q | A |
|---|---|
| In the offline-query-min-heap pattern, what two things happen at each query step? | (1) Add all intervals with `start <= query` to the heap. (2) Expire (pop) all intervals with `right < query`. The heap top is then the smallest valid covering interval. |
| Why must queries be processed in sorted order in LC 1851? | Because the interval pointer only moves forward (sweep), and the heap only accepts intervals with `start <= current query`. Processing out of order would require resetting the pointer, making it O(N*Q). |
| What is the SQL equivalent of cursor pagination and why is it fast? | `WHERE (created_at, id) < (cursor_ts, cursor_id) ORDER BY created_at DESC LIMIT N` — uses a composite index seek, which is O(log N) regardless of how deep into the dataset you are. |
| What header should a 429 Too Many Requests response include? | `Retry-After: <seconds>` (or an HTTP-date) — tells the client exactly how long to wait before retrying, enabling polite back-off without guessing. |
| In multi-source BFS (LC 1162, LC 542), why initialize all source cells in the queue before starting BFS? | It ensures all sources expand simultaneously at distance 0, so the BFS wave front correctly represents the shortest distance from any source — the same as a single-source BFS from a virtual super-source connected to all real sources with zero-weight edges. |

## Mock / Contest Log
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

## Week 24 Debrief
- Which hard OA problem type (sweep line / heap / binary search / offline query) needs more practice?
- Which LLD system could you not whiteboard from scratch in 10 minutes?
- Log your answers in `knowledge-base/revision-log.md`.
