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

```java
// Decode Ways II (LC 639) — skeleton showing state transition structure
class Solution {
    public int numDecodings(String s) {
        int MOD = 1_000_000_007;
        // Bootstrap from first character
        long e1 = s.charAt(0) != '0' ? (s.charAt(0) != '*' ? 1 : 0) : 0;
        long s1 = s.charAt(0) == '*' ? 9 : 0; // wildcard = 9 choices (1-9)

        for (int i = 1; i < s.length(); i++) {
            char c = s.charAt(i);
            char p = s.charAt(i - 1);
            long ne1 = 0, ns1 = 0;

            // Current char alone
            if (c == '*') {
                ns1 = 9 * (e1 + s1) % MOD; // * = any of 1-9
            } else if (c != '0') {
                ne1 = (e1 + s1) % MOD; // literal digit alone
            }

            // Two-char decode (current + previous)
            if (c == '*') {
                if (p == '1') ne1 = (ne1 + 9 * e1) % MOD;       // 11-19
                else if (p == '2') ne1 = (ne1 + 6 * e1) % MOD;  // 21-26
                else if (p == '*') ne1 = (ne1 + 15 * s1) % MOD; // *1-*9 where * is 1 or 2
            } else {
                int d = c - '0';
                if (p == '1') ne1 = (ne1 + e1) % MOD;            // 1d always valid
                else if (p == '2' && d <= 6) ne1 = (ne1 + e1) % MOD; // 2d valid if d<=6
                else if (p == '*') {
                    ne1 = (ne1 + e1) % MOD;                       // *=1: 1d always
                    if (d <= 6) ne1 = (ne1 + e1) % MOD;           // *=2: only if d<=6
                }
            }

            e1 = ne1;
            s1 = ns1;
        }

        return (int)((e1 + s1) % MOD);
    }
}
```

> **Study note:** This problem is notorious for off-by-one in state transitions. The skeleton above shows the structure; always trace through `"*1"`, `"1*"`, `"**"` by hand before submitting.

---

### Full Solution Walkthrough — Student Attendance Record II (LC 552)

**Problem:** Count length-n sequences over `{A, L, P}` with no 2+ A's and no 3+ consecutive L's.  
**State:** `dp[a][l]` = number of valid sequences ending with `a` absences used and `l` trailing lates.

```java
class Solution {
    public int checkRecord(int n) {
        int MOD = 1_000_000_007;
        // dp[a][l]: a in {0,1}, l in {0,1,2}
        // Initialize for length 0: one empty sequence
        long[][] dp = new long[2][3];
        dp[0][0] = 1; // empty sequence: 0 A, 0 trailing L

        for (int step = 0; step < n; step++) {
            long[][] ndp = new long[2][3];
            for (int a = 0; a < 2; a++) {
                for (int l = 0; l < 3; l++) {
                    if (dp[a][l] == 0) continue;
                    long v = dp[a][l];
                    // Append 'P' → resets trailing lates, keeps absences
                    ndp[a][0] = (ndp[a][0] + v) % MOD;
                    // Append 'L' → increments trailing lates (only if < 2)
                    if (l < 2) ndp[a][l + 1] = (ndp[a][l + 1] + v) % MOD;
                    // Append 'A' → resets trailing lates, uses one absence
                    if (a == 0) ndp[1][0] = (ndp[1][0] + v) % MOD;
                }
            }
            dp = ndp;
        }

        long total = 0;
        for (int a = 0; a < 2; a++)
            for (int l = 0; l < 3; l++)
                total = (total + dp[a][l]) % MOD;
        return (int) total;
    }
}
```

**Complexity:** O(6n) time, O(6) = O(1) space (only 6 states).

---

### Climbing Stairs Revision (LC 70) — From Memory

```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2) return n;
        int a = 1, b = 2;
        for (int i = 3; i <= n; i++) {
            int temp = a + b;
            a = b;
            b = temp;
        }
        return b;
    }
}
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
