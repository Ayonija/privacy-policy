# Day 85 — Backtracking: Combinations, Combination Sum, and Sudoku Solver
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Combinations generates all size-k subsets — same loop structure as Subsets but collect only at depth k. Combination Sum allows unlimited reuse of each element — recurse with the same index instead of index+1. Sudoku Solver is the canonical constraint-satisfaction backtracking problem: try each digit in empty cells, validate against row/col/box constraints, backtrack on failure.

---

## DSA (2 hours)
### Pattern: Fixed-Depth Backtracking + Unlimited Reuse + Constraint Satisfaction

**Combinations (LC 77):**
Choose k numbers from [1, n]. Same backtracking template as Subsets but only collect at depth k (`len(path) == k`). Pruning: early exit if remaining elements < remaining slots needed: `n - i + 1 < k - len(path)`.

**Combination Sum (LC 39):**
Find all combinations from `candidates` that sum to `target`. Each candidate can be used unlimited times. Backtracking: loop from `start` to end; recurse with the SAME `i` (allowing reuse); subtract candidate from target. Base case: `target == 0` → collect. Prune: `target < 0` → return.

**Sudoku Solver (LC 37):**
Fill in empty cells ('.') so every row, column, and 3×3 box contains digits 1–9 exactly once. Backtracking: find the next empty cell; try digits 1–9; validate (not in row/col/box); place and recurse; if recursion fails → undo (backtrack). Return True if board is fully filled.

**Trigger condition:**
- "all size-k subsets from [1, n]" → backtracking with depth == k leaf collection; prune when remaining elements < needed slots
- "all combinations from candidates that sum to target (unlimited reuse)" → backtracking; recurse with same index; base case: target == 0
- "fill in a constraint grid (Sudoku, N-Queens variant)" → constraint-satisfaction backtracking; validate before placing; backtrack on failure

**Time complexity:** LC 77: O(C(n,k) × k) | LC 39: O(target^(target/min_candidate)) | LC 37: O(9^m) where m = empty cells
**Space complexity:** O(k) / O(target/min_candidate) recursion depth / O(1) in-place

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Combinations | 77 | Medium | Backtracking, collect at depth k | Same loop as Subsets; collect when `len(path) == k`; prune early when remaining < needed |
| 2 | Combination Sum | 39 | Medium | Backtracking with reuse (same index) | Recurse with `i` not `i+1`; subtract from target; collect when target == 0 |
| 3 | Sudoku Solver | 37 | Hard | Constraint-satisfaction backtracking | Try 1–9 in each empty cell; validate row/col/box; backtrack if no digit works |

---

