# Day 81 — Greedy & Backtracking: Interval Merging, Non-overlapping, Course Schedule III
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Interval problems share a single prerequisite: sort by start (or end). Merge Intervals groups overlapping ranges. Non-overlapping Intervals greedily keeps intervals with the earliest end time — the activity-selection greedy. Course Schedule III uses a max-heap to greedily swap out the longest completed course when a new course can't fit in remaining time.

---

## DSA (2 hours)
### Pattern: Sort + Merge + Greedy End-Time Selection + Max-Heap Course Swap

**Merge Intervals (LC 56):**
Sort intervals by start time. Iterate: if the current interval overlaps the last merged interval (`curr.start <= last.end`), merge by updating `last.end = max(last.end, curr.end)`. Otherwise append the current interval as a new merged interval.

**Non-Overlapping Intervals (LC 435):**
Minimum intervals to remove so no two overlap. Greedy: sort by END time. Greedily keep intervals with the earliest end — they leave the most room for future intervals (activity-selection greedy). Count intervals that overlap with the last kept interval — these are the removed ones.

**Course Schedule III (LC 630):**
Each course has `(duration, deadline)`. Take the maximum number of courses. Greedy: sort by deadline (earliest deadline first). Use a max-heap of durations of taken courses. For each course: try adding it (total_time += duration; push duration to heap). If total_time > deadline: evict the longest course taken so far (pop heap; subtract from total_time). This swap keeps the maximum count while ensuring feasibility.

**Trigger condition:**
- "merge all overlapping intervals" → sort by start; iterate and merge when `curr.start <= last.end`
- "minimum removals to eliminate all overlaps" → sort by end; count intervals that start before last end (they must be removed)
- "maximum number of tasks/courses within individual deadlines" → sort by deadline; max-heap; swap out longest task when over budget

**Time complexity:** LC 56: O(n log n) | LC 435: O(n log n) | LC 630: O(n log n)
**Space complexity:** O(n) / O(1) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Merge Intervals | 56 | Medium | Sort by start; merge on overlap | `curr.start <= last.end` → merge by taking max of ends |
| 2 | Non-Overlapping Intervals | 435 | Medium | Sort by end; greedy activity selection | Count overlaps with last kept end; those are removals |
| 3 | Course Schedule III | 630 | Hard | Sort by deadline + max-heap swap | Add course; if over deadline, evict longest course in heap |

---

