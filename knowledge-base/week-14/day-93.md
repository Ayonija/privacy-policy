# Day 93 — 1D DP: Jump Game Pattern & DP on Sequences
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master jump game DP variants — problems where you compute the minimum number of steps or the feasibility of reaching a target — and connect the pattern to general DP-on-index problems where "can I reach position i?" is the core question.

---

## DSA (2 hours)

### Pattern: 1D DP — Reachability & Minimum Steps

**Core idea:**  
`dp[i]` = minimum jumps (or feasibility) to reach index `i`. Transition: for every index `j` where `j + nums[j] >= i` and `dp[j]` is valid, `dp[i] = min(dp[i], dp[j] + 1)`. Many jump game variants can be solved with greedy if the greedy exchange argument holds — but DP is the safe fallback.

**Trigger condition:**  
- "Can you reach the end?" or "Minimum steps to reach position N"
- Each position has a "reach" value (`nums[i]` = max jump length from `i`)
- When values per position define which future positions are reachable

**Complexity:**  
Naive DP: O(n²) | Greedy optimization: O(n) | Space: O(n) DP, O(1) greedy

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Jump Game VI | 1696 | Medium | DP + Monotonic Deque | `dp[i] = nums[i] + max(dp[i-k..i-1])`; sliding window max with deque keeps O(n) |
| 2 | Jump Game II | 45 | Medium | Greedy (or DP) | Track current farthest reachable; when you reach current boundary, increment jumps |
| 3 | Jump Game (revision) | 55 | Medium | Greedy feasibility | Track `max_reach`; if `i > max_reach` at any point, return False |

---

### Full Solution Walkthrough — Jump Game (LC 55) — Revision

```python
def canJump(nums: list[int]) -> bool:
    max_reach = 0
    for i, n in enumerate(nums):
        if i > max_reach:
            return False      # can't reach this position
        max_reach = max(max_reach, i + n)
    return True
```

**Trace on `[2,3,1,1,4]`:** max_reach: 0→2→4→4→5→5. Never blocked. → True  
**Trace on `[3,2,1,0,4]`:** max_reach: 0→3→4→4→4. At i=4: max_reach=4, stuck. → False

---

### Full Solution Walkthrough — Jump Game II (LC 45)

**Goal:** Minimum number of jumps to reach the last index. Greedy: at each jump, extend as far as possible.

```python
def jump(nums: list[int]) -> int:
    jumps = 0
    current_end = 0   # boundary of current jump range
    farthest = 0      # farthest we can reach from current range

    for i in range(len(nums) - 1):  # don't need to jump from last
        farthest = max(farthest, i + nums[i])
        if i == current_end:   # exhausted current jump's reach
            jumps += 1
            current_end = farthest
            if current_end >= len(nums) - 1:
                break

    return jumps
```

**Why greedy works:** At each "boundary crossing," we must use one jump. Among all positions reachable in the current jump, we choose the one reaching farthest — any other choice is suboptimal (exchange argument).

**Alternative DP for understanding:**
```python
def jump_dp(nums: list[int]) -> int:
    n = len(nums)
    dp = [float('inf')] * n
    dp[0] = 0
    for i in range(n):
        if dp[i] == float('inf'): continue
        for j in range(i+1, min(n, i + nums[i] + 1)):
            dp[j] = min(dp[j], dp[i] + 1)
    return dp[-1]
# O(n²) — use greedy O(n) in interview, but DP reveals the structure
```

---

### Full Solution Walkthrough — Jump Game VI (LC 1696) — Key Problem

**Problem:** Array `nums`; jump between 1 and k positions forward. Maximize the score (sum of visited elements including start and end).

**Recurrence:** `dp[i] = nums[i] + max(dp[i-k], ..., dp[i-1])`

**Naive:** O(nk) — TLE for large k.  
**Optimization:** Sliding window maximum with a monotonic deque.

```python
from collections import deque

def maxResult(nums: list[int], k: int) -> int:
    n = len(nums)
    dp = [float('-inf')] * n
    dp[0] = nums[0]
    dq = deque([0])   # stores indices; front = index of max dp value in window

    for i in range(1, n):
        # Remove indices outside the window [i-k, i-1]
        while dq and dq[0] < i - k:
            dq.popleft()

        # dp[i] = nums[i] + max dp in window
        dp[i] = nums[i] + dp[dq[0]]

        # Maintain decreasing deque (remove smaller dp values from back)
        while dq and dp[dq[-1]] <= dp[i]:
            dq.pop()
        dq.append(i)

    return dp[-1]
```

