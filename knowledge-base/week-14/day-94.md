# Day 94 — 1D DP: Stock Trading State Machines
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master the stock trading DP family — problems where state tracks `(day, transactions_used, holding_stock)` — and understand how the state machine generalizes from 1 transaction to k transactions to infinity with cooldown and fees.

---

## DSA (2 hours)

### Pattern: 1D DP — State Machine with Transaction States

**Core idea:**  
Each day you're in one of several states: `(holding, not_holding)` × `(transactions_remaining)`. Transitions: buy (not_holding → holding, uses a transaction), sell (holding → not_holding), or rest (stay in same state). `dp[d][t][h]` = max profit on day `d`, with `t` transactions completed, holding stock `h`.

**Trigger condition:**  
- Stock buy/sell problem with a limit on transactions
- "Cooldown" or "fee" variants add one more dimension or modify the transition cost
- If unlimited transactions: reduce to simple forward scan (only add positive consecutive differences)

**Complexity:**  
Time: O(n × k) where k = max transactions | Space: O(k) with rolling variables (two arrays of size k)

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Best Time to Buy and Sell Stock III | 123 | Hard | DP with k=2 state machine | Hardcode 4 states: buy1, sell1, buy2, sell2; each day update in reverse order |
| 2 | Best Time to Buy and Sell Stock IV | 188 | Hard | DP with k transactions | If k ≥ n//2, unlimited transactions; otherwise `dp[k+1]` arrays |
| 3 | Best Time to Buy and Sell Stock (revision) | 121 | Easy | Track min so far | `max_profit = max(max_profit, price - min_price)`; one pass |

---

### Full Solution Walkthrough — Best Time to Buy and Sell Stock (LC 121) — Revision

```python
def maxProfit(prices: list[int]) -> int:
    min_price = float('inf')
    max_profit = 0
    for p in prices:
        min_price = min(min_price, p)
        max_profit = max(max_profit, p - min_price)
    return max_profit
```

---

### Full Solution Walkthrough — Stock III (LC 123) — At Most 2 Transactions — Hard

**State machine with 4 states (after each day):**
- `buy1`: lowest effective cost after buying once (= `-price` at best)
- `sell1`: max profit after completing 1 transaction
- `buy2`: max profit after buying a second time (= `sell1 - price` at best)
- `sell2`: max profit after completing 2 transactions

```python
def maxProfit(prices: list[int]) -> int:
    buy1 = buy2 = float('-inf')  # cost basis (negate price)
    sell1 = sell2 = 0

    for p in prices:
        buy1  = max(buy1,  -p)           # buy first stock: pay p
        sell1 = max(sell1, buy1 + p)     # sell first stock: gain p
        buy2  = max(buy2,  sell1 - p)    # buy second stock: pay p, credit sell1
        sell2 = max(sell2, buy2 + p)     # sell second stock: gain p

    return sell2
```

**Why no explicit transaction count needed:** Each state inherently encodes "how many buys have occurred." `buy1` ≤ first buy; `sell1` = after first complete; `buy2` starts after `sell1`; `sell2` = final.

**Trace on `[3,3,5,0,0,3,1,4]`:**

| Day | price | buy1 | sell1 | buy2 | sell2 |
|-----|-------|------|-------|------|-------|
| — | — | -∞ | 0 | -∞ | 0 |
| 0 | 3 | -3 | 0 | -3 | 0 |
| 1 | 3 | -3 | 0 | -3 | 0 |
| 2 | 5 | -3 | 2 | -1 | 2 |
| 3 | 0 | 0 | 2 | 2 | 2 |
| 4 | 0 | 0 | 2 | 2 | 2 |
| 5 | 3 | 0 | 3 | 5 | 5 |
| 6 | 1 | 0 | 3 | 5 | 5 |
| 7 | 4 | 0 | 4 | 5 | **6** |

Answer: 6. ✓

---

### Full Solution Walkthrough — Stock IV (LC 188) — At Most k Transactions — Hard

```python
def maxProfit(k: int, prices: list[int]) -> int:
    n = len(prices)
    if not prices or k == 0:
        return 0

    # If k >= n//2, unlimited transactions
    if k >= n // 2:
        return sum(max(0, prices[i+1] - prices[i]) for i in range(n-1))

    # buy[t] = max profit after t-th buy (0-indexed); sell[t] = after t-th sell
    buy  = [float('-inf')] * (k + 1)
    sell = [0] * (k + 1)

    for p in prices:
        for t in range(1, k + 1):
            buy[t]  = max(buy[t],  sell[t-1] - p)  # buy t-th stock
            sell[t] = max(sell[t], buy[t]    + p)  # sell t-th stock

    return sell[k]
```

**Complexity:** O(nk) time, O(k) space.

**Key insight for the unlimited case:** When `k ≥ n/2`, there can't be more than `n/2` non-overlapping transactions anyway, so "unlimited" is the same constraint. Simply capture every positive price movement.

---

### Extension Variants (Know These Patterns)

```python
# Stock with Cooldown (LC 309): after selling, must wait 1 day
def maxProfit_cooldown(prices):
    held = float('-inf')    # holding stock
    sold = 0                # just sold (on cooldown)
    rest = 0                # resting (not holding, not cooldown)
    for p in prices:
        held, sold, rest = max(held, rest - p), held + p, max(rest, sold)
    return max(sold, rest)

# Stock with Transaction Fee (LC 714): fee per transaction
def maxProfit_fee(prices, fee):
    hold = float('-inf')
    cash = 0
    for p in prices:
        hold = max(hold, cash - p)
        cash = max(cash, hold + p - fee)
    return cash
```

