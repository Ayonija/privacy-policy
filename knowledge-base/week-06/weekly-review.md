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

## Checklist
- [ ] Reviewed all 35 flashcards
- [ ] Able to state the trigger condition for each pattern without looking
- [ ] Rewrote at least 3 skeletons from memory (choose: Contiguous Array, LFU Cache, Max Path Sum, BFS Serialize)
- [ ] Identified which problems you could not solve in 20 min and logged them