### Code Skeleton
```python
import heapq

# Merge Intervals (LC 56)
def merge(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:   # overlap
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged

# Non-Overlapping Intervals (LC 435)
def eraseOverlapIntervals(intervals):
    intervals.sort(key=lambda x: x[1])   # sort by END time
    removals = 0
    last_end = float('-inf')
    for start, end in intervals:
        if start < last_end:   # overlaps with last kept interval
            removals += 1       # remove this one
        else:
            last_end = end      # keep this interval
    return removals

# Course Schedule III (LC 630)
def scheduleCourse(courses):
    courses.sort(key=lambda x: x[1])   # sort by deadline
    heap = []   # max-heap of durations (negate for Python)
    total_time = 0
    for duration, deadline in courses:
        heapq.heappush(heap, -duration)
        total_time += duration
        if total_time > deadline:
            # Evict the longest course taken so far
            total_time += heapq.heappop(heap)   # heap stores negated, so add (which subtracts)
    return len(heap)
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Interval Merge / Non-overlapping / Course Schedule III in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three interval scheduling problems requiring different greedy strategies. Merge Intervals asks for the consolidated set of overlapping ranges. Non-Overlapping Intervals asks for the minimum removals to eliminate all overlaps. Course Schedule III asks for the maximum number of courses you can complete given per-course deadlines — where choosing a course now might block better courses later. The brute-force approaches check all O(2^n) subsets or O(n!) orderings — infeasible at scale."

**Task:** "My goal was to reduce each to O(n log n) by identifying the correct greedy invariant: sort key matters differently for each variant (start time vs end time vs deadline), and the data structure changes for Course Schedule III (max-heap for swapping, not just scanning)."

**Action:** Walk the interviewer through these steps:
1. *Identify the sort key for each:* "Merge Intervals: sort by start time — overlap occurs when `curr.start ≤ last.end`. Non-Overlapping: sort by END time — the activity-selection greedy; keeping the earliest-ending interval maximises space for future intervals. Course Schedule III: sort by deadline — process each course at the earliest possible time against its own deadline."
2. *Merge Intervals core:* "Maintain a `merged` list. For each interval: if `curr.start ≤ merged[-1][1]` → overlap → extend `merged[-1][1] = max(merged[-1][1], curr.end)`. Else append as a new interval. One pass, O(n)."
3. *Non-Overlapping Intervals core:* "Maintain `last_end`. For each interval: if `curr.start < last_end` → overlap → count as removal. Else update `last_end = curr.end`. We always keep the interval with the earlier end — proven optimal by exchange argument."
4. *Course Schedule III core:* "Max-heap of taken course durations. For each course `(duration, deadline)`: push to heap, add to `total_time`. If `total_time > deadline`: pop the largest-duration course from the heap (swap it out). This maintains maximum count while staying feasible — we prefer more courses with smaller durations over fewer courses with larger ones."
5. *Exchange argument for Course Schedule III:* "If we ever swap a shorter course for a longer one we already took, we gain time without losing a slot. The max-heap always gives us the optimal swap target. This greedy is optimal because any solution using the same or fewer courses can be converted to this solution without increasing total_time."

**Result:** "All three: O(n log n) — dominated by the sort. Merge Intervals O(n log n) vs O(n² × L) naive merging with linear search. Non-Overlapping O(n log n) vs O(2^n) subset enumeration. Course Schedule III O(n log n) vs O(2^n) exhaustive subset search. For n = 10^5 intervals, the heap-based Course Schedule III processes each interval in O(log n) — 10^5 × 17 ≈ 1.7 × 10^6 operations vs 2^(10^5) — intractable."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer this approach |
|-------------|---------------------------|--------------------------|
| Sweep line with segment tree for Merge | Range queries needed after merge | Sort + linear merge is O(n log n) with O(n) space; segment tree adds constant overhead without benefit for static input |
| DP for Non-Overlapping | Weighted intervals (keep intervals with weights) | Unweighted activity selection: O(n log n) greedy is optimal; DP is O(n²) without binary search; use DP only when intervals have weights (Max Profit Job Scheduling, Day 83) |
| Priority queue without swap for Course Schedule III | Taking only deadline-feasible courses naively | Without the max-heap swap, you'd miss valid solutions: a course you included early may be worse than one seen later; the swap is the key insight |

**Why NOT greedy by duration alone for Course Schedule III:** Taking shortest courses first doesn't account for deadlines — a short course with a far-future deadline should not displace a longer course with an imminent deadline. Sorting by deadline first and then swapping on overflow is the correct two-dimension greedy.

---

### Edge Cases to Trace Before Coding
- LC 56: single interval → return as-is; all intervals disjoint → return sorted list unchanged; all intervals identical → merge to one
- LC 435: no overlaps → return 0; all intervals overlap in a chain → must remove all but one starting from the rightmost
- LC 630: courses with duration > deadline → can never be taken (adding exceeds deadline → heap pops it back out immediately); single course → take if duration ≤ deadline

---

### Interview Pattern Drill

| Pattern | Sort key | Decision condition | Action |
|---------|---------|-------------------|--------|
| Merge intervals | Start time | `curr.start <= last.end` | Merge: update last.end |
| Non-overlapping | End time | `curr.start < last_end` | Remove: count; else update last_end |
| Activity selection | End time | `curr.start >= last_end` | Keep: update last_end |
| Course Schedule | Deadline | `total_time > deadline` | Swap: evict longest in heap |

**Activity-selection greedy correctness:** By sorting by end time and always keeping the interval with the earliest end, we maximise the number of non-overlapping intervals (or equivalently, minimise removals). Any interval we skip has a later end time — keeping it would only reduce future options.

---

## System Design (1 hour)
### Topic: Message Queues — Fundamentals and Kafka Basics

**What is a message queue?**
A durable buffer between producers (writers) and consumers (readers). Decouples components: the producer doesn't wait for the consumer; the consumer processes at its own pace. Examples: Apache Kafka, AWS SQS, RabbitMQ, Google Pub/Sub.

**Core concepts:**
- **Producer:** publishes messages to a topic
- **Topic:** a named stream of messages (like a table in a DB)
- **Partition:** a topic is split into N ordered partitions; each message goes to one partition
- **Consumer:** reads messages from one or more partitions
- **Consumer group:** N consumers sharing the work; each partition assigned to one consumer at a time
- **Offset:** the position of a message within a partition; consumers commit offsets to track progress

**Why message queues?**
1. **Decoupling:** Post Service doesn't need to know about Notification Service
2. **Load leveling:** consumer processes at its own pace; queue absorbs bursts
3. **Durability:** messages persisted to disk; survive consumer crashes
4. **Fan-out:** one message → many consumers (Pub/Sub pattern)
5. **Replay:** Kafka retains messages for days; consumers can re-read historical data

**Kafka topic and partition model:**
```
Topic "new_posts" (3 partitions)
  Partition 0: [msg0, msg1, msg4, ...]  → consumed by Worker A
  Partition 1: [msg2, msg5, msg8, ...]  → consumed by Worker B
  Partition 2: [msg3, msg6, msg9, ...]  → consumed by Worker C
