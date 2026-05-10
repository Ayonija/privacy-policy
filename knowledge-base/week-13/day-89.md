# Day 89 — Greedy: Partition Labels, Remove K Digits, and Candy
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Partition Labels greedily extends the current partition to include the last occurrence of every character seen so far — a single-pass boundary extension. Remove K Digits uses a monotone stack to greedily eliminate large digits from the left, minimising the resulting number. Candy uses a two-pass greedy: left-to-right satisfies increasing neighbours; right-to-left satisfies decreasing neighbours; take the max of both passes.

---

## DSA (2 hours)
### Pattern: Last-Occurrence Boundary Extension + Monotone Stack Greedy + Two-Pass Left-Right Greedy

**Partition Labels (LC 763):**
Partition string into as many parts as possible so each letter appears in at most one part. Greedy: precompute the last index of each character. Scan left to right, extending `end = max(end, last[c])`. When `i == end`: partition boundary found, record `i - start + 1`, set `start = i + 1`.

**Remove K Digits (LC 402):**
Remove k digits from a number string to produce the smallest possible number. Monotone stack: maintain a stack of digits in increasing order. When current digit < stack top AND k > 0: pop (remove the larger digit), decrement k. After scan: if k > 0, pop from the end. Strip leading zeros. Edge case: result empty → return "0".

**Candy (LC 135):**
N children in a line with ratings. Each child gets ≥ 1 candy. Child with higher rating than neighbour gets more candy. Minimum total candies.

Two-pass greedy:
1. Left-to-right: if `rating[i] > rating[i-1]` → `left[i] = left[i-1] + 1`, else `left[i] = 1`
2. Right-to-left: if `rating[i] > rating[i+1]` → `right[i] = right[i+1] + 1`, else `right[i] = 1`
3. `candies[i] = max(left[i], right[i])`

**Trigger condition:**
- "partition into maximum parts where each element is in one part only" → last-occurrence boundary extension; single pass
- "remove k digits to get smallest number" → monotone increasing stack; pop when current < top AND k > 0
- "distribute minimum resources satisfying left-right ordering constraints" → two-pass greedy; left pass + right pass; take max

**Time complexity:** LC 763: O(n) | LC 402: O(n) | LC 135: O(n)
**Space complexity:** O(1) / O(n) stack / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Partition Labels | 763 | Medium | Last-occurrence boundary extension | `end = max(end, last[c])`; partition when `i == end` |
| 2 | Remove K Digits | 402 | Medium | Monotone increasing stack | Pop larger digits when current is smaller AND k > 0; strip leading zeros |
| 3 | Candy | 135 | Hard | Two-pass left-right greedy | Left pass: ascending; right pass: descending; take `max` per child |

---

### Code Skeleton
```python
# Partition Labels (LC 763)
def partitionLabels(s):
    last = {c: i for i, c in enumerate(s)}   # last index of each char
    result = []
    start = end = 0
    for i, c in enumerate(s):
        end = max(end, last[c])
        if i == end:
            result.append(end - start + 1)
            start = i + 1
    return result

# Remove K Digits (LC 402)
def removeKdigits(num, k):
    stack = []
    for digit in num:
        while k > 0 and stack and stack[-1] > digit:
            stack.pop()
            k -= 1
        stack.append(digit)
    # If k > 0 after scan, remove from the right (largest suffix digits)
    stack = stack[:-k] if k else stack
    # Strip leading zeros and handle empty case
    result = ''.join(stack).lstrip('0')
    return result or '0'

# Candy (LC 135)
def candy(ratings):
    n = len(ratings)
    left = [1] * n
    right = [1] * n
    for i in range(1, n):
        if ratings[i] > ratings[i - 1]:
            left[i] = left[i - 1] + 1
    for i in range(n - 2, -1, -1):
        if ratings[i] > ratings[i + 1]:
            right[i] = right[i + 1] + 1
    return sum(max(left[i], right[i]) for i in range(n))
```

---

### Edge Cases to Trace Before Coding
- LC 763: single character → one partition of size 1; all same character → one partition of size n; all distinct characters → n partitions of size 1
- LC 402: `k == 0` → return num as-is; `k >= len(num)` → return "0"; num is already ascending (e.g., "1234") → remove last k digits; "10200" with k=1 → remove 1 → "0200" → strip → "200"
- LC 135: all equal ratings → all get 1 candy; strictly increasing → [1,2,3,...,n]; strictly decreasing → [n,...,2,1]; single child → 1 candy

---

### Interview Pattern Drill

**Partition Labels — why it produces maximum parts:**
By extending the boundary to include every character's last occurrence, we guarantee no character straddles two parts. We seal the partition as early as possible (when current index hits the current boundary) — this is the greedy optimum.

