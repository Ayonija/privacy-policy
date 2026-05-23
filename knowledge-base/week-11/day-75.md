# Day 75 — Heaps: Greedy Capital Allocation, Seat Manager, and IPO
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Longest Happy String is a greedy max-heap construction: always pick the most frequent available character unless it would create a triple-consecutive run. Seat Reservation Manager demonstrates the min-heap as a priority queue for ordered resource allocation. IPO is the canonical two-heap greedy: separate projects by capital requirement and use a max-heap to always pick the highest-profit affordable project.

---

## DSA (2 hours)
### Pattern: Greedy Max-Heap String Construction + Min-Heap Resource Queue + Two-Heap Greedy Selection

**Longest Happy String (LC 1405):**
A "happy" string has no "aaa", "bbb", or "ccc" as a substring. Maximise length using at most `a` a's, `b` b's, `c` c's. Greedy: always append the most frequent character available, as long as adding it won't create a triple run. If the top character would create a triple, use the second-most-frequent instead.

Max-heap of `(-count, char)`. Each step:
1. Pop the highest-frequency char (say c1). If last two chars in result == c1, pop the next (c2), use c2, push c1 back.
2. Push c1 (or c2) back with decremented count if still positive.

**Seat Reservation Manager (LC 1845):**
A seat manager for `n` seats (1-indexed). `reserve()` → return the smallest available seat number; `unreserve(seatNumber)` → make the seat available again.

Implementation: min-heap initialized with `[1, 2, ..., n]`. `reserve()` = heappop; `unreserve(s)` = heappush. O(log n) per operation.

**IPO (LC 502):**
Start with capital `w`. Each project has a `profit[i]` and `capital[i]` (minimum capital to start). Pick at most `k` projects to maximise final capital.

Greedy with two heaps:
1. Sort projects by capital (min-heap of `(capital, profit)`)
2. Max-heap of profits for "affordable" projects
3. Each of k rounds: move all affordable projects from capital-heap to profit-heap; pop max profit and add to `w`

**Trigger condition:**
- "construct the longest string with character limits and no triple run" → max-heap greedy; take the most frequent that won't create a triple, else take the second-most-frequent
- "always return the smallest/largest available resource number" → min/max heap of available resources; O(log n) per operation
- "select at most K items to maximise value, each with a threshold cost, items unlock as cost increases" → two heaps: threshold-sorted (pop when affordable) + value-sorted (pop max)

**Time complexity:** LC 1405: O(a+b+c) ≈ O(1) since only 3 chars | LC 1845: O(log n) per op | LC 502: O((n + k) log n)
**Space complexity:** O(1) / O(n) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Longest Happy String | 1405 | Medium | Greedy max-heap; avoid triple run | Pop top char; if creates triple, use second; push both back with decremented counts |
| 2 | Seat Reservation Manager | 1845 | Medium | Min-heap of available seats | reserve = heappop; unreserve = heappush; O(log n) per op |
| 3 | IPO | 502 | Hard | Two heaps: min-heap by capital + max-heap by profit | Each round: unlock affordable projects; pick max profit |

---

