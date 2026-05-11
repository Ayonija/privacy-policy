# Day 06 — Arrays & Strings: Pattern Integration
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Integrate two-pointer and sliding window into mixed-context problems to build pattern-recognition fluency before the review day.

## DSA (2 hours)
### Pattern: Sliding Window — Product & Anagram Variants
- Product constraint windows: instead of sum ≥ target, the condition is product ≥ k; use multiplication/division to update state.
- Anagram windows: fixed-size window; track freq diff with a counter; anagram found when all diffs are zero.
- Trigger condition: "find all windows" (return count or start indices) OR "product/count constraint on window" instead of sum constraint.
- Time complexity: O(n) | Space complexity: O(1) for product, O(26) for anagram

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove Duplicates from Sorted Array | 26 | Easy | Two Pointers (slow/fast) | Slow pointer marks next write position; fast pointer scans for new values |
| 2 | Find All Anagrams in a String | 438 | Medium | Fixed Sliding Window + Freq Map | Slide a window of len(p); record left index whenever freq maps match |
| 3 | Subarray Product Less Than K | 713 | Medium | Sliding Window (product) | For each right, count valid subarrays ending at right = `right − left + 1` when product < k |

### Code Skeleton
```java
class Solution {
    // Remove Duplicates (LC 26) — slow/fast two pointers
    public static int removeDuplicates(int[] nums) {
        int slow = 0;   // slow = last confirmed unique element's index
        for (int fast = 1; fast < nums.length; fast++) {
            if (nums[fast] != nums[slow]) {   // new unique element found
                slow++;
                nums[slow] = nums[fast];
            }
        }
        return slow + 1;   // count = last unique index + 1
    }

    // Subarray Product < K (LC 713)
    public static int numSubarrayProductLessThanK(int[] nums, int k) {
        if (k <= 1) {    // all positive ints >= 1, so no subarray product can be < 1
            return 0;
        }
        int product = 1, left = 0, count = 0;
        for (int right = 0; right < nums.length; right++) {
            product *= nums[right];
            while (product >= k) {
                product /= nums[left];
                left++;
            }
            count += right - left + 1;  // subarrays ending at right with start in [left..right]
        }
        return count;
    }
}
```

### Interview Tips

- **The counting formula `right - left + 1`:** explain it explicitly — "For each `right`, every subarray starting anywhere from `left` to `right` and ending at `right` is valid. That's `right - left + 1` subarrays." This is a reusable technique across many problems.
- **Early return `k <= 1` for LC 713:** state it at the top and explain why — "All array elements are positive (≥ 1), so the minimum product of any single element is 1, which is not strictly less than 1. If k ≤ 1, the answer is always 0." Showing you read constraints is a green flag.
- **LC 438 (Find All Anagrams) vs LC 567 (Permutation in String):** these are essentially the same problem — LC 567 returns bool (is there any anagram?), LC 438 returns all starting indices. Use the same `matches` counter technique; this family of problems appears repeatedly.
- **Common mistake:** in LC 26 (Remove Duplicates), returning `slow` instead of `slow + 1` — `slow` is the *index* of the last unique element, but the *count* is `slow + 1`.
- **Brute force for LC 713:** O(n²) scanning all subarrays and computing product each time (even O(n³) if not careful) — sliding window is O(n) with window product tracked incrementally.

### Edge Cases to Trace Before Coding
- LC 26: single element array → `slow = 0`, `fast` loop doesn't run, return 1 ✓
- LC 26: all elements identical → `slow` stays at 0, return 1 ✓
- LC 713: `k = 1` → return 0 (handled by early return)
- LC 713: single element `< k` → count = 1 after one iteration ✓
- LC 438: `len(p) > len(s)` → impossible to find anagram, result is empty list

## System Design (1 hour)
### Topic: Big-O — Counting Output Size
- Subarray Product Less Than K returns a *count*, but if you had to enumerate all subarrays the output itself is O(n²) — even an O(n) algorithm can produce O(n²) output.
- Always separate algorithm complexity from output complexity; in interviews, clarify whether you need the count, the list, or just existence.
- In API design: pagination exists for exactly this reason — returning a giant list of results is O(n) output even if the query is O(log n).
- Anagram search at scale: instead of a sliding window per query, build an inverted index of sorted substrings (like a search engine), trading O(n·|alphabet|) precomputation for O(1) lookup.
- Interview talking point: "If asked to find all occurrences of any permutation of a pattern in a text at scale, answer: pre-hash the pattern's sorted form; scan text with a rolling hash — O(n) with O(1) per window check."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you had to deliver results incrementally (e.g., ship part of a feature) rather than waiting to finish everything — mirroring counting valid subarrays ending at each right step rather than enumerating all at the end.
- Leadership principle: Deliver Results

## Flashcards

| Q | A |
|---|---|
| In the slow/fast two-pointer pattern for Remove Duplicates, what does the slow pointer represent? | The index of the last confirmed unique element written so far |
| How many valid subarrays end at index `right` when the product window is `[left, right]`? | `right − left + 1` — every subarray from any start in [left, right] to right is valid |
| Why must you return 0 early if k ≤ 1 in Subarray Product Less Than K? | Any single positive integer ≥ 1, so no subarray can have a product strictly less than 1 |
| How do you update the anagram match check in O(1) per window slide? | Maintain a `matches` counter: decrement when a char's window freq hits the target freq, increment when it leaves it |
| What is the difference between algorithm complexity and output complexity? | Algorithm complexity is how long the code runs; output complexity is how large the result is — they can differ (e.g., O(n) scan producing O(n²) pairs) |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
