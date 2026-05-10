# Day 84 — Backtracking: Subsets, Subsets II, and Partition K Equal Subsets
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Subsets is the foundational backtracking template: at each index, choose to include or exclude. Subsets II adds duplicate handling — sort first, then skip duplicate elements at the same depth level. Partition K Equal Subsets is a harder backtracking problem: assign each number to one of K buckets with equal-sum target, using bitmask visited state to prune efficiently.

---

## DSA (2 hours)
### Pattern: Include/Exclude Backtracking + Duplicate Pruning + Bitmask Backtracking

**Subsets (LC 78):**
Every element is either included or excluded → 2^n subsets. Backtracking: for each index `i` from `start` to end, add `nums[i]` to current path, recurse with `start = i + 1`, then remove `nums[i]` (backtrack). Collect the path at every recursion level (not just leaves).

**Subsets II (LC 90):**
Identical to Subsets but with duplicates. Sort `nums`. In the loop: skip `nums[i]` if `i > start AND nums[i] == nums[i-1]` — this skips duplicate elements at the same recursion level (same depth), while still allowing the same value if it appears at deeper recursion (different depth = different elements, distinct positions).

**Partition to K Equal Sum Subsets (LC 698):**
Partition `nums` into K subsets each with sum = `total / K`. Backtrack: assign each number to one of K "buckets". Optimizations: sort `nums` descending (fail fast on large numbers); use a bitmask to represent which numbers are used; prune if `bucket_sum + nums[i] > target`; if a bucket is empty, it's equivalent to the previous empty bucket (avoid symmetric states).

**Trigger condition:**
- "all subsets / power set of an array" → backtracking with include/exclude; collect path at every node
- "all subsets with duplicate elements" → sort first; skip `nums[i] == nums[i-1]` when `i > start`
- "can you partition array into K equal-sum groups?" → backtracking with bucket assignment; bitmask for visited state; sort desc for pruning

**Time complexity:** LC 78: O(2^n × n) | LC 90: O(2^n × n) | LC 698: O(k × 2^n)
**Space complexity:** O(n) recursion depth for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Subsets | 78 | Medium | Backtracking include/exclude | Collect path at every node; loop from start to end; recurse with start = i+1 |
| 2 | Subsets II | 90 | Medium | Sort + skip duplicates at same depth | Skip if `i > start AND nums[i] == nums[i-1]`; sort ensures duplicates are adjacent |
| 3 | Partition K Equal Subsets | 698 | Hard | Bucket backtracking + bitmask | Sort desc; try assigning each unused number to current bucket; prune identical empty buckets |

---

### Code Skeleton
```python
# Subsets (LC 78)
def subsets(nums):
    result = []
    def backtrack(start, path):
        result.append(path[:])   # collect at every node
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return result

# Subsets II (LC 90)
def subsetsWithDup(nums):
    nums.sort()   # sort to bring duplicates adjacent
    result = []
    def backtrack(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:   # skip duplicate at same level
                continue
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return result

# Partition to K Equal Sum Subsets (LC 698)
def canPartitionKSubsets(nums, k):
    total = sum(nums)
    if total % k != 0:
        return False
    target = total // k
    nums.sort(reverse=True)   # largest first: fail fast
    if nums[0] > target:
        return False

    buckets = [0] * k

    def backtrack(index):
        if index == len(nums):
            return all(b == target for b in buckets)
        seen = set()
        for i in range(k):
            if buckets[i] + nums[index] <= target and buckets[i] not in seen:
                seen.add(buckets[i])   # skip identical empty/equal buckets
                buckets[i] += nums[index]
                if backtrack(index + 1):
                    return True
                buckets[i] -= nums[index]
        return False

    return backtrack(0)
```

---

### Edge Cases to Trace Before Coding
- LC 78: empty array → return `[[]]` (one subset: the empty set); single element → `[[], [nums[0]]]`; all same elements → 2^n subsets but no deduplication needed (Subsets allows duplicates)
- LC 90: all duplicates e.g. `[1,1,1]` → subsets: `[], [1], [1,1], [1,1,1]`; single unique element → same as Subsets
- LC 698: k=1 → always True (all in one subset); k=len(nums) → True iff all equal; nums has element > target → False immediately; `total % k != 0` → False

---

### Interview Pattern Drill

**Backtracking template (generalized):**
```python
def backtrack(state):
    if is_complete(state):
        record(state)
        return
    for choice in get_choices(state):
        if is_valid(choice, state):
            apply(choice, state)
            backtrack(state)
            undo(choice, state)
```

**Subsets vs Combinations vs Permutations:**
| Problem | What changes | Loop starts at | Collect at |
|---------|-------------|----------------|-----------|
| Subsets | include/exclude each element | `start` (no reuse) | Every node |
| Combinations | choose exactly k elements | `start` (no reuse) | Leaf when `len == k` |
| Permutations | arrange all elements | 0 (use visited set) | Leaf when `len == n` |

**Duplicate skip rule — why `i > start` and not `i > 0`:**
`i > 0` would skip duplicates across ALL previous recursions, cutting off valid subsets at deeper levels. `i > start` skips duplicates only at the SAME depth/level of the tree — we allow the same value at different depths (representing different element positions), but not the same value twice at the same branching level.

