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

### STAR Interview Framework

> **How to use the STAR method when explaining Sliding Window — Product & Anagram Variants in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an array of positive integers and a target k, and asked to count the number of contiguous subarrays with product strictly less than k. The brute-force approach of checking all O(n²) subarrays and computing product for each is O(n²) or O(n³) if done naively — for n = 3×10⁴ that's up to ~2.7×10¹³ operations, clearly too slow."

**Task:** "My goal was to solve this in O(n) time and O(1) space by recognising that 'product of a contiguous subarray' can be maintained incrementally with multiplication and division — exactly the sliding window update pattern."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Contiguous subarray, product constraint on the window interior → sliding window. Product is maintained incrementally: multiply on expand, divide on shrink."
2. *Initialize:* "I return 0 immediately if `k <= 1` (all positive integers ≥ 1, so no product can be < 1). Otherwise `left = 0`, `product = 1`, `count = 0`."
3. *Core loop logic:* "For each `right`, multiply `product *= arr[right]`. While `product >= k`, divide `product /= arr[left]` and `left++`. Then add `right - left + 1` to `count` — that formula counts every subarray ending at `right` with start anywhere in `[left, right]`."
4. *Convergence guarantee:* "Each element enters once and exits at most once — O(n) amortised. The `right - left + 1` counting formula requires stating explicitly: 'every subarray starting from any index in [left, right] and ending at right is valid — there are `right - left + 1` such subarrays.'"
5. *Duplicate handling / edge case proactivity:* "The early return `if k <= 1 return 0` is non-obvious but critical. State it upfront and explain: with all elements ≥ 1, any product is ≥ 1, which is never strictly less than 1 or 0."

**Result:** "This reduces time complexity from O(n²) to O(n). The counting formula `right - left + 1` is a reusable technique that appears in at least 5 other LeetCode problems — naming it explicitly signals pattern depth. For n = 3×10⁴, O(n²) is ~10⁹ ops (~1 second, borderline); O(n) is ~6×10⁴ ops (~0.06ms)."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Sliding Window here |
|-------------|---------------------------|-------------------------------|
| Brute force — all subarrays | When n ≤ 500 | Sliding window is O(n); brute force is O(n²) and times out at n ≥ 10⁴ |
| Prefix product + binary search | When elements can be zero or negative (sliding window breaks) | All positive integers — sliding window works and is O(n) vs O(n log n) for prefix + binary search |

**Why NOT brute force:** O(n²) for n = 3×10⁴ is ~10⁹ operations — borderline to guaranteed timeout.
**Why NOT prefix product + binary search:** Prefix product division breaks when elements are zero; binary search adds O(log n) per step. For all-positive arrays, sliding window is strictly simpler and faster.

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

**Leadership Principle:** Deliver Results

**STAR Story: Shipping Feature Value Incrementally Instead of Waiting for Full Completion**

**Situation (20%):** "At my previous company, we were building a new search ranking system. The full feature set was a 4-month project. Three months in, leadership flagged that a competitor had launched a similar feature and asked if we could get any value to users sooner without throwing away the existing work."

**Task (part of S/T):** "I was the tech lead. My goal was to identify which completed components could ship independently and deliver measurable user value within 2 weeks — without blocking the remaining work or creating a maintenance burden."

**Action (60-70% — be specific about what YOU did):**
"First, I mapped out all completed components and evaluated each for independent deployability. I found that the query rewriting module — which improved result relevance for navigational queries — was fully tested and could ship behind a feature flag without any dependency on the remaining ranking components.
Then, I worked with QA to fast-track the query rewriter's test suite and get sign-off within 4 days.
Next, I proposed a phased rollout: 10% of traffic to the query rewriter alone, then measure precision and recall against our holdout set for 5 days before expanding. I personally monitored the dashboard during the initial rollout window each morning.
Finally, I coordinated with the PM to communicate the partial launch to stakeholders as a committed milestone — which reset expectations and bought the team 6 more weeks for the remaining components."

**Result (10-20%):** "The query rewriter shipped in 12 days and immediately lifted navigational search precision by 11%. Leadership pressure on the timeline reduced substantially once they saw concrete user impact. The remaining components shipped 6 weeks later, on the revised schedule. Shipping incrementally rather than all-at-once also surfaced two production edge cases early, which would have been much harder to debug in a big-bang release."

**Interview tip:** The key is showing you identified the deliverable boundary — what could ship independently — and executed the handoff cleanly. This mirrors counting subarrays ending at each step: you don't wait until the end to report results. Prepare for: delivering results, ownership, bias for action, or stakeholder management.

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