```

Messages within a partition are **strictly ordered**. Messages across partitions are NOT ordered relative to each other. To guarantee ordering for a specific entity (e.g., all posts from user 123): partition by `user_id` hash.

**Delivery guarantees overview:**
| Level | Behaviour | Risk |
|-------|-----------|------|
| At-most-once | Fire and forget; no retry | Messages lost on crash |
| At-least-once | Retry on failure; ack after processing | Duplicate messages possible |
| Exactly-once | Kafka transactions | Higher complexity and latency |

**Interview talking point:** "Message queues decouple producers from consumers and provide a durable buffer for handling traffic spikes. In the newsfeed system (Day 72), when Taylor Swift posts, the Post Service writes to Kafka and returns immediately. Fanout Workers pull from Kafka and push to Redis — no synchronous coupling. If fanout workers go down, messages accumulate in Kafka; on restart, workers pick up where they left off using committed offsets."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 56 Merge Intervals (target: 15 min)
- **Medium 2:** LC 435 Non-Overlapping Intervals (target: 15 min)
- **Hard:** LC 630 Course Schedule III (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)

**Leadership Principle:** Invent and Simplify

**STAR Story: Simplifying a Complex Deployment Pipeline by Decoupling CI and Release**

**Situation:** At my previous company, our deployment pipeline was tightly coupled: every CI build that passed automatically triggered a production release. This had been fine when the team was small and shipped once a week. As the team grew to 12 engineers shipping multiple times per day, the coupling created a serious problem: any flaky test or unrelated CI failure blocked the entire deployment queue. On average, we had 4–6 blocked deployments per day, each requiring manual intervention. Engineers spent roughly 45 minutes daily unblocking the pipeline — wasted time that had nothing to do with the code they were shipping.

**Task:** I was asked to fix the deployment bottleneck without disrupting the team's current workflow. My constraint: the solution had to be simple enough that any engineer on the team could operate it without reading documentation. We had previously considered a full GitOps migration (ArgoCD + declarative configs) but shelved it as too complex for the team's current maturity level.

**Action:**

*First,* I mapped the failure modes. The two main causes of pipeline blockage were: (1) flaky integration tests that passed 90% of the time but randomly failed, and (2) long-running end-to-end tests (12–15 minutes) that delayed downstream deployments even when they eventually passed. Both could be separated from the release decision without compromising safety.

*Then,* I proposed and implemented a two-stage decoupling: separate the CI stage (run all tests, produce an artifact) from the Release stage (deploy the artifact to production). The key change: CI failures would mark a build as "not yet release-ready" but would NOT block other builds from moving to the Release stage. Release required explicit promotion — a one-click action in the CI dashboard by any engineer.

*Next,* I introduced a release queue backed by a simple Redis sorted set (score = artifact timestamp) and a separate release daemon that picked the top artifact, ran a quick smoke test (30 seconds), and deployed. This replaced the previous implicit FIFO queue baked into the CI tool with an explicit, inspectable queue that engineers could understand and manipulate. Flaky tests became non-blocking: engineers could re-run a flaky test independently without re-running the full 15-minute pipeline.

*Finally,* I ran the new system in shadow mode for one sprint — it would have deployed in parallel with the old system — and compared outcomes. In shadow mode, the new system would have saved 38 hours of blocked-deployment time across the team in two weeks.

**Result:** After full rollout, blocked deployments dropped from 4–6 per day to 0–1 per day (average 0.4/day — a 90% reduction). Time spent on pipeline debugging fell from 45 minutes/day team-wide to under 5 minutes. Deployment frequency increased from an average of 8 per day to 14 per day (75% improvement) as engineers felt safer and less hesitant to ship. The solution required no new infrastructure beyond the Redis queue we already had — no new tools, no training, no documentation beyond a one-paragraph README.

*In an interview, say:* "The instinct was to adopt a sophisticated GitOps solution. But the real problem wasn't the tools — it was the tight coupling between test outcomes and deployment decisions. The invention was recognising that decoupling those two stages with a simple explicit queue was all we needed. Simplicity was the invention." Use this for "Tell me about a time you simplified a process," "Tell me about a time you improved engineering productivity," or "Tell me about a time you chose a simple solution over a complex one."

---

## Flashcards

| Q | A |
|---|---|
| How do you merge overlapping intervals after sorting? | Sort by start time. Keep a `merged` list. For each interval: if `curr.start <= merged[-1][1]` → overlap → update `merged[-1][1] = max(merged[-1][1], curr.end)`. Else append current interval as new entry. |
| How does the non-overlapping intervals greedy work? | Sort by end time. Scan left to right: if `curr.start < last_end` → this interval overlaps → count as removed. Else update `last_end = curr.end` (keep this interval). Return count. |
| How does Course Schedule III use a max-heap? | Sort courses by deadline. For each: add duration to total_time; push to max-heap. If `total_time > deadline`: evict the largest-duration course (heappop); this swap maintains the maximum count while staying feasible. |
| Why does activity-selection sort by END time rather than START time? | Sorting by end time and keeping the earliest-ending interval leaves the maximum remaining time for future intervals. An interval with a later end time — even if it starts earlier — "blocks" more future intervals. |
| What is the role of a Kafka offset? | The offset is the sequential position of a message within a partition. Consumers commit their offset after processing. On restart, they resume from the last committed offset — enabling at-least-once delivery without re-reading all historical messages. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/25 min, no hints)
- [ ] Rewrote merge intervals and Course Schedule III heap swap from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain Kafka partitions and delivery guarantees cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
