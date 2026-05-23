# Day 05 — Two Pointers + Sliding Window: Combined Application
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Apply both patterns to problems where the right choice between them is non-obvious — a key interview skill.

## DSA (2 hours)
### Pattern: Choosing Between Two Pointers and Sliding Window
- Two pointers: input is sorted OR you move pointers based on a comparison (sum too big/small) — no state needed inside the window.
- Sliding window: input is unsorted OR you track state inside the window (char counts, distinct elements, product).
- Both achieve O(n); the difference is in *how* the window contracts and *what* you maintain.
- Trigger condition: if shrinking depends on the window's internal state → sliding window; if shrinking depends only on end values → two pointers.
- Time complexity: O(n) | Space complexity: O(1) to O(k)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Squares of a Sorted Array | 977 | Easy | Two Pointers | Largest squares come from either end; fill result array from the back |
| 2 | Fruit Into Baskets | 904 | Medium | Sliding Window + Freq Map | Max window with at most 2 distinct values; shrink when distinct > 2 |
| 3 | Max Consecutive Ones III | 1004 | Medium | Sliding Window | Window is valid when zeros inside ≤ k; track zero count, shrink when it exceeds k |

### Code Skeleton
```java
class Solution {
    // Two Pointers — fill from back (LC 977)
    public static int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        int left = 0, right = n - 1, pos = n - 1;
        while (left <= right) {   // NOTE: <= not < because we need the last element too
            if (Math.abs(nums[left]) > Math.abs(nums[right])) {
                result[pos] = nums[left] * nums[left];
                left++;
            } else {
                result[pos] = nums[right] * nums[right];
                right--;
            }
            pos--;
        }
        return result;
    }

    // Sliding Window — at most k distinct (LC 904 pattern)
    public static int atMostKDistinct(int[] arr, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        int left = 0, result = 0;
        for (int right = 0; right < arr.length; right++) {
            freq.put(arr[right], freq.getOrDefault(arr[right], 0) + 1);
            while (freq.size() > k) {
                freq.put(arr[left], freq.get(arr[left]) - 1);
                if (freq.get(arr[left]) == 0) {
                    freq.remove(arr[left]);   // must delete, not just set to 0, or freq.size() is wrong
                }
                left++;
            }
            result = Math.max(result, right - left + 1);
        }
        return result;
    }
}
```

### Interview Tips

- **5-second pattern classifier:** Is the array sorted, and does the answer depend only on the two endpoint values? → Two pointers. Does the answer depend on something *inside* the window (counts, sums, distinct values)? → Sliding window. Practice this classification reflex until it's automatic.
- **LC 977 key insight to state:** "Largest squares always come from either the leftmost or rightmost element — we compare both ends and fill the result array from the back." Say the word "from the back" — interviewers listen for this.
- **Critical bug prevention for LC 904:** when removing from the freq map, `freq.remove(key)` (not just `freq.put(key, 0)`) is mandatory — otherwise `freq.size()` stays wrong and you'll never shrink the window correctly.
- **Brute force for LC 904:** O(n²) checking every subarray with a set — sliding window is O(n).
- **Common mistake:** using `left <= right` in LC 977's while loop — since we fill `pos` from back and `left` and `right` can meet on the last valid pair, `left <= right` is needed (not `left < right`).

### STAR Interview Framework

> **How to use the STAR method when explaining Choosing Between Two Pointers and Sliding Window in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to solve problems where both two-pointer and sliding window could plausibly apply — the key interview test is demonstrating you can classify which pattern fits in under 60 seconds and explain why. I'll show this with the 'Fruit Into Baskets' problem: given an array, find the longest subarray with at most 2 distinct values."

**Task:** "My goal was to recognize that 'at most 2 distinct' is an internal state constraint (counting distinct elements inside the window), which means the window cannot shrink based purely on endpoint values — that rules out two-pointer and selects sliding window."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Five-second classification: Does validity depend only on endpoint values? → Two pointers. Does it depend on internal state (distinct counts, character frequencies, product)? → Sliding window. Here, 'at most 2 distinct values' requires tracking all values inside — sliding window."
2. *Initialize:* "I set `left = 0`, `freq = {}` (empty frequency map), `result = 0`. The window [left, right] maintains the invariant `len(freq) <= 2`."
3. *Core loop logic:* "For each `right`, add `arr[right]` to `freq`. While `len(freq) > 2`, decrement `freq[arr[left]]` and if it hits 0 remove it, then `left++`. Then `result = max(result, right - left + 1)`."
4. *Convergence guarantee:* "Each element enters once via `right++` and exits at most once via `left++`. Total operations ≤ 2n = O(n) amortised."
5. *Duplicate handling / edge case proactivity:* "Must use `freq.remove(key)` — not `freq[key] = 0` — when a count hits zero. Leaving a zero-count key breaks `len(freq)` and the shrink condition never triggers."

