# Day 97 — 1D DP: Coin Change, Unbounded Knapsack & Interval DP
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master the Coin Change / Unbounded Knapsack pattern — "how many ways / minimum cost to fill a target using unlimited copies of items" — then tackle two Hard problems where the same recurrence structure appears in disguise (refueling stops, cutting a stick).

---

## DSA (2 hours)

### Pattern: 1D DP — Unbounded Knapsack & Target-Fill

**Core idea:**  
`dp[i]` = minimum coins (or number of ways) to make amount `i`. For minimum: `dp[i] = min(dp[i - coin] + 1)` for each coin ≤ i. For ways: `dp[i] += dp[i - coin]`. Fill left to right; each coin can be reused (unbounded). Process coins in outer loop (ways), amounts in inner loop — or amounts outer, coins inner for minimum.

**Trigger condition:**  
- "Minimum number of elements from a list to sum to target" (min coins)
- "Number of ways to sum to target with repetition allowed" (coin change II)
- "Partition target using items with costs" — any unbounded reuse problem

**Complexity:**  
Time: O(amount × len(coins)) | Space: O(amount)

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Number of Refueling Stops | 871 | Hard | Greedy + Max Heap (or DP) | At each stop, decide if you needed fuel retroactively — use a max-heap to pick the largest past station |
| 2 | Minimum Cost to Cut a Stick | 1547 | Hard | Interval DP (not Knapsack) | `dp[i][j]` = min cost to make all cuts between positions i and j; try each cut as the "last cut" |
| 3 | Coin Change (revision) | 322 | Medium | 1D Unbounded Knapsack | `dp[0]=0, dp[i]=min(dp[i-c]+1)` for each coin c ≤ i |

---

### Full Solution Walkthrough — Coin Change (LC 322) — Revision

```python
def coinChange(coins: list[int], amount: int) -> int:
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for i in range(1, amount + 1):
        for c in coins:
            if c <= i:
                dp[i] = min(dp[i], dp[i - c] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1
```

**Trace for `coins=[1,2,5], amount=11`:**  
`dp[0]=0, dp[1]=1, dp[2]=1, dp[3]=2, dp[4]=2, dp[5]=1, dp[6]=2, dp[7]=2, dp[8]=3, dp[9]=3, dp[10]=2, dp[11]=3`  
Answer: 3 (5+5+1) ✓

**Coin Change II — Number of Ways (LC 518):**
```python
def change(amount: int, coins: list[int]) -> int:
    dp = [0] * (amount + 1)
    dp[0] = 1   # one way to make 0: use nothing

    for c in coins:             # outer loop = coins (ensures no duplicates)
        for i in range(c, amount + 1):
            dp[i] += dp[i - c]

    return dp[amount]
```

**Key difference:** Coin Change II iterates coins in the outer loop to avoid counting permutations (e.g., [1,2] and [2,1] would both be counted if amounts were outer). Order matters!

---

### Full Solution Walkthrough — Minimum Number of Refueling Stops (LC 871) — Hard

**Problem:** Car starts with `startFuel`, travels distance `target`. Stations at positions `stations[i][0]` with fuel `stations[i][1]`. Minimum number of refuels to reach target.

**Greedy insight:** At any point where you run out of fuel, retroactively pick the largest fuel station you've passed. This minimizes the number of stops.

```python
import heapq

def minRefuelStops(target: int, startFuel: int, stations: list[list[int]]) -> int:
    heap = []          # max-heap (negate for min-heap)
    fuel = startFuel
    stops = 0
    prev_pos = 0

    for pos, station_fuel in stations + [[target, 0]]:
        fuel -= (pos - prev_pos)    # consume fuel to reach this station
        prev_pos = pos

        # While we can't reach this station, retroactively refuel at best past station
        while fuel < 0:
            if not heap:
                return -1           # impossible
            fuel += -heapq.heappop(heap)  # add largest past fuel
            stops += 1

        heapq.heappush(heap, -station_fuel)   # record this station as an option

    return stops
```

**Why greedy works:** Choosing the largest available past station maximizes fuel gained per stop — any other choice is dominated (exchange argument). 

**Alternative DP approach (O(n²)):**
```python
def minRefuelStops_dp(target, startFuel, stations):
    n = len(stations)
    # dp[i] = max distance reachable with exactly i stops
    dp = [0] * (n + 1)
    dp[0] = startFuel

    for i, (pos, fuel) in enumerate(stations):
        # Process in reverse: don't let station i use itself
        for t in range(i, -1, -1):
            if dp[t] >= pos:   # can reach station i with t stops
                dp[t+1] = max(dp[t+1], dp[t] + fuel)

    for i, d in enumerate(dp):
        if d >= target:
            return i
    return -1
```

