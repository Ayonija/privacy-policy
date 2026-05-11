# Day 88 — Backtracking: Word Search, Letter Tile Possibilities, Expression Add Operators
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Word Search uses grid DFS backtracking with in-place visited marking. Letter Tile Possibilities counts distinct sequences of any length from a multiset of tiles — permutation counting with duplicates. Expression Add Operators inserts +, −, × between digits to reach a target — backtracking on operator choice, with careful multiplication tracking to avoid re-evaluating previous terms.

---

## DSA (2 hours)
### Pattern: Grid DFS Backtracking + Multiset Permutation Counting + Expression Evaluation Backtracking

**Word Search (LC 79):**
Given a grid of characters and a word, find if the word exists as a connected path (horizontal/vertical, no cell reused). DFS from each cell matching `word[0]`: recursively match `word[k]` by moving to adjacent unvisited cells. Mark the current cell as visited in-place (e.g., `board[r][c] = '#'`); restore on backtrack.

**Letter Tile Possibilities (LC 1079):**
Count distinct non-empty sequences of any length from a multiset of tiles. Use a frequency counter. At each step, pick any tile with count > 0, add 1 to result (this sequence length is valid), decrement count, recurse, restore count. This counts all non-empty sequences without duplicates because we branch on distinct available characters (frequency map), not on positions.

**Expression Add Operators (LC 282):**
Insert +, −, × between digits of a string to reach target. Backtracking: at each split point, try all three operators. Track `current_eval` and `prev_multiplier` — when you encounter ×, you must undo the previous addition/subtraction and re-apply with multiplication: `new_eval = current_eval - prev + prev * curr`.

**Trigger condition:**
- "does a word exist as a path in a 2D grid?" → DFS from each cell; in-place mark + restore; check word character by character
- "count distinct sequences from a character multiset" → frequency counter; backtrack over distinct chars; count at every non-empty recursive call
- "insert operators between digits to reach target value" → backtracking; track prev_multiplier for × undo-redo

**Time complexity:** LC 79: O(m×n×4^L) | LC 1079: O(n! / (dups!)) | LC 282: O(4^n × n)
**Space complexity:** O(L) recursion depth / O(unique_chars) / O(n) for current expression

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Word Search | 79 | Medium | Grid DFS backtracking, in-place mark | Mark visited with `'#'`; restore after backtrack; DFS from every matching start cell |
| 2 | Letter Tile Possibilities | 1079 | Medium | Frequency counter backtracking | Count at every recursive call (any length valid); branch on distinct chars; no sorting needed |
| 3 | Expression Add Operators | 282 | Hard | Operator-insertion backtracking with prev_mult tracking | Track `eval` and `prev_mult`; on `*`: `eval = eval - prev_mult + prev_mult * curr` |

---

