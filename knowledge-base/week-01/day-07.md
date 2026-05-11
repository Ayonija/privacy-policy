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
```java
class Solution {
    // Dutch National Flag (LC 75)
    public static void sortColors(int[] nums) {
        int low = 0, mid = 0, high = nums.length - 1;
        // Invariant: [0, low) = 0s, [low, mid) = 1s, [high+1, n) = 2s
        while (mid <= high) {
            if (nums[mid] == 0) {
                int tmp = nums[low]; nums[low] = nums[mid]; nums[mid] = tmp;
                low++; mid++;
            } else if (nums[mid] == 1) {
                mid++;
            } else {
                int tmp = nums[mid]; nums[mid] = nums[high]; nums[high] = tmp;
                high--;  // do NOT increment mid — the swapped value from high is unknown
            }
        }
    }

    // Is Subsequence (LC 392)
    public static boolean isSubsequence(String s, String t) {
        int i = 0;   // pointer into s (the pattern)
        for (int j = 0; j < t.length(); j++) {
            if (i < s.length() && t.charAt(j) == s.charAt(i)) {
                i++;
            }
        }
        return i == s.length();
    }
}
```

### Interview Tips

- **Dutch National Flag — state the invariant:** "At every step: [0, low) contains 0s, [low, mid) contains 1s, [high+1, n) contains 2s, and [mid, high] is unknown." Interviewers at senior level expect you to name the invariant before writing code.
- **Why NOT increment `mid` after swap with `high`:** the value swapped from `high` is unknown — it could be 0, 1, or 2. `mid` must re-examine it. This is the most common DNF bug in interviews.
- **Review day strategy:** on review days, time yourself solving each problem from scratch without looking at prior solutions. If you can't produce a working solution in 15 minutes, add it to `revision-log.md`. Speed is a signal.
- **Is Subsequence variant (follow-up for L5+):** "If there are millions of pattern strings to check against the same t, precompute for each position in t the next occurrence of each character — O(n·26) precompute, O(m) per query." State this proactively.
- **Common mistake:** in Is Subsequence, checking `s` and `t` with two-pointer loops instead of the clean for-loop over `t` — both work but the for-loop is shorter and less error-prone.

### Edge Cases to Trace Before Coding
- LC 75: all 0s → `low = mid = 0`, all swaps go to back of "0s region"; `high` never moves; correct ✓
- LC 75: `[1, 0, 2]` → mid starts at 0 (value 1), mid++; mid at 1 (value 0), swap low↔mid, low++ mid++; mid at 2 > high, done ✓
- LC 392: empty `s` → is a subsequence of anything (vacuously true); i stays 0, return `0 == 0` ✓
- LC 392: `s` longer than `t` → impossible; i never reaches `s.length()`, return false ✓
- LC 1208: `maxCost = 0` → only windows where all characters are identical are valid

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
| What are the three pointer regions in Dutch National Flag and what invariant does each maintain? | `[0, low)` = confirmed 0s; `[low, mid)` = confirmed 1s; `[high+1, n)` = confirmed 2s; `[mid, high]` = unexplored territory |
| What is the key difference between a subsequence and a substring? | Substring: contiguous characters; Subsequence: any characters in order, gaps allowed |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
