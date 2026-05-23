# Day 03 — Sliding Window: Fixed & Variable Size
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Introduce the sliding window pattern — the go-to technique for contiguous subarray and substring problems.

## DSA (2 hours)
### Pattern: Sliding Window
- Maintain a window [left, right] that expands right on each step and shrinks left when a constraint is violated.
- Trigger condition: "longest/shortest subarray/substring satisfying a constraint" — whenever brute force would be O(n²) nested loops over a range.
- Time complexity: O(n) — each element enters and leaves the window at most once | Space complexity: O(k) where k is the window state (often O(1) or O(alphabet))

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Best Time to Buy and Sell Stock | 121 | Easy | Sliding Window (track min) | Track running min price; answer = max(current_price − min_so_far) |
| 2 | Longest Substring Without Repeating Characters | 3 | Medium | Variable Sliding Window + Set | Shrink left until duplicate is removed; use set or char→index map |
| 3 | Minimum Size Subarray Sum | 209 | Medium | Variable Sliding Window | Expand right until sum ≥ target, record length, shrink left; repeat |

### Code Skeleton
```java
class Solution {
    public static int slidingWindowVariable(int[] arr, int target) {
        int left = 0;
        int windowState = 0;  // e.g. current sum or freq map
        int best = Integer.MAX_VALUE;
        for (int right = 0; right < arr.length; right++) {
            windowState += arr[right];           // expand window
            while (windowState >= target) {      // shrink while condition holds
                best = Math.min(best, right - left + 1);
                windowState -= arr[left];
                left++;
            }
        }
        return best != Integer.MAX_VALUE ? best : 0;
    }
}
```

### Interview Tips

- **State the window invariant before coding:** "My window [left, right] always satisfies [condition]. I expand right on every step and shrink left only when [violation]." This one sentence proves to the interviewer you understand the algorithm.
- **`while` vs `if` for the shrink:** for minimum-window problems use `while` (keep shrinking as long as valid); for maximum-window problems where you just need to keep moving, `if` is often enough — mixing these up is a top-5 sliding window interview bug.
- **Brute force to state:** "I could check all O(n²) subarrays and sum each — O(n²) or O(n³). Sliding window maintains a running sum in O(1) per step."
- **Common mistake:** in LC 209 (Minimum Size Subarray Sum), computing `right - left + 1` *after* `left++` instead of before — the window size must be recorded while [left, right] is still the valid window.
- **Alternative approach for LC 3:** use a char→index map instead of a set to jump `left` directly to `last_seen[char] + 1` instead of incrementing one by one — still O(n) but avoids inner loop.

### STAR Interview Framework

> **How to use the STAR method when explaining Sliding Window in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an array of positive integers and a target, and asked to find the shortest contiguous subarray with sum ≥ target. The brute-force approach — checking all O(n²) subarrays and computing each sum — is O(n³) naively or O(n²) with a prefix sum, both of which time out for n = 10⁵ where constraints demand better."

**Task:** "My goal was to solve this in O(n) time and O(1) space by recognising that the constraint 'contiguous subarray satisfying a sum condition' is the sliding window trigger — I can maintain a running sum incrementally instead of recomputing from scratch."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "I noticed the problem asks for the shortest subarray where sum ≥ target, using a contiguous range. That's the variable sliding window trigger — expand right freely, shrink left when valid."
2. *Initialize:* "I set `left = 0`, `windowSum = 0`, `best = Integer.MAX_VALUE`. The window `[left, right]` always represents the current active range."
3. *Core loop logic:* "For each `right`, I add `arr[right]` to `windowSum`. Then, while `windowSum >= target`, I record `right - left + 1` as a candidate answer and shrink by adding `arr[left]` subtracted and `left++`. This greedily finds the smallest window satisfying the constraint ending at each `right`."
4. *Convergence guarantee:* "Each element enters the window at most once (via `right++`) and leaves at most once (via `left++`). Total pointer moves ≤ 2n, giving O(n) despite the nested `while` — this is the amortised argument."
5. *Duplicate handling / edge case proactivity:* "I record `right - left + 1` INSIDE the while loop, BEFORE `left++`. Recording it after `left++` gives the wrong window size — this is the top-5 sliding window bug."