### Code Skeleton
```java
import java.util.*;

// Word Search (LC 79)
class Solution {
    public boolean exist(char[][] board, String word) {
        int m = board.length, n = board[0].length;
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (dfs(board, r, c, word, 0, m, n)) return true;
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, int r, int c, String word, int k, int m, int n) {
        if (k == word.length()) return true;
        if (r < 0 || r >= m || c < 0 || c >= n || board[r][c] != word.charAt(k)) return false;
        char temp = board[r][c];
        board[r][c] = '#'; // mark visited
        boolean found = dfs(board, r+1, c, word, k+1, m, n)
                     || dfs(board, r-1, c, word, k+1, m, n)
                     || dfs(board, r, c+1, word, k+1, m, n)
                     || dfs(board, r, c-1, word, k+1, m, n);
        board[r][c] = temp; // restore
        return found;
    }
}

// Letter Tile Possibilities (LC 1079)
class Solution {
    public int numTilePossibilities(String tiles) {
        int[] count = new int[26];
        for (char c : tiles.toCharArray()) count[c - 'A']++;
        return backtrack(count);
    }

    private int backtrack(int[] count) {
        int total = 0;
        for (int i = 0; i < 26; i++) {
            if (count[i] > 0) {
                total += 1; // this sequence (of any length) is valid
                count[i]--;
                total += backtrack(count);
                count[i]++;
            }
        }
        return total;
    }
}

// Expression Add Operators (LC 282)
class Solution {
    public List<String> addOperators(String num, int target) {
        List<String> result = new ArrayList<>();
        backtrack(num, target, 0, "", 0, 0, result);
        return result;
    }

    private void backtrack(String num, int target, int index, String path, long evalVal, long prevMult, List<String> result) {
        if (index == num.length()) {
            if (evalVal == target) result.add(path);
            return;
        }
        for (int end = index + 1; end <= num.length(); end++) {
            String token = num.substring(index, end);
            if (token.length() > 1 && token.charAt(0) == '0') break; // no leading zeros
            long curr = Long.parseLong(token);
            if (index == 0) {
                backtrack(num, target, end, token, curr, curr, result);
            } else {
                // Add
                backtrack(num, target, end, path + "+" + token, evalVal + curr, curr, result);
                // Subtract
                backtrack(num, target, end, path + "-" + token, evalVal - curr, -curr, result);
                // Multiply: undo last addition, re-apply with multiplication
                backtrack(num, target, end, path + "*" + token,
                          evalVal - prevMult + prevMult * curr,
                          prevMult * curr, result);
            }
        }
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 79: single cell board with single char word → True if they match; word longer than total cells → impossible; word visits same cell twice in path → prevented by '#' marking
- LC 1079: single tile → 1 possibility; all same tiles e.g. "AAA" → 3 (sequences: "A", "AA", "AAA"); 26 unique tiles → sum of P(26,1) + P(26,2) + ... + P(26,26)
- LC 282: `num = "3456237490"`, `target = 9191` → no valid expression; `num = "105"`, `target = 5` → `"1*0+5"`, `"10-5"`; leading zero check prevents e.g. "05" as a multi-digit number

---

### Interview Pattern Drill

**Word Search optimisation — early termination:**
Before DFS: count character frequencies in the word. If the board doesn't have enough of any character → return False immediately. Also: if `word[0]` is rarer than `word[-1]` in the board, search for the reversed word (start from less common end for fewer DFS starts).

**Expression Add Operators — multiply tracking explained:**
When evaluating left to right with mixed operators, multiplication breaks associativity. Example:
- Path so far: `"2+3"`, `eval = 5`, `prev_mult = 3`
- Now multiply by 4: can't just do `5 * 4 = 20`
- Must "undo" the last addition: `eval - prev_mult = 5 - 3 = 2`
- Then multiply: `2 + (3 * 4) = 2 + 12 = 14`
- Formula: `new_eval = eval - prev_mult + prev_mult * curr`
- New `prev_mult = prev_mult * curr` (for chaining: `3*4*5` needs to undo `3*4`)

**Letter Tile Possibilities — why no sorting needed:**
Counting over the frequency map means we iterate over DISTINCT characters. Two tiles with the same letter are interchangeable — we don't need to sort to deduplicate; the frequency counter naturally handles this by treating all 'A' tiles as the same choice.

---

## System Design (1 hour)
### Topic: Message Queue Scaling — Throughput, Partitioning, and Backpressure

**Throughput bottlenecks in message queues:**

Producer side:
- Batch size (`batch.size`): larger batches reduce overhead; wait up to `linger.ms` for more records
- Compression (`compression.type=lz4/snappy`): reduces network and disk IO at cost of CPU
- In-flight requests: `max.in.flight.requests.per.connection=5` allows pipelining without waiting for ACK

Consumer side:
- `max.poll.records`: how many records per poll; too small = overhead; too large = long processing time before next heartbeat
- Consumer parallelism: limited by partition count — add partitions to scale consumers
- Processing time: if heavy computation per record, consider offloading to a thread pool

**Partitioning strategy for scaling:**
```
Topic: "user_events"
Partition count: 100
Partition key: user_id

→ All events from user 42 go to the same partition
→ User-specific ordering guaranteed
→ 100 consumer instances can process in parallel
→ Adding consumers beyond 100: idle (no partition to assign)
```

Rule: **set partition count high at creation** — Kafka allows increasing but not decreasing partitions. Increasing partitions breaks per-key ordering guarantees for existing data.

**Horizontal scaling of consumers:**
Kubernetes HPA (Horizontal Pod Autoscaler) + consumer lag metric:
```yaml
# Conceptual: scale consumer deployment based on Kafka lag
- metric: kafka_consumer_lag
  threshold: 5000       # scale up when lag > 5000 messages
  scaleUp: add 2 pods
  scaleDown: lag < 500 for 5 min → remove 1 pod
