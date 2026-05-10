# Day 87 — Backtracking: Palindrome Partitioning, Generate Parentheses, N-Queens
**Week 13 | Phase 1: DSA Mastery | Month 3**

## Focus
Palindrome Partitioning explores all ways to split a string into palindrome substrings using backtracking with precomputed palindrome checks. Generate Parentheses generates all valid combinations of n pairs of parentheses by tracking open and close counts and their constraints. N-Queens is the classic backtracking problem with three constraint sets (columns, diagonals) that determine valid queen placements.

---

## DSA (2 hours)
### Pattern: String Partition Backtracking + Count-Constrained Generation + Multi-Constraint Placement

**Palindrome Partitioning (LC 131):**
Partition string `s` into all possible lists of palindrome substrings. Backtracking: starting at index `start`, try every substring `s[start:end+1]`; if it's a palindrome, add to path and recurse from `end+1`. Collect path when `start == len(s)`. Precompute a 2D `is_palindrome[i][j]` table in O(n²) to make palindrome check O(1).

**Generate Parentheses (LC 22):**
Generate all valid combinations of n pairs of parentheses. Constraints: can add `(` if `open < n`; can add `)` if `close < open`. Backtracking: two branches at each step — add `(` or `)`. Collect when `len(path) == 2n`.

**N-Queens (LC 51):**
Place n queens on an n×n board so no two queens share a row, column, or diagonal. Backtracking row by row: at row r, try each column c; check if c is in `cols`, `(r-c)` in `diag1`, `(r+c)` in `diag2`; if not → place queen, recurse to row r+1, then remove. Collect when `r == n`.

**Trigger condition:**
- "all ways to partition a string into palindromes" → backtracking on string with is_palindrome check; DP table for O(1) check
- "generate all valid parentheses combinations" → count-constrained backtracking; `open < n` → add `(`; `close < open` → add `)`
- "place non-attacking queens on n×n board" → row-by-row backtracking; track forbidden columns and diagonals with sets

**Time complexity:** LC 131: O(n × 2^n) | LC 22: O(4^n / √n) Catalan number | LC 51: O(n!)
**Space complexity:** O(n²) for palindrome DP table / O(n) recursion / O(n) for constraint sets

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Palindrome Partitioning | 131 | Medium | String partition backtracking + DP palindrome table | Precompute `is_pal[i][j]` in O(n²); backtrack with O(1) palindrome check |
| 2 | Generate Parentheses | 22 | Medium | Count-constrained two-branch backtracking | `open < n` → add `(`; `close < open` → add `)`; O(4^n/√n) valid sequences |
| 3 | N-Queens | 51 | Hard | Row-by-row backtracking with column+diagonal sets | `cols`, `diag1 = r-c`, `diag2 = r+c`; place in each row, skip attacked columns |

---

### Code Skeleton
```python
# Palindrome Partitioning (LC 131)
def partition(s):
    n = len(s)
    # Precompute is_palindrome[i][j]
    is_pal = [[False] * n for _ in range(n)]
    for i in range(n - 1, -1, -1):
        for j in range(i, n):
            if s[i] == s[j] and (j - i <= 2 or is_pal[i + 1][j - 1]):
                is_pal[i][j] = True

    result = []
    def backtrack(start, path):
        if start == n:
            result.append(path[:])
            return
        for end in range(start, n):
            if is_pal[start][end]:
                path.append(s[start:end + 1])
                backtrack(end + 1, path)
                path.pop()
    backtrack(0, [])
    return result

# Generate Parentheses (LC 22)
def generateParenthesis(n):
    result = []
    def backtrack(path, open_count, close_count):
        if len(path) == 2 * n:
            result.append(''.join(path))
            return
        if open_count < n:
            path.append('(')
            backtrack(path, open_count + 1, close_count)
            path.pop()
        if close_count < open_count:
            path.append(')')
            backtrack(path, open_count, close_count + 1)
            path.pop()
    backtrack([], 0, 0)
    return result

# N-Queens (LC 51)
def solveNQueens(n):
    cols = set()
    diag1 = set()   # r - c
    diag2 = set()   # r + c
    board = [['.' for _ in range(n)] for _ in range(n)]
    result = []

    def backtrack(row):
        if row == n:
            result.append([''.join(r) for r in board])
            return
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue
            board[row][col] = 'Q'
            cols.add(col); diag1.add(row - col); diag2.add(row + col)
            backtrack(row + 1)
            board[row][col] = '.'
            cols.discard(col); diag1.discard(row - col); diag2.discard(row + col)
    backtrack(0)
    return result
```

---

### Edge Cases to Trace Before Coding
- LC 131: single character → one partition `[["a"]]`; all same chars e.g. "aaa" → `[["a","a","a"], ["a","aa"], ["aa","a"], ["aaa"]]`; string is itself a palindrome → one partition is the whole string
- LC 22: n=1 → `["()"]`; n=0 → `[""]` or `[]` (check problem constraints); n=3 → 5 valid strings (Catalan(3))
- LC 51: n=1 → `[["Q"]]`; n=2, 3 → no solution; n=4 → 2 solutions; n=8 → 92 solutions

---

### Interview Pattern Drill

**N-Queens diagonal invariants:**
- Two queens on the same ↘ diagonal share the same `row - col` value
- Two queens on the same ↗ diagonal share the same `row + col` value
- These two formulas compress 2n-1 diagonals into a single set lookup

**Generate Parentheses constraint logic:**
The two rules `open < n` and `close < open` are sufficient to generate ALL valid sequences and ONLY valid sequences:
- `open < n`: we haven't used all open brackets yet
- `close < open`: we can't close a bracket that hasn't been opened
Together they ensure every prefix is a valid partial sequence.

