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
```python
# Car Fleet (LC 853)
def car_fleet(target, position, speed):
    cars = sorted(zip(position, speed), reverse=True)
    stack = []
    for pos, spd in cars:
        time = (target - pos) / spd
        if not stack or time > stack[-1]:
            stack.append(time)
        # else: this car catches up — joins the leading fleet, don't push
    return len(stack)

# Sum of Subarray Ranges (LC 2104) — skeleton
def sub_array_ranges(nums):
    # sum_of_ranges = sum_of_subarray_maxima - sum_of_subarray_minima
    # use two monotonic stack passes (one increasing, one decreasing)
    # for each element: count subarrays where it is the min/max
    # contribution = element_value * left_count * right_count
    pass
```

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
- STAR prompt: Describe a time you had to choose between delivering a result quickly with some uncertainty vs. taking more time to ensure complete correctness — directly mirroring the PA/EL vs PC/EC trade-off.
- Leadership principle: Dive Deep

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