---

### Full Solution Walkthrough — Minimum Cost to Cut a Stick (LC 1547) — Hard

**Problem:** Stick of length n; make cuts at given positions. Cost of a cut = current stick length. Find minimum total cost to make all cuts.

**Interval DP:** Add sentinels 0 and n to cuts array. `dp[i][j]` = min cost to make all cuts between `cuts[i]` and `cuts[j]`.

```python
def minCost(n: int, cuts: list[int]) -> int:
    cuts = sorted([0] + cuts + [n])
    m = len(cuts)
    # dp[i][j] = min cost to make all cuts between cuts[i] and cuts[j]
    dp = [[0] * m for _ in range(m)]

    for length in range(2, m):          # length of interval in cut-index space
        for i in range(m - length):
            j = i + length
            dp[i][j] = float('inf')
            for k in range(i + 1, j):   # try each cut as the "last" cut made
                cost = cuts[j] - cuts[i]  # current stick length when this cut is made
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cost)

    return dp[0][m - 1]
```

**Why "last cut" framing?** In interval DP, we think of making the last cut `k` that splits `[i, j]` into `[i, k]` and `[k, j]`. The cost of that cut = `cuts[j] - cuts[i]` (the full stick at that point). Both sub-intervals are solved recursively.

**Complexity:** O(m³) where m = len(cuts) + 2.

---

## System Design (30 min)

### Topic: Notification System — Push Notification Deep Dive (APNs & FCM)

**Core components:**
1. **APNs (Apple Push Notification Service):** HTTP/2 provider API; up to 5000 notifications per connection; JWT auth per team; payload max 4KB; requires device token from iOS SDK
2. **FCM (Firebase Cloud Messaging):** HTTP v1 API; per-app service account auth; payload max 4KB (data) + notification payload; supports topics (publish-subscribe) and multicast (up to 500 tokens)
3. **Notification payload:** `{title, body, badge, sound, data (custom key-value), TTL, priority}`
4. **Token management:** On app launch → send fresh token to your backend; on `devicecheck` failure or `InvalidRegistration` response → delete token
5. **Priority:** High priority = wakes device immediately (battery cost); Normal = delivered when convenient. Use high priority for messages, normal for badge updates.

**Key trade-offs:**
- **Topic subscriptions (FCM) vs. multicast vs. per-token:** Topics = no backend token management, but no targeting control. Multicast = batch up to 500 tokens, full control. Per-token = full control but O(n) API calls for large audiences.
- **Notification vs. Data message:** Notification = system tray displayed by OS (simpler). Data = silent, app handles display (custom UI, dedup logic, grouping). Use data messages for chat to allow custom threading.
- **Delivery guarantee:** APNs/FCM are best-effort — no guarantee of delivery. Add in-app inbox as fallback (user fetches missed notifications on open).

**Interview talking point:**  
*"If asked how to handle notification delivery to users with multiple devices, answer: the Device Token Registry stores one row per (user_id, device_id). On notification send, fan-out to all rows for that user_id. Use FCM multicast (500 tokens per call) to batch sends. On `Unregistered` response for any token, delete that row. Users expect all devices to ring — per-device send is the only way to achieve this."*

---

## Assessment / Mock (1 hour)

### Activity: HackerRank OA Simulation — Greedy + DP Problem

**Goal:** Solve Minimum Refueling Stops (LC 871) in 35 min; use the greedy heap approach.

**Session structure:**
- 0:00 — Read LC 871; write pseudocode for the greedy heap approach (no code yet)
- 5:00 — Code the heap solution; trace on `target=100, startFuel=10, stations=[[10,60],[20,30],[30,30],[60,40]]`
- 25:00 — Submit; if wrong, trace fuel state at each station
- 30:00 — 5-min break; then attempt Minimum Cost to Cut a Stick (LC 1547) for remaining 30 min
- 60:00 — Debrief: which part of Interval DP took longest to reason about?

**Debrief prompt:** In LC 1547, could you explain *why* the cost at each cut is the full interval length `cuts[j] - cuts[i]`? If not, re-read the problem statement and re-derive it before tomorrow.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you disagreed with a technical decision made by someone more senior. What did you do?

**Target LP:** *Have Backbone; Disagree and Commit* (Amazon) — voice disagreement professionally; then fully commit once decided.

