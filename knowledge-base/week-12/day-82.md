# Day 82 — Greedy: Jump Game, Jump Game II, and Min Taps
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Jump Game patterns are interval-coverage greedy problems. Jump Game I asks: can you reach the end? Track the maximum index reachable so far. Jump Game II asks: minimum jumps? Greedily extend the coverage window, incrementing jumps only when you exhaust the current window. Minimum Taps to Open to Water a Garden is structurally identical to Jump Game II after converting tap intervals to coverage ranges.

---

## DSA (2 hours)
### Pattern: Greedy Coverage Window Expansion

**Jump Game (LC 55):**
Track `max_reach`: the furthest index reachable so far. For each index `i` from 0 to n-1: if `i > max_reach` → stuck, return False. Otherwise update `max_reach = max(max_reach, i + nums[i])`. Return True if loop completes.

**Jump Game II (LC 45):**
Minimum jumps to reach the last index. Greedy: maintain `current_end` (end of current jump window) and `farthest` (furthest reachable from anywhere in the current window). When `i == current_end`: we must jump — increment jumps, set `current_end = farthest`. Stop when `current_end >= n - 1`.

**Minimum Number of Taps (LC 1326):**
Each tap at position `i` with range `[i - ranges[i], i + ranges[i]]` covers an interval. Convert to the Jump Game II problem: for each tap, compute `left = max(0, i - ranges[i])` and `right = min(n, i + ranges[i])`. Create `jumps` array where `jumps[left] = max(jumps[left], right)`. This transforms the problem: from each index, the furthest reachable = `jumps[index]`. Apply Jump Game II logic on `jumps`.

**Trigger condition:**
- "can you reach the last index?" → greedy max_reach scan; return False if `i > max_reach`
- "minimum jumps to reach last index (each jump up to nums[i] steps)" → greedy window expansion; count window transitions
- "minimum taps to water entire garden (each tap covers a range)" → convert to interval coverage → Jump Game II on coverage array

**Time complexity:** LC 55: O(n) | LC 45: O(n) | LC 1326: O(n)
**Space complexity:** O(1) / O(1) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Jump Game | 55 | Medium | Greedy max_reach | If `i > max_reach` at any step → impossible; update `max_reach = max(max_reach, i + nums[i])` |
| 2 | Jump Game II | 45 | Medium | Greedy window expansion | Jump when window exhausted (`i == current_end`); track `farthest` within window |
| 3 | Min Taps to Water Garden | 1326 | Hard | Convert to Jump Game II | Map tap ranges to interval coverage array; apply greedy window scan |

---

### Code Skeleton
```python
# Jump Game (LC 55)
def canJump(nums):
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
    return True

# Jump Game II (LC 45)
def jump(nums):
    jumps = 0
    current_end = 0
    farthest = 0
    for i in range(len(nums) - 1):   # stop at n-2; last index needn't trigger jump
        farthest = max(farthest, i + nums[i])
        if i == current_end:         # exhausted current window → must jump
            jumps += 1
            current_end = farthest
            if current_end >= len(nums) - 1:
                break
    return jumps

# Minimum Number of Taps (LC 1326)
def minTaps(n, ranges):
    # Build coverage array: max_right[i] = furthest right endpoint reachable from position i
    max_right = [0] * (n + 1)
    for i, r in enumerate(ranges):
        left = max(0, i - r)
        right = min(n, i + r)
        max_right[left] = max(max_right[left], right)

    # Jump Game II on max_right
    taps = 0
    current_end = 0
    farthest = 0
    for i in range(n):
        farthest = max(farthest, max_right[i])
        if i == current_end:
            if farthest == current_end:   # no progress → impossible
                return -1
            taps += 1
            current_end = farthest
            if current_end >= n:
                break
    return taps
```

---

### Edge Cases to Trace Before Coding
- LC 55: `nums = [0]` → n=1, already at end → True; `nums = [0, 1]` → stuck at 0, can't reach index 1 → False; all zeros except start → only reach if n=1
- LC 45: `nums = [1]` → already at end, 0 jumps; `nums = [2, 3, 1, 1, 4]` → 2 jumps; array of length 1 → 0 jumps (stop at n-2 in loop)
- LC 1326: `ranges = [0, 0, 0, 0]` → some positions unreachable → -1; `ranges = [2, 0, 0, 0]` → tap 0 covers [0,2], still can't reach 3 → -1; `ranges = [3, 0, 0, 0, 0, 0, 0]` with n=6 → single tap covers all

---

### Interview Pattern Drill

| Variant | Track | Update | Jump condition |
|---------|-------|--------|----------------|
| Jump Game I (reachable?) | `max_reach` | `max(max_reach, i + nums[i])` | `i > max_reach` → False |
| Jump Game II (min jumps) | `current_end`, `farthest` | `farthest = max(farthest, i + nums[i])` | `i == current_end` → jump++ |
| Min Taps (interval cover) | same as JG II | precompute `max_right[left]` | `farthest == current_end` → -1 |

**Why greedy window expansion gives minimum jumps:**
At each window, we greedily choose the jump that extends our reach furthest. We never overshoot — we take exactly one jump per window boundary crossing. Any alternative that jumps earlier uses the same or more jumps. Any that defers jumps uses the same or more. This is the interval scheduling greedy: earliest-deadline first on windows.

