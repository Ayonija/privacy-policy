# Day 27 — Monotonic Stack: Car Fleet & Subarray Ranges
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Apply monotonic stack to fleet-merging and range-based aggregation problems — a step toward Hard-level stack usage.

## DSA (2 hours)
### Pattern: Monotonic Stack — Fleet Merging & Contribution Technique
- Car Fleet: sort by position descending; compute each car's time to reach target; a car that catches up to the car ahead joins its fleet (times are non-increasing in the stack); a car that takes longer starts a new fleet.
- Implement Stack using Queues: single-queue approach — after each push, rotate the queue so the new element is at the front.
- Sum of Subarray Ranges: for each element, find the number of subarrays where it is the minimum, and where it is the maximum; contribution = (as_max − as_min) × value.
- Trigger condition: "fleet merging / arrival time grouping" (car fleet) OR "aggregate subarray min/max contributions" (range/span problems).
- Time complexity: O(n log n) for car fleet (sort), O(n) for subarray ranges | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Implement Stack using Queues | 225 | Easy | Single-queue rotation | Push x, then rotate queue `size−1` times so x is at front; pop and peek are O(1) |
| 2 | Car Fleet | 853 | Medium | Monotonic stack on sorted times | Sort by position desc; push time-to-target; if new time ≤ stack top, it joins the fleet (don't push); stack size = fleet count |
| 3 | Sum of Subarray Ranges | 2104 | Medium | Monotonic stack (contribution technique) | Sum of ranges = sum of subarray maxima − sum of subarray minima; each computed in O(n) with two monotonic stack passes |

### Code Skeleton
```java
class Solution {
    // Car Fleet (LC 853)
    public static int carFleet(int target, int[] position, int[] speed) {
        int n = position.length;
        int[][] cars = new int[n][2];
        for (int i = 0; i < n; i++) {
            cars[i][0] = position[i];
            cars[i][1] = speed[i];
        }
        Arrays.sort(cars, (a, b) -> b[0] - a[0]); // sort by position descending
        Deque<Double> stack = new ArrayDeque<>();
        for (int[] car : cars) {
            double time = (double)(target - car[0]) / car[1];
            if (stack.isEmpty() || time > stack.peekLast()) {
                stack.addLast(time);
            }
            // else: this car catches up — joins the leading fleet, don't push
        }
        return stack.size();
    }

    // Sum of Subarray Ranges (LC 2104) — skeleton
    public static long subArrayRanges(int[] nums) {
        // sum_of_ranges = sum_of_subarray_maxima - sum_of_subarray_minima
        // use two monotonic stack passes (one increasing, one decreasing)
        // for each element: count subarrays where it is the min/max
        // contribution = element_value * left_count * right_count
        // TODO: implement
        return 0;
    }
}
```

### Interview Tips

- **Car Fleet: sort descending, not ascending:** "Sorting by position descending processes cars from nearest-to-target to farthest — this is the only order where a car can be checked against the fleet immediately ahead of it."
- **Why push only when time > stack top (not ≥):** "If two cars have exactly the same time-to-target, they arrive simultaneously and form a fleet — the rear car does NOT push; it joins. Use strict `>` to push, absorb on `≤`."
- **Contribution technique mental model:** "For sum of subarray ranges = sum of subarray maxima − sum of subarray minima. Compute each separately using two monotonic stack passes — one increasing (for minima), one decreasing (for maxima). The contribution of element i as min = `nums[i] * left_count * right_count` where counts use strict/non-strict boundaries to avoid double-counting."
- **Double-counting gotcha:** "For contribution technique: use strict < on the left boundary and ≤ on the right (or vice versa consistently) — never strict on both sides or non-strict on both sides, or you'll double-count elements with equal values."
- **Brute force for Sum of Subarray Ranges:** O(n²) enumerate all subarrays, compute min and max each time → monotonic stack is O(n).

### STAR Interview Framework

> **Monotonic Stack — Fleet Merging & Contribution Technique:** brute-force O(n²) → this approach O(n log n) / O(n) time, O(n) space

**S:** "Given positions and speeds of n cars heading to a target. Brute-force pairwise time comparison to count fleets is O(n²) — too slow for n=10^5. For subarray ranges, brute-force enumeration of all subarrays is also O(n²)."
**T:** "Need O(n log n) for car fleets (bottleneck is sorting) and O(n) for subarray ranges. Goal: count distinct fleets (car fleet) or compute sum of max−min across all subarrays (ranges)."
**A (60% of answer time):**
1. *Classify:* "Car Fleet trigger: 'group entities by arrival order when faster ones are blocked by slower ones ahead' — monotonic stack on sorted times. Subarray Ranges trigger: 'aggregate min or max contribution across all subarrays' — contribution technique."
2. *Init:* "Car Fleet: sort cars by position descending; maintain a stack of time-to-target values. Subarray Ranges: two passes with separate stacks; arrays `left[]` and `right[]` to store boundary distances."
3. *Loop/Step:* "Car Fleet: for each car, compute time; if time > stack top, push (new fleet); else skip (joins ahead fleet). Subarray Ranges: for each element find the nearest previous smaller (left boundary) and next smaller-or-equal (right boundary); contribution as minimum = `nums[i] * left[i] * right[i]`."
4. *Termination:* "Car Fleet: stack size equals fleet count. Subarray Ranges: sum contributions from both min and max passes; subtract min sum from max sum."
5. *Gotcha:* "In the contribution technique, use strict < on one side and ≤ on the other to avoid double-counting equal elements — this is the single most common wrong answer on this problem."
**R:** "O(n log n) / O(n) time, O(n) space. For n=10^5, the contribution technique runs in ~3ms vs ~10 seconds for brute force — the difference between a real-time analytics feature and a batch job."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Brute-force O(n²) nested loop | n ≤ 1,000 | Too slow for n=10^5 |
| Sorting + binary search per element | Finding a single boundary per element | Contribution technique needs both left and right boundaries simultaneously — stack is simpler and O(n) |

## System Design (1 hour)
### Topic: PACELC Theorem — Extending CAP with Latency
- **PACELC** (Daniel Abadi, 2012) extends CAP: "If Partition, choose Availability or Consistency; Else (normal operation), choose Latency or Consistency."
- CAP only describes behaviour during failures; PACELC adds the everyday trade-off: even without a partition, replicated systems must choose between low latency (async replication) and strong consistency (synchronous, higher latency).
- **PA/EL (high availability, low latency):** DynamoDB, Cassandra — fast reads/writes, eventually consistent.
- **PC/EC (strong consistency, higher latency):** Zookeeper, etcd — every operation goes through consensus, slower but always correct.
- **PC/EL hybrid:** Google Spanner — uses TrueTime API to achieve external consistency with relatively low latency by bounding clock uncertainty.
- Interview talking point: "If asked why strong consistency always has higher latency, answer: synchronous replication requires acknowledgement from a quorum of nodes before returning to the client; each round-trip adds network latency proportional to the distance to the farthest replica — for global deployments this can be 100s of milliseconds."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- Leadership principle: Dive Deep

**Full STAR Story — "Diagnosing a Fleet Merging Bug in a Logistics Dispatch System":**

**S (20%):** "At a logistics company, our dispatch system assigned delivery vehicles to routes using an estimated time-of-arrival model. Drivers reported that two vehicles were frequently being dispatched to the same delivery stop — a fleet-merging bug was causing one route to be silently absorbed into another, resulting in missed deliveries and ~$8K/week in re-dispatch costs."
**T:** "I was assigned to investigate. The bug was intermittent and had no error logs — goal was to find the root cause and fix it within one sprint without disrupting the live dispatch system."
**A (60% — use 'I' not 'we'):** "(1) I dug into the dispatch model and found it processed vehicles in order of GPS registration (effectively random) rather than by proximity to destination — the equivalent of sorting by arrival time without sorting by position first. (2) I traced three specific incident logs where the merging misfired and confirmed the root cause: two vehicles with nearly identical ETAs were being evaluated in the wrong order, causing the system to incorrectly conclude they were in the same fleet when they were not. (3) I rewrote the fleet-grouping logic to sort vehicles by descending distance to destination before running the ETA comparison stack — exactly the pre-sort step the monotonic stack approach requires. (4) I added a unit test suite of 40 cases covering equal ETAs, single-vehicle routes, and wrap-around dispatch zones."
**R (20%):** "The bug was eliminated — zero fleet-merging errors in the 6 weeks post-deployment. Re-dispatch costs dropped by $8K/week. My root-cause write-up was used by the team lead to justify a broader audit of the dispatch model's sorting assumptions."
*Works for LP questions on: Dive Deep; Ownership; Are Right, A Lot.*

## Flashcards

| Q | A |
|---|---|
| Why does Car Fleet sort by position descending before using a stack? | You want to process cars from front to back (closest to target first); a car behind can only catch a car in front, not vice versa |
| When does a car NOT create a new fleet in the Car Fleet problem? | When its time-to-target ≤ the fleet ahead (stack top) — it will catch up and join that fleet, so it is not pushed |
| How does the contribution technique compute sum of subarray minima? | For each element, find `left[i]` (distance to previous smaller) and `right[i]` (distance to next smaller or equal); contribution = `nums[i] * left[i] * right[i]` |
| What does the ELC part of PACELC stand for? | Else (no partition) — choose between Latency and Consistency in normal operation |
| Why does Google Spanner achieve relatively low latency despite external consistency? | Uses TrueTime (GPS + atomic clocks) to bound clock skew; transactions wait only for the uncertainty window (~7ms) rather than a full quorum round-trip |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
