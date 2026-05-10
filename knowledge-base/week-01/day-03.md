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
```python
def sliding_window_variable(arr, target):
    left = 0
    window_state = 0  # e.g. current sum or freq map
    best = float('inf')
    for right in range(len(arr)):
        window_state += arr[right]           # expand
        while window_state >= target:        # contract condition
            best = min(best, right - left + 1)
            window_state -= arr[left]
            left += 1
    return best if best != float('inf') else 0
```

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
- STAR prompt: Describe a time you found a more efficient approach to a recurring task that previously required doing the same work repeatedly (analogous to re-computing the window from scratch vs. incrementally updating it).
- Leadership principle: Invent and Simplify

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
