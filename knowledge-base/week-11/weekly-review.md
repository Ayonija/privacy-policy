# Week 11 Review — Days 71–77
**Phase 1: DSA Mastery | Month 3 | Slot 8 (Days 71–80)**

## Week Span
Days 71–75: Slot 8 — Heaps (Top-K, K-way merge, two-heap median, greedy heap allocation, IPO)
Days 76–77: Slot 8 — Tries (Implement, Replace Words, Word Search II, Wildcard, Suffix-wrapped)
System Design: Newsfeed architecture (requirements, fanout, DB schema, social graph, caching, notifications, ranked feed)

---

## Patterns This Week

### DSA Patterns

**Min-heap Top-K by frequency (Day 71 — LC 347):**
Push `(count, elem)`; pop when `len > k`; heap retains top K most frequent.

**Binary search K-element window (Day 71 — LC 658):**
Binary search on left boundary `lo`; invariant: `x - arr[mid] <= arr[mid+k] - x`; return `arr[lo:lo+k]`.

**K-way merge min-heap (Day 71 — LC 23):**
Push `(val, list_idx, node)` from each list; pop min, push node.next; `list_idx` breaks ties.

**Quickselect Kth largest (Day 72 — LC 215):**
Three-way partition around random pivot; recurse on side containing target index `n-k`; O(n) average.

**Max-heap frequency sort (Day 72 — LC 451):**
Counter + negate frequencies; drain heap appending `char * freq`.

**Two-heap running median (Day 72 — LC 295):**
`lo` = max-heap (negated, lower half); `hi` = min-heap (upper half). Invariant: `max(lo) ≤ min(hi)`, sizes differ by at most 1. Push to lo, rebalance to hi, rebalance sizes.

**Max-heap Top-K by distance (Day 73 — LC 973):**
Push `(-dist², x, y)` to max-heap; pop when `len > k`.

**Greedy max-heap two-at-a-time (Day 73 — LC 767):**
Pop two most frequent; append both; push back with decremented counts.

**Two-heap + lazy deletion sliding window median (Day 73 — LC 480):**
Mark evicted elements in `delayed` counter; skip when they reach heap top.

**Task Scheduler frequency formula (Day 74 — LC 621):**
`max((max_freq - 1) * (n + 1) + count_max, len(tasks))`.

**Min-heap greedy swap (Day 74 — LC 1642):**
Min-heap of size `l`; on new jump: if heap full and new > min → swap (pop min, use bricks for it, push new).

**K-way merge range tracking (Day 74 — LC 632):**
Min-heap + current_max; range = [heap_min, current_max]; pop min, push next from same list; stop when list exhausted.

**Greedy max-heap triple-run avoidance (Day 75 — LC 1405):**
Pop most frequent; if last two chars equal → instead pop second-most-frequent; push back with decremented counts.

**Min-heap resource queue (Day 75 — LC 1845):**
`reserve()` = heappop; `unreserve(s)` = heappush; O(log n) each.

**Two-heap greedy capital selection IPO (Day 75 — LC 502):**
Sort projects by capital; each round: move affordable to profit max-heap; pop max profit; add to w.

**Trie insert/search/startsWith (Day 76 — LC 208):**
TrieNode with `children = {}`, `is_end = False`. O(L) per operation.

**Trie prefix replace (Day 76 — LC 648):**
Traverse each word; stop and return prefix at first is_end; else keep original word.

**Trie + grid DFS with node pruning (Day 76 — LC 212):**
Build Trie of all target words; DFS from each grid cell traversing Trie simultaneously; set is_end=False after find; prune childless nodes.

**Trie BFS on word-only paths (Day 77 — LC 720):**
BFS from root, only follow edges where child.is_end = True; longest path = answer.

**Wildcard Trie DFS (Day 77 — LC 211):**
Literal char → traverse child; '.' → DFS all children; return True if any branch reaches end with is_end.

**Suffix-wrapped Trie (Day 77 — LC 745):**
Insert `"{suffix}#{word}"` for all suffixes; store latest index at each node; query `"{suffix}#{prefix}"`.

---

## Flashcard Deck — 35 Cards

### Day 71 — Top-K + K Closest + Merge K

**1.** How does a min-heap of size K find Top-K frequent elements?
Push `(frequency, element)` for each unique element; when `len > K`, pop the minimum. The K remaining elements are the most frequent.

