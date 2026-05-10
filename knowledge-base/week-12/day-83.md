# Day 83 — Greedy: Gas Station, Queue Reconstruction, Max Profit Job Scheduling
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Gas Station uses a global feasibility check (total gas ≥ total cost) plus a single-pass greedy reset to find the unique valid starting station. Queue Reconstruction by Height sorts by height descending then inserts by k-position — a counter-intuitive greedy that works because taller people don't affect shorter people's counts. Max Profit Job Scheduling combines sorting, binary search, and DP to select non-overlapping jobs for maximum profit.

---

## DSA (2 hours)
### Pattern: Single-Pass Greedy Reset + Sort-and-Insert Greedy + Binary Search DP on Intervals

**Gas Station (LC 134):**
If `sum(gas) < sum(cost)`: impossible, return -1. Otherwise a unique solution exists. Single pass: maintain `tank` (running net gas). When `tank < 0`: reset `tank = 0` and set `start = i + 1` (the current station can't be part of a valid loop from any previous start). The start station at the end is the answer.

**Queue Reconstruction by Height (LC 406):**
Each person is `[h, k]` where `h` = height and `k` = number of taller/equal-height people in front. Sort by height descending, breaking ties by k ascending. Insert each person at index `k` in the result list. Correctness: when inserting person `p`, all previously inserted people are ≥ p's height, so their positions count correctly. People inserted later are shorter — they don't affect p's count.

**Max Profit Job Scheduling (LC 1235):**
Jobs have `(start, end, profit)`. Select non-overlapping jobs to maximize total profit. Sort jobs by end time. DP: `dp[i]` = max profit using first i jobs. For each new job i: find the last job j that ends ≤ start_i (binary search on sorted end times). `dp[i] = max(dp[i-1], profit_i + dp[j])`. Return `dp[n]`.

**Trigger condition:**
- "find the starting gas station for a circular route" → check total gas ≥ cost; single pass with start reset when tank goes negative
- "reconstruct queue by height and count of taller people in front" → sort desc by height, asc by k; insert at index k
- "select non-overlapping intervals to maximize total profit/value" → sort by end; DP + binary search for last compatible job

**Time complexity:** LC 134: O(n) | LC 406: O(n²) insertion; O(n log n) sort | LC 1235: O(n log n)
**Space complexity:** O(1) / O(n) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Gas Station | 134 | Medium | Single-pass greedy reset | If total gas ≥ total cost → solution exists; start = index after last negative tank |
| 2 | Queue Reconstruction by Height | 406 | Medium | Sort desc by h, asc by k; insert at index k | Taller people placed first; shorter people inserting at k-index don't affect taller people's counts |
| 3 | Max Profit Job Scheduling | 1235 | Hard | Sort by end + DP + binary search | `dp[i] = max(dp[i-1], profit + dp[last_compatible])` |

---

### Code Skeleton
```python
import bisect

# Gas Station (LC 134)
def canCompleteCircuit(gas, cost):
    total_tank = 0
    tank = 0
    start = 0
    for i in range(len(gas)):
        net = gas[i] - cost[i]
        total_tank += net
        tank += net
        if tank < 0:
            start = i + 1
            tank = 0
    return start if total_tank >= 0 else -1

# Queue Reconstruction by Height (LC 406)
def reconstructQueue(people):
    # Sort: descending height, ascending k on ties
    people.sort(key=lambda x: (-x[0], x[1]))
    result = []
    for person in people:
        result.insert(person[1], person)   # insert at index k
    return result

# Max Profit Job Scheduling (LC 1235)
def jobScheduling(startTime, endTime, profit):
    jobs = sorted(zip(startTime, endTime, profit), key=lambda x: x[1])
    # dp[i] = max profit considering first i jobs (1-indexed)
    dp = [0] * (len(jobs) + 1)
    end_times = [0] + [job[1] for job in jobs]   # for binary search

    for i, (s, e, p) in enumerate(jobs):
        # Find last job j (1-indexed) with end_times[j] <= s
        j = bisect.bisect_right(end_times, s, 0, i + 1) - 1
        dp[i + 1] = max(dp[i], p + dp[j])

    return dp[len(jobs)]
```

---

### Edge Cases to Trace Before Coding
- LC 134: all stations valid → start = 0; single station with gas ≥ cost → return 0; total gas < total cost → -1; last station completes the circuit → start anywhere before
- LC 406: all same height → sort by k only; k = 0 for everyone → insert all at front, result is just the sorted order; single person → trivially correct
- LC 1235: all jobs overlap → take the most profitable single job; all jobs disjoint → take all; single job → dp[1] = profit[0]

---

### Interview Pattern Drill

**Gas Station greedy correctness proof:**
If the total gas ≥ total cost, a solution exists (circular problem has one solution). The key insight: if you can't complete the circuit starting from 0 with some prefix sum going negative at index i, then no starting station from 0 to i can work (they all pass through the same negative point with less gas). So the start must be i+1 or later. The single pass eliminates all impossible starts.

**Queue Reconstruction correctness:**
After sorting by descending height, when we process person `p = [h, k]`:
- All people already in the result are ≥ h → they all count for p's k-value
- Inserting at index k places exactly k people in front of p
- People not yet processed are shorter → when they're inserted at their k-position, they don't count for p's k (p won't see them as ≥ h)

**Max Profit Job Scheduling — why binary search works:**
Jobs sorted by end time. For job i with start time s: we want the last job j where `end[j] ≤ s` (non-overlapping). Binary search on the `end_times` array gives this in O(log n). Combined with DP, we get O(n log n) overall.

