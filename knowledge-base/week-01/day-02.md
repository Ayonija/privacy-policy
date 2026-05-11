# Day 02 — Two Pointers: Depth & Variations
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Solidify two-pointer fluency by applying it to geometry-style problems and multi-number sums.

## DSA (2 hours)
### Pattern: Two Pointers — Greedy Shrink
- On area/distance problems, always move the pointer at the smaller value — moving the larger side can only decrease the result.
- Trigger condition: maximise/minimise a value that depends on the gap between two indices and a height/value array.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Reverse String | 344 | Easy | Two Pointers | Swap in-place; classic pointer-crossing template |
| 2 | Container With Most Water | 11 | Medium | Two Pointers (greedy shrink) | Move the shorter bar inward — moving the taller bar can't improve area |
| 3 | 3Sum Closest | 16 | Medium | Two Pointers + Sort | Track abs diff between current sum and target; same duplicate-skip discipline as 3Sum |

### Code Skeleton
```python
def container_with_most_water(height):
    left, right = 0, len(height) - 1
    max_area = 0
    while left < right:
        area = min(height[left], height[right]) * (right - left)
        max_area = max(max_area, area)
        if height[left] < height[right]:
            left += 1   # shorter side limits the area; moving it is the only way to possibly improve
        else:
            right -= 1
    return max_area
```

### Interview Tips

- **Prove the greedy argument out loud:** "Moving the taller pointer can only decrease width and can't increase the limiting height, so area can only shrink. Therefore we always move the shorter pointer." Interviewers at L5+ expect you to justify the greedy choice, not just state it.
- **Brute force baseline:** O(n²) is checking every pair `(i, j)` — state this before explaining the O(n) approach so the interviewer sees you understand what you're improving.
- **3Sum Closest communication tip:** initialise `closest` with the first three elements, not `float('inf')`, to avoid issues with the `abs()` comparison initialisation.
- **Common mistake:** in Container With Most Water, updating `max_area` inside the `if` branch rather than unconditionally — you must record the area *before* moving any pointer.
- **Alternative approach for Reverse String:** the two-pointer swap is O(1) space; mention that Python's `s.reverse()` does the same but isn't allowed in-place questions.

### Edge Cases to Trace Before Coding
- `height` has only 2 elements → still works; one iteration captures the only possible area
- All heights equal → any pointer movement is valid; result is correctly computed
- 3Sum Closest: array with exactly 3 elements → return their sum immediately (no choice exists)
- Negative numbers in 3Sum Closest → sorting + two-pointer handles negatives naturally

## System Design (1 hour)
### Topic: RAM Model — Memory Hierarchy Basics
- Registers (~0.3 ns) → L1 cache (~1 ns) → L2 (~4 ns) → RAM (~100 ns) → Disk (~1 ms).
- Algorithm analysis assumes uniform O(1) memory access; real performance degrades with cache misses.
- Sequential access patterns (arrays) are cache-friendly; pointer chasing (linked lists) is not.
- Why it matters in design: large working sets that don't fit in L3 cache can make O(n) feel like O(n²) in practice.
- Interview talking point: "If asked whether a hash map or sorted array is faster for 10⁶ lookups, answer: sorted array with binary search has better cache locality; hash map wins on average but suffers on cache misses for large, sparse key spaces."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Tell me about a situation where you had to make a trade-off between two competing constraints (speed vs. memory, precision vs. speed) — similar to the greedy shrink decision in Container With Most Water.
- Leadership principle: Are Right, A Lot

## Flashcards

| Q | A |
|---|---|
| Why do you move the shorter pointer (not the taller) in Container With Most Water? | Moving the taller pointer can only keep width the same or smaller and can't increase the limiting height, so the area can only shrink |
| How do you keep track of the closest sum in 3Sum Closest? | Initialise `closest = nums[0]+nums[1]+nums[2]`; update whenever `abs(current_sum - target) < abs(closest - target)` |
| What is the in-place reverse string time and space complexity? | O(n) time, O(1) space |
| What is the memory access latency order from fastest to slowest? | Registers → L1 cache → L2 cache → RAM → Disk |
| How does cache locality affect algorithm choice? | Sequential array access is faster than pointer-chased linked list access even if Big-O is the same |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
