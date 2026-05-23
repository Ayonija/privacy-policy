# Day 98 — 1D DP: Longest Increasing Subsequence & Job Scheduling DP
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Introduce LIS (Longest Increasing Subsequence) — the gateway to O(n log n) patience sorting — and apply the same binary-search-on-DP idea to Maximum Profit in Job Scheduling, a Hard problem that bridges DP with binary search on intervals.

---

## DSA (2 hours)

### Pattern: 1D DP — Subsequence Optimization & Binary Search on DP

**Core idea:**  
LIS O(n²): `dp[i] = max(dp[j] + 1)` for all `j < i` where `nums[j] < nums[i]`. LIS O(n log n): maintain a "tails" array where `tails[k]` = smallest tail element of all increasing subsequences of length `k+1`. Binary search to update.

**Trigger condition:**  
- "Longest subsequence where elements satisfy a monotone condition"
- "Maximum profit / score from non-overlapping intervals" (binary search on sorted endpoints + DP)
- "Russian doll" / envelope / nested structures

**Complexity:**  
LIS O(n²) | LIS O(n log n) with patience sort | Job scheduling: O(n log n)

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Russian Doll Envelopes | 354 | Hard | LIS in 2D + patience sort | Sort by width ASC, then height DESC for equal widths; run LIS on heights only |
| 2 | Maximum Profit in Job Scheduling | 1235 | Hard | DP + binary search on intervals | Sort by end time; `dp[i]` = max profit using first i jobs; binary search for last compatible job |
| 3 | Longest Increasing Subsequence (revision) | 300 | Medium | 1D DP / patience sort | O(n²) DP for understanding; O(n log n) `bisect_left` on `tails` array |

---

### Full Solution Walkthrough — LIS (LC 300) — Revision

**O(n²) DP — understand structure:**
```python
def lengthOfLIS_dp(nums: list[int]) -> int:
    n = len(nums)
    dp = [1] * n  # every element is an LIS of length 1 alone
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

**O(n log n) — patience sort (interview gold):**
```python
from bisect import bisect_left

def lengthOfLIS(nums: list[int]) -> int:
    tails = []   # tails[k] = smallest tail of all increasing subseqs of length k+1
    for n in nums:
        pos = bisect_left(tails, n)   # find where n fits
        if pos == len(tails):
            tails.append(n)           # extends longest subseq
        else:
            tails[pos] = n            # replaces to get smaller tail (greedy)
    return len(tails)
```

**Why replacing works:** `tails` stays sorted. Replacing `tails[pos]` with a smaller value doesn't shrink the LIS length but makes future extensions more likely (smaller tail = easier to beat). The length of `tails` always equals the LIS length, even though `tails` itself may not be a valid LIS.

**Trace on `[10,9,2,5,3,7,101,18]`:**  
`[] → [10] → [9] → [2] → [2,5] → [2,3] → [2,3,7] → [2,3,7,101] → [2,3,7,18]`  
Length = 4 ✓

---

### Full Solution Walkthrough — Russian Doll Envelopes (LC 354) — Hard

**Problem:** Envelope `(w, h)` fits in another if both dimensions strictly increase. Find max nesting depth.

**Reduction to LIS:**  
1. Sort by width ASC; for equal widths, sort height DESC.
2. Run LIS on heights only.

**Why DESC for equal widths?** Prevents using two envelopes of the same width in the same sequence (e.g., `[3,4]` and `[3,5]` — we can't nest them, but if heights were `[4,5]`, the LIS would incorrectly count both).

```python
from bisect import bisect_left

def maxEnvelopes(envelopes: list[list[int]]) -> int:
    # Sort: width ASC, height DESC (for equal widths)
    envelopes.sort(key=lambda e: (e[0], -e[1]))

    # LIS on heights using patience sort
    tails = []
    for _, h in envelopes:
        pos = bisect_left(tails, h)
        if pos == len(tails):
            tails.append(h)
        else:
            tails[pos] = h

    return len(tails)
```

**Complexity:** O(n log n) sort + O(n log n) LIS = O(n log n).

---

### Full Solution Walkthrough — Maximum Profit in Job Scheduling (LC 1235) — Hard

**Problem:** Jobs `(start, end, profit)`; can't take overlapping jobs. Maximize total profit.

**Key approach:** Sort by end time. `dp[i]` = max profit considering first `i` jobs (sorted by end). For job `i`, binary search for the latest job `j` that ends before `job_i.start`. Then `dp[i] = max(dp[i-1], dp[j] + profit_i)`.

```python
from bisect import bisect_right

