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
```python
# Two Pointers — fill from back (LC 977)
def sorted_squares(nums):
    n = len(nums)
    result = [0] * n
    left, right, pos = 0, n - 1, n - 1
    while left <= right:   # NOTE: <= not < because we need the last element too
        if abs(nums[left]) > abs(nums[right]):
            result[pos] = nums[left] ** 2
            left += 1
        else:
            result[pos] = nums[right] ** 2
            right -= 1
        pos -= 1
    return result

# Sliding Window — at most k distinct (LC 904 pattern)
def at_most_k_distinct(arr, k):
    freq = {}
    left = result = 0
    for right in range(len(arr)):
        freq[arr[right]] = freq.get(arr[right], 0) + 1
        while len(freq) > k:
            freq[arr[left]] -= 1
            if freq[arr[left]] == 0:
                del freq[arr[left]]   # must delete, not just set to 0, or len(freq) is wrong
            left += 1
        result = max(result, right - left + 1)
    return result
```

### Interview Tips

- **5-second pattern classifier:** Is the array sorted, and does the answer depend only on the two endpoint values? → Two pointers. Does the answer depend on something *inside* the window (counts, sums, distinct values)? → Sliding window. Practice this classification reflex until it's automatic.
- **LC 977 key insight to state:** "Largest squares always come from either the leftmost or rightmost element — we compare both ends and fill the result array from the back." Say the word "from the back" — interviewers listen for this.
- **Critical bug prevention for LC 904:** when removing from the freq map, `del freq[key]` (not just `freq[key] = 0`) is mandatory — otherwise `len(freq)` stays wrong and you'll never shrink the window correctly.
- **Brute force for LC 904:** O(n²) checking every subarray with a set — sliding window is O(n).
- **Common mistake:** using `left <= right` in LC 977's while loop — since we fill `pos` from back and `left` and `right` can meet on the last valid pair, `left <= right` is needed (not `left < right`).

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
- Space optimisation: frequency map removal when count → 0 is mandatory to keep `len(freq)` accurate; forgetting this is a common bug.
- Interview talking point: "If asked how a sliding window rate limiter works, answer: maintain a deque or sorted list of request timestamps; evict timestamps older than the window size on each new request; reject if count ≥ limit."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Give an example where you had to work within a hard resource constraint (time, budget, headcount) and had to maximise output within that constraint — mirroring the "at most k" sliding window problem.
- Leadership principle: Frugality

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
