# Day 90 — Greedy Synthesis: Monotone Digits, Min Deletions, Max Events
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Slot 9 close-out synthesis. Monotone Increasing Digits greedily finds the largest monotone-increasing number ≤ N. Minimum Deletions to Make Character Frequencies Unique uses a greedy frequency reduction strategy with a set. Maximum Number of Events That Can Be Attended uses a min-heap sorted by end day — greedily attend the earliest-ending event each day. System design: full Message Queue / Pub/Sub decision framework synthesis.

---

## DSA (2 hours)
### Pattern: Right-to-Left Digit Greedy + Frequency Dedup Greedy + Min-Heap Event Selection

**Monotone Increasing Digits (LC 738):**
Find the largest number ≤ N with monotone non-decreasing digits. Process digits right to left: when `digits[i] < digits[i-1]`, decrement `digits[i-1]` and mark all digits from i onward as '9'. Left-to-right marking with a `mark` pointer.

**Minimum Deletions to Make Character Frequencies Unique (LC 1647):**
Each character's frequency must be unique. Greedy: sort frequencies descending. Maintain a `seen` set of used frequencies. For each frequency: while `freq > 0 AND freq in seen`: decrement freq, increment deletions. Add final freq to seen (if > 0).

**Maximum Number of Events That Can Be Attended (LC 1353):**
Each event has a `[start, end]` day range; you can attend at most one event per day. Greedy: sort events by start day. For each day d from 1 to max_day: add all events starting on day d to a min-heap (keyed by end day). Remove expired events (end < d). Attend the earliest-ending available event (heappop); increment count.

**Trigger condition:**
- "largest number ≤ N with non-decreasing digits" → right-to-left scan; decrement at violation; fill 9s from violation point
- "minimum deletions so all character frequencies are distinct" → sort desc; greedy reduce each freq; count reductions
- "maximum events attendable (one per day, each event has a window)" → min-heap by end day; process day by day; always attend earliest-ending

**Time complexity:** LC 738: O(d) where d = digits | LC 1647: O(n log n) | LC 1353: O(n log n)
**Space complexity:** O(d) / O(26) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Monotone Increasing Digits | 738 | Medium | Right-to-left digit greedy | When `digits[i] < digits[i-1]`: decrement `digits[i-1]`; fill 9s from i onward |
| 2 | Min Deletions Unique Freq | 1647 | Medium | Sort frequencies; greedy reduce with seen set | Sort desc; reduce each freq until unique; count reductions |
| 3 | Maximum Events Attended | 1353 | Hard | Min-heap by end day; greedy attend earliest-ending | Sort by start; add to heap on start day; remove expired; attend min-end each day |

---

### Code Skeleton
```python
import heapq
from collections import Counter

# Monotone Increasing Digits (LC 738)
def monotoneIncreasingDigits(n):
    digits = list(str(n))
    mark = len(digits)   # position from which we fill '9'
    for i in range(len(digits) - 1, 0, -1):
        if digits[i] < digits[i - 1]:
            digits[i - 1] = str(int(digits[i - 1]) - 1)
            mark = i
    for i in range(mark, len(digits)):
        digits[i] = '9'
    return int(''.join(digits))

# Minimum Deletions to Make Character Frequencies Unique (LC 1647)
def minDeletions(s):
    freqs = sorted(Counter(s).values(), reverse=True)
    seen = set()
    deletions = 0
    for freq in freqs:
        while freq > 0 and freq in seen:
            freq -= 1
            deletions += 1
        if freq > 0:
            seen.add(freq)
    return deletions

# Maximum Number of Events That Can Be Attended (LC 1353)
def maxEvents(events):
    events.sort()   # sort by start day
    heap = []       # min-heap of end days
    day = 0
    attended = 0
    i = 0
    n = len(events)
    max_day = max(e[1] for e in events)

    for day in range(1, max_day + 1):
        # Add all events starting today
        while i < n and events[i][0] == day:
            heapq.heappush(heap, events[i][1])
            i += 1
        # Remove expired events
        while heap and heap[0] < day:
            heapq.heappop(heap)
        # Attend the event ending soonest
        if heap:
            heapq.heappop(heap)
            attended += 1

    return attended
```

---

### Edge Cases to Trace Before Coding
- LC 738: `n = 10` → 9; `n = 332` → 299; `n = 1` → 1; already monotone → return n unchanged
- LC 1647: all characters same → 0 deletions (one unique freq); all unique chars with freq 1 → already unique → 0 deletions; s = "aab" → freqs [2,1] → unique → 0
- LC 1353: all events on same day → attend 1; events that don't overlap → attend all; `events = [[1,2],[2,3],[3,4]]` → attend all 3 (different days used)

