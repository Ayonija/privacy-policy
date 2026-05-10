# Day 37 — Hashing: Pair Counting & Palindrome Pairs
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Use HashMaps to count valid pairs in O(n) — including the hardest string-pair problem on LeetCode that combines hashing with palindrome checking.

---

## DSA (2 hours)
### Pattern: HashMap for Pair/Complement Counting

**4Sum II (split into two halves):**
Reduce a 4-array sum to a 2-array problem: precompute all `a+b` sums in a HashMap; for each `c+d`, look up `-(c+d)` in the map. Time: O(n²) instead of O(n⁴).

**Pairs of Songs with Durations Divisible by 60:**
Classic complement counting: for each duration `d`, the complement is `(60 - d % 60) % 60`. The mod-60 pattern mirrors the Two-Sum complement. Counts of previous durations with that remainder give the number of valid pairs.

**Palindrome Pairs (LC 336):**
For each word `w` in the dictionary, a concatenation `w + other` is a palindrome if:
- Case 1: `reverse(other) == w` (exact reverse match)
- Case 2: `w[:k]` is a palindrome and `reverse(w[k:])` is in the dict
- Case 3: `w[k:]` is a palindrome and `reverse(w[:k])` is in the dict

Build a `word → index` HashMap; for each word check all 3 cases using palindrome checks on prefixes and suffixes.

**Trigger condition:**
- "count pairs from two arrays with sum = target" → split + HashMap lookup
- "count pairs from one array with sum divisible by k" → complement mod
- "concatenate two words to form palindrome" → reverse lookup + split into prefix/suffix checks

**Time complexity:** O(n²) for 4Sum II | O(n) for song pairs | O(n·L²) for palindrome pairs (L = avg word length)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | 4Sum II | 454 | Medium | Split into two halves + HashMap | Map all `a+b` sums; count `-(c+d)` lookups — O(n²) not O(n⁴) |
| 2 | Pairs of Songs With Total Durations Divisible by 60 | 1010 | Medium | Complement mod 60 | `complement = (60 - d % 60) % 60`; add `count_map[complement]` to result; then increment `count_map[d % 60]` |
| 3 | Palindrome Pairs | 336 | Hard | Reverse HashMap + prefix/suffix palindrome checks | For each word, check: exact reverse in dict; prefix palindrome + reverse suffix in dict; suffix palindrome + reverse prefix in dict |

---

### Code Skeleton
```python
# 4Sum II (LC 454)
def four_sum_count(nums1, nums2, nums3, nums4):
    ab_map = {}
    for a in nums1:
        for b in nums2:
            ab_map[a + b] = ab_map.get(a + b, 0) + 1
    count = 0
    for c in nums3:
        for d in nums4:
            count += ab_map.get(-(c + d), 0)
    return count

# Pairs of Songs Divisible by 60 (LC 1010)
def num_pairs_divisible_by60(time):
    count_map = [0] * 60   # index = duration mod 60
    result = 0
    for t in time:
        complement = (60 - t % 60) % 60
        result += count_map[complement]
        count_map[t % 60] += 1
    return result

# Palindrome Pairs (LC 336) — skeleton
def palindrome_pairs(words):
    word_map = {w: i for i, w in enumerate(words)}
    result = []
    def is_palindrome(s): return s == s[::-1]
    for i, word in enumerate(words):
        for k in range(len(word) + 1):
            prefix, suffix = word[:k], word[k:]
            # Case 1/2: prefix is palindrome, reverse of suffix is in map
            # Case 3: suffix is palindrome, reverse of prefix is in map
            # (be careful about empty string and duplicate pairs)
        pass
    return result
```

---

### Edge Cases to Trace Before Coding
- LC 454: all zeros — `0+0 = 0`, `-(0+0) = 0`; map will have entry 0 with count n²
- LC 1010: duration exactly 60 → `60 % 60 = 0`; complement `(60 - 0) % 60 = 0` (self-pairing allowed)
- LC 336: empty string `""` pairs with every palindrome; same-word reversed pair should not pair with itself

---

## System Design (1 hour)
### Topic: Index Selectivity & the Query Planner

**Index selectivity:**
Selectivity = (distinct values) / (total rows). A column with high selectivity (e.g., email — almost every value is unique) benefits enormously from an index. A column with low selectivity (e.g., boolean `is_active` with 90% rows true) does NOT — the DB would find it cheaper to full-scan than to jump around the disk following index pointers.

**Rule of thumb:**
- Selectivity > 10–20%: index is likely to help
- Selectivity < 1% on a boolean/enum column: index is usually a hindrance — query planner will ignore it

**How the query planner decides:**
1. Estimates the number of rows the WHERE clause will return (using statistics — column histograms)
2. Compares estimated cost of: (a) index scan + random I/O vs. (b) sequential full table scan
3. For large result sets (>10–15% of table), sequential scan is often cheaper due to random I/O cost of index lookups

**EXPLAIN / EXPLAIN ANALYZE:**
Always use `EXPLAIN` to verify the query planner chose your index:
```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
-- Look for: "Index Scan using orders_user_id_idx" (good)
-- vs: "Seq Scan" (index was ignored — check selectivity and statistics)
```

**Stale statistics problem:**
If a table has many new rows since the last `ANALYZE`, the planner may use wrong estimates and choose a bad plan. Run `ANALYZE table_name` after bulk inserts.

**Interview talking point:** "If asked why an index you created is being ignored by the query planner, answer: three common reasons — (1) low selectivity: the column has few distinct values, so a sequential scan is cheaper; (2) stale statistics: run ANALYZE to refresh; (3) implicit type coercion: `WHERE user_id = '123'` (string) won't use an integer index — the types must match exactly."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you found that a solution you prepared for a problem was actually ignored in practice because its cost outweighed its benefit — analogous to an index that the query planner bypasses due to low selectivity.
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does 4Sum II reduce O(n⁴) to O(n²)? | Precompute all n² sums of arrays 1+2 in a HashMap; for each of the n² sums from arrays 3+4, look up its negation — O(n²) total |
| How do you handle the modular complement in Pairs of Songs? | `complement = (60 - t % 60) % 60`; the outer `% 60` handles the case where `t % 60 == 0` (complement would be 60, not 0) |
| What are the three cases for Palindrome Pairs? | 1. Exact reverse match (full word reverse is in dict). 2. Prefix is palindrome + reverse of suffix is in dict. 3. Suffix is palindrome + reverse of prefix is in dict |
| What is index selectivity and why does it matter? | Selectivity = distinct_values / total_rows; low selectivity (boolean, gender) means the index finds too many rows — sequential scan is often faster |
| Why might the query planner ignore a valid index? | (1) Low selectivity — too many matching rows. (2) Stale statistics — run ANALYZE. (3) Type mismatch — implicit coercion prevents index use |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