**Result:** "This reduces the time complexity from O(n²) to O(n). For n = 10⁵, the difference is roughly 10¹⁰ operations (~10 seconds) vs 2×10⁵ operations (~0.2ms). The amortised argument — each element enters and exits at most once — is the key justification to state when the interviewer asks why the nested while doesn't make this O(n²)."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Sliding Window here |
|-------------|---------------------------|-------------------------------|
| Brute force — all subarrays O(n²/n³) | When n ≤ 500 | Sliding window is O(n); brute force times out at n ≥ 10⁴ |
| Prefix sum + binary search O(n log n) | When elements can be negative (sliding window doesn't shrink correctly for negatives) | All-positive inputs — sliding window is O(n) and simpler; prefix + binary search needed only for mixed signs |

**Why NOT brute force:** O(n²) for n = 10⁵ is 10¹⁰ operations — guaranteed timeout. Prefix sum reduces recomputation to O(1) per subarray but still O(n²) total.
**Why NOT prefix + binary search:** This works for negative numbers, but adds O(n log n) complexity and code complexity. When all values are positive (as in LC 209), sliding window is strictly better.

### Edge Cases to Trace Before Coding
- LC 121 (Stock): monotonically decreasing prices → profit is 0; handle by returning `Math.max(0, bestProfit)`
- LC 3 (No Repeating): all same character (e.g., `"aaa"`) → window never expands past 1; answer is 1
- LC 209 (Min Subarray Sum): no valid subarray (sum of entire array < target) → return 0
- Empty array → return 0 for all three problems

## System Design (1 hour)
### Topic: Big-O in Practice — Reading Complexity From Code
- Nested loops → O(n²); single pass with inner shrink (sliding window) → O(n).
- Sorting dominates many algorithms: adding a sort step makes the floor O(n log n).
- Recursive calls: T(n) = T(n/2) + O(1) → O(log n); T(n) = 2T(n/2) + O(n) → O(n log n) (merge sort).
- Amortised analysis: each element is processed at most twice in a sliding window → O(n) total even though the while loop is nested.
- Interview talking point: "If asked how sliding window achieves O(n) despite a nested while loop, answer: amortised — left pointer advances at most n total steps across all iterations of the outer loop."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Invent and Simplify

**STAR Story: Eliminating Redundant Recomputation in a Metrics Aggregation Service**

**Situation (20%):** "At my previous company, we had a real-time analytics dashboard that computed rolling 5-minute averages for 200 metrics every 10 seconds. Each refresh re-scanned the last 5 minutes of raw events from scratch — it was re-reading and summing the same data repeatedly. As event volume grew to 50K events per metric per minute, the aggregation job took 45 seconds per cycle, causing the dashboard to show data that was nearly a minute stale."

**Task (part of S/T):** "I was the owner of the metrics pipeline. My goal was to reduce the aggregation cycle to under 5 seconds so the dashboard could reflect data within a 10-second window — without increasing infrastructure costs."

**Action (60-70% — be specific about what YOU did):**
"First, I profiled the service and confirmed that 92% of compute was spent re-summing events that were included in the previous cycle's window — analogous to recomputing an array sum from scratch when a sliding window could maintain it incrementally.
Then, I redesigned the aggregation to use a sliding window approach: I maintained a running sum and a deque of timestamped event values, adding new events to the running sum as they arrived and subtracting events that slid out of the 5-minute window as time advanced.
Next, I benchmarked the new design on a staging environment with synthetic load matching 2× production volume — the cycle time dropped from 45 seconds to 1.8 seconds.
Finally, I deployed with a feature flag, validated dashboard correctness against ground truth for 24 hours, then enabled it for all 200 metrics."

**Result (10-20%):** "The aggregation cycle fell from 45 seconds to 1.8 seconds — a 96% reduction. Dashboard staleness dropped from ~55 seconds to under 5 seconds. We handled a 3× event volume increase without any infrastructure changes. I documented the incremental state update pattern, and it was applied to two other services in the next sprint."

**Interview tip:** Emphasize the before/after numbers and what *you* specifically changed in the code. Prepare this story for questions about: simplification, efficiency, initiative, or ownership.

## Flashcards

| Q | A |
|---|---|
| How do you detect when to use a sliding window vs. two pointers? | Two pointers work on sorted arrays with directional shrink; sliding window works on any array/string where you need a contiguous range satisfying a constraint |
| What is the amortised argument for why sliding window is O(n)? | Each element is added to the window once (right++) and removed at most once (left++), so total operations ≤ 2n |
| How do you track the current window's character frequencies efficiently? | Use an array of size 26 (for lowercase letters) or a hashmap; increment on expand, decrement on shrink |
| In Best Time to Buy and Sell Stock, why is a single pass sufficient? | You only need the minimum price seen before the current day; one left-to-right scan tracks both simultaneously |
| What is the window invariant in Minimum Size Subarray Sum? | All elements in [left, right] sum to less than target; right expands until the sum meets target, then left shrinks |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
