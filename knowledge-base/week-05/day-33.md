# Day 33 — Hashing: Prefix Sum + HashMap
**Week 05 | Phase 1: DSA Mastery | Month 2**

## Focus
Combine prefix sums with HashMaps to answer subarray sum questions in O(n) — a pattern that appears in 20%+ of medium array problems.

---

## DSA (2 hours)
### Pattern: Prefix Sum + HashMap (Count of Prefix Sums)

**Core idea:**
Define `prefix[i]` = sum of `nums[0..i]`. A subarray `nums[l..r]` has sum `prefix[r] - prefix[l-1]`. To count subarrays with sum = k, you need to count pairs where `prefix[r] - prefix[l-1] = k`, i.e., `prefix[l-1] = prefix[r] - k`. As you scan right-to-left, the HashMap stores *how many times each prefix sum has appeared so far*.

**Key trick — modular arithmetic variant:**
For "subarray sum divisible by k": a subarray sum is divisible by k iff `(prefix[r] - prefix[l-1]) % k == 0`, i.e., `prefix[r] % k == prefix[l-1] % k`. So count pairs with the same prefix mod value. Handle negative mods: `(prefix % k + k) % k`.

**Trigger condition:**
- "number of subarrays with sum = k" → prefix sum count map
- "subarray sum divisible by k" → prefix mod map
- "2D version: submatrices summing to target" → fix top/bottom row, reduce to 1D prefix sum problem

**Time complexity:** O(n) for 1D | O(n²·m) for 2D submatrix | Space complexity: O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Subarray Sum Equals K | 560 | Medium | Prefix sum count map | `count += prefix_map[prefix - k]`; initialise `prefix_map[0] = 1` for subarrays starting at index 0 |
| 2 | Continuous Subarray Sum | 523 | Medium | Prefix mod map | `(prefix[r] - prefix[l]) % k == 0` iff same mod; require subarray length ≥ 2 (store first-seen index) |
| 3 | Number of Submatrices That Sum to Target | 1074 | Hard | Fix row pair + 1D prefix sum | Fix top row r1 and bottom row r2; compress 2D to 1D column sums; run LC 560 on that 1D array |

---

### Code Skeleton
```java
class Solution {
    // Subarray Sum Equals K (LC 560)
    public static int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> prefixMap = new HashMap<>();
        prefixMap.put(0, 1);   // prefix_sum → count of times seen
        int prefix = 0;
        int count = 0;
        for (int n : nums) {
            prefix += n;
            count += prefixMap.getOrDefault(prefix - k, 0);
            prefixMap.put(prefix, prefixMap.getOrDefault(prefix, 0) + 1);
        }
        return count;
    }

    // Continuous Subarray Sum (LC 523)
    public static boolean checkSubarraySum(int[] nums, int k) {
        Map<Integer, Integer> modMap = new HashMap<>();
        modMap.put(0, -1);   // remainder → earliest index seen
        int prefix = 0;
        for (int i = 0; i < nums.length; i++) {
            prefix = (prefix + nums[i]) % k;
            if (modMap.containsKey(prefix)) {
                if (i - modMap.get(prefix) >= 2) {   // length >= 2
                    return true;
                }
            } else {
                modMap.put(prefix, i);
            }
        }
        return false;
    }

    // Number of Submatrices That Sum to Target (LC 1074) — skeleton
    public static int numSubmatrixSumTarget(int[][] matrix, int target) {
        int rows = matrix.length, cols = matrix[0].length;
        int result = 0;
        for (int r1 = 0; r1 < rows; r1++) {
            int[] colSum = new int[cols];    // compressed column sums for rows r1..r2
            for (int r2 = r1; r2 < rows; r2++) {
                for (int c = 0; c < cols; c++) {
                    colSum[c] += matrix[r2][c];
                }
                // now run LC 560 on colSum with target
                // (prefix sum + count map)
                // TODO: implement
            }
        }
        return result;
    }
}
```

---

### STAR Interview Framework

> **Prefix Sum + HashMap (Count of Prefix Sums):** brute-force O(n²) → this approach O(n) time, O(n) space

**S:** "Given an integer array of n elements, count subarrays with sum exactly k. Brute force checks all O(n²) subarrays — for n=10⁵ that's 10¹⁰ operations, ~10s at 10⁹ ops/sec."
**T:** "Need O(n) by converting the subarray sum question into a prefix-sum lookup: 'how many previous prefix sums equal prefix[r] - k?'"
**A (60% of answer time):**
1. *Classify:* "'Number of subarrays with sum = k' → prefix sum count map. 'Subarray sum divisible by k' → prefix mod map. '2D submatrix sum = target' → fix row pair, reduce to 1D prefix sum."
2. *Init:* "`prefix_map = {0: 1}` (the empty prefix). Running `prefix = 0`, `count = 0`."
3. *Loop/Step:* "`prefix += nums[i]`; `count += prefix_map.get(prefix - k, 0)`; `prefix_map[prefix] += 1`. For mod variant: `prefix = (prefix + nums[i]) % k`; store first-seen index (not count) to verify length ≥ 2."
4. *Termination:* "Single left-to-right pass — O(n). HashMap lookups O(1) average."
5. *Gotcha:* "Initialising `prefix_map = {0: 1}` is the most commonly missed step. Without it, subarrays starting at index 0 (where `prefix == k`) are not counted. State this before writing any code."
**R:** "O(n) time, O(n) space vs O(n²) brute force. At n=10⁵: ~1ms vs ~10s. The 2D extension reduces to O(n²·m) — fix O(n²) row pairs, run O(m) prefix scan on each."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Sliding window | All elements non-negative (shrink is valid) | Negative numbers invalidate the shrink decision — prefix map handles them correctly |
| Two-pointer on prefix array | Sorted prefix sums (no negatives) | Sorting destroys index ordering needed to count valid subarrays |
| Segment tree with range sum | Online updates + range queries | O(n log n) — overkill for a static one-pass count |

