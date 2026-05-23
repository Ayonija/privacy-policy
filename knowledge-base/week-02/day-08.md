# Day 08 — Two Pointers: Harder Variants
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Push two-pointer fluency into less-obvious setups: multi-number sums, interval merging, and non-obvious array partitioning.

## DSA (2 hours)
### Pattern: Two Pointers — 4Sum & Interval Intersection
- 4Sum extends 3Sum: fix two outer indices, run two pointers inside — O(n³) total, but still polynomial and expected in interviews.
- Interval intersection with two pointers: advance the pointer whose interval ends earlier; the current intersection (if any) is `[max(start), min(end)]`.
- Trigger condition: k-sum problems (generalise the fix-and-inner-scan approach) OR merge/intersect two sorted interval lists.
- Time complexity: O(n³) for 4Sum | O(m+n) for interval intersection | Space complexity: O(1) extra

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Move Zeroes | 283 | Easy | Two Pointers (slow/fast) | Slow pointer = next write position for non-zero; fast pointer scans; swap or overwrite then fill zeros |
| 2 | 4Sum | 18 | Medium | Two Pointers + Sort (nested) | Sort, fix i and j with two loops, run two pointers for inner pair; skip all duplicate indices |
| 3 | Interval List Intersections | 986 | Medium | Two Pointers on two arrays | Advance the pointer whose interval ends first; overlap exists when max(start) ≤ min(end) |

### Code Skeleton
```java
class Solution {
    // 4Sum (LC 18)
    public static List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        int n = nums.length;
        for (int i = 0; i < n - 3; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;       // skip dup fixed i
            for (int j = i + 1; j < n - 2; j++) {
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;  // skip dup fixed j
                int left = j + 1, right = n - 1;
                while (left < right) {
                    long s = (long) nums[i] + nums[j] + nums[left] + nums[right];
                    if (s == target) {
                        result.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                        while (left < right && nums[left] == nums[left + 1]) left++;
                        while (left < right && nums[right] == nums[right - 1]) right--;
                        left++; right--;
                    } else if (s < target) {
                        left++;
                    } else {
                        right--;
                    }
                }
            }
        }
        return result;
    }

    // Interval Intersection (LC 986)
    public static int[][] intervalIntersection(int[][] A, int[][] B) {
        int i = 0, j = 0;
        List<int[]> result = new ArrayList<>();
        while (i < A.length && j < B.length) {
            int lo = Math.max(A[i][0], B[j][0]);   // overlap start = later of two starts
            int hi = Math.min(A[i][1], B[j][1]);   // overlap end = earlier of two ends
            if (lo <= hi) {                          // actual overlap exists
                result.add(new int[]{lo, hi});
            }
            if (A[i][1] < B[j][1]) {               // advance the interval that ends sooner
                i++;
            } else {
                j++;
            }
        }
        return result.toArray(new int[0][]);
    }
}
```

### Interview Tips

- **4Sum duplicate-skip guard difference:** for the fixed `i` index use `i > 0`; for the fixed `j` index use `j > i + 1` (not `j > 0`). Using the wrong guard is the most common 4Sum bug. Explain why: j resets to `i+1` each outer iteration.
- **Integer overflow in k-sum:** with 32-bit integers, `nums[i] + nums[j] + nums[left] + nums[right]` can overflow if values are large — in Java/C++ cast to `long`. In Python this is handled automatically; mention it to show language awareness.
- **Interval intersection formula:** "Two intervals [a,b] and [c,d] overlap iff `max(a,c) ≤ min(b,d)`. If this holds, their intersection is exactly `[max(a,c), min(b,d)]`." Stating this formula before drawing will impress interviewers.
- **Brute force for 4Sum:** O(n⁴) checking all quadruples → sorting + two nested loops + two pointers = O(n³). Always ask the interviewer for expected n to know if O(n³) is acceptable.
- **Common mistake:** in Move Zeroes, swapping every non-zero with the slow-pointer position when you could just overwrite non-zeros left-to-right and fill zeros at the end — both O(n) but fewer writes.

### STAR Interview Framework

> **How to use the STAR method when explaining Two Pointers — 4Sum & Interval Intersection in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given two sorted interval lists and asked to find all their intersecting intervals. A brute-force approach of comparing every interval from list A against every interval from list B is O(m×n) — for m = n = 10⁴ that's 10⁸ comparisons, borderline but still too slow and unnecessary given the sorted order."

**Task:** "My goal was to solve this in O(m+n) by recognising that two sorted interval lists can be scanned with two pointers — we advance the pointer whose interval ends sooner, since it can no longer intersect with anything ahead in the other list."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "Two sorted input sequences, pairwise comparison — two-pointer on two arrays. The advance rule is: move the pointer with the smaller end value."
2. *Initialize:* "I set `i = 0, j = 0`. At each step, I compute the potential intersection: `lo = max(A[i][0], B[j][0])` and `hi = min(A[i][1], B[j][1])`."
3. *Core loop logic:* "If `lo <= hi`, the intervals overlap — record `[lo, hi]`. Then advance the pointer whose interval ends sooner: if `A[i][1] < B[j][1]`, increment `i`; otherwise increment `j`."
4. *Convergence guarantee:* "Each step advances at least one pointer. At most m+n total advances — O(m+n). We correctly skip intervals that cannot produce any future intersections."
5. *Duplicate handling / edge case proactivity:* "Intervals touching at a single point — `[1,2]` and `[2,3]` — produce `lo=2, hi=2`. Since `lo <= hi`, this is a valid intersection `[2,2]`. Mention this upfront."