**Complexity:** O(n) time, O(k) space.

**Deque invariant:** Front = index with the maximum dp value in the window. Back = most recently added. Always maintain decreasing `dp` values front to back.

**Pattern recognition:** *Any time you see "max/min of a sliding window of size k applied in a DP recurrence," reach for a monotonic deque.*

---

### Side-by-Side: Jump Game Pattern Variants

| Problem | Key Question | Approach | Complexity |
|---------|-------------|----------|------------|
| LC 55 — Jump Game | Can you reach end? | Greedy max_reach | O(n) |
| LC 45 — Jump Game II | Min jumps to end? | Greedy boundary | O(n) |
| LC 1696 — Jump Game VI | Max score jumping 1–k | DP + Monotonic Deque | O(n) |
| LC 1871 — Jump Game VII | Can reach end jumping within [min,max]? | BFS / prefix sum | O(n) |

---

## System Design (30 min)

### Topic: Chat Service — Real-Time Delivery (WebSocket Architecture)

**Core components:**
1. **WebSocket servers** — stateful; one persistent TCP connection per client; each server handles ~10K–100K connections
2. **Heartbeat / Ping-Pong** — client sends ping every 30s; server closes connection if no ping within 60s
3. **Horizontal scaling issue** — user A (server 1) sends to user B (server 2); need cross-server routing
4. **Message routing layer** — Redis Pub/Sub: when server 1 receives a message for user B, it publishes to channel `user_B`; server 2 (subscribed to `user_B`) delivers it over B's open WebSocket
5. **Offline users** — if user not connected, message goes to Notification Service (APNs/FCM push)

**Key trade-offs:**
- **Long polling vs. WebSocket:** Long polling: client polls every ~1s, get response when message arrives. WebSocket: single persistent connection, server pushes. WebSocket is ~10× more efficient for active users.
- **Redis Pub/Sub vs. Message Queue for routing:** Pub/Sub is fast (no durability needed for in-flight routing). If server crashes mid-delivery, message already persisted in Cassandra — client can fetch on reconnect.
- **Connection sticky routing:** Use consistent hashing or a Redis lookup to always route a user to the same server (reduces cross-server pub/sub messages).

**Interview talking point:**  
*"If asked how to handle 1 million concurrent WebSocket connections, answer: horizontal scale the WebSocket servers (each handles ~50K connections). Use Redis Pub/Sub for cross-server delivery routing. The bottleneck is typically memory per connection (~50KB), not CPU. At 50K connections per server, 20 servers handle 1M connections with headroom."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode — Sliding Window + DP Combination Set

**Goal:** Practice the deque-optimized DP pattern on one new problem. Try LC 239 (Sliding Window Maximum) first if unfamiliar with the deque, then return to Jump Game VI.

**Session structure:**
- 0:00 — LC 239 Sliding Window Maximum (Medium): implement monotonic deque (15 min)
- 15:00 — LC 1696 Jump Game VI (Medium): apply deque insight to DP (25 min)
- 40:00 — Review: trace through the deque state at each step on a small example
- 55:00 — Write the deque invariant in one sentence from memory

**Debrief prompt:** Could you explain why the deque front always holds the maximum? If not, re-trace the deque state update logic step-by-step before tomorrow.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you proposed a simpler solution when your team was over-engineering. How did you communicate the trade-off?

**Target LP:** *Frugality* (Amazon) — accomplish more with less; simplicity is a feature.