**Result:** "Both patterns achieve O(n) — the classifier determines correctness, not speed. Misclassifying (using two pointers when you need internal state) produces a buggy O(n) solution. For n = 10⁵, an O(n²) brute force (check all subarrays with a set) takes ~10¹⁰ operations (~10s); the sliding window takes ~2×10⁵ operations (~0.2ms)."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Sliding Window here |
|-------------|---------------------------|-------------------------------|
| Two Pointers | When input is sorted and validity depends only on endpoints | Fruit Into Baskets input is unsorted and requires internal distinct-count tracking — two pointers is incorrect |
| Brute force O(n²) with a set | When n ≤ 500 | Sliding window is O(n); brute force times out at n ≥ 10⁴ |

**Why NOT two pointers here:** Two pointers requires that moving a pointer inward always makes the window either more or less valid based on only the removed value. But validity here depends on the entire frequency map — removing one element of a duplicate type doesn't change the distinct count, so two-pointer shrink doesn't work.
**Why NOT brute force:** O(n²) for n = 10⁵ is 10¹⁰ ops — timeout.

### Edge Cases to Trace Before Coding
- LC 977: all negative numbers (e.g., `[-4,-3,-2]`) → sorted squares are `[4,9,16]` filled from back correctly
- LC 977: single element → left == right, one iteration, result is `[elem²]`
- LC 904: single fruit type (`k=1`) → longest run of the same fruit
- LC 1004 (Max Consecutive Ones III): `k = 0` → answer is the longest existing run of 1s; `k ≥ n` → answer is `n`

## System Design (1 hour)
### Topic: Big-O — Analysing Real Code Patterns
- Back-filled array technique (LC 977): avoids re-sorting a result; O(n) with two passes eliminated.
- "At most k distinct" is a generalised sliding window that underlies substring problems in interviews.
- In systems: bounded-resource windows map to rate limiters — a sliding window rate limiter keeps a window of recent requests, exactly like LC 904 but with timestamps.
- Space optimisation: frequency map removal when count → 0 is mandatory to keep `freq.size()` accurate; forgetting this is a common bug.
- Interview talking point: "If asked how a sliding window rate limiter works, answer: maintain a deque or sorted list of request timestamps; evict timestamps older than the window size on each new request; reject if count ≥ limit."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Frugality

**STAR Story: Maximising Output Within a Hard Headcount Constraint**

**Situation (20%):** "During a hiring freeze at my previous company, our team of 4 engineers was asked to deliver a feature set originally scoped for 8 engineers in the same 6-week timeline. The original plan had each engineer owning one feature end-to-end — doubling up was expected to simply halve throughput, but the business deadline was fixed."

**Task (part of S/T):** "As the tech lead, I was responsible for ensuring we shipped the highest-value features within the constraint. My goal was to identify which 4 features out of 8 gave the most business impact, then structure the work so we weren't blocked on shared dependencies."

**Action (60-70% — be specific about what YOU did):**
"First, I worked with the PM to score each feature by business impact (user reach × revenue lift) and development effort. I built a simple priority score: `impact / effort` — analogous to the sliding window constraint 'at most k distinct items', I was finding the maximum-value set within a fixed-capacity window.
Then, I dropped the 4 features with the lowest priority scores and proposed the cut to leadership with data. I got buy-in within 48 hours.
Next, I restructured the remaining 4 features so the shared API layer was built first by one engineer, then the other three features could proceed in parallel without blocking each other.
Finally, I held daily 10-minute standups where any blocked engineer could surface impediments immediately — the constraint forced us to be ruthless about scope creep and dependencies."

**Result (10-20%):** "We shipped all 4 prioritized features on time with full test coverage. Post-launch analysis showed those 4 features drove 83% of the projected revenue impact despite being only half the scope. The remaining 4 features were deprioritized and two of them were ultimately cancelled — they would have been low-value even with full headcount. The structured prioritization approach became our team's standard process for subsequent planning cycles."

**Interview tip:** Frugality questions reward demonstrating judgment about what NOT to do. Show you made a deliberate, data-driven choice to constrain scope rather than cutting quality. Prepare for questions about: trade-offs, prioritization, working under constraints, or resource management.

## Flashcards

| Q | A |
|---|---|
| How do you decide between two pointers and sliding window when both could apply? | If the window's validity depends only on the two endpoint values → two pointers; if it depends on internal state (counts, sums, distinct values) → sliding window |
| How do you solve Squares of a Sorted Array in O(n)? | Use two pointers at both ends; always place the larger absolute-value square at the current back position, then move that pointer inward |
| What is the at-most-k-distinct sliding window invariant? | `len(freq) ≤ k` at all times; shrink left until this holds whenever a new right element would violate it |
| In Max Consecutive Ones III, what does the window state track? | The count of zeros inside the current window; window is valid when zero_count ≤ k |
| How does a sliding window rate limiter mirror LC 904? | It keeps a window of at most k recent events; evict events outside the time window exactly like evicting elements that exceed the k-distinct limit |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