def jobScheduling(startTime, endTime, profit):
    jobs = sorted(zip(startTime, endTime, profit), key=lambda x: x[1])
    
    # dp as list of (end_time, max_profit) pairs for binary search
    dp = [(0, 0)]   # (end_time=0, profit=0) sentinel
    
    for s, e, p in jobs:
        # Find latest job that ends ≤ s (current job start)
        # Binary search on end times in dp
        lo, hi = 0, len(dp) - 1
        while lo < hi:
            mid = (lo + hi + 1) // 2
            if dp[mid][0] <= s:
                lo = mid
            else:
                hi = mid - 1
        
        new_profit = dp[lo][1] + p
        if new_profit > dp[-1][1]:   # only append if improvement
            dp.append((e, new_profit))
    
    return dp[-1][1]
```

**Cleaner with `bisect_right` on end times:**
```python
from bisect import bisect_right

def jobScheduling(startTime, endTime, profit):
    jobs = sorted(zip(endTime, startTime, profit))
    # ends[i] = end time of i-th sorted job (for binary search)
    ends = [0]
    dp = [0]

    for e, s, p in jobs:
        # Last job ending ≤ s
        i = bisect_right(ends, s) - 1
        new_profit = dp[i] + p
        if new_profit > dp[-1]:
            dp.append(new_profit)
            ends.append(e)
        # If not better, don't extend (dp stays non-decreasing)

    return dp[-1]
