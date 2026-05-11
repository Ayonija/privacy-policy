# Day 86 — Backtracking: Permutations, Permutations II, Permutation Sequence
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Permutations generates all orderings using a visited array — every element can go in every position. Permutations II handles duplicates by sorting and skipping identical elements at the same depth. Permutation Sequence uses math (factorial number system) to directly compute the k-th permutation in O(n²) without generating all n! permutations.

---

## DSA (2 hours)
### Pattern: Visited-Array Permutation + Duplicate Skip + Factorial Number System

**Permutations (LC 46):**
All orderings of a list of distinct integers. Backtracking: at each depth, loop over ALL elements; use a `visited` boolean array to avoid reusing elements in the same path. Collect when `len(path) == n`.

**Permutations II (LC 47):**
All unique permutations with duplicates. Sort first. In the loop: skip `nums[i]` if:
- `visited[i]` is True (already in path), OR
- `i > 0 AND nums[i] == nums[i-1] AND NOT visited[i-1]`

The second condition skips a duplicate that comes after an identical unvisited element — this ensures each group of duplicates is placed in sorted order (avoiding duplicate permutations).

**Permutation Sequence (LC 60):**
Return the k-th permutation (1-indexed) of [1, 2, ..., n]. Use the factorial number system: there are `(n-1)!` permutations starting with each digit. Given k (convert to 0-indexed): position in first slot = `k // (n-1)!`. Then recurse for the remaining slots with `k = k % (n-1)!`. Pick and remove the chosen digit each time.

**Trigger condition:**
- "all arrangements / orderings of distinct elements" → visited-array backtracking; loop from 0 to n
- "all unique permutations with duplicate elements" → sort; skip if `nums[i] == nums[i-1] AND NOT visited[i-1]`
- "the k-th permutation directly (without generating all)" → factorial number system; divide-and-mod on k; O(n²)

**Time complexity:** LC 46: O(n! × n) | LC 47: O(n! × n) worst case | LC 60: O(n²)
**Space complexity:** O(n) recursion depth for all

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Permutations | 46 | Medium | Visited-array backtracking | Loop from 0; skip if visited; collect at len(path) == n |
| 2 | Permutations II | 47 | Medium | Sort + skip duplicate at same level | Skip if `nums[i] == nums[i-1] AND NOT visited[i-1]` |
| 3 | Permutation Sequence | 60 | Hard | Factorial number system | At each position: `idx = k // (n-1)!`; pick `digits[idx]`; remove; `k %= (n-1)!` |

---

### Code Skeleton
```java
import java.util.*;

// Permutations (LC 46)
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        boolean[] visited = new boolean[nums.length];
        backtrack(nums, visited, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] visited, List<Integer> path, List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (visited[i]) continue;
            visited[i] = true;
            path.add(nums[i]);
            backtrack(nums, visited, path, result);
            path.remove(path.size() - 1);
            visited[i] = false;
        }
    }
}

// Permutations II (LC 47)
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        boolean[] visited = new boolean[nums.length];
        backtrack(nums, visited, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] visited, List<Integer> path, List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (visited[i]) continue;
            // Skip duplicate: same value as previous, and previous is NOT in path
            if (i > 0 && nums[i] == nums[i - 1] && !visited[i - 1]) continue;
            visited[i] = true;
            path.add(nums[i]);
            backtrack(nums, visited, path, result);
            path.remove(path.size() - 1);
            visited[i] = false;
        }
    }
}

// Permutation Sequence (LC 60)
class Solution {
    public String getPermutation(int n, int k) {
        List<Integer> digits = new ArrayList<>();
        int factorial = 1;
        for (int i = 1; i <= n; i++) {
            digits.add(i);
            factorial *= i;
        }
        k -= 1; // convert to 0-indexed
        StringBuilder result = new StringBuilder();
        for (int i = n; i > 0; i--) {
            factorial /= i;
            int idx = k / factorial;
            result.append(digits.get(idx));
            digits.remove(idx);
            k %= factorial;
        }
        return result.toString();
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 46: single element → one permutation `[[nums[0]]]`; two elements → two permutations
- LC 47: all identical e.g. `[1,1,1]` → one permutation `[[1,1,1]]`; `[1,1,2]` → three permutations
- LC 60: `k=1` → always the sorted permutation; `k=n!` → the reverse-sorted permutation; `n=1` → always "1"

---

### Interview Pattern Drill

**Permutations vs Subsets loop structure:**
| | Start index | Loop range | Visited array |
|---|------------|-----------|--------------|
| Subsets / Combinations | `start` (monotone) | `[start, n)` | No |
| Permutations | 0 (any element) | `[0, n)` | Yes |

**Why the Permutations II duplicate condition works:**
`nums[i] == nums[i-1] AND NOT visited[i-1]` — "the same value came before me and it's not in the current path."

This means: we're at the same depth, and we already tried placing `nums[i-1]` (same value) at this position in a previous branch. Placing `nums[i]` here would produce an identical subtree. The `NOT visited[i-1]` part distinguishes this from the case where `nums[i-1]` IS in the path at a different depth (then placing `nums[i]` produces a genuinely different permutation).

**Permutation Sequence derivation:**
For n=4, k=9 (0-indexed: k=8):
- 3! = 6 permutations per first digit
- idx = 8 // 6 = 1 → first digit = `digits[1]` = 2; remaining digits = [1, 3, 4]; k = 8 % 6 = 2
- 2! = 2 permutations per second digit
- idx = 2 // 2 = 1 → second digit = `digits[1]` = 3; remaining = [1, 4]; k = 2 % 2 = 0
- 1! = 1 permutation per third digit
- idx = 0 // 1 = 0 → third digit = `digits[0]` = 1; remaining = [4]; k = 0
- Last digit = 4
- Result: "2314"

---

## System Design (1 hour)
### Topic: Consumer Groups, Partition Assignment, and Offset Management

**Consumer group rebalancing:**
When a consumer joins or leaves a group, Kafka triggers a rebalance: all partitions are reassigned among the current group members.

```
Before rebalance (2 consumers, 4 partitions):
  Consumer A: Partition 0, 1
  Consumer B: Partition 2, 3

