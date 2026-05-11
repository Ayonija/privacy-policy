# Day 99 — 1D DP: Odd Even Jump & Miscellaneous Hard Patterns
**Week 15 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Tackle Odd Even Jump (LC 975) — a Hard problem combining monotonic stack, TreeMap-style ordered lookup, and dual-state DP — and use today to consolidate less common but high-signal DP patterns that appear in FAANG OA rounds.

---

## DSA (2 hours)

### Pattern: Dual-State 1D DP with Ordered Jump Targets

**Core idea:**  
Some problems require tracking two alternating states simultaneously (e.g., odd jump / even jump). Precompute "next reachable index for odd jumps" and "next reachable index for even jumps" using a sorted structure; then run a DP where `odd[i]` = can you reach the end starting from index `i` on an odd jump, `even[i]` similarly.

**Trigger condition:**  
- Alternating rules that depend on current "step parity"
- "Next element satisfying condition X in a sorted order" → sorted map + monotonic stack
- State = (position, parity) — extend the standard 1D DP by adding one binary dimension

**Complexity:**  
Preprocessing: O(n log n) | DP: O(n) | Space: O(n)

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Odd Even Jump | 975 | Hard | Monotonic stack + dual-state DP | Precompute `next_odd[i]` / `next_even[i]` with sorted map + decreasing stack; then DP from right to left |
| 2 | Strange Printer | 664 | Hard | Interval DP | `dp[i][j]` = min turns to print `s[i..j]`; if `s[k] == s[i]`, the first turn covers both |
| 3 | Perfect Squares (revision) | 279 | Medium | 1D DP (unbounded coin change variant) | `dp[i] = min(dp[i - k²] + 1)` for all perfect squares k² ≤ i |

---

### Full Solution Walkthrough — Perfect Squares (LC 279) — Revision

```python
def numSquares(n: int) -> int:
    squares = [i*i for i in range(1, int(n**0.5) + 1)]
    dp = [float('inf')] * (n + 1)
    dp[0] = 0
    for i in range(1, n + 1):
        for sq in squares:
            if sq > i: break
            dp[i] = min(dp[i], dp[i - sq] + 1)
    return dp[n]
```

**Insight:** Identical structure to Coin Change (LC 322), but coins = `{1, 4, 9, 16, ...}`.  
**Lagrange's four-square theorem:** Answer is always ≤ 4. If `n` is of the form `4^a(8b+7)`, answer = 4.

---

### Full Solution Walkthrough — Odd Even Jump (LC 975) — Hard

**Problem:** From index `i`, odd jump: go to the smallest `nums[j] >= nums[i]` with `j > i` (nearest valid, ties: smallest index). Even jump: go to the largest `nums[j] <= nums[i]` with `j > i` (nearest valid, ties: smallest index). Starting from any index with an odd jump, count how many starting indices can reach the end.

**Step 1: Precompute `next_odd[i]` and `next_even[i]`**  
Use a sorted structure (by value) processed right to left. For `next_odd`: iterate indices sorted by value ascending; for each, find the next-greater-or-equal index among those to the right (monotonic stack on indices).

```python
from sortedcontainers import SortedList

def oddEvenJumps(arr: list[int]) -> int:
    n = len(arr)

    def make_next(sorted_indices) -> list[int]:
        # sorted_indices: indices processed in some sorted order
        # For each, next[i] = the next index to the right in sorted order
        result = [None] * n
        stack = []   # monotonic decreasing stack of indices
        for i in sorted_indices:
            while stack and stack[-1] < i:
                result[stack.pop()] = i
            stack.append(i)
        return result

    # For odd jumps: sort by (value ASC, index ASC)
    sorted_odd = sorted(range(n), key=lambda i: (arr[i], i))
    next_odd = make_next(sorted_odd)

    # For even jumps: sort by (value DESC, index ASC)
    sorted_even = sorted(range(n), key=lambda i: (-arr[i], i))
    next_even = make_next(sorted_even)

    # DP from right to left
    # odd[i] = can reach end starting from i with an odd jump
    # even[i] = can reach end starting from i with an even jump
    odd = [False] * n
    even = [False] * n
    odd[-1] = even[-1] = True   # end is always reachable from itself

    for i in range(n - 2, -1, -1):
        if next_odd[i] is not None:
            odd[i] = even[next_odd[i]]   # odd jump lands → next move is even
        if next_even[i] is not None:
            even[i] = odd[next_even[i]]  # even jump lands → next move is odd

    return sum(odd)  # count starts where an odd jump (first jump) can reach end
```

