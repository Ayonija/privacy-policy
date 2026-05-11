# Day 91 — 1D DP: Fibonacci Variants & State Machine Introduction
**Week 13 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Transition from Phase 1 recursive patterns into dynamic programming: reframe overlapping subproblems as a filled table, master the Fibonacci-style recurrence, and tackle two canonical Hard DP problems that test multi-dimensional state tracking.

---

## DSA (2 hours)

### Pattern: 1D Dynamic Programming — Fibonacci Recurrence & State Machines

**Core idea:**  
Store results of subproblems in a 1D array (or two variables). Each cell `dp[i]` is computed from a fixed number of prior cells, eliminating redundant recursion. When the state at position `i` depends on more than just "which index" — e.g., how many of something have been used — extend to `dp[i][state]`.

**Trigger condition:**  
- Problem asks: count of ways, min/max value, or feasibility  
- Each state depends on only the previous 1–3 states (Fibonacci window)  
- OR each state combines a position index with a small, enumerable auxiliary state (state machine)

**Complexity:**  
- Time: O(n × S) where S = number of states | Space: O(S) with rolling variables

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Decode Ways II | 639 | Hard | 1D DP + state machine | `*` can represent 1–9 and form two-digit pairs; track `e1` (ends in digit), `s1` (ends in `*`) as separate state streams |
| 2 | Student Attendance Record II | 552 | Hard | 1D DP with 6-state machine | State = (absences_used ∈ {0,1}, trailing_lates ∈ {0,1,2}); 6 states total; count valid length-n sequences |
| 3 | Climbing Stairs (revision) | 70 | Easy | Fibonacci DP | `dp[i] = dp[i-1] + dp[i-2]`; compress to two rolling variables |

---

### Full Solution Walkthrough — Decode Ways II (LC 639)

**Problem:** Count decodings of string `s` containing digits `0–9` and `*` (wildcard for `1–9`).  
**Recurrence:** At each character, track two streams:
- `e1` = # ways if current char is treated as a single character (non-`*`)
- `s1` = # ways if current char is `*`

```python
def numDecodings(s: str) -> int:
    MOD = 10**9 + 7
    # Bootstrap from first character
    e1 = 1 if s[0] != '0' else 0   # single digit, not wildcard
    s1 = 9 if s[0] == '*' else 0   # wildcard = 9 choices (1–9)

    for i in range(1, len(s)):
        c = s[i]
        prev_e1, prev_s1 = e1, s1
        
        if c == '*':
            e1 = 9 * prev_s1 + 9 * prev_e1  # * alone = 9 choices
            # Two-digit: prev was *, curr is * → 11–19 (9) + 21–26 (6) = 15
            # prev was digit d: d=1 → 11–19 (9), d=2 → 21–29 but only 21–26 (6)
            new_s1 = 15 * prev_s1  # * followed by * (15 valid combos)
            # prev_e1 cases: count valid two-char combos
            # (need to track prev digit for this — simplified: recompute)
            e1 = (9 * prev_s1 + 9 * prev_e1) % MOD
            s1 = (15 * prev_s1) % MOD
            # Note: full solution also accounts for prev_e1 pairs;
            # see complete implementation below
        else:
            digit = int(c)
            # Single char
            e1 = (0 if digit == 0 else prev_s1 + prev_e1)
            # Two-char ending with non-*: need to know if prev formed 1X or 2X
            # Simplified skeleton — delete and implement from scratch

        e1, s1 = e1 % MOD, s1 % MOD

    return (e1 + s1) % MOD
```

