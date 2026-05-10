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
```python
# Remove Duplicates (LC 26) — slow/fast two pointers
def remove_duplicates(nums):
    slow = 0
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]
    return slow + 1

# Subarray Product < K (LC 713)
def num_subarray_product_less_than_k(nums, k):
    if k <= 1:
        return 0
    product, left, count = 1, 0, 0
    for right in range(len(nums)):
        product *= nums[right]
        while product >= k:
            product //= nums[left]
            left += 1
        count += right - left + 1  # all subarrays ending at right
    return count
```

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