**Partition K: symmetric bucket pruning:**
If bucket A and bucket B are both empty (or have identical sums), assigning `nums[i]` to A vs B produces identical subtrees. The `seen` set skips this symmetry, cutting the search space from k! to much smaller.

---

## System Design (1 hour)
### Topic: RabbitMQ vs Kafka — When to Use Which

**RabbitMQ overview:**
Traditional message broker. Implements AMQP protocol. Push-based: broker delivers messages to consumers. Messages deleted after ACK. Best for: task queues, RPC, routing-heavy workloads.

**Kafka overview:**
Distributed streaming log. Pull-based: consumers pull from partitions at their own pace. Messages retained on disk for configurable period (days/weeks). Best for: event streaming, audit logs, multi-consumer fan-out, replay.

**Head-to-head comparison:**

| Property | RabbitMQ | Kafka |
|----------|----------|-------|
| Model | Push (broker → consumer) | Pull (consumer → broker) |
| Retention | Deleted on ACK | Log retention (time or size) |
| Replay | Not supported | Full replay within retention window |
| Ordering | FIFO within queue | Strict within partition |
| Throughput | ~50K msgs/sec per node | 1 M msgs/sec per broker |
| Routing | Exchanges + bindings (flexible) | Topic + partition key |
| Consumer model | Competing consumers | Consumer groups (independent per group) |
| Protocol | AMQP | Custom TCP binary protocol |
| Use case | Task distribution, RPC, routing | Event streaming, analytics, audit log |

**When to choose RabbitMQ:**
- Complex routing logic (header-based, topic-based fanout with bindings)
- Traditional task queues (email sending, image processing)
- Need for message TTL, priority queues, dead-letter exchanges
- Request/reply (RPC) patterns with short-lived messages

**When to choose Kafka:**
- High-throughput event streaming (millions of events/second)
- Multiple independent consumer groups each processing all events
- Need replay: re-processing historical events, bootstrapping new services
- Event sourcing: the Kafka log is the source of truth
- Analytics pipeline: Kafka → Flink/Spark → ClickHouse

**Hybrid architecture:**
```
User-facing microservices → Kafka (high throughput, fan-out)
                                │
                    ┌───────────┴──────────────┐
                    ▼                           ▼
           Analytics workers            Kafka → RabbitMQ bridge
           (ClickHouse pipeline)              │
                                    Task-specific workers
                                    (email, SMS — push model)
```

**AWS equivalents:**
- RabbitMQ ≈ Amazon SQS (standard or FIFO)
- Kafka ≈ Amazon Kinesis or Amazon MSK (Managed Streaming for Kafka)

**Decision framework:**
```
Do you need replay?
  YES → Kafka
  NO → Do you need multiple independent consumers (pub/sub)?
         YES → Kafka (consumer groups)
         NO → Do you need complex routing?
                YES → RabbitMQ
                NO → SQS (simpler, fully managed)
```

**Interview talking point:** "For our newsfeed system I'd choose Kafka over RabbitMQ for three reasons: (1) We need fan-out to multiple consumer groups — feed workers, notification workers, analytics — each independently reading all events. RabbitMQ's competing consumer model would require separate queues per consumer type. (2) We need replay: when we launch a new search indexer, it needs to process historical events from the last 7 days. Kafka's log retention enables this. (3) Throughput: at 1M posts/day, Kafka's 1M msg/sec capacity gives us headroom for spikes. RabbitMQ would require horizontal sharding at that scale."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 78 Subsets (target: 12 min)
- **Medium 2:** LC 90 Subsets II (target: 15 min)
- **Hard:** LC 698 Partition K Equal Subsets (target: 28 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you chose a specific technology or tool over an alternative — what was your decision framework and what trade-offs did you accept?
- Leadership principle: Are Right, A Lot

---

## Flashcards

| Q | A |
|---|---|
| How does the Subsets backtracking collect all power-set subsets? | At every recursion call (every node of the tree): append `path[:]` to result. Loop from `start` to end: add `nums[i]`, recurse with `start = i+1`, remove `nums[i]`. Collect 2^n subsets total. |
| How does Subsets II skip duplicates at the same recursion level? | Sort `nums` first. In the loop: `if i > start and nums[i] == nums[i-1]: continue`. The `i > start` check (not `i > 0`) ensures we only skip within the same depth, not across deeper recursion levels. |
| What are the two key pruning strategies for Partition K Equal Subsets? | (1) Sort descending: large numbers fail fast when they exceed bucket target. (2) Symmetric bucket pruning: use a `seen` set of bucket sums; skip assigning to a bucket if another bucket with the same current sum was already tried at this level. |
| When should you choose Kafka over RabbitMQ? | Choose Kafka when you need: (1) multiple independent consumer groups each receiving all messages (pub/sub fan-out), (2) message replay / log retention, (3) very high throughput (millions/sec). Choose RabbitMQ for complex routing, traditional task queues, or request/reply (RPC) patterns. |
| What is the backtracking template for all subset/combination/permutation problems? | `def backtrack(state): record if complete; for each valid choice: apply choice → recurse → undo choice`. The key variation: Subsets/Combinations loop from `start` (no reuse); Permutations loop from 0 with a `visited` set. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/15/28 min, no hints)
- [ ] Rewrote Subsets II duplicate skip and Partition K bucket backtracking from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain Kafka vs RabbitMQ decision framework cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