### Edge Cases to Trace Before Coding
- LC 560: k=0 — valid; `prefix_map[0] = 1` handles subarray from index 0 summing to 0
- LC 523: two zeros in a row `[0, 0]` with k=anything → valid (length ≥ 2, same remainder 0)
- LC 1074: single cell matrix; negative values in matrix

---

## System Design (1 hour)
### Topic: B+Tree Operations — Insert, Split, and Range Queries in Detail

**Search — O(log_m n):**
Start at root; at each internal node binary-search the key array to find the correct child pointer; descend until leaf; scan leaf for the exact key.

**Insert — O(log_m n) amortised:**
1. Find the correct leaf via search.
2. Insert key+pointer in sorted position within the leaf.
3. If leaf overflows (> m−1 entries): **split** — move median key up to parent, create two half-full leaf nodes; relink the leaf chain.
4. If parent overflows: split parent recursively; in the worst case split propagates to root → root splits and tree height increases by 1.

**Why splits are cheap in practice:**
Each node holds ~100 keys → a node fills up slowly. Bulk loads (sorted inserts) fill nodes to ~69% capacity (standard B+Tree fill factor). Splits are rare in steady-state.

**Range query — O(log_m n + k):**
```
Find leaf containing start_key → O(log n)
While leaf.key <= end_key:
    collect key+pointer pairs
    if end of leaf: leaf = leaf.next   # O(1) pointer follow
```
This is why B+Trees are ideal for range queries — the sorted linked leaf list makes sequential reads very cache-friendly.

**Delete — O(log_m n) amortised:**
Remove key from leaf; if leaf underflows (< ⌈m/2⌉ entries), either borrow from a sibling or merge with a sibling (and remove separator key from parent). Merges can cascade up.

**Interview talking point:** "If asked what happens internally when you insert a million rows into an indexed column, answer: each insert finds the correct leaf in O(log n) and inserts in sorted order. Leaf splits are infrequent (triggered at capacity). For bulk inserts, databases use a bulk-load operation — sort the data first, fill leaves to 70%, build internal nodes bottom-up — dramatically faster than n individual inserts."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Invent and Simplify

**Full STAR Story — "Precomputing Partial Results to Eliminate Redundant Computation":**
**S (20%):** "At a metrics platform, a fraud-detection service answered ~500 real-time queries per second asking: 'How many transactions in this rolling window sum to exactly T?' Each query triggered a full O(n) scan of a 100K-row transaction buffer — the service was saturating the CPU at peak."
**T:** "I was responsible for the query layer. Goal: reduce per-query CPU cost by 10× without altering the upstream data model."
**A (60% — 'I' not 'we'):** "(1) I identified that every query was recomputing the same prefix sums from scratch — O(n) per query, 500 times per second = 50M operations/sec. (2) I introduced a running prefix-sum array maintained incrementally: on each new transaction, I appended `prefix[i] = prefix[i-1] + amount` — O(1) per update. (3) I replaced the full-scan query with a HashMap lookup: `count += prefix_map[prefix - target]`, also O(1) per query. (4) I validated correctness on 10 edge cases including negative amounts and zero-sum windows before enabling in production via a feature flag."
**R (20%):** "Per-query CPU dropped from O(n)=100K ops to O(1). Throughput headroom increased from 20% to 85% at peak load. The pattern was extended to a 2D submatrix variant for regional fraud aggregation three months later."
*Works for: Invent and Simplify, Dive Deep, Deliver Results.*

---

## Flashcards

| Q | A |
|---|---|
| Why must you initialise `prefix_map = {0: 1}` in Subarray Sum Equals K? | It accounts for subarrays starting at index 0 — when `prefix == k`, you need `prefix - k = 0` to already have a count of 1 in the map |
| Why does Continuous Subarray Sum store the *first-seen* index of each mod, not the count? | You need to verify the subarray length is ≥ 2; storing the earliest index lets you compute `current_index − stored_index ≥ 2` |
| How do you reduce Number of Submatrices to a 1D problem? | Fix top row r1 and bottom row r2; for each column c accumulate `col_sum[c] += matrix[r][c]` for r in [r1, r2]; then run Subarray Sum Equals K on `col_sum` |
| What triggers a B+Tree leaf split during insertion? | When inserting would cause the leaf to exceed m−1 entries; the leaf is split at the median, and the median key is pushed up to the parent |
| Why are B+Tree range scans cache-friendly? | All data is in leaf nodes linked in a sorted doubly-linked list; a range scan reads contiguous memory pages sequentially — ideal for CPU prefetcher and OS readahead |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