| Pattern | Sort key | DP transition |
|---------|---------|--------------|
| Max Profit Job Scheduling | End time | `dp[i] = max(dp[i-1], profit + dp[last_compatible])` |
| Weighted Interval Scheduling | End time | Same template |
| LIS (comparison) | Start time | `dp[i] = max(dp[j] + 1)` for all j < i with compatible end |

---

## System Design (1 hour)
### Topic: Pub/Sub Fundamentals and Messaging Patterns

**What is Pub/Sub?**
Publishers send messages to a topic; subscribers receive all messages from subscribed topics. Unlike a point-to-point queue (one consumer per message), Pub/Sub delivers each message to ALL subscribers. Classic examples: Google Pub/Sub, AWS SNS, Redis Pub/Sub.

**Queue vs Pub/Sub:**
| Property | Message Queue | Pub/Sub |
|----------|--------------|---------|
| Delivery | One consumer receives each message | All subscribers receive each message |
| Persistence | Queue holds messages until consumed | Push delivery; limited persistence in most systems |
| Use case | Task distribution, work queues | Event broadcasting, fan-out notifications |
| Kafka model | Consumer groups (queue model) | Multiple consumer groups (pub/sub model) |

Note: Kafka supports BOTH models. Within a consumer group = queue model (partitioned work). Across consumer groups = pub/sub model (broadcast).

**Common messaging patterns:**

1. **Point-to-point (queue):** Order service → queue → one payment worker
2. **Pub/Sub (fan-out):** New post event → topic → [notification worker, analytics worker, search indexer, feed fanout worker]
3. **Request/Reply:** Client sends request with `reply_to` header; server responds on that queue. Enables async RPC.
4. **Dead Letter Queue (DLQ):** Messages that fail processing N times → moved to DLQ for investigation. Prevents poison-pill messages from blocking the queue forever.

**Dead Letter Queue implementation:**
```
[Main Queue] → Consumer attempts processing
                ├─ Success → ACK; message gone
                └─ Failure → NACK; retry
                     └─ After max_retries → DLQ
                            └─ Alert + manual investigation / replay
```

**Event sourcing vs message passing:**
- Message passing: fire-and-forget; consumer processes and discards
- Event sourcing: the event log IS the source of truth; services replay events to rebuild state; Kafka log retention enables this

**Fan-out patterns in newsfeed:**
```
New Post Event
    │
Kafka "new_posts"
    ├─ Consumer Group "feed-fanout" (one worker per partition)
    │    → ZADD feed:{follower_id} in Redis
    ├─ Consumer Group "notification-sender"
    │    → Push notifications via FCM/APNS
    ├─ Consumer Group "search-indexer"
    │    → Index post content in Elasticsearch
    └─ Consumer Group "analytics"
         → Stream to ClickHouse for engagement metrics
```

Each consumer group gets an independent copy of every message → pure Pub/Sub fan-out. Within each group, messages are load-balanced across workers → queue model for parallelism.

**Message ordering guarantees:**
- Kafka: strict ordering within a partition; no ordering across partitions
- SQS FIFO: strict ordering within a message group
- SQS Standard: best-effort ordering (faster, cheaper)
- RabbitMQ: FIFO within a single queue; no cross-queue ordering

**Backpressure:**
When consumers can't keep up with producers, the queue grows unboundedly. Solutions:
1. Rate-limit producers (slow publishing)
2. Scale out consumers (add more worker pods)
3. Drop low-priority messages
4. Apply circuit breaker: stop accepting new messages when queue depth > threshold

**Interview talking point:** "In our notification system, we use Kafka with multiple consumer groups as a Pub/Sub bus. A single 'new_post' event fans out to feed workers, notification workers, and search indexers — each group independently processes every event at its own pace. If the search indexer falls behind during a traffic spike, it lags on its own consumer group offset without affecting feed fanout or notifications. This isolation is a key advantage of Kafka over a shared SQS queue for multi-consumer scenarios."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 134 Gas Station (target: 15 min)
- **Medium 2:** LC 406 Queue Reconstruction by Height (target: 15 min)
- **Hard:** LC 1235 Max Profit Job Scheduling (target: 28 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you built a system that needed to fan out a single event to multiple downstream consumers — how did you handle failures in one consumer without affecting others?
- Leadership principle: Ownership

---

## Flashcards

| Q | A |
|---|---|
| How does Gas Station find the valid starting station in O(n)? | Maintain running `tank`. When `tank < 0`: set `start = i + 1`, reset `tank = 0`. After the loop: return `start` if `total_tank ≥ 0`, else -1. The reset works because no station between the previous start and i can be the valid start. |
| Why does Queue Reconstruction sort descending by height? | When inserting person `[h, k]`, all previously inserted people are ≥ h → they all count toward k. People inserted later are shorter → won't affect the count. Sorting descending ensures this invariant holds at every insertion step. |
| What is the DP recurrence for Max Profit Job Scheduling? | Sort by end time. `dp[i] = max(dp[i-1], profit[i] + dp[j])` where j = last job with `end[j] ≤ start[i]` (found via binary search on end times). `dp[0] = 0` (no jobs). |
| What is the difference between a message queue and Pub/Sub? | Queue: one consumer receives each message (competing consumers, load-balanced). Pub/Sub: ALL subscribers receive each message (fan-out). Kafka supports both: within a consumer group = queue; across consumer groups = pub/sub. |
| What is a Dead Letter Queue and why is it important? | DLQ receives messages that fail processing after max retries. Prevents poison-pill messages from blocking the main queue forever. Enables debugging, alerting, and manual replay. Essential for production message queue systems. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/28 min, no hints)
- [ ] Rewrote Gas Station single-pass and Max Profit job scheduling DP from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain queue vs pub/sub and Kafka fan-out cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