**2.** What is the binary search invariant for Find K Closest Elements?
`x - arr[mid] <= arr[mid+k] - x` → left side is closer or equal → `hi = mid`. Otherwise `lo = mid + 1`. Loop until `lo == hi`; return `arr[lo:lo+k]`.

**3.** How does Merge K Sorted Lists handle stable comparison in the heap?
Push `(node.val, list_index, node)`. The integer `list_index` breaks ties when values are equal, avoiding TypeError from comparing ListNode objects.

**4.** What is the Newsfeed hybrid fanout model?
Normal users (< threshold followers): fanout on write → push post_id to Redis sorted sets. Celebrity accounts: no fanout → merge at read time from `user_posts:{celebrity_id}`. Balances write amplification vs read latency.

**5.** What Redis data structure stores the pre-computed newsfeed?
Sorted set `feed:{user_id}` with score = timestamp, member = post_id. `ZADD` on new posts; `ZREVRANGE` to read latest; `ZREMRANGEBYRANK` to cap at 1000 entries.

### Day 72 — Kth Largest + Frequency Sort + Two-Heap Median

**6.** How does quickselect find Kth largest in O(n) average?
Three-way partition around random pivot into <, ==, > groups. If target index in == group → found. Recurse on side containing target. Average O(n) since each partition halves the search space.

**7.** How does Sort Characters By Frequency use a max-heap?
Counter + push `(-freq, char)` pairs; drain: each pop gives `(char, freq)`; append `char * freq` to result.

**8.** State the two invariants of the two-heap MedianFinder.
(1) `max(lo) ≤ min(hi)` — lower half ≤ upper half. (2) `len(lo) == len(hi)` or `len(lo) == len(hi) + 1` — lo holds the extra element.

**9.** How does the celebrity problem manifest in fanout on write?
A celebrity with 100 M followers triggers 100 M Redis writes per post. At a rate-limited 1000 writes/sec, one post takes 33 hours to propagate — unacceptable.

**10.** How does feed cold start work?
Fetch all followee IDs → for each followee, fetch last 20 posts from `user_posts:{followee_id}` sorted set → merge and sort → populate `feed:{user_id}` Redis cache with 7-day TTL.

### Day 73 — K Closest Points + Reorganize + Sliding Window Median

**11.** How does K Closest Points to Origin avoid computing sqrt?
Distance² = x² + y² preserves ordering. Push `(-x²-y², x, y)` to a max-heap of size K; pop when size > K.

**12.** How does Reorganize String guarantee no triple run?
Pop top two most-frequent chars simultaneously; append both to result; push back with decremented counts. Guaranteed alternation without explicit last-char check.

**13.** What is lazy deletion in the sliding window median?
Mark evicted elements in `delayed` counter instead of removing from heap interior. When marked element reaches top, discard and re-balance. Amortized O(log k) per removal.

**14.** Why is Cassandra used for the follow graph?
Access pattern: "get all followers of user X" = partition scan by followee_id. Cassandra wide-row model provides O(1) partition scan; no joins needed; scales to billions of rows.

**15.** Why cap feed Redis sorted sets at 1000 entries and only cache active users?
1000 entries per user × 8 bytes × 1 B users = 8 TB — too large. Cap bounds memory; inactive users (> 30 days) get cold-start rebuild on next login.

### Day 74 — Task Scheduler + Furthest Building + Smallest K-list Range

**16.** State the Task Scheduler math formula.
`max((max_freq - 1) * (n + 1) + count_max, len(tasks))`. `max_freq` = highest task count; `count_max` = number of tasks with that count. Floor at `len(tasks)` for no-idle cases.

**17.** How does Furthest Building decide to swap a ladder for bricks?
Min-heap of the `l` largest jumps used so far. When heap full and new jump > heap min: pop heap min (use bricks for it), push new jump (assign ladder to it).

**18.** What stops the Smallest Range from K Lists algorithm?
When the list providing the current minimum is exhausted — no element from that list can extend the range. At that point, record the minimum range seen and return.

**19.** What is the dual Cassandra table design for the social graph?
Table 1 partitioned by followee_id (for listing followers during fanout). Table 2 partitioned by follower_id (for listing followees during feed cold-start). Dual-write on follow/unfollow.

**20.** What is affinity score in feed ranking?
Measures how often the viewer has interacted with a poster (likes, comments, DMs). Computed from interaction history; decays over time so recent interactions weigh more.

### Day 75 — Longest Happy String + Seat Manager + IPO