**Why `even[next_odd[i]]`?** After making an odd jump to index `j`, the *next* jump from `j` must be even. So reachability from `i` via odd = reachability from `j` via even = `even[j]`.

**Complexity:** O(n log n) for sorting + O(n) for DP.

**Key insight to state in interview:**  
"I decompose the problem: first precompute where each index jumps to under odd/even rules using a sorted structure + monotonic stack. Then the DP is straightforward — two boolean arrays, one pass right-to-left."

---

### Full Solution Walkthrough — Strange Printer (LC 664) — Hard

**Problem:** Printer prints a string of the same character in one turn, can overwrite. Minimum turns to print string `s`.

**Interval DP:** `dp[i][j]` = minimum turns to print `s[i..j]`.

```python
def strangePrinter(s: str) -> int:
    n = len(s)
    # dp[i][j] = min turns to print s[i..j]
    dp = [[0] * n for _ in range(n)]

    # Base: single characters
    for i in range(n):
        dp[i][i] = 1

    # Fill by increasing length
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = dp[i][j-1] + 1   # worst case: print s[j] separately

            # Optimization: if any s[k] == s[j] for k in [i, j-1],
            # we can extend the turn that prints s[k] to also cover s[j]
            for k in range(i, j):
                if s[k] == s[j]:
                    # Printing s[k..j] where s[k]==s[j]: the turn printing s[k]
                    # also covers s[j], so we only need to handle s[k+1..j-1]
                    candidate = dp[i][k] + (dp[k+1][j-1] if k+1 <= j-1 else 0)
                    dp[i][j] = min(dp[i][j], candidate)

    return dp[0][n-1]
```

**Intuition:** If `s[k] == s[j]`, the printer can print from `k` to `j` in one stroke, and we only need extra turns for the middle `s[k+1..j-1]`. So we "merge" the two same-character positions.

**Complexity:** O(n³) — standard interval DP.

---

## System Design (30 min)

### Topic: Chat + Notification Systems Integration — Full End-to-End Flow

**Complete message flow (1:1 chat):**
1. Sender's app → WebSocket → Chat Server A → message stored in Cassandra
2. Chat Server A → publishes to Redis Pub/Sub channel `user_B`
3. Chat Server B (subscribed to `user_B`) → pushes via WebSocket to Receiver B's device
4. If B is offline → Chat Server checks Redis → user_B not connected → publishes to Notification Service via Kafka
5. Notification Service → looks up B's device tokens → sends APNs/FCM push notification
6. B opens app → WebSocket reconnects → fetches missed messages from Cassandra using `last_seen_message_id` cursor
7. B's client sends READ receipt → Chat Server → notifies A → double blue tick

**Failure scenarios and handling:**
| Failure | Detection | Recovery |
|---------|-----------|----------|
| Sender WebSocket drops mid-send | Client gets no ACK in 5s | Client retries with same idempotency key |
| Chat Server crashes | Redis user-server mapping becomes stale | Reconnect logic removes stale entry; client reconnects to any available server |
| APNs/FCM returns 5xx | Notification worker catches exception | Re-enqueue with exponential backoff; max 3 retries; then in-app inbox fallback |
| Message store unavailable | Write fails | Return error to client; client shows "send failed"; user retries |