---

### Slot 9 Complete Algorithm Reference

| Pattern | Sort key | Data structure | Core operation | Use when |
|---------|---------|---------------|---------------|---------|
| Interval merge | Start time | List | Merge when `curr.start <= last.end` | Consolidate overlapping ranges |
| Non-overlapping | End time | None | Count `curr.start < last_end` | Minimum interval removals |
| Activity selection | End time | None | Keep earliest-ending; count kept | Maximum non-overlapping events |
| Course Schedule III | Deadline | Max-heap | Evict longest when over budget | Max tasks within deadlines |
| Jump Game I | — | None | Track max_reach; fail if `i > max_reach` | Reachability |
| Jump Game II | — | None | Window expansion; count boundary crossings | Minimum jumps |
| Gas Station | — | None | Running tank; reset start on negative | Circular route feasibility |
| Max Profit Jobs | End time | Binary search + DP | `dp[i] = max(dp[i-1], p + dp[j])` | Weighted interval scheduling |
| Subsets | — | Backtracking | Collect at every node; `start = i+1` | All power-set subsets |
| Permutations | — | Backtracking + visited | Loop 0..n; collect at depth n | All orderings |
| N-Queens | — | Backtracking + sets | col, diag1, diag2 sets; row by row | Constraint satisfaction placement |
| Event attendance | Start day | Min-heap (end day) | Attend earliest-ending each day | Maximize one-per-day event attendance |
| Remove K Digits | — | Monotone stack | Pop when stack[-1] > curr AND k > 0 | Smallest number after k removals |
| Candy | — | Two arrays | Left pass + right pass; max each | Minimum allocation with ordering |

---

### Edge Cases Pattern Synthesis

**Backtracking problems — universal checklist:**
1. What do I collect and when? (every node / leaf / at depth k)
2. What is my loop range? (0 to n for permutations; start to n for combinations)
3. Do I need a visited array, or a start index?
4. How do I handle duplicates? (sort + skip rule)
5. What is the base case? (target==0, len==k, index==n)

**Greedy problems — universal checklist:**
1. What is the greedy choice and why is it safe?
2. Does the problem have optimal substructure?
3. What is my sort order, if any?
4. What am I tracking per step? (max_reach, last_end, running sum)
5. What is my termination condition?

---

## System Design (1 hour)
### Topic: Full Message Queue / Pub/Sub Decision Framework

**Master decision tree:**
```
Need async messaging between services?
  │
  ├─ Do you need multiple independent consumers (fan-out)?
  │    YES → Kafka (consumer groups = independent pub/sub)
  │    NO → Single queue
  │           ├─ Complex routing (topic/header exchange)? → RabbitMQ
  │           └─ Simple FIFO, fully managed? → SQS
  │
  ├─ Do you need message replay?
  │    YES → Kafka (log retention: days to weeks)
  │    NO → RabbitMQ / SQS (message deleted on ACK)
  │
  ├─ What delivery guarantee?
  │    At-most-once → any system; producer acks=0, consumer pre-commit
  │    At-least-once → Kafka (acks=1 or all, post-process commit); SQS default
  │    Exactly-once → Kafka transactions + idempotent consumers (or DB dedup)
  │
  ├─ How do you handle failures?
  │    Retries → exponential backoff + jitter in consumer
  │    Poison pills → DLQ after N retries; alert on DLQ depth
  │    Consumer crash → re-read from last committed offset (Kafka) or visibility timeout (SQS)
  │
  └─ How do you scale?
       More consumers → add pods (bounded by partition count in Kafka)
       More partitions → increase at topic creation (breaking change if added later)
       Cross-region → MirrorMaker 2 (Kafka); SNS cross-region (AWS)
```

**The 6 questions to answer cold for any message queue design:**

1. **What is the delivery guarantee?** At-most/at-least/exactly-once. Most: at-least-once + consumer idempotency.
2. **Who are the consumers?** Single consumer = queue. Multiple independent consumers = Kafka consumer groups.
3. **Do consumers need replay?** Historical reprocessing → Kafka log retention. No replay needed → SQS/RabbitMQ.
4. **How do you handle failures?** Retry policy + DLQ + alerting on DLQ depth.
5. **How do you handle backpressure?** Pull model (Kafka) = natural. Push model = prefetch limit + rate limiting.
6. **How do you scale?** Partition count (Kafka) or queue parallelism (SQS) bounds consumer concurrency.