**21.** How does Longest Happy String avoid creating "aaa"?
Pop most-frequent char; if last two chars in result are the same → instead pop second-most-frequent, use it, push first back unchanged. This avoids the triple without explicit last-char tracking.

**22.** How is SeatManager.reserve() implemented in O(log n)?
Min-heap of available seat numbers. `reserve()` = `heappop()` → returns smallest available. `unreserve(s)` = `heappush(s)`. Initialise with `heapify([1..n])` in O(n).

**23.** What are the two heaps in IPO and what does each contain?
(1) Min-heap sorted by capital required (projects not yet affordable). (2) Max-heap of profits (negated, for projects we can start now). Each round: move affordable projects from (1) to (2); pop max profit from (2).

**24.** How does the newsfeed handle pagination?
Redis sorted set with `ZREVRANGEBYSCORE feed:{user_id} {cursor_timestamp} -inf LIMIT 0 20`. Client stores the timestamp of the last seen post as cursor for the next request.

**25.** Why use a max-heap for feed scoring instead of re-sorting the full list?
Re-sorting N posts is O(N log N) per feed view. A pre-computed sorted set in Redis with scores updated incrementally (on new engagement) amortises the sort cost. Feed view is O(1) ZREVRANGE.

### Day 76 — Implement Trie + Replace Words + Word Search II

**26.** What are the three Trie operations and their time complexities?
`insert(word)`, `search(word)`, `startsWith(prefix)` — all O(L) where L = word/prefix length. Trie node has `children = {}` and `is_end = False`.

**27.** How does Replace Words find the shortest matching root?
Insert all roots into Trie. For each word, traverse Trie char by char; at the first `is_end = True` → return the prefix built so far (shortest root match). If path breaks → return original word.

**28.** Two optimisations that make Word Search II efficient?
(1) Set `is_end = False` after finding a word → prevents duplicate results. (2) Delete Trie child when no children + no is_end → prunes dead branches from future DFS calls.

**29.** Why is Trie better than HashSet for prefix queries?
HashSet can check exact membership in O(L) but `startsWith` requires O(n×L) scan of all entries. Trie answers both in O(L) by sharing prefix nodes.

**30.** How does the notification system avoid spamming a user with thousands of "liked your post" notifications?
Redis counter with TTL: on first like → send notification + set counter with 5-min TTL. Subsequent likes increment counter but don't trigger new notifications. At threshold counts (10, 100, 1000) → send batched "X others liked" notification.

### Day 77 — Longest Word + Wildcard Trie + Suffix-Wrapped Trie

**31.** How does Longest Word in Dictionary's BFS guarantee every prefix is valid?
BFS from root, but only follow edges where `child.is_end = True`. Every node visited on the path has is_end set → every prefix of the found word is a valid dictionary word.

**32.** How does WordDictionary.search handle '.' wildcard?
DFS with branching. At '.': recursively call DFS for ALL children of the current node. Return True if any branch reaches the end of the pattern with `is_end = True`.

**33.** How does WordFilter support prefix AND suffix queries?
For each word w at index i: insert `"{suffix}#{word}"` for all suffixes (including empty). Each node stores `latest_index`. Query `f(prefix, suffix)` looks up `"{suffix}#{prefix}"` and returns the stored index.

**34.** Chronological vs ranked feed — key trade-offs?
Chronological: simple, transparent, favours frequent posters. Ranked: more engaging (higher session time), requires ML, risks filter bubbles. Mitigation: topic diversity constraints + "why am I seeing this?" explanation.

**35.** How is a ranked feed score updated incrementally for new engagement?
On new like/comment: async worker updates `ZADD feed:{viewer_id} {new_score} {post_id}` for the viewer's feed (if in active feeds). Batch decay job every 15 min re-scores top 1000 items to account for recency decay.

---

## Patterns Mastered This Week
Rate yourself 1–5 after this review.

- [ ] Min-heap Top-K template — LC 347
- [ ] Two-heap running median (MedianFinder) — LC 295
- [ ] K-way merge min-heap — LC 23
- [ ] Quickselect (three-way partition) — LC 215
- [ ] Sliding window median with lazy deletion — LC 480
- [ ] Task Scheduler formula — LC 621
- [ ] Two-heap greedy IPO — LC 502
- [ ] Trie insert/search/startsWith template — LC 208
- [ ] Grid DFS + Trie pruning (Word Search II) — LC 212
- [ ] Wildcard Trie DFS — LC 211

---

## Problems Needing Drill
| Pattern | Problem | Retry date |
|---------|---------|------------|
| (learner fills in) | | |
