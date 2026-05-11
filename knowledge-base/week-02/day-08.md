# Day 08 — Two Pointers: Harder Variants
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Push two-pointer fluency into less-obvious setups: multi-number sums, interval merging, and non-obvious array partitioning.

## DSA (2 hours)
### Pattern: Two Pointers — 4Sum & Interval Intersection
- 4Sum extends 3Sum: fix two outer indices, run two pointers inside — O(n³) total, but still polynomial and expected in interviews.
- Interval intersection with two pointers: advance the pointer whose interval ends earlier; the current intersection (if any) is `[max(start), min(end)]`.
- Trigger condition: k-sum problems (generalise the fix-and-inner-scan approach) OR merge/intersect two sorted interval lists.
- Time complexity: O(n³) for 4Sum | O(m+n) for interval intersection | Space complexity: O(1) extra

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Move Zeroes | 283 | Easy | Two Pointers (slow/fast) | Slow pointer = next write position for non-zero; fast pointer scans; swap or overwrite then fill zeros |
| 2 | 4Sum | 18 | Medium | Two Pointers + Sort (nested) | Sort, fix i and j with two loops, run two pointers for inner pair; skip all duplicate indices |
| 3 | Interval List Intersections | 986 | Medium | Two Pointers on two arrays | Advance the pointer whose interval ends first; overlap exists when max(start) ≤ min(end) |

### Code Skeleton
```python
# 4Sum (LC 18)
def four_sum(nums, target):
    nums.sort()
    result = []
    n = len(nums)
    for i in range(n - 3):
        if i > 0 and nums[i] == nums[i-1]: continue       # skip dup fixed i
        for j in range(i + 1, n - 2):
            if j > i + 1 and nums[j] == nums[j-1]: continue  # skip dup fixed j
            left, right = j + 1, n - 1
            while left < right:
                s = nums[i] + nums[j] + nums[left] + nums[right]
                if s == target:
                    result.append([nums[i], nums[j], nums[left], nums[right]])
                    while left < right and nums[left] == nums[left+1]: left += 1
                    while left < right and nums[right] == nums[right-1]: right -= 1
                    left += 1; right -= 1
                elif s < target:
                    left += 1
                else:
                    right -= 1
    return result

# Interval Intersection (LC 986)
def interval_intersection(A, B):
    i = j = 0
    result = []
    while i < len(A) and j < len(B):
        lo = max(A[i][0], B[j][0])   # overlap start = later of two starts
        hi = min(A[i][1], B[j][1])   # overlap end = earlier of two ends
        if lo <= hi:                  # actual overlap exists
            result.append([lo, hi])
        if A[i][1] < B[j][1]:        # advance the interval that ends sooner
            i += 1
        else:
            j += 1
    return result
```

### Interview Tips

- **4Sum duplicate-skip guard difference:** for the fixed `i` index use `i > 0`; for the fixed `j` index use `j > i + 1` (not `j > 0`). Using the wrong guard is the most common 4Sum bug. Explain why: j resets to `i+1` each outer iteration.
- **Integer overflow in k-sum:** with 32-bit integers, `nums[i] + nums[j] + nums[left] + nums[right]` can overflow if values are large — in Java/C++ cast to `long`. In Python this is handled automatically; mention it to show language awareness.
- **Interval intersection formula:** "Two intervals [a,b] and [c,d] overlap iff `max(a,c) ≤ min(b,d)`. If this holds, their intersection is exactly `[max(a,c), min(b,d)]`." Stating this formula before drawing will impress interviewers.
- **Brute force for 4Sum:** O(n⁴) checking all quadruples → sorting + two nested loops + two pointers = O(n³). Always ask the interviewer for expected n to know if O(n³) is acceptable.
- **Common mistake:** in Move Zeroes, swapping every non-zero with the slow-pointer position when you could just overwrite non-zeros left-to-right and fill zeros at the end — both O(n) but fewer writes.

### Edge Cases to Trace Before Coding
- LC 18 (4Sum): fewer than 4 elements → return `[]`; all same element (e.g., `[0,0,0,0]`, target=0) → one result `[[0,0,0,0]]`
- LC 18: target is very large or negative → sorting handles negative numbers correctly
- LC 986: one list is empty → while loop condition is false immediately, return `[]`
- LC 986: intervals touch at a single point (e.g., `[1,2]` and `[2,3]`) → `lo=2, hi=2`, `lo <= hi` so `[2,2]` is a valid intersection
- LC 283 (Move Zeroes): all zeros → slow never advances, all writes are to the same position (no-op)

## System Design (1 hour)
### Topic: Big-O Revisit — When O(n³) is Acceptable
- 4Sum is O(n³): for n=100 that's 10⁶ ops, fine; for n=10⁶ it's 10¹⁸, unusable.
- Always ask the interviewer the expected input size; it changes the acceptable algorithm class.
- k-sum generalisation: k=2 → O(n), k=3 → O(n²), k=4 → O(n³); each extra dimension costs a factor of n.
- Interval merging/intersection at scale: sort by start time O(n log n) once, then single-pass O(n); used in calendar systems and IP range lookups.
- Interview talking point: "If asked to find free meeting slots for two people from their calendar arrays, answer: merge each person's intervals, then run the two-pointer interval intersection — O(m log m + n log n) for sorting, O(m+n) for the scan."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Tell me about a time you had to coordinate between two independent streams of work (two teams, two systems) and find points of overlap or conflict — mirroring interval list intersection.
- Leadership principle: Earn Trust

## Flashcards

| Q | A |
|---|---|
| How do you generalise two-pointer pair-sum to k-sum? | Fix (k−2) elements with nested loops, then run one two-pointer pass for the remaining pair; time = O(n^(k-1)) |
| What is the duplicate-skip rule for the fixed indices in 4Sum? | `if i > 0 and nums[i] == nums[i-1]: continue` and similarly for j (with `j > i+1` guard) |
| In interval intersection, which pointer advances after checking overlap? | The pointer whose interval has the smaller end value — it can no longer intersect anything further right in the other list |
| How do you determine if two intervals [a,b] and [c,d] overlap? | They overlap if and only if `max(a,c) ≤ min(b,d)` |
| In Move Zeroes, how do you preserve relative order of non-zero elements? | Use slow pointer as a write position; overwrite with non-zero elements left to right (no swap needed if you fill zeros separately at the end) |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