**System design interview synthesis:**

| Scenario | Choice | Reason |
|----------|--------|--------|
| Newsfeed fanout | Kafka | Multiple consumer groups (feed, notifications, analytics, search) |
| Payment processing | Kafka | Replay for reconciliation; audit log; high throughput |
| Email delivery | SQS + DLQ | Simple FIFO; fully managed; DLQ for bounce handling |
| Real-time event analytics | Kafka → Flink → ClickHouse | Streaming aggregation; replay for backfill |
| Microservice RPC | RabbitMQ | Request/reply pattern; routing; short-lived messages |
| IoT sensor ingestion | Kafka | Very high throughput; time-series replay; multiple processors |

**Kafka capacity and sizing cheat sheet:**
- Single broker: ~1 GB/s write throughput (with RF=1)
- With RF=3: effective write capacity ~333 MB/s per broker
- Rule of thumb: 10 MB/s per partition (conservative)
- Partition count: `max_expected_throughput_MB_s / 10`; round up to next power of 2
- Retention: `throughput × retention_days × 86400 × replication_factor` = total storage
- Consumer lag alert threshold: 3–5× normal processing time worth of messages

**Trade-off summary:**

| Choice | Trade-off |
|--------|-----------|
| Kafka vs SQS | Kafka: more powerful (replay, fan-out, high throughput) but operationally complex. SQS: fully managed, simpler, no replay, lower max throughput. |
| At-least-once vs exactly-once | At-least-once: simpler, lower latency. Exactly-once: complex, ~2× latency overhead from transactions. Use idempotent consumers instead. |
| Push vs pull | Push: lower latency. Pull: natural backpressure, consumer controls rate. |
| High partition count | More parallelism. Drawback: more file handles, longer rebalance time, Zookeeper/KRaft load. |
| Log compaction | Key-value semantics (latest value per key). Drawback: no ordered message replay of all events; only latest state per key. |

**Interview talking point:** "The message queue choice ultimately comes down to three questions: do you need fan-out (multiple consumers), do you need replay, and what's your throughput requirement? For our newsfeed system, Kafka wins on all three: fan-out to 4 consumer groups, replay for bootstrapping the search indexer on new launches, and 1B events/day at ~11 MB/s. For our transactional email delivery — where we need guaranteed delivery but no replay, low volume, and simple FIFO — SQS with a DLQ is the right choice. Always use the simplest tool that meets your requirements."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — Slot 9 synthesis
- **Medium 1:** LC 738 Monotone Increasing Digits (target: 12 min)
- **Medium 2:** LC 1647 Min Deletions Unique Freq (target: 12 min)
- **Hard:** LC 1353 Maximum Events (target: 28 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Walk through a system you built that used asynchronous messaging — describe the key architectural decisions, delivery guarantees chosen, and trade-offs accepted.
- Leadership principle: Invent and Simplify

---

## Flashcards

| Q | A |
|---|---|
| How does Monotone Increasing Digits work right-to-left? | Scan from right to left: when `digits[i] < digits[i-1]`, decrement `digits[i-1]` and mark position i as the start of '9' fill. After scan, fill all positions from `mark` to end with '9'. This gives the largest number ≤ N with non-decreasing digits. |
| How does Min Deletions Make Frequencies Unique work? | Sort frequencies descending. Maintain a `seen` set. For each freq: while `freq > 0 AND freq in seen` → `freq--; deletions++`. Add final freq to seen. This greedily assigns each character the highest available unique frequency. |
| How does Maximum Events use a min-heap to maximise attendance? | Sort events by start day. For each day d: add all events starting today to a min-heap (keyed by end day). Remove expired events (end < d). Attend (pop) the event ending soonest. Total pops = answer. |
| State the 6 questions for a message queue system design. | (1) Delivery guarantee? (2) Single or multiple independent consumers? (3) Need replay? (4) Failure handling: DLQ + retry? (5) Backpressure: push or pull? (6) Scale: partition count or queue parallelism? |
| When should you use Kafka vs SQS vs RabbitMQ? | Kafka: fan-out (multiple consumer groups), replay, high throughput. SQS: fully managed, simple FIFO, no replay, lower throughput. RabbitMQ: complex routing (exchanges/bindings), request/reply (RPC), traditional task queues. Always pick the simplest tool that meets your requirements. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/12/28 min, no hints)
- [ ] Rewrote Monotone Increasing Digits right-to-left pass and Max Events heap from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Recited the Slot 9 algorithm reference table from memory
- [ ] Completed system design — recited the 6-question MQ decision framework cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