**Min Taps reduction to Jump Game II:**
A tap at position `i` covering `[i-r, i+r]` means: from any starting position in `[0, i-r]`, we can extend our coverage to at least `i+r`. The key insight: `max_right[left]` captures the best right endpoint achievable from `left`. Once built, it's identical to `nums` in Jump Game II where `nums[i] = max_right[i] - i`.

---

## System Design (1 hour)
### Topic: Kafka Internals — Partitions, Replication, and Consumer Groups

**Partition internals:**
Each partition is a write-ahead log — an append-only sequence of records on disk. Records are immutable once written. The partition has a single leader broker and 0 or more follower replicas.

```
Partition 0 on Broker 1 (leader):
  Offset: 0   1   2   3   4   5   ...
  Data:  [m0][m1][m2][m3][m4][m5]...

Partition 0 on Broker 2 (follower — replica):
  Lag behind leader; catches up via replication
```

**Replication:**
- Replication factor `RF=3`: 1 leader + 2 followers per partition
- Leader handles all reads and writes
- Followers replicate asynchronously from the leader's log
- **ISR (In-Sync Replicas):** followers that are caught up within `replica.lag.time.max.ms`. Only ISR replicas are eligible to become leader on failover
- `acks=all` (strongest): producer waits for ACK from all ISR replicas before considering the write committed

**Consumer groups deep dive:**
```
Topic "payments" (3 partitions)
Consumer Group "payment-processor":
  Consumer A → Partition 0
  Consumer B → Partition 1
  Consumer C → Partition 2

Consumer Group "audit-log":
  Consumer D → Partition 0, 1, 2   (single consumer reads all)
```

Rules:
- At most one consumer per partition within a group (fan-out to groups, not within)
- If consumers > partitions: some consumers idle
- If consumers < partitions: one consumer handles multiple partitions
- **Rebalance:** triggered when a consumer joins or leaves the group; partitions reassigned

**Offset management:**
- Consumers commit offsets to Kafka's internal `__consumer_offsets` topic
- `auto.commit.enable=true`: offsets committed automatically every `auto.commit.interval.ms`
- Manual commit (`commitSync` / `commitAsync`): more control; commit after processing ensures at-least-once
- Commit before processing: at-most-once (message lost on crash before processing)

**Log retention and compaction:**
```
Retention modes:
1. Time-based: delete records older than retention.ms (default 7 days)
2. Size-based: delete oldest records when partition exceeds retention.bytes
3. Log compaction: keep only the latest value per key → key-value store semantics
```

**Producer acknowledgement modes:**
| `acks` | Behaviour | Durability | Throughput |
|--------|-----------|-----------|-----------|
| `0` | Fire and forget | Lowest (data loss possible) | Highest |
| `1` | Leader only ACK | Medium (leader crash = loss) | Medium |
| `all` / `-1` | All ISR ACK | Highest (no loss if ISR ≥ 2) | Lowest |

**Exactly-once semantics (EOS):**
Producer idempotence: each producer gets a PID; broker deduplicates retries using `(PID, sequence_number)`. Transactions: atomic multi-partition writes + offset commits. Use `transactional.id` to enable. Higher latency (~2×) but no duplicates.

**Interview talking point:** "Kafka achieves high throughput through sequential disk writes (the partition log), zero-copy data transfer (sendfile syscall bypasses kernel-user copy), and batching. A single Kafka broker can handle 1 GB/s of writes. For our payment processing system, we use RF=3, acks=all, and manual offset commit after processing to achieve at-least-once delivery. For idempotent payment processing, each payment message carries a UUID that the payments service deduplicates in its database."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 55 Jump Game (target: 10 min)
- **Medium 2:** LC 45 Jump Game II (target: 15 min)
- **Hard:** LC 1326 Minimum Number of Taps (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to balance reliability (ensuring every event is processed) against performance (not re-processing duplicates) in a distributed system.
- Leadership principle: Bias for Action

---

## Flashcards

| Q | A |
|---|---|
| How does Jump Game I greedily determine reachability? | Maintain `max_reach`. For each index `i`: if `i > max_reach` → return False (stuck). Update `max_reach = max(max_reach, i + nums[i])`. Return True if loop completes without getting stuck. |
| How does Jump Game II count minimum jumps? | Maintain `current_end` (current window boundary), `farthest` (best reach in window), `jumps`. For each i < n-1: update `farthest`. When `i == current_end`: jump++, `current_end = farthest`. O(n) single pass. |
| How do you reduce Min Taps to Jump Game II? | Build `max_right[left] = max coverage right endpoint from position left`. For tap `i` with range `r`: `max_right[max(0, i-r)] = max(..., min(n, i+r))`. Then run Jump Game II on `max_right`. |
| What is the Kafka ISR (In-Sync Replicas) and why does it matter? | ISR = set of follower replicas caught up within `replica.lag.time.max.ms`. `acks=all` waits for all ISR to ACK. If a leader crashes, only ISR replicas are eligible to take over — ensuring no committed data is lost. |
| What is the difference between committing offsets before vs after processing in Kafka? | Commit before processing: at-most-once delivery (message lost if consumer crashes before processing). Commit after processing: at-least-once delivery (message re-processed on crash before commit). Exactly-once requires transactional APIs. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 10/15/25 min, no hints)
- [ ] Rewrote Jump Game II window expansion and Min Taps reduction from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain Kafka ISR, replication, and offset management cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
