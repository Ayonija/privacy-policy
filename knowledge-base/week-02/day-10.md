# Day 10 — Arrays & Strings: Slot 1 Pattern Mastery Check
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Slot 1 close-out: verify pattern recognition speed, handle interval problems with two pointers, and conquer a deque-based sliding window.

## DSA (2 hours)
### Pattern: Sliding Window with Deque & Two Pointers on Ranges
- Deque-based sliding window: maintain a monotonic deque (increasing or decreasing) to track the window's max or min in O(1); deque stores *indices*, not values.
- Range/interval two pointers: when both arrays are sorted and you want all overlapping or adjacent pairs, advance the pointer whose range ends soonest.
- Trigger condition: "sliding window maximum/minimum" OR "longest window where max−min ≤ limit" → deque; "range union/intersection" → two-pointer on sorted intervals.
- Time complexity: O(n) for deque window | Space complexity: O(k) for deque

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Summary Ranges | 228 | Easy | Two Pointers (range collapse) | Advance fast pointer while consecutive; record [slow, fast] as a range; move slow = fast+1 |
| 2 | Interval List Intersections (revisit challenge) | 986 | Medium | Two Pointers | Re-solve from memory without skeleton; target: under 10 minutes |
| 3 | Longest Continuous Subarray With Absolute Diff ≤ Limit | 1438 | Medium | Sliding Window + Monotonic Deque | Maintain max_deque (decreasing) and min_deque (increasing); shrink left when max−min > limit |