### Code Skeleton
```java
import java.util.*;

// Combinations (LC 77)
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(n, k, 1, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int n, int k, int start, List<Integer> path, List<List<Integer>> result) {
        if (path.size() == k) {
            result.add(new ArrayList<>(path));
            return;
        }
        // Prune: need (k - path.size()) more elements; only n - i + 1 available
        for (int i = start; i <= n - (k - path.size()) + 1; i++) {
            path.add(i);
            backtrack(n, k, i + 1, path, result);
            path.remove(path.size() - 1);
        }
    }
}

// Combination Sum (LC 39)
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(candidates); // sort to enable early break
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remaining, int start, List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            int c = candidates[i];
            if (c > remaining) break; // candidates sorted; no point continuing
            path.add(c);
            backtrack(candidates, remaining - c, i, path, result); // i not i+1: allow reuse
            path.remove(path.size() - 1);
        }
    }
}

// Sudoku Solver (LC 37)
class Solution {
    public void solveSudoku(char[][] board) {
        Set<Character>[] rows = new HashSet[9];
        Set<Character>[] cols = new HashSet[9];
        Set<Character>[] boxes = new HashSet[9];
        for (int i = 0; i < 9; i++) {
            rows[i] = new HashSet<>();
            cols[i] = new HashSet<>();
            boxes[i] = new HashSet<>();
        }

        // Initialize constraint sets
        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] != '.') {
                    char d = board[r][c];
                    rows[r].add(d);
                    cols[c].add(d);
                    boxes[(r / 3) * 3 + c / 3].add(d);
                }
            }
        }
        backtrack(board, rows, cols, boxes);
    }

    private boolean backtrack(char[][] board, Set<Character>[] rows, Set<Character>[] cols, Set<Character>[] boxes) {
        // Find next empty cell
        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] == '.') {
                    int boxId = (r / 3) * 3 + c / 3;
                    for (char d = '1'; d <= '9'; d++) {
                        if (!rows[r].contains(d) && !cols[c].contains(d) && !boxes[boxId].contains(d)) {
                            board[r][c] = d;
                            rows[r].add(d); cols[c].add(d); boxes[boxId].add(d);
                            if (backtrack(board, rows, cols, boxes)) return true;
                            board[r][c] = '.';
                            rows[r].remove(d); cols[c].remove(d); boxes[boxId].remove(d);
                        }
                    }
                    return false; // no digit works → backtrack
                }
            }
        }
        return true; // no empty cell → solved
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 77: `k == n` → only one combination (all numbers); `k == 1` → n combinations (each single number); `k == 0` → return `[[]]`
- LC 39: `candidates = [1]`, `target = 100` → single combination with 100 ones; single candidate equal to target → `[[target]]`; no valid combination → empty result
- LC 37: board with only one empty cell → try all 9 digits, exactly one valid; already-solved board → return immediately

---

### Interview Pattern Drill

**Subsets vs Combinations vs Combination Sum:**
| Problem | Reuse? | Collect at | Start index for recursion |
|---------|--------|-----------|--------------------------|
| Subsets | No | Every node | `i + 1` |
| Combinations | No | Depth == k | `i + 1` |
| Combination Sum | Yes | `remaining == 0` | `i` (same) |
| Combination Sum II | No (unique) | `remaining == 0` | `i + 1`, skip dups |

**Sudoku box indexing:**
`box_id = (row // 3) * 3 + (col // 3)` maps each cell to one of 9 boxes (0–8). Memorise this formula.

**Pruning in Combinations:**
`range(start, n - (k - len(path)) + 2)` — the upper bound ensures at least `k - len(path)` elements remain after the current pick. Without this, we'd try impossible prefixes. The `+2` accounts for 1-indexed range and Python's exclusive upper bound.

**Sudoku backtracking efficiency:**
Pre-initialising `rows[]`, `cols[]`, `boxes[]` sets makes validation O(1) per digit check instead of O(9) scanning. After placing a digit: add to all three sets. After removing (backtrack): discard from all three.

---

## System Design (1 hour)
### Topic: At-Least-Once and Exactly-Once Delivery in Message Queues

**The three delivery guarantees:**

**At-most-once (fire and forget):**
- Producer sends; no retry on failure
- Consumer ACKs before processing
- Messages can be lost (producer failure, consumer crash before processing)
- Use when: metrics, logs — losing some data is acceptable; throughput > correctness

**At-least-once (the default for most systems):**
- Producer retries on no-ACK (idempotency key prevents duplicates at broker)
- Consumer commits offset AFTER processing
- Messages can be re-delivered if consumer crashes after processing but before committing offset
- Consumer must handle duplicates: idempotency checks in the database
- Use when: payments, notifications — must not lose; can handle dedup logic

**Exactly-once (strongest guarantee):**
Kafka transactions:
1. Producer gets a unique `transactional.id`; broker assigns a PID + epoch
2. Producer opens transaction: `beginTransaction()`
3. Sends messages to multiple partitions atomically
4. Commits offset AND output in one atomic transaction: `commitTransaction()`
5. Consumers configured with `isolation.level=read_committed`: only see committed messages

```
Producer A (transactional.id="payment-processor-1"):
  beginTransaction()
  send("payments-output", payment_event)
  sendOffsets(consumer_group, {partition_0: offset_42})
  commitTransaction()   → atomic: either all succeed or all fail
```

**Exactly-once in practice:**
- Kafka Streams: built-in exactly-once (enable with `processing.guarantee=exactly_once`)
- Custom consumers: difficult; often use database transactions (process + mark as done atomically)
- Flink: checkpointing + barriers provide exactly-once semantics over Kafka sources

**Idempotency as a substitute for exactly-once:**
For most systems, exactly-once is complex and expensive. Instead: at-least-once delivery + idempotent consumers.
- Payment service: check `payment_id` in DB before charging → skip if already processed
- Feed fanout: `ZADD` is idempotent (re-adding same post_id with same score is a no-op)
- Email sender: store `notification_id` in Redis; skip if already sent

**Practical delivery configuration matrix:**

| System | Guarantee | Why |
|--------|-----------|-----|
| Payment processing | At-least-once + DB idempotency | Can't lose payments; DB dedup handles duplicates |
| Feed fanout | At-least-once | Redis ZADD is idempotent; duplicate fanout is harmless |
| Analytics events | At-most-once | Losing 0.01% of click events is acceptable |
| Email notifications | At-least-once + dedup | Can't miss emails; sending twice is annoying but not catastrophic |
| Financial audit log | Exactly-once | Strict compliance; no duplicates in ledger |

**Kafka consumer offset commit strategies:**

```java
// At-most-once: commit BEFORE processing
consumer.poll();
consumer.commitSync();   // commit first
process(messages);       // if crash here: message lost

// At-least-once: commit AFTER processing
consumer.poll();
process(messages);       // if crash here: message re-delivered on restart
consumer.commitSync();   // commit after

// Exactly-once: atomic process + commit (Kafka transactions)
// try (transaction) {
//     processAndWriteToOutput(messages);
//     commitOffset(consumerGroup, partition, offset);
// }
```

**Interview talking point:** "For our payment processing pipeline, we use at-least-once delivery with database idempotency rather than Kafka's exactly-once transactions. Each payment message carries a UUID; before charging the customer, we do an INSERT ... ON CONFLICT DO NOTHING on the `processed_payments` table. If the consumer crashes and re-processes the same message, the idempotency check prevents double-charging. This is simpler than Kafka transactions and achieves the same effective guarantee with lower latency."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 77 Combinations (target: 12 min)
- **Medium 2:** LC 39 Combination Sum (target: 15 min)
- **Hard:** LC 37 Sudoku Solver (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you designed a system where data correctness (no duplicates, no missing records) was critical — what delivery guarantees did you implement and how did you test them?
- Leadership principle: Insist on the Highest Standards

---

## Flashcards

| Q | A |
|---|---|
| How does Combination Sum differ from Combinations in the backtracking loop? | Combination Sum recurses with `i` (same index) allowing reuse of the same candidate. Combinations recurse with `i + 1` (no reuse). Both loop from `start` to end; Combination Sum sorts candidates and breaks early when `c > remaining`. |
| What is the pruning condition in Combinations (LC 77)? | `range(start, n - (k - len(path)) + 2)` — upper bound ensures at least `k - len(path)` elements remain after picking the current one. Without this, we'd explore impossible prefixes where remaining elements < remaining slots. |
| How do you compute the box index in Sudoku Solver? | `box_id = (row // 3) * 3 + (col // 3)`. This maps each of the 81 cells to one of 9 boxes (0–8). Pre-build sets for all 9 boxes to get O(1) validation. |
| What is the difference between at-least-once and exactly-once delivery? | At-least-once: consumer commits offset after processing; crashes cause re-delivery (duplicates possible). Exactly-once: Kafka transactions atomically commit offset + output; no duplicates, but higher latency and complexity. Most systems use at-least-once + idempotent consumers instead. |
| How do you implement idempotent message processing without Kafka transactions? | Each message carries a unique ID. Before processing: check if ID exists in a processed-IDs store (DB or Redis). If yes: skip. If no: process + store ID atomically. This gives effective exactly-once semantics with simpler infrastructure. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/15/30 min, no hints)
- [ ] Rewrote Combination Sum and Sudoku Solver backtracking from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain at-least-once vs exactly-once and idempotency pattern cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