**Full STAR Story — "O(n) Greedy vs. O(n²) DP for Jump Game Routing":**
**S (20%):** "At RouteOptCo, the team was building a path-feasibility checker using an O(n²) DP that was taking 800ms for 10K-node graphs — 4× over the 200ms SLA."
**T:** "I proposed replacing the DP with an O(n) greedy max-reach scan, claiming it would bring latency under 50ms. I needed to convince two skeptical seniors."
**A (60% — 'I' not 'we'):** "(1) I proved the greedy exchange argument in writing: at each index, expanding max_reach is always at least as good as any alternative choice — no future decision depends on which path led to the current index. (2) I benchmarked both on 100K-node synthetic graphs: O(n) greedy ran in 12ms vs. 8.2 seconds for O(n²) DP. (3) I acknowledged the DP's advantage — it can reconstruct the exact path — and proposed adding optional path reconstruction as a separate O(n) back-trace pass only when explicitly needed. (4) I ran the greedy solution through all existing test cases and added 5 new edge cases for zero-length jumps and stranded positions."
**R (20%):** "Team adopted the greedy solution. P99 latency dropped from 800ms to 18ms. The optional path reconstruction was never needed in production — saving the team from building unnecessary complexity."
*Works for: Frugality, Invent and Simplify, Are Right A Lot.*

### STAR Interview Framework

> **Jump Game VI deque-optimized DP:** naive O(nk) → monotonic deque DP O(n) time, O(k) space

**S:** "Array nums; jump 1 to k positions forward, maximize score. Naive O(nk): for each position check all k prior positions for max dp value."
**T:** "Need O(n) by maintaining a sliding window maximum over the prior k dp values using a monotonic deque."
**A (60%):**
1. *Classify:* "Max score with sliding window recurrence → monotonic deque DP."
2. *Init:* "dp[0]=nums[0]; deque holding index 0."
3. *Loop/Recurrence:* "For i 1 to n-1: remove indices outside window [i-k, i-1] from front. dp[i]=nums[i]+dp[deque.front]. Remove from back all indices with dp ≤ dp[i]. Append i."
4. *Termination:* "Return dp[n-1]."
5. *Gotcha:* "Pop from deque BACK before appending — maintain decreasing dp values front-to-back. Forgetting this makes the deque give wrong max values."
**R:** "O(n) time, O(k) space. Pattern: 'max/min of sliding window in a DP recurrence' → always reach for monotonic deque."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Segment tree | Non-sliding window range max | Overkill; deque achieves same O(n) for sliding window |
| Sparse table | Static array range max | DP array is built dynamically — can't precompute |

---

## Flashcards

| Q | A |
|---|---|
| How does Jump Game (LC 55) track reachability in O(n)? | Maintain `max_reach`. At each index `i`: if `i > max_reach`, return False. Otherwise, `max_reach = max(max_reach, i + nums[i])`. Return True at the end. |
| What is the greedy insight in Jump Game II (LC 45)? | Track the farthest position reachable from all positions in the current "jump." When you exhaust the current jump's boundary, increment jump count and set new boundary = farthest. Greedy: always extend as far as possible per jump. |
| Write the recurrence for Jump Game VI. | `dp[i] = nums[i] + max(dp[max(0, i-k) .. i-1])`. The sliding window max is maintained by a monotonic deque of indices with decreasing dp values. |
| What is the monotonic deque invariant in Jump Game VI? | Front = index of maximum dp value in the window `[i-k, i-1]`. Values in deque (by dp) are non-increasing front to back. When adding `dp[i]`: pop from back all indices with smaller dp values (they'll never be the window max again). |
| In Chat Service, why use Redis Pub/Sub for WebSocket routing instead of a message queue? | Routing is ephemeral — we just need to fan a message to the right server fast. Pub/Sub has microsecond latency and no storage overhead. Durability is handled separately by Cassandra. Using a persistent queue here would add unnecessary write amplification. |

---

## Checklist

- [ ] Jump Game (LC 55): wrote greedy solution from memory, traced `[3,2,1,0,4]` by hand
- [ ] Jump Game II (LC 45): explained greedy boundary insight before coding
- [ ] Jump Game VI (LC 1696): implemented deque-optimized DP; traced deque state on `[1,-1,-2,4,-7,3]`, k=2
- [ ] Filled in the "Side-by-Side" table from memory (closed this file, recalled variants)
- [ ] Sketched WebSocket routing diagram: client → WS Server → Redis Pub/Sub → peer server → peer client
- [ ] Completed 1-hour assessment session (LC 239 + LC 1696)
- [ ] Behavioral answer delivered aloud, timed
- [ ] Reviewed all 5 flashcards
- [ ] Logged any unsolved problems to `knowledge-base/revision-log.md`