### Code Skeleton
```python
import heapq

# Longest Happy String (LC 1405)
def longestDiverseString(a, b, c):
    heap = [(-cnt, char) for cnt, char in [(a, 'a'), (b, 'b'), (c, 'c')] if cnt > 0]
    heapq.heapify(heap)
    result = []
    while heap:
        cnt1, c1 = heapq.heappop(heap)
        # Check if adding c1 would create "ccc"
        if len(result) >= 2 and result[-1] == result[-2] == c1:
            if not heap: break   # no other char available → stop
            cnt2, c2 = heapq.heappop(heap)
            result.append(c2)
            if cnt2 + 1 < 0: heapq.heappush(heap, (cnt2 + 1, c2))
            heapq.heappush(heap, (cnt1, c1))   # push c1 back unchanged
        else:
            result.append(c1)
            if cnt1 + 1 < 0: heapq.heappush(heap, (cnt1 + 1, c1))
    return ''.join(result)

# Seat Reservation Manager (LC 1845)
class SeatManager:
    def __init__(self, n):
        self.available = list(range(1, n + 1))
        heapq.heapify(self.available)   # already sorted, heapify is O(n)

    def reserve(self):
        return heapq.heappop(self.available)

    def unreserve(self, seatNumber):
        heapq.heappush(self.available, seatNumber)

# IPO (LC 502)
def findMaximizedCapital(k, w, profits, capital):
    # Min-heap by capital required (unlocks as w grows)
    cap_heap = sorted(zip(capital, profits))   # or heapify
    cap_idx = 0
    profit_heap = []   # max-heap of profits (negate)
    for _ in range(k):
        # Move all affordable projects to profit heap
        while cap_idx < len(cap_heap) and cap_heap[cap_idx][0] <= w:
            heapq.heappush(profit_heap, -cap_heap[cap_idx][1])
            cap_idx += 1
        if not profit_heap: break   # no affordable projects
        w += -heapq.heappop(profit_heap)   # take most profitable
    return w
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Greedy Max-Heap String Construction / Min-Heap Resource Queue / Two-Heap Greedy Capital Selection in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three problems requiring greedy choices on dynamically evolving option sets: building the longest valid string without triple-character runs (LC 1405), allocating seats with the smallest available number (LC 1845), and selecting profitable projects to maximise capital given a budget constraint that grows as projects are completed (LC 502). Brute force — checking all arrangements, scanning all seats, or trying all project orderings — is exponential or O(n²)."

**Task:** "My goal was to recognise that each problem reduces to a heap operation on the 'best available choice at this moment': max-heap for the most frequent character, min-heap for the smallest available seat, and a two-heap system for the most profitable affordable project."

**Action:** Walk the interviewer through these steps:
1. *Classify the pattern:* "No-triple-run string → max-heap greedy (always pick the two most frequent chars). Seat manager → min-heap of available seats (reserve = pop; unreserve = push). IPO → two heaps: a min-heap sorted by capital requirement (unlocks projects as budget grows) plus a max-heap of profits for currently affordable projects."
2. *Initialize:* "Longest Happy String: push `(-count, char)` for a, b, c. Seat Manager: heapify `[1..n]`. IPO: sort projects by capital; start with an empty profit max-heap."
3. *Core loop logic:* "Longest Happy String: pop top; if last two chars in result match the top char, pop second-most-frequent instead, place it, push top back. IPO: each of k rounds, move ALL projects with `capital ≤ w` from the capital min-heap to the profit max-heap, then pop the max profit and add to `w`."
4. *Convergence guarantee:* "Longest Happy String: the heap empties naturally; termination is when no char can be placed without creating a triple. IPO: k rounds enforced by the outer loop; inner loop draining affordable projects is bounded by the total project count n."
5. *Duplicate handling / edge case proactivity:* "IPO edge: if `profit_heap` is empty when we try to pop (no affordable projects even after draining), we `break` early — spending all k rounds is not required. Seat Manager: `unreserve(s)` after all seats are taken — the heap handles this correctly since push is always valid."

**Result:** "Seat Manager: O(log n) per operation vs O(n) linear scan. IPO: O((n + k) log n) vs O(k × n) brute force — for n = 10^5 projects and k = 10^4, that's 10^9 vs 1.5 × 10^6 operations. Longest Happy String: O(constant) since only 3 possible characters, so O(a + b + c) total."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer heap here |
|-------------|---------------------------|----------------------|
| Linear scan for available seat | n ≤ 1000 and operations are infrequent | O(n) per reserve; heap gives O(log n) — critical for n = 10^5 with 10^5 operations |
| Greedy sort for IPO | k ≥ n (take all affordable projects) | When k is large, run the capital-heap drain once; for k << n, two-heap per-round is more efficient |
| Recursive for Longest Happy String | String length ≤ 100 | O(3^n) exponential; heap greedy is O(n) and correct |

**Why NOT linear scan for Seat Manager:** At n = 10^5 seats and 10^5 operations, O(n) per operation = O(10^10) total. Heap gives O(n log n) total.
**Why NOT sort-all for IPO:** Sorting by profit alone ignores the capital constraint — you must afford a project before picking it; the two-heap correctly unlocks projects incrementally.

---

### Edge Cases to Trace Before Coding
- LC 1405: a=7, b=1, c=0 → result = "aabaa" (can't use more than 2 consecutive a's); a=1, b=1, c=1 → "abc" (all used); one char has count = 0 → ignore
- LC 1845: reserve all n seats → heap is empty; unreserve after all reserved → heap has one element; reserve then unreserve alternately → heap size stays bounded at n
- LC 502: k=0 → return initial w; no affordable projects ever → return w unchanged; all projects affordable from start → pick top k by profit

---

### Interview Pattern Drill

| Problem | Capital heap | Profit heap | When to switch |
|---------|-------------|-------------|----------------|
| IPO | Min-heap (capital, profit) | Max-heap (-profit) | After all affordable moved to profit heap |
| Task with dependency | Min-heap (start_time) | Max-heap (-value) | When time ≥ start_time of next task |

**Two-heap greedy template:**
```
sorted_by_cost = [(cost_i, value_i), ...]
value_heap = max-heap (negated)
for k rounds:
    while cost_heap is affordable:
        move to value_heap
    pick max value from value_heap
    update budget/resource
