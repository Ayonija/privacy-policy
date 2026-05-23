# Week 06 Weekly Review — Days 36–42
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Week Summary
This week closed out the Hashing slot (Days 36–40) and opened the Trees & BST slot (Days 41–42).

**Topics covered:**
- Prefix XOR/Sum Mod + In-Place Hashing (Day 36)
- Pair Counting with HashMap complements (Day 37)
- Frequency Multiset Equivalence + LFU Cache (Day 38)
- Sliding Window with Last-Seen Index + Randomised Collection (Day 39)
- HashMap Slot 4 Synthesis: multi-pass counting, slope normalisation (Day 40)
- Tree DFS: depth tracking, gain propagation (Day 41)
- Tree BFS: level order, zigzag, serialize/deserialize (Day 42)

**System design arc:** Completed DB Indexing unit (Days 36–40) covering hash indexes, index selectivity, query planner, full-text inverted index, partial indexes, when NOT to index, and the full decision framework. Started Caching unit (Days 41–42): fundamentals, hit rate, cache-aside pattern, Redis data structures.

---

## 35 Flashcards — Full Week Review

### Day 36 — Prefix Sum II & In-Place Hashing
| Q | A |
|---|---|
| How do you find the longest subarray with equal 0s and 1s in O(n)? | Replace 0 → −1; store first-seen index for each prefix sum; longest subarray summing to 0 = `current_index − first_seen[prefix]` |
| Why do you store the *first* seen index (not the last) for each prefix sum in Contiguous Array? | You want the longest subarray — using the first occurrence of a prefix sum maximises the distance to the current index |
| What are the three steps of the First Missing Positive in-place algorithm? | 1. Clean: replace out-of-range values with n+1. 2. Mark: negate `nums[v-1]` for each v ∈ [1,n]. 3. Scan: first positive index i → answer is i+1 |
| What key operation does a hash index NOT support that a B+Tree does? | Range queries (BETWEEN, >, <) — the hash function destroys key ordering |
| What is InnoDB's Adaptive Hash Index (AHI)? | An automatic in-memory cache that InnoDB builds on top of its B+Tree for frequently accessed pages — not configurable or persistent; transparent to the user |

### Day 37 — Pair Counting & Palindrome Pairs
| Q | A |
|---|---|
| How does 4Sum II reduce O(n⁴) to O(n²)? | Precompute all n² sums of arrays 1+2 in a HashMap; for each of the n² sums from arrays 3+4, look up its negation — O(n²) total |
| How do you handle the modular complement in Pairs of Songs? | `complement = (60 - t % 60) % 60`; the outer `% 60` handles the case where `t % 60 == 0` (complement would be 60, not 0) |
| What are the three cases for Palindrome Pairs? | 1. Exact reverse match. 2. Prefix is palindrome + reverse of suffix is in dict. 3. Suffix is palindrome + reverse of prefix is in dict |
| What is index selectivity and why does it matter? | Selectivity = distinct_values / total_rows; low selectivity (boolean, gender) means the index finds too many rows — sequential scan is often faster |
| Why might the query planner ignore a valid index? | (1) Low selectivity — too many matching rows. (2) Stale statistics — run ANALYZE. (3) Type mismatch — implicit coercion prevents index use |

### Day 38 — String Analysis & LFU Cache
| Q | A |
|---|---|
| What are the two conditions for "Two Strings Are Close"? | (1) Same set of unique characters. (2) Same sorted multiset of character frequencies — both conditions must hold |
| Why must `min_freq` only reset to 1 on a NEW key insert (not on `get`)? | On a `get` or `put` update, min_freq might increase if its bucket empties, but never drops below 1; only a brand-new insertion resets it to 1 |
| How does LFU use OrderedDict for LRU tie-breaking? | OrderedDict preserves insertion order; `popitem(last=False)` removes the oldest-inserted key at the minimum frequency — LRU among equal-frequency keys |
| What is an inverted index and what problem does it solve? | Maps each word (token) to the list of documents containing it; enables O(log V) word search — impossible with a B+Tree which can't handle `LIKE '%word%'` |
| How do you intersect two posting lists in an AND query? | Merge two sorted posting lists using two pointers — O(n₁ + n₂); posting lists are kept sorted by document ID to enable this merge |

### Day 39 — Advanced Sliding Window & Randomised Collection
| Q | A |
|---|---|
| How does Minimum Consecutive Cards to Pick Up find the minimum window? | For each card: if seen before, record `i - last_seen[card] + 1`; always update `last_seen[card] = i`; return the minimum recorded window |
| Why must you also update `last_seen` for cards that have no duplicate yet? | The same card may form a shorter window with a later occurrence; always updating ensures you compare against the most recent prior position |
| What is the critical edge case when removing an element equal to the last array element in LC 381? | `remove_idx == len(arr) - 1` — the "swap" is with itself; you must still remove the index from `val_to_indices[val]` and NOT accidentally re-add it |
| When should you NOT add an index to a column? | Low cardinality, small table (< ~1000 rows), column never used in WHERE/JOIN, or table is extremely write-heavy |
| What is a partial index and when is it most useful? | An index with a WHERE clause — only indexes rows matching the condition; ideal when most rows don't qualify (e.g., 5 % of rows are "active") |

