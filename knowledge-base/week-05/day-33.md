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
- STAR prompt: Describe a time you found a way to answer a recurring question in O(1) by precomputing and storing partial results — analogous to maintaining a running prefix sum so you never recompute from scratch.
- Leadership principle: Invent and Simplify

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