```
This pattern applies to: IPO (capital budget), Task Scheduling with deadlines, Meeting scheduling to maximize meetings attended.

---

## System Design (1 hour)
### Topic: Newsfeed — Feed Caching Strategy and Pre-computation

**Pre-computed feed cache:**
Each user's feed is stored in Redis as a sorted set `feed:{user_id}` with post_ids scored by timestamp. This cache is the primary source for all feed reads.

**Cache population triggers:**
1. **On new post (fanout worker):** for each normal follower, `ZADD feed:{follower_id} {timestamp} {post_id}`; cap at 1000 entries with `ZREMRANGEBYRANK feed:{follower_id} 0 -1001`
2. **On login after cache miss (cold start worker):** async job rebuilds feed from scratch

**Feed read path:**
```
User requests feed (GET /feed?cursor=xxx)
    │
    ├─ Redis ZREVRANGE feed:{user_id} 0 49  → 50 post_ids
    │   (if Redis miss → cold start, wait for rebuild, or return empty)
    │
    ├─ Batch GET post details from Post Service (parallel)
    │    ├─ Redis cache: post:{post_id} → cached JSON
    │    └─ DB fallback: SELECT * FROM posts WHERE post_id IN (...)
    │
    ├─ Merge celebrity posts (if user follows celebrities)
    │    └─ ZREVRANGE user_posts:{celebrity_id} 0 19 for each celebrity followed
    │
    └─ Rank + return top N posts (chronological or scored)
