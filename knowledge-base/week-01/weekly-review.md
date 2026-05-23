# Week 01 Review — Days 1–7

## Phase
Phase 1: DSA Mastery | Month 1

## STAR Framework — Week 1 Consolidated Patterns

> This section provides a consolidated STAR framework covering the core patterns from Week 1. Use it to practice rapid pattern narration — a critical interview skill.

---

### Two Pointers (Days 1–2): STAR Summary

**Situation:** "Given a sorted array and a sum/area constraint, brute force nested loops are O(n²) — for n = 10⁶ that's ~17 minutes. The sorted structure means we can make a guaranteed directional decision on every step."

**Task:** "Reduce pair-sum to O(n) and area/greedy problems to O(n) by converging two boundary pointers inward based on the comparison at each step."

**Action (key steps to narrate):**
1. State `left = 0, right = n-1`. For pair-sum: if sum too small, `left++`; too large, `right--`. For area (Container With Most Water): always move the shorter pointer — moving the taller bar can only decrease the result.
2. For 3Sum: sort first, fix one element with an outer loop, run two pointers inside, skip duplicates at both the fixed index AND inside the inner loop.
3. Terminate when `left >= right`.

**Result:** "O(n) vs O(n²). For n = 10⁶: ~1ms vs ~17 minutes for pair-sum."

---

### Sliding Window (Days 3–6): STAR Summary

**Situation:** "Given an unsorted array and a subarray constraint (sum, character count, product), brute force checks all O(n²) subarrays in O(n²)–O(n³) time — timing out for n ≥ 10⁴."

**Task:** "Maintain a window [left, right] with incremental state updates — O(1) per step — so total complexity is O(n) amortised."

**Action (key steps to narrate):**
1. Expand `right` on every step, updating window state (sum, freq map, product, zero count).
2. Shrink `left` while the window violates the constraint — use `while` for minimum windows, `if` or monotone approach for maximum windows.
3. Record the answer (length, count, or indices) at the right moment: before shrinking for minimum-window problems, after expanding for maximum-window problems.
4. Amortised argument: each element enters once and exits at most once → total ≤ 2n operations.

**Result:** "O(n) via amortised analysis. The amortised argument — 'each element enters once and exits at most once' — is the senior-level justification the interviewer expects when asking why the nested while doesn't make this O(n²)."

---

### Dutch National Flag / Three Pointers (Day 7): STAR Summary

**Situation:** "Partition an array of 0s, 1s, and 2s in-place with a single pass — built-in sort is O(n log n), counting sort is two passes."

**Task:** "Maintain three pointer invariants simultaneously: [0, low) = 0s, [low, mid) = 1s, [high+1, n) = 2s, [mid, high] = unexplored."

**Action (key steps to narrate):**
1. On `nums[mid] == 0`: swap with `low`, advance both `low` and `mid`.
2. On `nums[mid] == 1`: advance `mid` only.
3. On `nums[mid] == 2`: swap with `high`, advance `high` only — do NOT advance `mid` (swapped value is unknown).

**Result:** "O(n) single pass, O(1) space. The critical insight: do not increment `mid` after swapping with `high` — this is the invariant that separates correct solutions from plausible-looking bugs."

---

## Patterns Covered This Week
- Two Pointers (basic: pair sum, palindrome, reverse)
- Two Pointers (greedy shrink: container/area problems)
- Two Pointers (slow/fast: remove duplicates, is subsequence)
- Two Pointers (three-pointer: Dutch National Flag)
- Sliding Window — fixed size (stock price, average, anagram)
- Sliding Window — variable size (sum constraint, product constraint, budget constraint)
- Sliding Window + Character Frequency Map (longest repeating replacement, permutation in string, find all anagrams)
- Sliding Window — at-most-k-distinct (fruit into baskets)
- Sliding Window — zero-count (max consecutive ones III)

## System Design Topics Covered
**Big-O Notation & the RAM Model:** Covered asymptotic complexity classes (O(1) through O(2ⁿ)), best/average/worst case distinctions, amortised analysis for sliding window, and the RAM model's uniform cost assumption. Explored real-world limits of Big-O: cache hierarchy effects on algorithm performance, and the gap between algorithm complexity and output complexity.

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Two Pointers — basic | | |
| Two Pointers — greedy shrink | | |
| Two Pointers — slow/fast | | |
| Three Pointers (Dutch National Flag) | | |
| Sliding Window — fixed | | |
| Sliding Window — variable sum | | |
| Sliding Window — freq map | | |
| Sliding Window — k-distinct | | |
| Big-O analysis | | |

## Weekly Flashcard Deck
Aggregate of all 5-per-day cards from Days 1–7 (35 cards):