**Palindrome DP recurrence:**
`is_pal[i][j]` is True if `s[i] == s[j]` AND either `j - i <= 2` (single char or pair) OR `is_pal[i+1][j-1]` is True. Iterate i from right to left (so `is_pal[i+1][j-1]` is already computed when we compute `is_pal[i][j]`).

**Backtracking efficiency comparison:**
| Problem | Pruning mechanism | What gets pruned |
|---------|-------------------|-----------------|
| Palindrome Partitioning | Skip non-palindrome substrings | Non-palindrome prefixes |
| Generate Parentheses | `open < n`, `close < open` constraints | All invalid prefix sequences |
| N-Queens | Column + diagonal sets | Attacked positions (most cells) |

---

## System Design (1 hour)
### Topic: Event Streaming Patterns — Event Sourcing, CQRS, and Outbox Pattern

**Event Sourcing:**
Instead of storing current state (UPDATE users SET balance = 100), store every event that led to the state:
```
UserCreated {user_id: 1, name: "Alice"}
DepositMade {user_id: 1, amount: 200}
WithdrawalMade {user_id: 1, amount: 100}
```
Current state = replay of all events. Kafka (or an event store) is the source of truth.

**Benefits of event sourcing:**
- Complete audit trail (regulatory compliance)
- Rebuild any past state by replaying to a timestamp
- Event-driven architecture: downstream services react to events
- Time-travel debugging: reproduce any bug by replaying events

**Drawbacks:**
- Query complexity: need read-optimised projections (read models)
- Storage: event log grows indefinitely (mitigate with snapshots)
- Schema evolution: old events must remain readable as schemas change

**CQRS (Command Query Responsibility Segregation):**
Separate the write model (commands) from the read model (queries).
```
Write path: POST /transfer → Command Handler → Event Store (Kafka)
                                                      │
                                               [Projection Worker]
                                                      │
Read path: GET /balance → Query Handler → Read DB (denormalized, fast)
```
Read DB is a materialized view updated by consuming events from Kafka. Reads are fast (pre-computed); writes are async (eventual consistency).

**Outbox Pattern:**
Problem: writing to a DB and publishing to Kafka in the same request without a distributed transaction. Risk: DB write succeeds but Kafka publish fails → data inconsistency.

Solution: write the event to an `outbox` table in the SAME DB transaction as the business entity:
```sql
BEGIN;
  INSERT INTO orders (id, ...) VALUES (...);
  INSERT INTO outbox (event_type, payload) VALUES ('OrderCreated', '{...}');
COMMIT;
```
A separate **outbox worker** reads from the outbox table and publishes to Kafka, then marks as sent. This guarantees: if the DB transaction commits, the event will eventually be published.

```
┌─────────────────────┐
│   Application DB    │
│  orders table       │  ← written atomically in same TX
│  outbox table       │
└──────────┬──────────┘
           │ outbox worker polls
           ▼
    [Kafka Publisher]
           │
    Kafka Topic "orders"
```

**Change Data Capture (CDC) as alternative to Outbox:**
Tools like Debezium watch the DB's write-ahead log (WAL/binlog) and stream changes to Kafka automatically. No outbox table needed; lower application complexity; higher operational complexity (Debezium connector management).

**Interview talking point:** "The Outbox Pattern solves the dual-write problem — the hardest distributed systems challenge in microservices. Without it, a payment service might write a payment to MySQL but fail to publish to Kafka, leaving the analytics pipeline and notification system unaware of the payment. The Outbox pattern makes the Kafka publish an eventually-consistent side effect of the DB commit rather than a separate operation that can fail independently. We use it in our payment pipeline with a 100ms polling outbox worker; for lower latency we could switch to Debezium CDC."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 131 Palindrome Partitioning (target: 20 min)
- **Medium 2:** LC 22 Generate Parentheses (target: 12 min)
- **Hard:** LC 51 N-Queens (target: 28 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you identified a data consistency issue between two systems — how did you diagnose it and what was your fix?
- Leadership principle: Earn Trust

---

## Flashcards

| Q | A |
|---|---|
| How do you precompute the palindrome DP table for Palindrome Partitioning? | `is_pal[i][j] = (s[i] == s[j]) AND (j - i <= 2 OR is_pal[i+1][j-1])`. Iterate i from `n-1` down to 0; j from i to n-1. Enables O(1) palindrome check during backtracking instead of O(n) per check. |
| What are the two constraints that make Generate Parentheses backtracking generate only valid sequences? | (1) `open < n`: can add `(` only if we haven't used all n open brackets. (2) `close < open`: can add `)` only if there's an unmatched open bracket. Together they ensure every prefix is a valid partial sequence. |
| How do you track attacked diagonals in N-Queens without a 2D grid? | Two sets: `diag1` stores `r - c` values (↘ diagonals); `diag2` stores `r + c` values (↗ diagonals). Two queens attack each other diagonally iff they share the same `r - c` or `r + c`. O(1) lookup with sets. |
| What is the Outbox Pattern and what problem does it solve? | Solves dual-write consistency: write the Kafka event to an `outbox` table in the SAME DB transaction as the business data. A separate worker reads the outbox and publishes to Kafka. Guarantees: if DB commits → event will eventually be published. No distributed transaction needed. |
| What is the difference between Event Sourcing and CQRS? | Event Sourcing: state is derived by replaying events; the event log is the source of truth. CQRS: separates write model (commands) from read model (queries, materialized views). They're complementary: Event Sourcing generates events; CQRS builds read projections from those events. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 20/12/28 min, no hints)
- [ ] Rewrote Palindrome Partitioning DP table and N-Queens diagonal sets from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain Outbox Pattern and CQRS cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