```

**Trace on `start=[1,2,3,3], end=[3,4,5,6], profit=[50,10,40,70]`:**  
Sorted by end: `(3,1,50), (4,2,10), (5,3,40), (6,3,70)`  
`ends=[0,3], dp=[0,50]`  
Job (4,2,10): bisect_right([0,3], 2)=1 → i=0 → new=10; 10 < 50: no append  
Job (5,3,40): bisect_right([0,3], 3)=2 → i=1 → new=50+40=90; 90>50: append  
`ends=[0,3,5], dp=[0,50,90]`  
Job (6,3,70): bisect_right([0,3,5], 3)=2 → i=1 → new=50+70=120; 120>90: append  
Answer: 120 ✓

---

## System Design (30 min)

### Topic: Notification System — Fan-out at Scale & Delivery Guarantees

**Core components:**
1. **Kafka partitioned by user_id** — fan-out events are produced per user; consumers read from their partition
2. **Notification Worker Pool** — each worker: fetch event, lookup user preferences, call APNs/FCM adapter; stateless; scale horizontally
3. **Retry queue** — on APNs/FCM failure (5xx, timeout): re-enqueue with exponential backoff; max 3 retries; after final failure → mark as undeliverable in delivery log
4. **In-app inbox fallback** — notifications that fail push delivery are written to a user inbox table (Cassandra); app fetches on next open
5. **Deduplication** — idempotency key = `(user_id, message_id, device_id)`; checked in Redis before each send attempt; prevents double-sends on retry

**Key trade-offs:**
- **Push vs. in-app inbox:** Push is lossy (device offline, token stale, OS suppresses). In-app inbox is reliable but only seen when user opens app. Always implement both for critical notifications.
- **Fan-out worker design:** Should fan-out be one worker per notification type (email, push, SMS) or one combined worker? Separate workers = independent scaling, failure isolation. Combined = simpler deployment. Separate is preferred at scale.
- **Thundering herd on fan-out:** If 1M users follow a celebrity who posts, 1M notifications fire simultaneously. Mitigation: rate-limit fan-out worker, jitter the retry delays, use priority queues (important notifications go first).

**Interview talking point:**  
*"If asked how to prevent duplicate notifications after a worker crash, answer: before sending, check Redis: `SET notif:{user_id}:{message_id} 1 NX EX 86400`. If key already exists → already sent, skip. This makes each notification send idempotent. TTL = 24 hours covers the retry window. Redis SET NX is atomic — no race condition."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode Mock Interview — DP + Binary Search Set

**Goal:** Solve Maximum Profit in Job Scheduling (LC 1235) under interview conditions.

**Session structure:**
- 0:00 — Read problem; describe the approach before writing code: "Sort by end time. Build dp array where dp[i] = max profit using first i jobs. Binary search for the last non-overlapping job."
- 5:00 — Code; trace through the small example above
- 35:00 — Submit; debug if needed
- 45:00 — Explain Russian Doll reduction to LIS verbally (why DESC sort for equal widths?)
- 55:00 — Write LIS patience sort from memory in 5 min flat

**Debrief prompt:** For Job Scheduling, did you keep `dp` non-decreasing? If your `dp` can decrease, binary search gives wrong results. Write the invariant: *"`dp[i]` represents the maximum achievable profit — it must be non-decreasing as we consider more jobs."*

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a project where you had to balance quality and speed. How did you decide what to cut?

**Target LP:** *Deliver Results* / *Frugality* — make hard trade-offs; deliver something valuable under constraint.

**Full STAR Story — "LIS-Based Job Scheduling Shipped in One Sprint":**
**S (20%):** "At SchedulerCo, I had one sprint to ship a job scheduling optimization feature that reduced idle CPU in our distributed build cluster by selecting the maximum-profit non-overlapping jobs. The PM wanted it in 2 weeks; the full solution with persistence, monitoring, and rollback would take 6."
**T:** "I needed to deliver a working, correct optimization engine in 2 weeks while explicitly choosing what to defer."
**A (60% — 'I' not 'we'):** "(1) I implemented the core DP + binary search job scheduling algorithm first — this was the irreversible correctness decision. (2) I cut persistence (jobs recomputed on restart) and monitoring dashboards from scope, documenting these as follow-up tasks with concrete acceptance criteria. (3) I kept integration tests for the scheduling correctness but skipped unit tests for the binary search wrapper since the logic was a direct call to a well-tested library. (4) I shipped with a feature flag so the new scheduler could be enabled per-cluster without a rollback procedure."
**R (20%):** "Shipped on day 13. Cluster CPU idle time dropped from 18% to 6% in the first week. The 4 deferred items were all completed in the following sprint — the feature flag made rollout safe. PM and engineering lead both praised the explicit scope negotiation."
*Works for: Deliver Results, Frugality, Bias for Action.*

### STAR Interview Framework

> **LIS patience sort O(n log n):** naive O(n²) DP → patience sort with binary search O(n log n) time, O(n) space

**S:** "Longest Increasing Subsequence. Naive O(n²): for each index, scan all prior indices for smaller values."
**T:** "Need O(n log n) by maintaining a 'tails' array where tails[k] = smallest tail of all increasing subsequences of length k+1."
**A (60%):**
1. *Classify:* "LIS → patience sort with tails array and bisect_left."
2. *Init:* "tails = [] (empty)."
3. *Loop/Recurrence:* "For each num: pos=bisect_left(tails, num). If pos==len(tails): append (extends longest). Else: tails[pos]=num (greedy replacement — smaller tail enables future extensions)."
4. *Termination:* "Return len(tails)."
5. *Gotcha:* "tails is NOT a valid LIS — it's a maintenance structure. Its LENGTH equals the LIS length but its elements may not form the actual subsequence. Stating this clearly in an interview shows deep understanding."
**R:** "O(n log n) time, O(n) space. Russian Doll Envelopes: sort by (width ASC, height DESC for equal widths) then LIS on heights — DESC sort prevents same-width pairs from extending the subsequence."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| O(n²) DP | Need to reconstruct the actual subsequence path | Only length needed — patience sort is sufficient and faster |
| Segment tree on compressed values | LIS with custom comparators | Overkill; bisect_left handles standard LIS in 2 lines |

---

## Flashcards

| Q | A |
|---|---|
| Describe the patience sort (O(n log n) LIS) algorithm in one sentence, then its invariant. | For each number, `bisect_left` into `tails`; if it extends, append; if not, replace. Invariant: `tails[k]` = smallest possible tail element of any increasing subsequence of length `k+1`. |
| Why does Russian Doll Envelopes sort equal-width envelopes by height DESC? | Prevents selecting two same-width envelopes in the subsequence. With DESC heights for the same width, the LIS on heights will pick at most one from each width group, because a later same-width element's height < earlier → it replaces in tails (doesn't extend). |
| Describe the Job Scheduling DP in two sentences. | Sort jobs by end time. `dp[i]` = max profit using first i jobs; for each new job, binary search for the latest job ending before its start, then `dp[i] = max(dp[i-1], dp[found] + profit_i)`. Keep `dp` non-decreasing; only append when new profit exceeds current max. |
| What is the key invariant for `dp` in Job Scheduling binary search? | `dp` must be non-decreasing (it represents achievable profits). If a new profit doesn't exceed `dp[-1]`, don't append — the binary search relies on this monotonicity to return the correct index. |
| In Notification System design, how do you prevent duplicate sends after a worker crashes mid-fan-out? | Use Redis SET NX with a TTL: `SET notif:{user_id}:{message_id} 1 NX EX 86400`. If the key already exists, the notification was already sent — skip. This is O(1) per check and race-condition-free due to Redis atomic SET NX. |

---

## Checklist

- [ ] LIS (LC 300): traced patience sort on `[10,9,2,5,3,7,101,18]` step by step
- [ ] Russian Doll Envelopes (LC 354): explained the DESC sort trick from memory, then coded
- [ ] Maximum Profit in Job Scheduling (LC 1235): traced the example in this file by hand before coding
- [ ] Described in-app inbox fallback and why push alone is insufficient
- [ ] Described Redis SET NX deduplication pattern for notification fan-out
- [ ] Completed 1-hour mock interview session with verbal approach description
- [ ] Behavioral STAR delivered; named specific trade-off with reasoning
- [ ] Reviewed all 5 flashcards
- [ ] Logged uncertain problems to `knowledge-base/revision-log.md`