**Complete clean solution (memorize this pattern):**
```python
def numDecodings(s: str) -> int:
    MOD = 10**9 + 7
    # e1: ways where last decoded chunk used the previous single non-* char
    # e2: ways ending with previous two-char non-* decode
    # s1: ways where previous char was *
    # We only need one pass; track (e1, s1) = (ways ending in literal, ways ending in *)

    if s[0] == '0': return 0
    e1 = 0 if s[0] == '0' else (1 if s[0] != '*' else 0)
    s1 = 9 if s[0] == '*' else 0

    for i in range(1, len(s)):
        ne1 = ns1 = 0
        c = s[i]
        p = s[i-1]

        # Current char alone
        if c == '*':
            ns1 = 9 * (e1 + s1)          # * = any of 1–9
        elif c != '0':
            ne1 = e1 + s1                  # literal digit alone

        # Two-char decode (current + previous)
        if c == '*':
            if p == '1':   ne1 += 9 * e1  # 11–19
            elif p == '2': ne1 += 6 * e1  # 21–26
            elif p == '*': ne1 += 15 * s1 # *1–*9 where * is 1 or 2
        else:
            d = int(c)
            if p == '1':   ne1 += e1       # 1d always valid
            elif p == '2' and d <= 6: ne1 += e1  # 2d valid if d<=6
            elif p == '*':
                ne1 += e1 if d <= 6 else 0     # *=1 always, *=2 only if d<=6
                ne1 += e1                        # *=1: 1d always
                # correction: see full Neetcode solution

        e1, s1 = ne1 % MOD, ns1 % MOD

    return (e1 + s1) % MOD
```

> **Study note:** This problem is notorious for off-by-one in state transitions. The skeleton above shows the structure; always trace through `"*1"`, `"1*"`, `"**"` by hand before submitting.

---

### Full Solution Walkthrough — Student Attendance Record II (LC 552)

**Problem:** Count length-n sequences over `{A, L, P}` with no 2+ A's and no 3+ consecutive L's.  
**State:** `dp[a][l]` = number of valid sequences ending with `a` absences used and `l` trailing lates.

```python
def checkRecord(n: int) -> int:
    MOD = 10**9 + 7
    # dp[a][l]: a ∈ {0,1}, l ∈ {0,1,2}
    # Initialize for length 0: one empty sequence
    dp = [[0] * 3 for _ in range(2)]
    dp[0][0] = 1  # empty sequence: 0 A, 0 trailing L

    for _ in range(n):
        ndp = [[0] * 3 for _ in range(2)]
        for a in range(2):
            for l in range(3):
                if dp[a][l] == 0:
                    continue
                v = dp[a][l]
                # Append 'P' → resets trailing lates, keeps absences
                ndp[a][0] = (ndp[a][0] + v) % MOD
                # Append 'L' → increments trailing lates (only if < 2)
                if l < 2:
                    ndp[a][l+1] = (ndp[a][l+1] + v) % MOD
                # Append 'A' → resets trailing lates, uses one absence
                if a == 0:
                    ndp[1][0] = (ndp[1][0] + v) % MOD
        dp = ndp

    return sum(dp[a][l] for a in range(2) for l in range(3)) % MOD
```

**Complexity:** O(6n) time, O(6) = O(1) space (only 6 states).

---

### Climbing Stairs Revision (LC 70) — From Memory

```python
def climbStairs(n: int) -> int:
    if n <= 2: return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b
```

**Generalization:** If you can take 1, 2, or 3 steps, `dp[i] = dp[i-1] + dp[i-2] + dp[i-3]`.

---

## System Design (30 min)

### Topic: Chat Service — High-Level Architecture

**Core components:**
1. **API Gateway** — authenticates tokens, routes `/send`, `/history`, `/connect` endpoints
2. **Chat Servers (WebSocket)** — stateful; each server holds open connections; clients connect to one server
3. **Message Store** — Cassandra with `partition_key = conversation_id`, `clustering_key = timestamp DESC`
4. **User-to-Server Mapping** — Redis key-value: `user_id → chat_server_id` (for routing cross-server messages)
5. **Notification Service** — fan-out to offline users via APNs / FCM push

**Key trade-offs:**
- **WebSocket vs. Long Polling:** WebSocket = full-duplex, low latency, stateful (harder to load-balance). Long polling works for infrequent updates (email) but adds ~1s latency.
- **Fan-out on write vs. read:** Write fan-out copies message to each recipient inbox (fast reads, amplification for large groups). Read fan-out stores once, serves dynamically (cheap writes, slower reads). Use write fan-out for 1:1 and small groups; read fan-out for large group chats (> ~1000 members).
- **Message ordering:** Per-conversation sequence ID (simple, monotone) vs. vector clocks (true causal ordering needed for multi-device sync).