```

**Backpressure mechanisms:**

1. **Rate limiting producers:** API gateway throttles POST requests when queue depth > threshold
2. **Bounded queues:** in-memory buffers between consumer and processing thread; block producer-to-consumer when full (natural backpressure)
3. **Circuit breaker:** consumer stops polling when downstream service (DB) is slow; queue accumulates; alerting fires; downstream service recovers
4. **Reactive streams:** protocols like RSocket or Akka Streams have built-in backpressure signalling — consumer explicitly requests N items at a time

**Multi-region Kafka:**
```
us-east-1 (primary cluster)     eu-west-1 (replica cluster)
Producer → Kafka US             Kafka EU (MirrorMaker 2 replicates)
              │                          │
         Consumers US              Consumers EU
         (analytics, feed)         (EU compliance feeds)
```

MirrorMaker 2 replicates topics across clusters. Offsets are translated (EU offset ≠ US offset for same message). Used for: geo-redundancy, compliance (EU data stays in EU), local latency reduction.

**Capacity planning formula:**
```
Messages/day = 1B
Avg message size = 1 KB
Throughput = 1B × 1KB / 86400s ≈ 11.5 MB/s

Kafka partition throughput ≈ 10 MB/s (conservative)
Minimum partitions = 11.5 / 10 ≈ 2 (round up: 8 for headroom)

With RF=3: storage per day = 1B × 1KB × 3 = 3 TB
With 7-day retention: 21 TB total storage across cluster
```

**Interview talking point:** "When we see consumer lag growing in our feed fanout pipeline, our first response is horizontal scaling: we have a Kubernetes HPA that watches the `kafka_consumer_lag` metric from Prometheus and adds fanout worker pods when lag exceeds 10K messages. The partition count (100) determines the maximum parallelism. If we've hit that ceiling, we need to increase partition count — but we do this in a maintenance window since it breaks per-user ordering guarantees. For the short term, we can also increase `max.poll.records` per worker if the bottleneck is polling overhead rather than processing."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 79 Word Search (target: 15 min)
- **Medium 2:** LC 1079 Letter Tile Possibilities (target: 15 min)
- **Hard:** LC 282 Expression Add Operators (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to scale a system under unexpected load — what signals told you there was a problem, what did you do, and what did you learn?
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| How does Word Search mark and unmark visited cells during DFS? | Temporarily replace `board[r][c]` with `'#'` (or any non-letter sentinel) before recursing. After recursion returns, restore `board[r][c] = temp`. This avoids an extra O(m×n) visited array and is safe because `'#'` won't match any letter in `word`. |
| How does Letter Tile Possibilities avoid double-counting duplicate tiles? | Uses a frequency Counter instead of iterating over positions. At each step, loop over DISTINCT characters with count > 0. Two 'A' tiles are treated as the same choice; no sorting needed. Add 1 before recursing (current sequence is valid at any length). |
| What is the multiply tracking trick in Expression Add Operators? | Track `prev_mult` (the last term's contribution). When adding `*` before `curr`: `new_eval = eval - prev_mult + prev_mult * curr`. This "undoes" the last addition and "re-applies" it with multiplication. New `prev_mult = prev_mult * curr` to support chaining. |
| What limits the maximum parallelism of a Kafka consumer group? | The number of partitions in the topic. Each partition is assigned to at most one consumer in a group. If consumers > partitions: extra consumers are idle. To increase parallelism beyond current consumers, increase partition count (plan ahead — increasing breaks per-key ordering for existing data). |
| What is backpressure in a messaging system and how do you implement it? | Backpressure slows down producers when consumers can't keep up. Implementations: (1) rate-limit producers at the API gateway based on queue depth; (2) bounded in-memory buffers that block when full; (3) circuit breaker that stops consumer polling when downstream is slow; (4) reactive streams protocols (RSocket) with explicit request-N semantics. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/30 min, no hints)
- [ ] Rewrote Word Search in-place DFS and Expression Add Operators multiply tracking from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain Kafka partition scaling and backpressure cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