| Q | A |
|---|---|
| How do you decide when to use the two-pointer pattern? | Sorted array OR need to find a pair/triplet with a sum/difference constraint, or palindrome verification |
| How do you eliminate duplicates in 3Sum without a set? | Sort first; skip `nums[i] == nums[i-1]` for the fixed index; skip duplicate left/right moves inside the inner while loop |
| What is the time complexity of two pointers on a sorted array of size n? | O(n) — each pointer travels at most n positions total |
| How does O(n²) scale when input doubles from n to 2n? | Work quadruples — (2n)² = 4n² |
| Write the loop condition that keeps two pointers from crossing. | `while left < right` |
| Why do you move the shorter pointer (not the taller) in Container With Most Water? | Moving the taller pointer can only keep width the same or smaller and can't increase the limiting height, so the area can only shrink |
| How do you keep track of the closest sum in 3Sum Closest? | Initialise `closest = nums[0]+nums[1]+nums[2]`; update whenever `abs(current_sum - target) < abs(closest - target)` |
| What is the in-place reverse string time and space complexity? | O(n) time, O(1) space |
| What is the memory access latency order from fastest to slowest? | Registers → L1 cache → L2 cache → RAM → Disk |
| How does cache locality affect algorithm choice? | Sequential array access is faster than pointer-chased linked list access even if Big-O is the same |
| How do you detect when to use a sliding window vs. two pointers? | Two pointers work on sorted arrays with directional shrink; sliding window works on any array/string where you need a contiguous range satisfying a constraint |
| What is the amortised argument for why sliding window is O(n)? | Each element is added to the window once (right++) and removed at most once (left++), so total operations ≤ 2n |
| How do you track the current window's character frequencies efficiently? | Use an array of size 26 (for lowercase letters) or a hashmap; increment on expand, decrement on shrink |
| In Best Time to Buy and Sell Stock, why is a single pass sufficient? | You only need the minimum price seen before the current day; one left-to-right scan tracks both simultaneously |
| What is the window invariant in Minimum Size Subarray Sum? | All elements in [left, right] sum to less than target; right expands until the sum meets target, then left shrinks |
| How do you check if a fixed-size window is an anagram of s1 in O(1) per step? | Maintain a `matches` counter; increment when a char's freq in window equals s1, decrement when it diverges; answer when matches == 26 |
| Why can max_freq in LC 424 only increase and never need to decrease? | You're looking for the longest valid window; if max_freq would decrease, the window can only stay the same size — it never needs to shrink below the current best |
| What is the validity condition for the window in LC 424? | `window_length − max_freq ≤ k` |
| How does a fixed sliding window differ from a variable one? | Fixed: window size is constant (k elements); slide by removing leftmost and adding new right. Variable: window grows/shrinks based on a constraint |
| Name a real-world system design analogue of the frequency map trick. | A denormalised read replica or precomputed aggregate table: pay extra write cost to make reads O(1) |
| How do you decide between two pointers and sliding window when both could apply? | If window validity depends only on endpoint values → two pointers; if it depends on internal state (counts, sums, distinct values) → sliding window |
| How do you solve Squares of a Sorted Array in O(n)? | Use two pointers at both ends; always place the larger absolute-value square at the current back position, then move that pointer inward |
| What is the at-most-k-distinct sliding window invariant? | `len(freq) ≤ k` at all times; shrink left until this holds whenever a new right element would violate it |
| In Max Consecutive Ones III, what does the window state track? | The count of zeros inside the current window; window is valid when zero_count ≤ k |
| How does a sliding window rate limiter mirror LC 904? | It keeps a window of at most k recent events; evict events outside the time window exactly like evicting elements that exceed the k-distinct limit |
| In the slow/fast two-pointer pattern for Remove Duplicates, what does the slow pointer represent? | The index of the last confirmed unique element written so far |
| How many valid subarrays end at index `right` when the product window is `[left, right]`? | `right − left + 1` — every subarray from any start in [left, right] to right is valid |
| Why must you return 0 early if k ≤ 1 in Subarray Product Less Than K? | Any single positive integer ≥ 1, so no subarray can have a product strictly less than 1 |
| How do you update the anagram match check in O(1) per window slide? | Maintain a `matches` counter: decrement when a char's window freq hits the target freq, increment when it leaves it |
| What is the difference between algorithm complexity and output complexity? | Algorithm complexity is how long the code runs; output complexity is how large the result is — they can differ |
| How do you check if s is a subsequence of t in O(n)? | Two pointers: i scans s, j scans t; increment i only when t[j] == s[i]; return i == len(s) |
| In Dutch National Flag, why do you NOT increment mid after swapping with high? | The element swapped from high is unknown — it could be 0, 1, or 2, so mid must re-examine it |
| What is the window invariant in Get Equal Substrings Within Budget? | Sum of abs(s[i]−t[i]) over the window ≤ maxCost; shrink left when it exceeds the budget |
| What is the key difference between a subsequence and a substring? | Substring: contiguous characters; Subsequence: any characters in order, gaps allowed |
| How many total pointer movements does three-pointer Dutch National Flag perform? | At most 3n — each of low, mid, high moves at most n steps, giving O(n) time |

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| — | — | — | — | — |

---

## STAR Quick Reference — Week 1

| Pattern | One-line STAR hook |
|---------|-------------------|
| Two Pointers (pair sum) | "Given a sorted array of 10M integers, applied converging two pointers → O(n) vs O(n²) — 1ms vs 17min." |
| Two Pointers (area/greedy) | "Given container widths, always moved the shorter pointer → O(n) globally optimal by elimination argument." |
| Three Pointers (Dutch National Flag) | "Given 0s/1s/2s, maintained three invariant zones simultaneously → O(n) single pass vs O(n log n) sort." |
| Sliding Window (fixed) | "Given k-day stock window, maintained a running sum with O(1) slide → O(n) vs O(n·k) brute force." |
| Sliding Window (variable sum) | "Given subarray sum target, used amortised shrink — each element enters and exits once → O(n) total." |
| Sliding Window (freq map) | "Given character replacement budget, maintained a match counter → O(1) validity check per step." |
| Sliding Window (k-distinct) | "Given at-most-k-distinct constraint, evicted excess keys on shrink → O(n) vs O(n²) nested scan." |

**Career story titles:**
1. "Two Pointers — cut pair-search latency from 17 min to 1 ms on a 10M-row sorted customer table."
2. "Sliding Window — reduced O(n²) log-window scan to O(n) amortised by maintaining incremental state."
3. "Dutch National Flag — partitioned mixed-status records in a single pass, eliminating a secondary sort step."
