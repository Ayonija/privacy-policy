# Day 01 — Two Pointers: Core Pattern
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Build the two-pointer mental model — the most reusable O(n) technique for sorted arrays and string problems.

## DSA (2 hours)
### Pattern: Two Pointers
- Maintain two indices moving toward or away from each other to collapse an O(n²) search into O(n).
- Trigger condition: sorted input, palindrome check, or pair/triplet sum where shrinking the search space directionally is valid.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Valid Palindrome | 125 | Easy | Two Pointers | Skip non-alphanumeric in-place; compare lowercased chars at both ends |
| 2 | Two Sum II - Input Array Is Sorted | 167 | Medium | Two Pointers | Sorted → if sum > target shrink right, if sum < target grow left |
| 3 | 3Sum | 15 | Medium | Two Pointers + Sort | Fix one element with outer loop, run two pointers inside; skip duplicates after sorting |

### Code Skeleton
```java
class Solution {
    public static void twoPointersTemplate(int[] arr, int target) {
        int left = 0, right = arr.length - 1;
        while (left < right) {
            int current = arr[left] + arr[right];   // example: pair sum
            if (current < target) {
                left++;   // need larger value
            } else if (current > target) {
                right--;  // need smaller value
            } else {
                // found answer — record, then skip duplicates
                left++;
                right--;
            }
        }
    }
}
```

### Interview Tips

- **State brute force first:** "The naive approach is O(n²) nested loops checking every pair — two pointers reduces this to O(n) by exploiting sorted order." Say this aloud before touching code.
- **Classify in under 60 seconds:** ask yourself — is the array sorted? Am I looking for a pair/triplet? Both yes → reach for two pointers.
- **Narrate pointer movement direction:** before each iteration say "left moves right because sum is too small" — interviewers credit communication as heavily as correctness.
- **Duplicate skipping in 3Sum:** always sort first; skip `nums[i] == nums[i-1]` at the fixed index AND skip equal moves inside the inner while loop. Missing either one is the most common 3Sum bug.
- **Common mistake:** using `left <= right` instead of `left < right` for pair problems — when pointers meet, pairing an element with itself is invalid for most pair-sum problems.

### Edge Cases to Trace Before Coding
- Empty array or single element → return immediately (no valid pair)
- All same elements (e.g., `[2, 2, 2]` for 3Sum) → must de-dup correctly without skipping valid triplets
- Negative numbers in 3Sum → sorting handles them naturally; two-pointer still works
- Overflow: `nums[i] + nums[j]` with large integers → cast to `long` in Java/C++ (Python is safe)

## System Design (1 hour)
### Topic: Big-O Notation & the RAM Model
- Big-O measures how runtime or memory scales with input size n, dropping constants and lower-order terms.
- Common classes in order: O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ).
- RAM model assumption: any single memory access or arithmetic op costs O(1); used to reason about algorithm complexity on a single machine.
- Best / average / worst case are distinct — always state which one you're reporting in interviews.
- Interview talking point: "If asked why O(n²) is unacceptable at n=10⁶, answer: ~10¹² operations at 10⁹ ops/sec ≈ 17 minutes; O(n log n) finishes in ~20 ms."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you broke a large, seemingly impossible problem into a series of small, ordered steps — analogous to how two pointers turn an O(n²) search into O(n).
- Leadership principle: Bias for Action

## Flashcards

| Q | A |
|---|---|
| How do you decide when to use the two-pointer pattern? | Sorted array or need to find a pair/triplet with a sum/difference constraint, or palindrome verification |
| How do you eliminate duplicates in 3Sum without a set? | Sort first; skip `nums[i] == nums[i-1]` for the fixed index; skip duplicate left/right moves inside the inner while loop |
| What is the time complexity of two pointers on a sorted array of size n? | O(n) — each pointer travels at most n positions total |
| How does O(n²) scale when input doubles from n to 2n? | Work quadruples — (2n)² = 4n² |
| Write the loop condition that keeps two pointers from crossing. | `while left < right` |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