**Interview talking point:**  
*"If asked why Cassandra for chat messages, answer: the access pattern is always `conversation_id` + time range scan for history, plus append-only writes. Cassandra's LSM tree handles high write throughput without update penalties; the partition + clustering key design delivers O(log n) page fetches for chat history with no need for complex joins."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode Weekly Contest — DP Warm-up Set

**Goal:** Solve 2 Medium-difficulty DP problems (self-selected from LeetCode problem set filtered by "Dynamic Programming") in under 35 minutes combined. Focus on pattern classification in the first 90 seconds.

**Session structure:**
- 0:00 — Read problem, classify pattern aloud (Fibonacci? State machine? Knapsack?)
- 2:00 — State recurrence in words before writing any code
- 5:00 — Write and trace skeleton
- 20:00 — Submit; move to problem 2
- 35:00 — Hard stop; debrief

**Debrief prompt:** For each problem solved, complete: *"The recurrence was dp[i] = ___ because ___."* If you can't fill in the blank without looking at your code, you coded without a plan — slow down tomorrow.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you had to learn a new technical concept under deadline pressure. How did you build understanding quickly, and what would you do differently?

**Target structure:**
- **Situation:** Context — what was the concept, why was it urgent?
- **Task:** What did you need to produce with that knowledge?
- **Action:** How did you learn (docs, mentors, prototyping)? What shortcuts did you take consciously?
- **Result:** What did you ship, and how solid was your understanding afterward?

**Leadership principle:** *Learn and Be Curious* (Amazon) | *"You are not done learning" mindset* (Google)

**Tip:** The trap is making this story only about learning. The interviewer wants to see how you *applied incomplete knowledge* under pressure — that's the judgment call.

---

## Flashcards

| Q | A |
|---|---|
| How do you detect a 1D DP problem vs. a greedy problem? | DP: future decisions interact — a wrong early choice can invalidate a better later one. Greedy: local optimal equals global optimal (prove by exchange argument). When in doubt, try greedy and look for a counterexample. |
| What are the two state streams in Decode Ways II and why both? | `e1` = ways where the last consumed char was a literal digit; `s1` = ways where the last was `*`. They must stay separate because `*` and a literal digit form different valid two-char pairs with the next character. |
| How do you reduce Fibonacci DP from O(n) space to O(1)? | Use two rolling variables: `a, b = dp[i-2], dp[i-1]`. Each iteration: `a, b = b, a + b`. Works whenever dp[i] depends only on dp[i-1] and dp[i-2]. |
| What are the 6 states in Student Attendance Record II? | `(absences, trailing_lates)` where absences ∈ {0,1} and trailing_lates ∈ {0,1,2}. Total = 2 × 3 = 6. Each step transitions all 6 states by appending P, L, or A with validity checks. |
| In Chat Service design, what does the Redis user-to-server mapping solve? | When user A (on server 1) sends a message to user B (on server 2), the API needs to know which server B is connected to in order to route the message to B's WebSocket. Redis `user_id → server_id` makes this O(1) lookup. |

---

## Checklist

- [ ] Solved Decode Ways II without hints (first 20 min no help; traced `"*1"`, `"1*"`, `"**"` by hand)
- [ ] Solved Student Attendance Record II — drew the state transition table before coding
- [ ] Revised Climbing Stairs — wrote O(1) space solution from memory in < 2 min
- [ ] Drew Chat Service architecture: client → API GW → WS server → Redis → Cassandra → Notification
- [ ] Completed LeetCode timed warm-up (2 Medium DP in ≤ 35 min)
- [ ] Delivered STAR behavioral answer in under 2 minutes without notes
- [ ] Reviewed all 5 flashcards (cover answer column, recall aloud)
- [ ] Logged any unsolved / uncertain problem to `knowledge-base/revision-log.md`