**Remove K Digits — monotone stack intuition:**
For a number to be small, we want small digits in the most significant positions. When we see a digit smaller than the previous one, the previous digit makes the number larger than necessary — remove it (if k > 0). The stack maintains an increasing sequence: each new digit is at least as large as the previous (or we've used k removals).

**Candy — why both passes are needed:**
Left pass handles cases where child has higher rating than LEFT neighbour. Right pass handles higher rating than RIGHT neighbour. A child in the middle of a valley needs to satisfy BOTH neighbours — taking `max(left, right)` ensures both constraints are met.

Example: `ratings = [1, 3, 2, 2, 1]`
- Left:  `[1, 2, 1, 1, 1]`
- Right: `[1, 2, 1, 2, 1]`  ← child at index 3 has higher rating than child at index 4
- Max:   `[1, 2, 1, 2, 1]` = 7 total

**Greedy correctness pattern comparison:**

| Problem | Greedy choice | Why it's safe |
|---------|--------------|--------------|
| Partition Labels | Extend boundary to last occurrence | Any earlier cut would split a character across parts |
| Remove K Digits | Remove the first digit larger than its right neighbour | Earlier large digit has higher significance → removing it reduces the number |
| Candy | Two independent greedy passes | Each pass satisfies one side's constraint; max combines both |

---

## System Design (1 hour)
### Topic: Push vs Pull, Dead Letter Queues, and Backpressure in Production

**Push vs Pull delivery models:**

**Push (RabbitMQ, HTTP webhooks, SNS):**
Broker proactively delivers messages to consumers. Consumer registers and receives callbacks.
- Pros: low latency (no polling delay), simple consumer code
- Cons: consumer must be always available; broker responsible for managing delivery rate; consumer can be overwhelmed (backpressure harder)
- Consumer controls: prefetch count (RabbitMQ `basic.qos`) limits unACKed messages in flight

**Pull (Kafka, SQS):**
Consumer polls the broker for messages at its own pace.
- Pros: consumer controls rate; natural backpressure (stop polling when busy); consumer can pause/resume independently
- Cons: polling latency (configurable with `fetch.min.bytes` and `fetch.max.wait.ms`); broker doesn't know consumer state
- Kafka `fetch.max.wait.ms=500`: broker waits up to 500ms for `fetch.min.bytes` of data before responding → reduces empty polls

**Dead Letter Queue (DLQ) — production configuration:**

```
Main Queue → Consumer
               ├─ Success → ACK; message deleted
               └─ Failure → NACK + retry count++
                     └─ Retry count > max_retries → DLQ

DLQ → Alert (PagerDuty / OpsGenie)
     → Monitoring dashboard (message type, error, count)
     → Manual replay tool (re-enqueue from DLQ after fix)
```

**DLQ message attributes to preserve:**
- Original message payload (unchanged)
- Error reason and stack trace (appended as metadata)
- Retry count, first-failure timestamp, last-failure timestamp
- Original queue name (so replay knows where to send it back)

**Backpressure patterns in production:**

1. **Token bucket rate limiting:** producer acquires a token per message; tokens replenish at rate R; producer blocks when bucket empty → natural backpressure upstream
2. **Consumer prefetch limit:** RabbitMQ `basic.qos(prefetch_count=10)` → broker holds at most 10 unACKed messages per consumer; prevents consumer memory overflow
3. **Adaptive batching:** Kafka `fetch.min.bytes`: consumer requests at least N bytes; broker waits rather than sending tiny batches → amortises round-trip cost
4. **Queue depth circuit breaker:** API layer checks queue depth before accepting requests; returns 429 when depth > threshold; sheds load upstream

**Message TTL (Time-to-Live):**
Messages older than TTL are expired. Used when stale messages are worse than no message:
- Feed fanout: if a user's feed update is 1 hour old before being processed, it's no longer relevant → TTL = 10 minutes
- Email notifications: if undeliverable for 24 hours → expire; notification superseded
- Payment retries: set TTL = refund window (do not retry charges 30 days later)

**Interview talking point:** "For our feed fanout workers, we use Kafka's pull model with `max.poll.records=500` and a 2-minute processing SLA per batch. The pull model gives us natural backpressure: if our Redis cluster is slow, workers simply stop polling. Kafka accumulates messages during the slowdown; workers catch up when Redis recovers. We use a DLQ for messages that fail after 3 retries — typically malformed post_ids. Our on-call alert fires when DLQ depth > 100; the runbook instructs engineers to inspect the DLQ, fix the upstream bug, and replay via a CLI tool that re-enqueues DLQ messages to the main topic."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 763 Partition Labels (target: 12 min)
- **Medium 2:** LC 402 Remove K Digits (target: 15 min)
- **Hard:** LC 135 Candy (target: 22 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you set up alerting and on-call procedures for a critical system — what signals did you monitor, and how did you reduce alert fatigue?
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does Partition Labels find partition boundaries in a single pass? | Precompute `last[c]` = last index of char c. Scan left to right: `end = max(end, last[c])`. When `i == end`: record partition size `end - start + 1`, set `start = i + 1`. This extends each partition to include all characters' last occurrences. |
| How does Remove K Digits use a monotone stack to minimise the number? | Maintain a monotone increasing stack. For each digit: while `k > 0` and stack is non-empty and `stack[-1] > digit` → pop (removes a larger significant digit) and `k--`. After scan: if `k > 0`, remove last k digits. Strip leading zeros; return "0" if empty. |
| Why does Candy need two passes instead of one? | Each child must satisfy constraints relative to BOTH neighbours. Left pass satisfies `rating[i] > rating[i-1]` constraints; right pass satisfies `rating[i] > rating[i+1]` constraints. A child in a valley may need more candies from one side — `max(left, right)` combines both requirements. |
| What is the difference between push and pull in messaging systems? | Push: broker proactively delivers to consumers (RabbitMQ, webhooks); low latency; consumer can be overwhelmed. Pull: consumer polls at own pace (Kafka, SQS); natural backpressure; consumer controls rate; small polling latency. |
| What information should a DLQ message preserve, and why? | Original payload (for replay), error reason + stack trace (for diagnosis), retry count, first/last failure timestamp, original queue name (so replay tool knows where to re-enqueue). Without these, engineers can't diagnose root cause or safely replay. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/15/22 min, no hints)
- [ ] Rewrote Partition Labels boundary extension and Remove K Digits monotone stack from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain DLQ design and push vs pull trade-offs cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
