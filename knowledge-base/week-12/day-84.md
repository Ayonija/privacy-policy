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

### STAR Interview Framework

> **How to use the STAR method when explaining Subsets / Subsets II / Partition K Equal Subsets in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three backtracking problems that form a progression in difficulty. Subsets is the foundational include/exclude template. Subsets II adds duplicate handling — a subtle invariant about when to skip vs when to continue. Partition K Equal Subsets is a combinatorial search problem where pruning is critical: naively trying all 2^n × k! bucket assignments is intractable for n ≥ 20."

**Task:** "My goal was to master the backtracking template and its critical variations: collecting at every node (Subsets) vs at leaves only (Combinations), the `i > start` duplicate skip guard, and the symmetric bucket pruning that makes Partition K feasible."

**Action:** Walk the interviewer through these steps:
1. *Subsets — foundational template:* "Define `backtrack(start, path)`: collect `path[:]` at every call (not just leaves — this is the key difference from Combinations). Loop from `start` to `len(nums)-1`: append `nums[i]`, recurse with `start = i+1`, pop. This produces 2^n subsets — every possible binary include/exclude decision."
2. *Subsets II — duplicate skip invariant:* "Sort `nums` first (brings duplicates adjacent). In the loop, add the guard `if i > start and nums[i] == nums[i-1]: continue`. The critical detail: `i > start`, NOT `i > 0`. Using `i > 0` would incorrectly skip valid subsets at deeper recursion levels where the same value appears at a different position. `i > start` limits the skip to the current depth level."
3. *Partition K — bucket backtracking:* "For each number (sorted descending for fast failure), try assigning it to each of the k buckets. Prune: if `bucket[j] + nums[i] > target` → skip. Symmetric pruning: use a `seen` set of bucket sums at the current level — if we've already tried assigning to a bucket with sum `s` and it failed, skip all other buckets with the same sum `s`. This eliminates entire symmetric subtrees."
4. *Why `seen` pruning works for Partition K:* "If two buckets have the same current sum, assigning `nums[i]` to either produces an identical subproblem. If one failed, the other will fail for the same reason. The `seen` set catches this symmetry in O(1) per bucket check."
5. *Complexity analysis:* "Subsets: exactly 2^n subsets, each takes O(n) to copy → O(n × 2^n). Subsets II: same worst case (all distinct), better for duplicates. Partition K: O(k × 2^n) — each of 2^n assignments is tried across k buckets. With pruning, average case is much better."

**Result:** "Subsets: O(n × 2^n) is optimal — you need to output all 2^n subsets, each of length up to n. Subsets II: the sort + `i > start` guard produces exactly the correct number of distinct subsets — for `[1,1,2]`: 6 subsets instead of 8 (no duplicates). Partition K: for n = 16 and k = 4, naive is 4^16 ≈ 4 × 10^9 — TLE. With descending sort + symmetric pruning, it runs in milliseconds because most branches are pruned before depth 3."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer this approach |
|-------------|---------------------------|--------------------------|
| Bitmask iteration for Subsets | Need random access to any subset by index | `for mask in range(1 << n)` generates all subsets iteratively; cleaner for subset enumeration without recursion; same O(n × 2^n) complexity |
| HashSet deduplication for Subsets II | Don't want to think about the skip guard | Convert each path to a tuple and add to a set; simpler to reason about but O(n × 2^n) extra memory for the set |
| Bitmask DP for Partition K | n ≤ 20 (bitmask fits in an int) | Bitmask DP: O(k × 2^n) with O(2^n) space, iterative; backtracking with pruning is often faster in practice due to early termination |

**Why NOT bitmask DP for Partition K in interviews:** Bitmask DP requires maintaining a `dp[mask]` array of size 2^n and reasoning about valid partial assignments — harder to construct correctly under time pressure. Backtracking with the two pruning rules (sort desc + `seen` set) is easier to derive and explain. Both are O(k × 2^n) worst case.

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

**Leadership Principle:** Are Right, A Lot

**STAR Story: Choosing Kafka Over RabbitMQ for a Multi-Team Event Pipeline — and Defending That Decision**

**Situation:** At my previous company, we were rebuilding the event pipeline for our e-commerce platform. At the time, we were using RabbitMQ for all message passing — it was the existing company standard, and three teams were comfortable with it. A new initiative required a pipeline that would fan out a single "order placed" event to five downstream consumers (inventory, fulfillment, email, analytics, search indexer), with the additional requirement that the search indexer team could replay the last 30 days of events whenever they deployed a new indexing schema. Two senior engineers and our engineering manager advocated for staying with RabbitMQ, since it was already deployed and teams knew it. I pushed for Kafka.

**Task:** I needed to make the right architectural decision for a system that would process hundreds of millions of events per year and support the search indexer's replay requirement — even though the consensus in the room was against me. I had to be right AND persuade the team without relying on authority.

**Action:**

*First,* I structured my evaluation around the specific requirements, not preferences. I listed the three key requirements: (1) fan-out to 5 independent consumers, (2) 30-day event replay for schema migrations, (3) throughput headroom for 10× growth. Then I evaluated each option against each requirement with concrete data.

*Then,* I acknowledged the RabbitMQ case fairly: for fan-out to 5 consumers, RabbitMQ fanout exchanges work — you'd create 5 separate queues, one per consumer. For replay, RabbitMQ does not support it natively; you'd need to persist events to a separate data store and replay from there — adding an entire extra system. For throughput, RabbitMQ handles ~50K msgs/sec per node; our projected peak was 40K msgs/sec, close to the limit.

*Next,* I built a decision matrix with every engineer in the room contributing requirements they cared about. This shifted the conversation from "which tool do we prefer" to "which tool satisfies this requirement." The replay requirement was the decisive one: RabbitMQ would require a separate event store (more infra, more failure modes), while Kafka's log retention natively solves it with one configuration parameter (`log.retention.hours=720`).

*Finally,* I proposed a 2-week spike: deploy Kafka on our staging environment, wire one consumer group to it, and measure throughput and latency under realistic load. This removed the theoretical debate and gave us empirical data. The spike results: Kafka processed 250K msgs/sec on a 3-node cluster with sub-5ms P99 latency. The team aligned behind Kafka at the end of the spike.

**Result:** Kafka was adopted for the event pipeline. Over 18 months, the search indexer team replayed historical events four times for schema migrations — each replay was a one-command Kafka consumer offset reset. Estimated cost of those four replays in a RabbitMQ architecture (building and maintaining a separate event store): approximately 6 engineer-weeks. Kafka eliminated that cost entirely. Throughput during peak sales events reached 180K msgs/sec — within Kafka's comfortable operating range but would have exceeded RabbitMQ's single-node limit. The decision matrix framework I introduced became a team standard for infrastructure evaluations.

*In an interview, say:* "I want to be clear: being right isn't about stubbornness. It's about having a rigorous framework for evaluation, presenting the evidence clearly, and being willing to test your assumptions empirically. The 2-week spike was the most important thing I did — it converted a debate about opinions into a question with a measurable answer." Use this for "Tell me about a time you disagreed with your team and were right," "Tell me about a technical decision you were proud of," or "Tell me about a time you changed the direction of a technical discussion with data."

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
