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
```python
# Sliding Window with Monotonic Deque (LC 1438)
from collections import deque

def longest_subarray(nums, limit):
    max_d, min_d = deque(), deque()  # store indices
    left = result = 0
    for right in range(len(nums)):
        # maintain decreasing deque for max
        while max_d and nums[max_d[-1]] <= nums[right]:
            max_d.pop()
        max_d.append(right)
        # maintain increasing deque for min
        while min_d and nums[min_d[-1]] >= nums[right]:
            min_d.pop()
        min_d.append(right)
        # shrink left if constraint violated
        while nums[max_d[0]] - nums[min_d[0]] > limit:
            left += 1
            if max_d[0] < left: max_d.popleft()
            if min_d[0] < left: min_d.popleft()
        result = max(result, right - left + 1)
    return result
```

## System Design (1 hour)
### Topic: Big-O & RAM Model — Slot 1 Synthesis
- Monotonic deque achieves O(1) amortised window max/min: each element is pushed and popped at most once.
- Deque vs. sorted structure: deque is O(n) total; maintaining a sorted multiset (like `SortedList`) is O(n log n) — use deque when you only need max/min of the window.
- System design application: sliding window maximum mirrors a real-time anomaly detector — "flag if any reading in the last k seconds exceeds the min reading by more than X."
- Recap all Big-O rules learned: amortised O(n) for pointers that never backtrack, O(1) per step for incremental state updates, O(n log n) floor when sorting is unavoidable.
- Interview talking point: "If asked to find the maximum of every sliding window of size k, answer: use a monotonic decreasing deque of indices; pop from back when new element is larger, pop from front when index is out of window — O(n) total."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you maintained a "priority view" of a changing situation (e.g., kept track of the top issues in a fast-moving project) with minimal rework — analogous to a monotonic deque maintaining max without re-scanning the window.
- Leadership principle: Customer Obsession

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