**Full STAR Story — "Greedy Heap vs. DP for Refueling Cost Optimization":**
**S (20%):** "At LogisticsCo, the tech lead proposed an O(n²) DP solution for route fuel optimization. I believed an O(n log n) greedy heap approach was correct and worth the debate — the DP was failing on large routes (20K stations, 3-second timeout)."
**T:** "I needed to make my case respectfully, get a decision, and commit to whichever approach was chosen."
**A (60% — 'I' not 'we'):** "(1) I formalized the greedy correctness argument in writing: when you run out of fuel, picking the largest past station is always at least as good as any other choice — provable by exchange argument. (2) I presented this to the tech lead with a complexity comparison and benchmarks: O(n log n) greedy ran 20K stations in 18ms vs. O(n²) DP in 3.1 seconds. (3) I acknowledged the DP's advantage — it naturally handles edge cases and is easier to reason about — and offered to write exhaustive edge case tests for both. (4) The tech lead chose the greedy approach; I wrote the full test suite covering empty heap scenarios, unreachable targets, and all-equal-fuel stations."
**R (20%):** "Greedy solution shipped, passing all 43 test cases. Route optimization latency dropped from 3.1s to 18ms. The exchange argument writeup became a team reference for future greedy vs. DP debates."
*Works for: Have Backbone Disagree and Commit, Are Right A Lot, Earn Trust.*

### STAR Interview Framework

> **Interval DP (Minimum Cost to Cut a Stick):** naive O(m! × m) → interval DP O(m³) time, O(m²) space

**S:** "Stick of length n, make cuts at given positions. Cost of each cut = current stick length. Naive: try all cut orderings O(m! × m) is infeasible."
**T:** "Need O(m³) interval DP by thinking of the 'last cut' in each subinterval as the choice variable."
**A (60%):**
1. *Classify:* "Optimal cost for cuts on a 1D range → interval DP."
2. *Init:* "Add sentinels: cuts = sorted([0] + cuts + [n]). dp[i][j] = 0 for adjacent cuts."
3. *Loop/Recurrence:* "For length 2..m: for i in 0..m-length: j=i+length. dp[i][j]=min(dp[i][k]+dp[k][j]+(cuts[j]-cuts[i])) for k in (i,j)."
4. *Termination:* "Return dp[0][m-1]."
5. *Gotcha:* "The cost of a cut is cuts[j]-cuts[i] — the CURRENT stick length at the time of that cut (the enclosing interval), not the original n. Forgetting the enclosing length is the most common bug."
**R:** "O(m³) time, O(m²) space where m = len(cuts)+2. 'Last cut' framing: splitting [i,j] at k costs cuts[j]-cuts[i] plus the cost of both sub-intervals."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Greedy (cut smallest first) | Uniform cost | Non-uniform stick lengths make greedy suboptimal |
| Memoized top-down | Deep recursion with repeated states | Both work; bottom-up by length avoids stack overflow |

---

## Flashcards

| Q | A |
|---|---|
| Write the Coin Change DP recurrence and initialization. | `dp[0] = 0`. For `i` in `1..amount`: `dp[i] = min(dp[i-c] + 1)` for each `coin c ≤ i`. Initialize all `dp[i] = inf`. Return `dp[amount]` if finite, else -1. |
| Why does Coin Change II put coins in the outer loop? | To count combinations (unordered): each coin is considered once. If amounts were outer, you'd count [1,2] and [2,1] as two different ways — that's permutations. Coins outer → each combination counted exactly once. |
| What is the greedy insight for Minimum Refueling Stops? | When you run out of fuel, retroactively refuel at the largest station you've passed. A max-heap of passed station fuels lets you pick the optimal station in O(log n) per stop. |
| Write the interval DP recurrence for Minimum Cost to Cut a Stick. | Add sentinels 0 and n. `dp[i][j]` = min cost for all cuts between `cuts[i]` and `cuts[j]`. `dp[i][j] = min(dp[i][k] + dp[k][j] + cuts[j] - cuts[i])` for each cut `k` in `(i,j)`. Fill by increasing interval length. |
| In APNs/FCM, when should you use a data message vs. a notification message? | Data message: silent delivery; app processes the payload and displays custom UI. Notification message: OS displays automatically. Use data for chat (custom threading, dedup, badge logic). Use notification for simple alerts where OS display is acceptable. |

---

## Checklist

- [ ] Coin Change (LC 322): wrote from memory, traced coins=[1,2,5], amount=11
- [ ] Coin Change II (LC 518): explained outer-coin loop vs outer-amount loop difference
- [ ] Minimum Refueling Stops (LC 871): traced heap state at each station for the example above
- [ ] Minimum Cost to Cut a Stick (LC 1547): drew the interval DP table for small example (cuts=[1,3,4,5], n=7)
- [ ] Described APNs vs FCM differences and when to use data vs notification message
- [ ] Completed 1-hour OA simulation with debrief
- [ ] Behavioral STAR delivered; showed disagree + commit structure
- [ ] Reviewed all 5 flashcards
- [ ] Logged uncertain problems to `knowledge-base/revision-log.md`
