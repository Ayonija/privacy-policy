# Day 07 — Review Day: Arrays & Strings Week 1
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Consolidate two-pointer and sliding window patterns; identify gaps before advancing. System design: close out Big-O + RAM model unit.

## DSA (2 hours)
### Pattern: Review — Two Pointers & Sliding Window
- Drill pattern recognition: given a problem description, classify it (two pointers vs. sliding window) before reading constraints.
- For each problem you struggled with this week, re-solve it from scratch without looking at your previous solution.
- If you cannot write the skeleton in 2 minutes, add to revision log.
- Time complexity: (problem-specific) | Space complexity: (problem-specific)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Is Subsequence | 392 | Easy | Two Pointers | Fast pointer scans t; slow pointer advances only on match with s |
| 2 | Sort Colors | 75 | Medium | Three Pointers (Dutch National Flag) | Three regions: [0,low) = 0s, [low,mid) = 1s, [high+1,n) = 2s; mid scans forward |
| 3 | Get Equal Substrings Within Budget | 1208 | Medium | Sliding Window (cost budget) | Cost = abs(s[i]-t[i]); find max window where total cost ≤ maxCost |

### Code Skeleton
```python
# Dutch National Flag (LC 75)
def sort_colors(nums):
    low, mid, high = 0, 0, len(nums) - 1
    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1; mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1  # do NOT increment mid here

# Is Subsequence (LC 392)
def is_subsequence(s, t):
    i = 0
    for ch in t:
        if i < len(s) and ch == s[i]:
            i += 1
    return i == len(s)
```

## System Design (1 hour)
### Topic: Big-O & RAM Model — Week 1 Wrap-Up
- Recap the complexity class ladder: O(1) → O(log n) → O(n) → O(n log n) → O(n²) → O(2ⁿ).
- Three-pointer problems are still O(n): each pointer moves at most n steps; total work ≤ 3n.
- RAM model limitation: assumes uniform memory cost; real systems must account for cache hierarchy, NUMA, and I/O.
- System design teaser for next week: horizontal vs. vertical scaling — the same "more resources" intuition as increasing n; next session will formalise this.
- Interview talking point: "If asked the complexity of Dutch National Flag, answer: O(n) time, O(1) space — three pointers scan the array once; mid only advances or swaps, never backtracks past low."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you had to sort or organise a messy situation into clear categories quickly with limited resources — analogous to the Dutch National Flag in-place partition.
- Leadership principle: Ownership

## Flashcards

| Q | A |
|---|---|
| How do you check if s is a subsequence of t in O(n)? | Two pointers: i scans s, j scans t; increment i only when t[j] == s[i]; return i == len(s) |
| In Dutch National Flag, why do you NOT increment mid after swapping with high? | The element swapped from high is unknown — it could be 0, 1, or 2, so mid must re-examine it |
| What is the window invariant in Get Equal Substrings Within Budget? | Sum of abs(s[i]−t[i]) over the window ≤ maxCost; shrink left when it exceeds the budget |
| Name the five problems covered this week and their patterns. | LC 125 Two Pointers, LC 167 Two Pointers, LC 15 Two Pointers+Sort, LC 11 Greedy Shrink, LC 16 Two Pointers+Sort, LC 3 Variable Window, LC 209 Variable Window, LC 643 Fixed Window, LC 424 Window+Freq, LC 567 Fixed+Freq, LC 977 Fill-from-back, LC 904 k-Distinct, LC 1004 Zero-count, LC 26 Slow/Fast, LC 438 Anagram, LC 713 Product, LC 392 Subseq, LC 75 DNF, LC 1208 Budget |
| What is the key difference between a subsequence and a substring? | Substring: contiguous characters; Subsequence: any characters in order, gaps allowed |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