### Code Skeleton
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    // Sliding Window with Monotonic Deque (LC 1438)
    public static int longestSubarray(int[] nums, int limit) {
        Deque<Integer> maxD = new ArrayDeque<>();  // indices in decreasing value order; front = current window max index
        Deque<Integer> minD = new ArrayDeque<>();  // indices in increasing value order; front = current window min index
        int left = 0, result = 0;

        for (int right = 0; right < nums.length; right++) {
            // Expand: add right to both deques (maintaining monotonic property)
            while (!maxD.isEmpty() && nums[maxD.peekLast()] <= nums[right]) {
                maxD.pollLast();
            }
            maxD.addLast(right);

            while (!minD.isEmpty() && nums[minD.peekLast()] >= nums[right]) {
                minD.pollLast();
            }
            minD.addLast(right);

            // Shrink: move left until max - min <= limit
            while (nums[maxD.peekFirst()] - nums[minD.peekFirst()] > limit) {
                left++;
                if (maxD.peekFirst() < left) maxD.pollFirst();   // front expired
                if (minD.peekFirst() < left) minD.pollFirst();
            }

            result = Math.max(result, right - left + 1);
        }
        return result;
    }
}
```

### Interview Tips

- **Two deques, not one:** LC 1438 needs both max and min simultaneously — use two separate monotonic deques. Explain this choice before coding: "I need the window max and min in O(1), so I'll maintain a decreasing deque for max and an increasing deque for min."
- **Always store indices, not values:** "I store indices so I can check if the front has fallen outside the current window [left, right] — if I stored values I'd lose this check." This is the single most important insight for any deque sliding window problem.
- **Deque operation order:** expand first (pop back, then append right), then check constraint, then shrink. Getting the order wrong produces incorrect results.
- **Brute force baseline:** O(n²) checking all subarrays and computing max-min with `max()` and `min()` each time (O(n) per call = O(n³) total) → two deques = O(n).
- **Common mistake:** checking `if maxD.peekFirst() < left` instead of `while maxD.peekFirst() < left` — but since we add one element per step and shrink one at a time, `if` is actually sufficient here (at most one can expire per iteration). Both work, but be able to explain why.

### STAR Interview Framework

> **How to use the STAR method when explaining Sliding Window with Monotonic Deque in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an array and a limit, and asked to find the longest subarray where the difference between the maximum and minimum is at most `limit`. The brute-force approach of checking all O(n²) subarrays and computing max-min via `max()` and `min()` is O(n³) — for n = 10⁵ that's 10¹⁵ operations. Even with a precomputed sliding range using a sorted structure it's O(n log n)."

**Task:** "My goal was to solve this in O(n) time and O(n) space by recognising that I need O(1)-per-step access to both the window's max and min — achievable with two simultaneous monotonic deques."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Sliding window where validity depends on `max - min` — an internal state I can't compute in O(1) without auxiliary structure. Two monotonic deques give O(1) max and O(1) min simultaneously."
2. *Initialize:* "I maintain `maxD` (decreasing deque, front = current window max index) and `minD` (increasing deque, front = current window min index). Set `left = 0`, `result = 0`."
3. *Core loop logic:* "For each `right`: pop from back of `maxD` while `nums[maxD.back] <= nums[right]` (maintain decreasing); push `right` to `maxD`. Similarly maintain `minD` (pop while `nums[minD.back] >= nums[right]`). Then while `nums[maxD.front] - nums[minD.front] > limit`: `left++`, and pop front of each deque if it's now out of window. Record `right - left + 1`."
4. *Convergence guarantee:* "Each element is pushed and popped from each deque at most once — O(n) total for all deque operations, O(n) amortised."
5. *Duplicate handling / edge case proactivity:* "Always store indices in the deque, not values — this is the critical insight for all deque sliding window problems. Storing values loses the ability to check if the front has slid out of the window."

**Result:** "This achieves O(n) time vs O(n log n) with a TreeMap or O(n³) brute force. For n = 10⁵, the difference between O(n) and O(n log n) is ~10⁵ vs ~1.7M operations — both fast, but the deque approach proves your algorithmic depth to the interviewer."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Monotonic Deque here |
|-------------|---------------------------|-------------------------------|
| Brute force O(n³) or O(n²) | When n ≤ 200 | O(n) deque approach vs O(n²)–O(n³); clear winner at n ≥ 10³ |
| Sorted multiset / TreeMap O(n log n) | When window minimum AND maximum are needed and n ≤ 10⁵ | TreeMap gives O(log n) per operation; deque gives O(1) amortised — deque is strictly faster |

**Why NOT brute force:** O(n²)–O(n³) for n = 10⁵ — timeout.
**Why NOT TreeMap/sorted set:** O(n log n) total with O(log n) per insert/delete — correct but ~17× slower than deque for n = 10⁵. Use TreeMap only when you need full sorted order of the window, not just max/min.

### Edge Cases to Trace Before Coding
- `limit = 0` → only windows where all elements are equal are valid; answer = length of longest equal-value run
- Single element → window size 1, always valid (max - min = 0 ≤ any limit ≥ 0)
- All same elements → maxD and minD both have same values; max - min = 0; entire array is valid
- Strictly decreasing array with limit = 0 → every window > 1 is invalid; answer = 1

## System Design (1 hour)
### Topic: Big-O & RAM Model — Slot 1 Synthesis
- Monotonic deque achieves O(1) amortised window max/min: each element is pushed and popped at most once.
- Deque vs. sorted structure: deque is O(n) total; maintaining a sorted multiset (like `TreeMap`) is O(n log n) — use deque when you only need max/min of the window.
- System design application: sliding window maximum mirrors a real-time anomaly detector — "flag if any reading in the last k seconds exceeds the min reading by more than X."
- Recap all Big-O rules learned: amortised O(n) for pointers that never backtrack, O(1) per step for incremental state updates, O(n log n) floor when sorting is unavoidable.
- Interview talking point: "If asked to find the maximum of every sliding window of size k, answer: use a monotonic decreasing deque of indices; pop from back when new element is larger, pop from front when index is out of window — O(n) total."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Customer Obsession

**STAR Story: Maintaining a Real-Time Priority View of Customer Incidents Without Rescanning**

**Situation (20%):** "At my previous company, our on-call team received 50–200 alerts per hour during peak traffic periods. The on-call engineer had to manually scroll through a Slack channel and re-read all recent alerts to identify the current highest-severity issue — this 'full scan' took 5–10 minutes, during which new critical incidents could arrive unnoticed. Customer impact windows were stretching to 20–30 minutes because of this prioritization delay."

**Task (part of S/T):** "I was responsible for on-call tooling. My goal was to give the on-call engineer a continuously updated 'current highest severity' view that stayed accurate as new alerts arrived and old ones resolved — without requiring manual re-scanning."

**Action (60-70% — be specific about what YOU did):**
"First, I designed a priority alert tracker as a monotonic data structure: a priority queue keyed by severity level, updated incrementally as alerts arrived and resolved. When a new alert arrived, it was inserted in O(log n) — when an alert was resolved, it was removed and the next highest severity was immediately surfaced.
Then, I built a Slack bot that maintained this live 'top incident' view and posted updates only when the top-severity incident changed — not on every alert. Engineers saw the current worst problem without reading every alert.
Next, I piloted the bot with 2 on-call engineers for 2 weeks and collected feedback on false positives and missed escalations.
Finally, I refined the severity scoring weights based on their feedback and deployed to the full on-call rotation."

**Result (10-20%):** "Mean time to acknowledge the highest-severity incident dropped from 18 minutes to 3 minutes — an 83% reduction. Customer impact window shortened by an average of 12 minutes. The on-call engineers reported 40% less cognitive load in post-rotation surveys. The bot was adopted by 3 other teams within the quarter."

**Interview tip:** Customer Obsession questions should connect your technical decision directly to customer impact. "The bot reduced customer impact windows by 12 minutes" is more powerful than "engineers liked it." Prepare for: customer obsession, delivering results, ownership, or bias for action.

## Flashcards

| Q | A |
|---|---|
| How does a monotonic decreasing deque find the window maximum in O(1)? | Front of deque always holds the index of the current window's max; pop from back any element ≤ new element before appending |
| When do you pop from the front of the deque? | When the front index is < left (it has slid out of the current window) |
| What two deques are needed for LC 1438 and what invariant does each maintain? | max_deque (decreasing values, front = window max) and min_deque (increasing values, front = window min) |
| How do you form the output for Summary Ranges? | When fast+1 is not consecutive or fast reaches end: if fast == slow output "slow", else output "slow→fast"; then set slow = fast+1 |
| Why prefer a monotonic deque over a sorted structure for sliding window max? | Deque is O(n) total (each element pushed/popped once); sorted structure costs O(n log n) and is only needed when you require the full sorted order |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
