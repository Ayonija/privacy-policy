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

### STAR Interview Framework

> **How to use the STAR method when explaining Two Pointers in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a sorted array and asked to find all pairs (or triplets) summing to a target. The brute-force approach of checking every combination with nested loops gives O(n²) for pairs and O(n³) for triplets, which for n = 10⁶ means roughly 10¹² operations — at 10⁹ ops/sec, that's around 17 minutes for pairs and infeasible for triplets."

**Task:** "My goal was to reduce this to O(n) for pair-sum and O(n²) for 3Sum by recognising that sorted order lets us make a guaranteed progress decision on each step instead of blind enumeration."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "I noticed the array is sorted and the problem asks for pairs satisfying a sum constraint — that's the canonical two-pointer trigger. I immediately reach for `left = 0, right = n-1`."
2. *Initialize:* "I place `left` at index 0 (smallest element) and `right` at index n-1 (largest element). The current sum is `nums[left] + nums[right]`."
3. *Core loop logic:* "If the sum equals the target, I record the pair and advance both pointers inward. If the sum is too small, I move `left` right to increase the sum. If the sum is too large, I move `right` left to decrease it. Only one pointer moves per iteration."
4. *Convergence guarantee:* "Each iteration moves at least one pointer, and pointers never cross backward, so we perform at most n total pointer moves — the algorithm terminates in O(n)."
5. *Duplicate handling / edge case proactivity:* "For 3Sum, after sorting, I skip `nums[i] == nums[i-1]` at the fixed index AND skip equal moves inside the inner while loop after recording a triplet. Missing either skip produces duplicate triplets — the most common 3Sum bug."

**Result:** "This reduces the pair-sum from O(n²) to O(n) and 3Sum from O(n³) to O(n²). For n = 10⁶, the pair-sum difference is finishing in ~1 ms versus timing out after 17 minutes. For 3Sum with n = 3000 (typical constraint), O(n²) finishes in ~9M ops while O(n³) would be 27B ops."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Two Pointers here |
|-------------|---------------------------|-------------------------------|
| Brute force / nested loops | When n ≤ 100 and code clarity matters more than speed | Two pointers is O(n) vs O(n²); for n ≥ 10⁴ brute force times out |
| Hash set / HashMap | When the input is unsorted, or you need index preservation | Two pointers needs sorted input but gives O(1) space vs O(n) for a hash set |

**Why NOT brute force:** O(n²) on n = 10⁶ is ~10¹² operations — about 17 minutes at 10⁹ ops/sec; constraints typically force n ≤ 10⁵ or mandate O(n log n) or better.
**Why NOT hash set:** A hash set costs O(n) extra space and doesn't naturally handle duplicates for k-sum problems; two pointers achieves O(1) space with sorted input and handles duplicates with a simple skip.

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

**Leadership Principle:** Bias for Action

**STAR Story: Turning an O(n²) Data Migration into an O(n) Pipeline**

**Situation (20%):** "In my previous role as a software engineer, our team had a data backfill job that processed a 2-million-row customer dataset by comparing each row against every other row to find matching records — an O(n²) nested-loop design. At our current data volume, the job took 9+ hours and was blocking our nightly release window."

**Task (part of S/T):** "I was responsible for the data pipeline's reliability. My goal was to reduce the backfill runtime to under 30 minutes without changing the output or adding new infrastructure."

**Action (60-70% — be specific about what YOU did):**
"First, I profiled the job and confirmed the nested loop accounted for 96% of runtime — it was doing 4 trillion comparisons.
Then, I noticed the matching key was a normalized identifier that could be sorted — exactly the two-pointer trigger condition. I proposed sorting both datasets once (O(n log n)) and using a two-pointer merge scan instead of the nested loop.
Next, I implemented the new approach in a branch, validated output correctness against a 100K-row sample where both old and new produced identical results.
Finally, I deployed the change, monitored the first production run end-to-end, and documented the pattern in our team wiki so two other similar jobs could be updated the same week."

**Result (10-20%):** "The backfill job went from 9+ hours to 18 minutes — a 97% reduction. The nightly release window was unblocked, and the two follow-up jobs I documented were updated within the week, saving an additional 6 hours of compute time per night. I now apply complexity analysis before committing to any nested loop in our core pipelines."

**Interview tip:** Interviewers want to hear about *your* contribution. Say "I profiled", "I implemented", "I validated" — not "we fixed it". Prepare this story for questions about: optimization, initiative, problem-solving under constraint, or technical depth.

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