```

**Pagination with cursor:**
Use the timestamp as the cursor. `GET /feed?before={timestamp}` → `ZREVRANGEBYSCORE feed:{user_id} {timestamp} -inf LIMIT 0 20`. Client stores the timestamp of the last seen post to request the next page.

**Cache warming heuristics:**
- New user signup → cold start immediately (warm feed on sign-up)
- User inactive > 7 days → feed TTL expires; warm on next login
- Trending post → proactively push to feeds of users likely to be active (ML prediction)

**Read-your-writes consistency:**
After a user posts, they should see their own post immediately. Two options:
1. Short TTL on the user's own feed (forces re-read from DB for their own posts)
2. Client-side optimistic update: append the post to the rendered feed immediately before server confirmation

**Interview talking point:** "Feed reads go through three layers: Redis for pre-computed post_ids (sub-millisecond), a batch-fetch for post details (also cached in Redis per-post), and a celebrity merge step. The slowest case — cache miss → cold start — takes 2–5 seconds as we fetch from all followees. We mitigate this by warming the cache on login and keeping feeds for active users always warm (< 7 day TTL)."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 1405 Longest Happy String (target: 18 min)
- **Medium 2:** LC 1845 Seat Reservation Manager (target: 10 min)
- **Hard:** LC 502 IPO (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)

**Leadership Principle:** Are Right, A Lot

**STAR Story: Greedy Feature Prioritisation That Maximised ROI Across a Constrained Roadmap**

**Situation (20%):** "Our startup had raised a Series A and was under pressure to ship 12 features in 6 months. Each feature had an estimated engineering cost (weeks) and a projected revenue uplift (in dollar ARR). Our product manager wanted to build features in order of discovery — a roughly chronological approach that ignored the cost-to-value ratio entirely."

**Task (part of S/T):** "I was asked to provide an alternative prioritisation. My constraint was that the team's total capacity was 24 engineer-weeks. My goal was to maximise the total projected ARR shipped within that budget — a bounded-knapsack problem analogous to IPO."

**Action (60-70% — be specific about what YOU did):**
"First, I challenged the assumption that we needed to build all 12 features — I wanted to know which subset maximised revenue given our capacity constraint, not just which features were cheapest or highest-value in isolation.
Then, I sorted features by their 'capital requirement' (minimum team capacity milestone needed before we could build them, due to dependencies) and built a greedy simulation: at each capacity checkpoint, I selected the highest-ARR-projected feature we could afford.
Next, I stress-tested the greedy approach with an exchange argument: for any feature pair where I'd chosen feature A over B, I showed that swapping would reduce projected ARR because B's ARR per engineer-week was always lower.
Finally, I presented the analysis to leadership with two charts: the greedy schedule vs. the PM's original order, showing a 34% higher projected ARR with the same budget."

**Result (10-20%):** "Leadership adopted the greedy schedule. By month 6, we had shipped 9 of the prioritised 10 features (one was delayed due to an external API dependency, not the schedule). The projected ARR from the delivered features was 31% higher than the original plan would have achieved with the same team. The PM later told me this was the first time they had seen a rigorous trade-off analysis instead of gut-feel roadmap ordering."

**Interview tip:** Interviewers want to hear about *your* contribution. Say "I challenged", "I sorted", "I stress-tested", "I presented" — not "we did". Prepare this story for questions about: Are Right A Lot, data-driven decisions, constructive dissent, and delivering results.

---

## Flashcards

| Q | A |
|---|---|
| How does Longest Happy String avoid creating "aaa"? | Pop the most-frequent char. If the last two result chars are the same as this char → would create triple → instead pop and use the second-most-frequent char; push the first back unchanged. |
| How is SeatManager.reserve() implemented with O(log n) time? | Min-heap of available seat numbers. `reserve()` = `heappop()` → returns smallest. `unreserve(s)` = `heappush(s)`. Initialised with `heapify([1, ..., n])` in O(n). |
| What are the two heaps in IPO and what does each contain? | (1) Min-heap by capital required — sorted by minimum capital to start each project. (2) Max-heap of profits — contains only affordable projects (capital ≤ current w). Each round: drain affordable from (1) into (2); pick max from (2). |
| How does the feed read path handle celebrity followees? | Celebrity posts are NOT in the pre-computed Redis feed. At read time, the Feed Service fetches the celebrity's last N posts from `user_posts:{celebrity_id}` sorted set and merges them with the regular feed by timestamp. |
| How is feed pagination implemented with a Redis sorted set? | Use timestamp as the cursor. `ZREVRANGEBYSCORE feed:{user_id} {cursor_timestamp} -inf LIMIT 0 20` returns the next page of posts before the cursor. Client stores the timestamp of the last post seen. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/10/25 min, no hints)
- [ ] Rewrote two-heap IPO template and Longest Happy String greedy from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can describe feed read path with cursor pagination cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