---

## System Design (30 min)

### Topic: Chat Service — Group Messaging & Fan-out Strategy

**Core components:**
1. **Group message store** — one record per message (not per recipient), stored in Cassandra
2. **Group membership table** — `(group_id, user_id, joined_at)` — used for fan-out
3. **Fan-out service** — when a group message arrives, look up all members, route to online members via WebSocket servers, queue offline members for push
4. **Online presence cache** — Redis: `user_id → (server_id, last_seen)` with short TTL (60s)
5. **Notification batch** — for large groups (1K+ members), fan-out happens asynchronously via a worker queue

**Key trade-offs:**
- **Synchronous vs. async fan-out:** Synchronous → sender gets delivery confirmation but response is slow for large groups. Async → fast sender ACK, eventual delivery. For group chats: async with delivery receipts sent separately.
- **Fan-out on write vs. read for groups:** Write fan-out (copy message to each inbox): O(n) writes per message, O(1) reads. Read fan-out: O(1) writes, O(n) reads on load. Use read fan-out for very large groups (> 1000 members) to avoid write amplification.
- **Message ordering in groups:** Use a per-group sequence counter (in Redis with atomic increment) to stamp messages. Clients render by sequence number.

**Interview talking point:**  
*"If asked how WhatsApp handles group messaging at scale, answer: store message once in Cassandra; fan-out asynchronously using a worker pool that reads the group member list and routes to each member's WebSocket server (for online members) or Notification Service (for offline). For groups > a threshold (e.g., 1000), switch to pull model — members poll for new messages using a `last_seen_sequence_id` cursor."*

---

## Assessment / Mock (1 hour)

### Activity: HackerRank OA Simulation — Stock DP Gauntlet

**Goal:** Reproduce all 5 stock variants from memory under time pressure.

**Session structure:**
- 0:00 — From memory, write LC 121 (1 transaction) in 3 min
- 3:00 — From memory, write LC 122 (unlimited) in 3 min
- 6:00 — From memory, write LC 123 (2 transactions) in 8 min
- 14:00 — From memory, write LC 309 (cooldown) in 8 min
- 22:00 — From memory, write LC 714 (fee) in 8 min
- 30:00 — Rest / review; check any that felt shaky
- 45:00 — Trace LC 123 on `[1,2,3,4,5]` (answer = 4, buy1 sell2) by hand
- 60:00 — Write the "unlimited case shortcut" for LC 188 and explain it

**Debrief prompt:** Which variant took longest? That's the one to drill tomorrow with 5 more test cases.

---

## Behavioral (30 min)

**STAR prompt:**  
Describe a situation where you identified a risk or technical debt that others weren't prioritizing. How did you advocate for addressing it?

**Target LP:** *Are Right, A Lot* — make good judgment calls; don't just follow consensus.

**Tip:** Be specific about the *evidence* you had. Vague "I noticed a potential issue" is weak. Strong: "Our error rate in this service was 0.3% and trending up — I modeled what that meant at 10× load and brought the projection to the team."

---

## Flashcards

| Q | A |
|---|---|
| What are the 4 DP states in Best Time to Buy and Sell Stock III? | `buy1` (best cost basis after 1st buy), `sell1` (max profit after 1st sell), `buy2` (max profit after 2nd buy = sell1 - price), `sell2` (final max profit after 2nd sell). Updated in this order each day. |
| Why does Stock IV short-circuit when k ≥ n//2? | Maximum distinct non-overlapping transactions in n days is n//2. So k ≥ n//2 is effectively unlimited. Use greedy: sum all positive consecutive differences. |
| Write the state transitions for Stock with Cooldown (LC 309). | `held = max(held, rest - p)` (buy if resting); `sold = held + p` (sell from held); `rest = max(rest, sold)` (enter rest from sold or stay). All three update simultaneously each day. |
| What is the general state machine template for stock problems? | States: (holding/not, transactions_remaining). Transitions: buy (not→hold, transactions−1), sell (hold→not), rest (stay). `dp[t][h]` where `t` = transactions used, `h` = 0/1 holding. |
| In Chat group design, when do you switch fan-out strategy from write to read? | Typically at group size > ~1000 members. Write fan-out costs O(members) writes per message — at large scale this causes write amplification. Read fan-out stores once; each member's client polls using a cursor (last seen sequence ID). |

---

## Checklist

- [ ] Wrote Stock I (LC 121) from memory in < 3 min
- [ ] Wrote Stock III (LC 123) state machine and traced the table above by hand
- [ ] Wrote Stock IV (LC 188) with the k ≥ n//2 shortcut
- [ ] Wrote Cooldown (LC 309) and Fee (LC 714) variants from memory
- [ ] Explained fan-out on write vs. read for group chats with a threshold
- [ ] Completed 1-hour OA simulation gauntlet with debrief
- [ ] Behavioral STAR delivered aloud with specific data point
- [ ] Reviewed all 5 flashcards
- [ ] Logged any uncertain variants to `knowledge-base/revision-log.md`
