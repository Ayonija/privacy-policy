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
```java
class Solution {
    public static int containerWithMostWater(int[] height) {
        int left = 0, right = height.length - 1;
        int maxArea = 0;
        while (left < right) {
            int area = Math.min(height[left], height[right]) * (right - left);
            maxArea = Math.max(maxArea, area);
            if (height[left] < height[right]) {
                left++;   // shorter side limits the area; moving it is the only way to possibly improve
            } else {
                right--;
            }
        }
        return maxArea;
    }
}
```

### Interview Tips

- **Prove the greedy argument out loud:** "Moving the taller pointer can only decrease width and can't increase the limiting height, so area can only shrink. Therefore we always move the shorter pointer." Interviewers at L5+ expect you to justify the greedy choice, not just state it.
- **Brute force baseline:** O(n²) is checking every pair `(i, j)` — state this before explaining the O(n) approach so the interviewer sees you understand what you're improving.
- **3Sum Closest communication tip:** initialise `closest` with the first three elements, not `Integer.MAX_VALUE`, to avoid issues with the `Math.abs()` comparison initialisation.
- **Common mistake:** in Container With Most Water, updating `maxArea` inside the `if` branch rather than unconditionally — you must record the area *before* moving any pointer.
- **Alternative approach for Reverse String:** the two-pointer swap is O(1) space; mention that Java's `StringBuilder.reverse()` does the same but isn't allowed in-place questions.

### STAR Interview Framework

> **How to use the STAR method when explaining Two Pointers — Greedy Shrink in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given an array of heights representing bars and asked to find two bars that form a container holding the most water. The brute-force approach of checking every pair `(i, j)` would give O(n²), which for n = 10⁵ means 10¹⁰ operations — about 10 seconds, right at the edge of timing out."

**Task:** "My goal was to solve this in O(n) time and O(1) space by recognising that sorted order — or in this case, directional knowledge — lets us make a guaranteed greedy choice at each step: always move the shorter bar inward."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "I noticed the area depends on only the two endpoint values — the minimum height and the width. That's the two-pointer greedy-shrink trigger: no internal window state needed."
2. *Initialize:* "I set `left = 0` and `right = n-1` to start at maximum width. I record `maxArea = min(height[left], height[right]) * (right - left)`."
3. *Core loop logic:* "At each step, I compute the current area and update `maxArea`. Then I ask: which pointer should I move? Moving the taller bar can only reduce the width AND can't increase the bottleneck height — so area can only decrease. Therefore I always move the shorter bar inward."
4. *Convergence guarantee:* "Each iteration moves exactly one pointer inward. The two pointers start n apart and meet in the middle — at most n-1 iterations. The algorithm is O(n)."
5. *Duplicate handling / edge case proactivity:* "I update `maxArea` unconditionally before moving any pointer — a common bug is updating inside the `if` branch and missing the area from one side."

**Result:** "This reduces the time complexity from O(n²) to O(n). For n = 10⁵, brute force takes ~10B operations (~10 seconds); the greedy shrink takes ~10⁵ operations (~0.1ms). The greedy correctness proof — moving the taller bar cannot help — is the key insight to state aloud."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Greedy Shrink here |
|-------------|---------------------------|-------------------------------|
| Brute force — all pairs O(n²) | When n ≤ 200 and interview context allows | O(n²) times out at n = 10⁵; greedy shrink is O(n) with a provably correct greedy choice |
| Divide & conquer | When the problem has a recursive substructure | Container With Most Water doesn't have overlapping subproblems; greedy shrink is simpler and equally optimal |

**Why NOT brute force:** For n = 10⁵, checking every pair is 10¹⁰ operations — borderline timeout or definite timeout depending on the judge.
**Why NOT divide & conquer:** It would achieve O(n log n) and require significant implementation complexity with no benefit over the O(n) greedy approach.

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

**Leadership Principle:** Are Right, A Lot

**STAR Story: Greedy Trade-off Between Speed and Accuracy in a Recommendation System**

**Situation (20%):** "In my previous role, our team was running a recommendation engine that re-ranked results using an exact pairwise scoring model — it was O(n²) per query, and as our catalog grew to 500K items, P95 latency hit 4 seconds. Users were abandoning the recommendation page and the business was losing engagement."

**Task (part of S/T):** "I was responsible for the ranking service's performance. My goal was to reduce response latency to under 200ms without degrading recommendation quality by more than 2% on our offline eval metric."

**Action (60-70% — be specific about what YOU did):**
"First, I analyzed the scoring model and identified that the pairwise comparisons were the bottleneck — analogous to checking every bar pair in the container problem.
Then, I proposed a greedy two-stage approach: a fast pre-filter using a lightweight score to reduce the candidate set from 500K to 500, followed by the full pairwise model on just the top 500. The key insight was that the fast pre-filter's error rate only affected the final result in edge cases — similar to proving the greedy shrink is correct by arguing that discarding the shorter bar can't yield a better answer.
Next, I ran an A/B experiment over 5 days with 10% of traffic, measuring both latency and offline quality. Quality dropped only 0.8%, well within the 2% budget.
Finally, I rolled out to 100% of traffic, reduced our compute cost by 60%, and wrote up the greedy pre-filter pattern as a team design principle."

**Result (10-20%):** "P95 latency dropped from 4 seconds to 130ms — a 97% improvement. Engagement on the recommendation page increased 22% in the month following rollout. Compute costs fell by 60%. The greedy two-stage pattern was adopted by two other ranking services in the org the following quarter."

**Interview tip:** Say "I analyzed", "I proposed", "I ran", "I rolled out" — not "the team decided". Prepare this story for questions about: data-driven decisions, trade-offs, algorithm design, or performance optimization.

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
