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

### STAR Interview Framework

> **How to use the STAR method when explaining the Dutch National Flag (Three-Pointer) pattern in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an array containing only 0s, 1s, and 2s, and asked to sort it in-place in a single pass without using the built-in sort. A comparison-based sort achieves O(n log n); even a counting sort is O(n) but requires two passes. The Dutch National Flag algorithm does it in O(n) with a single pass and O(1) space."

**Task:** "My goal was to sort the array in a single pass by maintaining three strict invariants: [0, low) contains only 0s, [low, mid) contains only 1s, [high+1, n) contains only 2s, and [mid, high] is unexplored. I partition in-place with three pointers."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Three values, in-place partition, single pass → Dutch National Flag (three-pointer variant of two pointers). The trigger is 'partition into exactly k groups in-place.'"
2. *Initialize:* "I set `low = 0`, `mid = 0`, `high = n-1`. The invariant holds trivially at start: all regions are empty."
3. *Core loop logic:* "While `mid <= high`: if `nums[mid] == 0`, swap `nums[low]` and `nums[mid]`, then `low++; mid++`. If `nums[mid] == 1`, just `mid++`. If `nums[mid] == 2`, swap `nums[mid]` and `nums[high]`, then `high--` — CRITICALLY, do NOT increment `mid` here."
4. *Convergence guarantee:* "Each iteration either advances `mid` or decreases `high`, so the unexplored region [mid, high] strictly shrinks. After at most n iterations, `mid > high` and the algorithm terminates."
5. *Duplicate handling / edge case proactivity:* "The most common bug: incrementing `mid` after swapping with `high`. The value swapped from `high` is unknown — it could be 0, 1, or 2. `mid` must re-examine it. This is the critical invariant that most candidates miss."

**Result:** "This achieves O(n) time and O(1) space in a single pass. Compare with `Arrays.sort()` which is O(n log n), or a counting sort which requires two passes. For n = 10⁵, sorting takes ~1.7M ops vs the flag algorithm's ~10⁵ ops. More importantly, the problem cannot be solved correctly in a single pass without the three-invariant reasoning."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Dutch National Flag here |
|-------------|---------------------------|-------------------------------|
| `Arrays.sort()` / built-in sort | When in-place single-pass is not required | O(n log n) vs O(n); also disallowed by the problem statement |
| Counting sort (two passes) | When values aren't restricted to {0,1,2} | Two passes vs one; Dutch Flag is strictly better when exactly 3 values are known |

**Why NOT built-in sort:** O(n log n) and explicitly disallowed by "sort in a single pass" constraint. Also misses the DSA pattern the interviewer is testing.
**Why NOT counting sort:** Counting sort is O(n) but requires two passes (count then rewrite). Dutch National Flag is O(n) in a single pass — strictly fewer operations, O(1) space.

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

**Leadership Principle:** Ownership

**STAR Story: Triaging and Categorizing a 300-Bug Backlog Under Launch Pressure**

**Situation (20%):** "Two weeks before a major product launch, our team inherited ownership of a legacy service with a backlog of 300 open bugs — a mix of P0 critical blockers, cosmetic issues, and everything in between. No one had triaged the backlog in 6 months and the team didn't know which bugs could kill the launch."

**Task (part of S/T):** "I stepped up as the bug triage lead. My goal was to categorize all 300 bugs into three buckets — 'must fix before launch', 'fix post-launch', and 'close as won't fix' — in 3 days, so the team could focus on blockers immediately."

**Action (60-70% — be specific about what YOU did):**
"First, I defined the three criteria upfront: P0 = data corruption or crash in a critical user path; P1 = degraded experience affecting ≥ 10% of users; P2 = cosmetic or edge case. This was my 'three-pointer invariant' — every bug would land in exactly one bucket.
Then, I ran through all 300 bugs in one day using a timelimited review: 3 minutes per bug, classify immediately and move on. I did not deep-dive any individual bug during triage — I flagged ambiguous cases for a 30-minute group review session.
Next, I called the group review with 3 engineers for the 40 ambiguous cases, and we resolved all of them in 90 minutes using the defined criteria.
Finally, I updated the bug tracker with all categorizations, notified the team of the 28 P0 blockers, and assigned owners to each P0 with a due date of T-7 days."

**Result (10-20%):** "All 28 P0 blockers were fixed before launch. We closed 180 bugs as 'won't fix' with documented rationale, and scheduled 92 as post-launch work. The launch went smoothly — zero P0 bugs reported in the first 48 hours. Leadership recognized the triage as a model for how to handle inherited technical debt, and the process was adopted by two other teams preparing for their own launches."

**Interview tip:** Show that you took ownership without being asked, defined clear criteria before starting, and moved quickly without perfect information. Prepare for: ownership, bias for action, insisting on the highest standards, or delivering results.

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
