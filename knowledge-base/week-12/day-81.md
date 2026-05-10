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
- STAR prompt: Describe a time you designed a workflow that decoupled two systems to improve reliability — what were the trade-offs of introducing the intermediary?
- Leadership principle: Invent and Simplify

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