Consumer C joins → rebalance triggered:
  Consumer A: Partition 0
  Consumer B: Partition 1, 2
  Consumer C: Partition 3
```

Rebalance impact: during rebalance, consumption is paused (stop-the-world). Strategies to minimise:
- **Cooperative/incremental rebalancing** (Kafka 2.4+): only reassign partitions that need to move; others keep consuming
- **Static membership** (`group.instance.id`): consumer gets same partition assignment on restart without triggering rebalance

**Partition assignment strategies:**
1. **Range:** assign contiguous partitions to each consumer (default for topics with same partition count)
2. **Round-robin:** distribute partitions evenly across consumers
3. **Sticky:** minimise partition movement during rebalances

**Offset commit strategies:**
```java
// Auto-commit (enabled by default, risky)
// Offsets committed every auto.commit.interval.ms (5s)
// Risk: if consumer crashes between commit and processing → data re-processed

// Manual sync commit (after processing batch)
for (ConsumerRecord<?,?> message : consumer.poll(Duration.ofMillis(100))) {
    process(message);
}
consumer.commitSync(); // blocks until broker ACKs commit

// Manual async commit (higher throughput, less safe)
consumer.commitAsync(onCommitCallback);

// Committing specific offsets (per-partition fine-grained control)
Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
offsets.put(new TopicPartition(topic, partition), new OffsetAndMetadata(offset + 1));
consumer.commitSync(offsets);
```

**Consumer lag monitoring:**
`consumer_lag = log_end_offset - committed_offset` per partition.
- Lag = 0: consumer is caught up
- Growing lag: consumer is falling behind; scale out or investigate slow processing
- Tools: Kafka consumer lag metrics via JMX, Kafka Manager, Grafana + Prometheus exporter

**Exactly-once with consumer groups:**
Transactional consumers must configure `isolation.level=read_committed` — they only read messages from committed transactions. Uncommitted messages (from a failed transaction) are invisible to the consumer.

**Dead consumer handling:**
Each consumer sends heartbeats to the group coordinator. If no heartbeat within `session.timeout.ms` (default 10s), consumer is considered dead:
1. Group coordinator triggers rebalance
2. Dead consumer's partitions reassigned to healthy consumers
3. Committed offsets are used as resume point
4. Uncommitted messages in the dead consumer's last poll are re-delivered

**Session timeout tuning:**
- Low `session.timeout.ms` (3s): fast failover, but false positives on GC pauses
- High `session.timeout.ms` (30s): tolerates GC pauses, but slower failover on real failures
- `max.poll.interval.ms` must be ≥ longest batch processing time; violation = forced leave

**Interview talking point:** "Consumer lag is the most important operational metric for a Kafka-based system. If the feed fanout workers develop lag during a celebrity post spike, follower feeds are delayed. We address this by auto-scaling the consumer group (adding pods when avg lag > 5K messages) and by tuning `max.poll.records` so each poll fetches a manageable batch. For our exactly-once payment processor, we use manual offset commits inside a database transaction: `INSERT INTO payments; UPDATE offset_table; COMMIT` — the DB transaction makes offset commit and payment record atomic without needing Kafka transactions."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 46 Permutations (target: 12 min)
- **Medium 2:** LC 47 Permutations II (target: 18 min)
- **Hard:** LC 60 Permutation Sequence (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to scale a consumer or worker pool to keep up with a message backlog — what was the bottleneck, how did you identify it, and how did you resolve it?
- Leadership principle: Deliver Results

---

## Flashcards

| Q | A |
|---|---|
| How does Permutations backtracking differ from Combinations? | Permutations loops from 0 (any element at each position) with a `visited` array. Combinations loop from `start` (monotone, no reuse). Permutations collect at `len(path) == n`; both use the same apply-recurse-undo template. |
| What is the duplicate skip condition in Permutations II and why does it work? | `if i > 0 and nums[i] == nums[i-1] and not visited[i-1]: continue`. Skips when the same value appears earlier in the sorted array AND that earlier element is NOT currently in the path — meaning we already explored that subtree in a previous branch at this depth. |
| How does the factorial number system compute the k-th permutation? | Convert k to 0-indexed. At each of n positions: `idx = k // (n-1)!`, pick `digits[idx]`, remove it, `k %= (n-1)!`. Repeat for n-1, n-2, ... 0 factorial. O(n²) — no backtracking needed. |
| What is consumer lag in Kafka and how do you monitor it? | Lag = `log_end_offset - committed_offset` per partition. Measures how far behind a consumer group is. Monitor via JMX metrics or tools like Grafana + Prometheus. Growing lag indicates the consumer is falling behind the producer rate. |
| What triggers a Kafka consumer group rebalance and how can you minimise its impact? | Triggered when a consumer joins, leaves, or times out (heartbeat miss). Minimise with: (1) cooperative/incremental rebalancing (Kafka 2.4+) — only moves affected partitions. (2) static membership (`group.instance.id`) — consumer rejoins with same partition assignment without full rebalance. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/18/25 min, no hints)
- [ ] Rewrote Permutations II duplicate condition and Permutation Sequence factorial logic from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain consumer group rebalancing and offset commit strategies cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