### Day 40 — Hashing Synthesis
| Q | A |
|---|---|
| How do you count cows in Bulls and Cows without re-counting bulls? | Build freq maps only for non-bull positions; cows = sum of `min(s_freq[c], g_freq[c])` for each char in g_freq |
| How do you represent slopes exactly in Max Points on a Line? | `(dy // gcd, dx // gcd)` with sign normalised so dx > 0 (or (1, 0) for vertical) — avoids floating-point errors |
| Why does Max Points on a Line add 1 to the slope count? | The slope map counts *other* points; the origin point itself must be included — answer = `slope_count + 1` |
| What is the complete index decision tree for a slow query? | Full-text → inverted index. Equality-only → hash. Range/ORDER BY/prefix → B+Tree. Multi-col → composite (equality first, range last). Sparse rows → partial. Read-optimised → covering |
| What are the 5 index interview questions you must answer cold? | (1) Why is full scan slow? (2) How does B+Tree achieve O(log n)? (3) Why only one clustered? (4) Why does planner ignore index? (5) When to avoid indexing? |

### Day 41 — Tree DFS & Maximum Path Sum
| Q | A |
|---|---|
| How does DFS right-side view work without BFS? | Visit right child before left; track depth; the first node reached at each new depth is the rightmost — append to result |
| What does Count Good Nodes propagate downward through DFS? | The maximum value seen on the path from root to the current node; a node is "good" if its value ≥ this running maximum |
| In Maximum Path Sum, why take `max(gain, 0)` for each child's return value? | Negative-gain subtrees should be pruned — clamping at 0 means "don't extend the path into loss territory" |
| What is the cache-aside (lazy loading) pattern? | On miss: query DB, populate cache with TTL, return. On hit: return cached value. Cache is only written on demand — no wasted space for cold data |
| What does the hit-rate vs. cache-size curve look like? | Logarithmic — each doubling of cache size yields diminishing hit-rate gains; profile before over-provisioning |

### Day 42 — Tree BFS & Serialize/Deserialize
| Q | A |
|---|---|
| In BFS level-order traversal, how do you detect level boundaries? | Snapshot `level_size = len(queue)` before the inner loop; process exactly that many nodes before the next level begins |
| How does Zigzag Level Order differ from standard BFS level order? | Identical BFS structure; maintain a `left_to_right` flag; reverse each level list when the flag is False; toggle after every level |
| How does BFS serialization handle missing children in LC 297? | Write `"null"` for None nodes; during deserialization, only non-null nodes are added to the parent queue — nulls consume no children tokens |
| What Redis data structure implements a real-time leaderboard and why? | Sorted Set — `ZADD` updates score in O(log n); `ZREVRANGE` retrieves top-K; `ZREVRANK` returns a player's rank |
| What Redis structure gives unique visitor counts for 100 M users at fixed memory? | HyperLogLog — `PFADD`/`PFCOUNT`; fixed 12 KB per key regardless of cardinality; ~0.81 % error acceptable for analytics |

---

## Pattern Quick Reference — Week 06

| Pattern | Trigger phrase | Complexity |
|---------|---------------|-----------|
| Prefix sum + first-seen map | Longest subarray summing to 0 / target | O(n) time, O(n) space |
| Prefix mod map | Subarrays divisible by k | O(n) time, O(k) space |
| In-place sign marking | First missing positive, O(1) space | O(n) time, O(1) space |
| HashMap pair complement | Count pairs with sum = target across two arrays | O(n²) — O(n) depending |
| Gain-propagation DFS | Max path sum in binary tree | O(n) time, O(h) space |
| BFS with level snapshot | Level order, zigzag, right side view | O(n) time, O(n) space |
| BFS serialization | Encode tree to string preserving structure | O(n) time, O(n) space |

---

---

## STAR Framework — Week 06 Pattern Synthesis

> Use these consolidated STAR talking points to bridge DSA patterns to real-world interview stories. Each maps to the week's core algorithm family.

---

### Pattern 1: Prefix Sum / In-Place Hashing (Day 36)

**Trigger phrase:** "find in O(n) time and O(1) space" or "longest subarray with equal counts"

**STAR talking point:**
- **S/T:** "I had a 50M-event batch pipeline that was allocating a 400 MB auxiliary boolean array to track which IDs had been processed — pushing the server into swap and inflating runtime from 8 minutes to 40 minutes."
- **A:** "I recognised the input array was idle after the first read pass. I used the in-place sign-marking trick: normalise out-of-range values (pass 1), negate `arr[id-1]` for each seen ID (pass 2), scan for positive entries (pass 3). Zero auxiliary allocation."
- **R:** "Memory footprint halved from 850 MB to 420 MB. Runtime: 6 minutes (6.7× speedup). Avoided an $18,000/year hardware upgrade."

---

### Pattern 2: HashMap Pair / Complement Counting (Day 37)

**Trigger phrase:** "count pairs summing to target" or "split a 4-way problem into two 2-way problems"

