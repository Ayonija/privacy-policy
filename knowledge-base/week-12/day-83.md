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

### STAR Interview Framework

> **How to use the STAR method when explaining Gas Station / Queue Reconstruction / Max Profit Job Scheduling in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three greedy problems with very different flavours. Gas Station requires a global feasibility check plus a single-pass greedy reset to find a circular route starting station. Queue Reconstruction by Height uses a counter-intuitive sort (descending height, not ascending) and inserts at a k-index — correct only because of the order guarantees. Max Profit Job Scheduling combines greedy sorting with DP and binary search for non-overlapping weighted interval selection. Each is subtle: the naive approaches (try all starts for Gas Station, try all orderings for Queue Reconstruction, try all subsets for Max Profit) are O(n²) to O(2^n)."

**Task:** "My goal was to identify the specific invariant that makes each greedy correct: for Gas Station, if total gas ≥ total cost, the start after the last negative-tank reset must work. For Queue Reconstruction, taller-first insertion means every insert sees exactly the right count. For Max Profit, sort by end time + binary search enables O(n log n) DP."

**Action:** Walk the interviewer through these steps:
1. *Gas Station — global feasibility + reset:* "Compute `total_tank` in parallel. When running `tank < 0`: no station between the previous start and current index can be a valid start (they all pass through this same negative point with even less starting gas). Reset `tank = 0`, set `start = i + 1`. At the end: if `total_tank ≥ 0`, return `start`; else return -1."
2. *Queue Reconstruction — sort and insert:* "Sort by height descending, tie-break by k ascending. Insert each person at index k in the result list. Correctness: when we insert `[h, k]`, all already-inserted people are ≥ h (because we process tallest first) — they all count toward k. People not yet inserted are shorter — when they insert at their own k-position, they don't affect `[h, k]`'s count because `[h, k]` doesn't count shorter people."
3. *Max Profit Job Scheduling — sort by end + binary search DP:* "Sort jobs by end time. Build `end_times = [0] + [job.end for job in jobs]`. For each job i: use `bisect_right(end_times, start[i])` to find j = last job ending ≤ start[i]. DP: `dp[i+1] = max(dp[i], profit[i] + dp[j])`. O(log n) per binary search, O(n) DP states → O(n log n) total."
4. *Why NOT try all starts for Gas Station:* "Trying all n starts: O(n²). But the global check (total_tank ≥ 0) tells us a valid start EXISTS if the check passes. The single-pass reset finds it in O(n) by eliminating all starts up to the index where the tank went negative."
5. *Why descending sort for Queue Reconstruction:* "If we sorted ascending by height, when we insert a shorter person at index k, they might affect the k-count of taller people inserted earlier (taller people are in front of shorter ones, so shorter inserted at k shifts taller people's positions). Descending sort avoids this by always inserting into a list that only contains taller or equal people."

**Result:** "Gas Station: O(n) vs O(n²) try-all-starts. Queue Reconstruction: O(n² log n) — sort O(n log n) + n insertions at O(n) each (list insertion is O(n)) — this is the unavoidable cost of list insert, but the algorithm complexity is correct for the approach. Max Profit Job Scheduling: O(n log n) vs O(2^n) subset enumeration. For n = 10^5 jobs, the DP + binary search gives 10^5 × 17 ≈ 1.7 × 10^6 operations vs 2^(10^5) — completely intractable."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer this approach |
|-------------|---------------------------|--------------------------|
| Try all starting stations for Gas Station | n ≤ 100 | O(n²) — fine for n ≤ 10^3; single-pass is O(n) always; use the greedy once you've seen it |
| Binary indexed tree for Queue Reconstruction | Need O(n log n) total including insertions | Segment tree can give O(log n) inserts; standard list insert is O(n); for interview, list insert is accepted |
| Priority queue DP for Max Profit | Online (jobs arrive in stream) | Offline (all jobs known): sort + binary search is simpler; online streaming: use a sorted structure like SortedList for O(log n) insertion and lookup |

**Why NOT sort by start time for Max Profit Job Scheduling:** DP requires knowing the last compatible job (ending ≤ current start). If jobs are sorted by start time, the end times are unsorted — binary search for `last job ending ≤ start[i]` requires sorting end times separately anyway. Sorting by end time directly enables the binary search invariant.

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

**Leadership Principle:** Ownership

**STAR Story: Taking Full Ownership of a Fan-Out System Failure That Spanned Three Teams**

**Situation:** At my previous company, we ran a Kafka-based event pipeline that fanned out a single "order completed" event to four downstream consumers: an email notification worker, an inventory update worker, a warehouse fulfillment worker, and an analytics worker. During a peak sales event, the inventory update worker began crashing in a tight restart loop due to a downstream database deadlock. Because all four workers shared a single Kafka consumer group, the crashing inventory worker's repeated rebalances disrupted partition assignments for the other three consumers — email notifications and fulfillment confirmations began arriving 20–45 minutes late. Customers were receiving no order confirmation emails and warehouse teams saw empty fulfillment queues. The incident lasted 3 hours before anyone fully understood the scope.

**Task:** I was an engineer on the analytics team — I was not the owner of the inventory worker, the email worker, or the fulfillment system. But as the person on call that night and the person with the deepest Kafka knowledge on the team, I took ownership of coordinating the full incident response. My goal was to restore email and fulfillment within the hour and then prevent recurrence, regardless of which team "owned" which component.

**Action:**

*First,* I identified the root cause within 15 minutes of being paged. The inventory worker crash loop was triggering Kafka consumer group rebalances every 30 seconds — the Kafka session timeout was 30 seconds, so each crash caused a full partition reassignment across all consumers in the group. I confirmed this by pulling consumer group lag metrics from Kafka's admin API and watching the rebalance counter increment in real time.

*Then,* I made the immediate tactical decision — which required me to act across team boundaries. I manually removed the inventory consumer from the consumer group using the Kafka consumer group management CLI, allowing the remaining three consumers to stabilise with their assigned partitions. Email notifications resumed within 4 minutes. Fulfillment resumed within 6 minutes. I escalated to the inventory team lead that their worker was isolated and they had full ownership of the fix on their end.

*Next,* I drafted and sent a customer impact summary to the CX team so they could proactively contact customers who had not received confirmation emails (approximately 3,200 customers), rather than waiting for inbound support tickets.

*Finally,* I wrote the post-mortem and proposed the structural fix: each downstream consumer should have its own consumer group (pure pub/sub fan-out), not share a consumer group with unrelated consumers. I also proposed a Kafka session timeout increase from 30 seconds to 90 seconds to reduce rebalance frequency on transient failures. Both changes were implemented within the following sprint across all four consumer groups — a cross-team change requiring coordination with three other teams.

**Result:** Email and fulfillment delays resolved from 20–45 minutes to 0 within 10 minutes of intervention. Approximately 3,200 customers received a proactive apology email rather than discovering the delay themselves — CX reported the ticket volume for this incident was 40% lower than for a comparable outage 6 months prior. The consumer-group isolation change reduced rebalance-related incidents to zero over the following 6 months. The incident also became the basis for a company-wide "consumer group isolation" guideline I published in our internal engineering wiki.

*In an interview, say:* "I want to be direct: this was not my system. I owned the analytics consumer, not the inventory worker or the email system. But when the incident spanned multiple teams and nobody was coordinating the response, I stepped in. Ownership, to me, means acting like this is your problem even when the org chart says otherwise — especially during a live customer impact event." Use this for "Tell me about a time you owned a problem outside your scope," "Tell me about a production incident you led," or "Tell me about a time you took initiative beyond your job description."

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