**Result:** "This reduces the time complexity from O(m×n) to O(m+n). For m = n = 10⁴, that's 10⁸ vs 2×10⁴ operations. The advance rule — move the interval that ends sooner — is the key insight, because a finished interval cannot intersect anything further right."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Two Pointers here |
|-------------|---------------------------|-------------------------------|
| Brute force O(m×n) | When both lists are very short (m,n ≤ 50) | Two pointers is O(m+n); brute force becomes impractical at 10⁴ each |
| Sort both lists then merge | When input lists are NOT sorted | If already sorted (as given), sorting would add O(n log n) unnecessarily; two pointers exploits existing sort |

**Why NOT brute force:** O(m×n) = 10⁸ for m=n=10⁴ — borderline or guaranteed timeout.
**Why NOT re-sorting:** The input is already sorted; re-sorting adds O(n log n) cost for no benefit.

### Edge Cases to Trace Before Coding
- LC 18 (4Sum): fewer than 4 elements → return `[]`; all same element (e.g., `[0,0,0,0]`, target=0) → one result `[[0,0,0,0]]`
- LC 18: target is very large or negative → sorting handles negative numbers correctly
- LC 986: one list is empty → while loop condition is false immediately, return `[]`
- LC 986: intervals touch at a single point (e.g., `[1,2]` and `[2,3]`) → `lo=2, hi=2`, `lo <= hi` so `[2,2]` is a valid intersection
- LC 283 (Move Zeroes): all zeros → slow never advances, all writes are to the same position (no-op)

## System Design (1 hour)
### Topic: Big-O Revisit — When O(n³) is Acceptable
- 4Sum is O(n³): for n=100 that's 10⁶ ops, fine; for n=10⁶ it's 10¹⁸, unusable.
- Always ask the interviewer the expected input size; it changes the acceptable algorithm class.
- k-sum generalisation: k=2 → O(n), k=3 → O(n²), k=4 → O(n³); each extra dimension costs a factor of n.
- Interval merging/intersection at scale: sort by start time O(n log n) once, then single-pass O(n); used in calendar systems and IP range lookups.
- Interview talking point: "If asked to find free meeting slots for two people from their calendar arrays, answer: merge each person's intervals, then run the two-pointer interval intersection — O(m log m + n log n) for sorting, O(m+n) for the scan."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Earn Trust

**STAR Story: Finding Overlap Between Two Teams' Independent Release Timelines**

**Situation (20%):** "Two independent engineering teams at my company — the payments team and the fraud detection team — were both planning major infrastructure migrations scheduled 6 weeks apart. Neither team had consulted the other, and I discovered through a routine architecture review that the migrations had a critical shared dependency: they both required downtime on the same database cluster at overlapping windows."

**Task (part of S/T):** "I was the platform architect on call. My goal was to surface the conflict, mediate between both teams, and find a resolution that let both migrations proceed without either team losing more than 1 additional week of schedule."

**Action (60-70% — be specific about what YOU did):**
"First, I mapped out both teams' maintenance window requests and identified the exact overlap — 3 hours on a Saturday morning where both migrations required exclusive write access to the cluster.
Then, I arranged a joint meeting with both tech leads and presented my findings with a concrete timeline diagram. I framed it as 'here's what I found and here are three options' rather than assigning blame.
Next, I proposed the three options: (1) sequence the migrations with a 1-week gap, (2) restructure the payments migration to avoid the shared table (would require 2 additional days of engineering), or (3) run both migrations in a single coordinated 6-hour window with a joint runbook. I let both teams evaluate the trade-offs.
Finally, I volunteered to write the joint runbook for Option 3 if both teams chose it — removing the main coordination cost that was blocking agreement."

**Result (10-20%):** "Both teams chose Option 3. I wrote the joint runbook in 1 day. The migrations ran concurrently in a single 5-hour window with no incidents. Both teams stayed on schedule, and the joint coordination built enough trust that the two teams now share a monthly cross-team architecture sync they requested themselves."

**Interview tip:** Earn Trust questions reward proactive conflict identification and facilitated resolution. "I identified the overlap, presented options neutrally, and volunteered to do the coordination work" is the formula. Prepare for: earn trust, conflict resolution, collaboration, or influencing without authority.

## Flashcards

| Q | A |
|---|---|
| How do you generalise two-pointer pair-sum to k-sum? | Fix (k−2) elements with nested loops, then run one two-pointer pass for the remaining pair; time = O(n^(k-1)) |
| What is the duplicate-skip rule for the fixed indices in 4Sum? | `if i > 0 and nums[i] == nums[i-1]: continue` and similarly for j (with `j > i+1` guard) |
| In interval intersection, which pointer advances after checking overlap? | The pointer whose interval has the smaller end value — it can no longer intersect anything further right in the other list |
| How do you determine if two intervals [a,b] and [c,d] overlap? | They overlap if and only if `max(a,c) ≤ min(b,d)` |
| In Move Zeroes, how do you preserve relative order of non-zero elements? | Use slow pointer as a write position; overwrite with non-zero elements left to right (no swap needed if you fill zeros separately at the end) |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