**Interview talking point:**  
*"If asked to design the complete flow from 'user sends message' to 'recipient reads it,' walk through these 7 steps. Mention the two key storage systems: Cassandra for messages (persistent, queryable) and Redis for ephemeral routing state (user-to-server, presence, dedup keys). The separation of concerns — persistent vs. ephemeral — is what makes the system maintainable."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode Contest Simulation — Hard DP Problem Cold-Start

**Goal:** Attempt Strange Printer (LC 664) from a cold start — no notes, no review — under strict interview conditions.

**Session structure:**
- 0:00 — Read problem; classify: "This is interval DP — I need `dp[i][j]`."
- 2:00 — Write recurrence in plain English before coding
- 5:00 — Code; start with base cases (single chars), then length ≥ 2
- 30:00 — If not passing: trace `dp` table for `s="aaab"` on paper
- 40:00 — Attempt Odd Even Jump (LC 975) explanation only — describe the algorithm verbally (no code needed today)
- 55:00 — Debrief: which part of interval DP caused the most confusion?

**Debrief prompt:** In Strange Printer, can you explain *why* `dp[i][j] = dp[i][k] + dp[k+1][j-1]` when `s[k] == s[j]`? The `dp[k+1][j-1]` is the cost of the "middle" that the shared turn (k to j) must overwrite. Write this in your own words.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you had to work with a difficult team member or stakeholder. How did you handle the relationship?

**Target LP:** *Earn Trust* (Amazon) — build trust through transparency, follow-through, and good intent.

**Tip:** Don't make the other person the villain. The strongest answers focus on what *you* did to improve the dynamic, not on cataloging the other person's faults.

---

## Flashcards

| Q | A |
|---|---|
| Describe the two-step approach to Odd Even Jump (LC 975). | Step 1: Precompute `next_odd[i]` and `next_even[i]` — for each index, where does it jump under odd/even rules? Use sorted order + monotonic stack. Step 2: DP from right to left — `odd[i] = even[next_odd[i]]`; `even[i] = odd[next_even[i]]`. Count indices where `odd[i] = True`. |
| Why is the make_next function for Odd Even Jump called with different sort orders? | `next_odd`: sort indices by `(value ASC, index ASC)` — odd jump goes to smallest value ≥ current. `next_even`: sort by `(value DESC, index ASC)` — even jump goes to largest value ≤ current. In each order, a monotonic stack finds the next index to the right for each position. |
| Write the Strange Printer base recurrence for a single character and for length ≥ 2. | Single char: `dp[i][i] = 1`. Length ≥ 2: `dp[i][j] = dp[i][j-1] + 1` (baseline: print s[j] separately). Optimize: for each `k ∈ [i,j-1]` where `s[k] == s[j]`: `dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j-1])`. |
| In Strange Printer, what does it mean that s[k] == s[j] saves a turn? | The printer stroke that prints s[k] can extend all the way to s[j] in one turn (same character). The cost is: turns to print s[i..k] (which sets up s[k]) + turns to print and overwrite s[k+1..j-1] (the middle). The s[j] character is "free" — covered by the k-to-j stroke. |
| In the integrated Chat + Notification design, why is Redis used for routing but Cassandra for messages? | Redis: ephemeral, low-latency key-value lookups (user-to-server mapping, presence, dedup keys); data can be rebuilt on failure. Cassandra: durable, high-write-throughput, time-ordered messages that must survive crashes and be queryable by conversation + time range. |

---

## Checklist

- [ ] Perfect Squares (LC 279): wrote from memory, traced dp[0..12]
- [ ] Odd Even Jump (LC 975): traced the full algorithm on `[10,13,12,14,15]` (expected: 2)
- [ ] Strange Printer (LC 664): traced `dp` table for `s="aaab"` by hand; answer = 2
- [ ] Drew the complete 7-step message flow (send → read receipt) without notes
- [ ] Completed 1-hour cold-start contest simulation with debrief written
- [ ] Behavioral STAR focused on own actions, not other person's behavior
- [ ] Reviewed all 5 flashcards
- [ ] Logged uncertain problems to `knowledge-base/revision-log.md`