**STAR talking point:**
- **S/T:** "A 4-array tuple-count problem was O(n⁴) — 62 billion ops for n = 500. Needed O(n²) or less."
- **A:** "Split into two halves: precompute all n² sums from arrays 1+2 into a HashMap, then for each n² sum from arrays 3+4 look up its negation in O(1). Two nested loops instead of four."
- **R:** "From ~62 seconds (O(n⁴)) to milliseconds (O(n²)) for n = 500. Same pattern applied to Pairs of Songs (mod-60 complement) and 4Sum II."

---

### Pattern 3: LFU Cache — freq→OrderedDict (Day 38)

**Trigger phrase:** "O(1) eviction of least frequently used key"

**STAR talking point:**
- **S/T:** "A recommendation cache was using LRU, which kept seasonal products alive long after the season ended. Evergreen products kept getting evicted, causing 400 ms P95 latency for 80% of queries."
- **A:** "Switched eviction policy to LFU using freq→OrderedDict + min_freq. OrderedDict at each frequency level maintains LRU tie-breaking in O(1). Shadow-tested both policies on 5% of traffic for a week."
- **R:** "P95 recommendation latency: 400 ms → 31 ms. Cache hit rate: 74% → 91% for evergreen products. $12,000/month DB cost reduction."

---

### Pattern 4: Sliding Window + Last-Seen Index (Day 39)

**Trigger phrase:** "minimum window containing a duplicate" or "maximum sum distinct subarray"

**STAR talking point:**
- **S/T:** "Needed to find minimum consecutive card pickup containing a pair. Brute-force O(n²) was too slow for n = 10⁵."
- **A:** "Maintained a `last_seen` HashMap. On duplicate at index i: candidate window = `i - last_seen[card] + 1`. Always update `last_seen[card] = i` (even for first occurrence) to track the most recent position."
- **R:** "O(n) time — 100,000 ops vs. 10 billion for brute force. Pattern generalises to the fixed-window distinct-max variant with a `freq` map and `distinct_count` tracker."

---

### Pattern 5: Gain Propagation DFS (Day 41)

**Trigger phrase:** "maximum sum of any path in a binary tree" or "path need not pass through root"

**STAR talking point:**
- **S/T:** "Given a tree with negative values, needed the max-sum path between any two nodes — not just root-to-leaf. Enumeration is O(n²)."
- **A:** "Each node plays two roles: (1) apex of a bent path: update global max with `node.val + left_gain + right_gain`. (2) Contributor to parent: return `node.val + max(left_gain, right_gain)`. Clamp gains at 0 to prune negative subtrees. Initialise `max_sum = root.val` not 0 to handle all-negative trees."
- **R:** "O(n) time, O(h) space. Single DFS pass. Generalises to Diameter of Binary Tree, Longest Path, and rerooting variants."

---

### Pattern 6: BFS Level-Order / Serialization (Day 42)

**Trigger phrase:** "process nodes level by level" or "encode tree to string and reconstruct exactly"

**STAR talking point:**
- **S/T:** "Multi-step onboarding workflow was a decision tree — users who abandoned had to restart from scratch, causing 23% abandonment."
- **A:** "Serialized the in-progress session tree via BFS with null sentinels: each node writes its value or 'null', children enqueued only for non-null nodes. Deserialized by parsing tokens in BFS order — queue of parents each consuming two child tokens. Verified 10,000 round-trip tests with property-based testing."
- **R:** "Abandonment rate: 23% → 9% (61% reduction). Completed onboarding +18%. Serialize + deserialize each under 2 ms for 15-level trees."

---

## Checklist
- [ ] Reviewed all 35 flashcards
- [ ] Able to state the trigger condition for each pattern without looking
- [ ] Rewrote at least 3 skeletons from memory (choose: Contiguous Array, LFU Cache, Max Path Sum, BFS Serialize)
- [ ] Identified which problems you could not solve in 20 min and logged them
- [ ] Practised delivering each STAR talking point above cold in under 90 seconds

---

## STAR Quick Reference — Week 6

| Pattern | One-line STAR hook |
|---------|-------------------|
| Prefix Sum + First-Seen Map | "Given equal-0s-1s subarray, replaced O(n²) scan with O(n) first-seen map → 6.7× speedup." |
| HashMap Pair Complement | "Given 4-array tuple count, split into two O(n²) halves → from 62B ops to milliseconds." |
| LFU Cache freq→OrderedDict | "Given seasonal product cache, switched LRU→LFU → P95 latency 400 ms → 31 ms." |
| Sliding Window + Last-Seen | "Given minimum card pickup window, O(n²) → O(n) with last-seen HashMap." |
| Gain Propagation DFS | "Given max-sum path in binary tree, O(n²) enumeration → O(n) single DFS pass." |
| BFS Level-Order / Serialization | "Given session tree persistence, BFS serialize/deserialize → abandonment 23% → 9%." |

**Career stories:**
1. "LFU Cache — switched recommendation cache eviction policy; P95 latency dropped 13×; hit rate +17 pp."
2. "BFS Serialization — persisted user onboarding session trees; completion rate up 18%; 61% abandonment drop."
