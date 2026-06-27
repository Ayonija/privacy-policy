# Month 5 Review — Phase 3: Mock Interviews

## Phase
**Goal:** Simulate real FAANG interview conditions daily. Solve 2 Hard problems cold in 45 minutes total, deliver a complete system design in 45 minutes, and answer behavioral questions using polished STAR stories. Exit Month 5 interview-ready.

**Context:** Phase 3 spans Slots 13–15 (Days 121–150). This review covers all three slots:
- Slot 13 (Days 121–130): Graph Hard + Twitter/X System Design
- Slot 14 (Days 131–140): Advanced DP + Uber/Lyft System Design
- Slot 15 (Days 141–150): DP Hard + Backtracking Hard + YouTube/Netflix System Design

---

## Patterns Mastered (comfort ≥ 4/5)
Rate yourself on the patterns below. Mark those you can cold-solve in < 25 minutes.

### Graph Patterns (Slot 13)
- [ ] Dijkstra's algorithm with priority queue
- [ ] Bellman-Ford for negative edges
- [ ] Floyd-Warshall for all-pairs shortest path
- [ ] Topological sort (Kahn's BFS + DFS)
- [ ] Strongly Connected Components (Kosaraju / Tarjan)
- [ ] Bipartite graph check
- [ ] Union-Find with path compression and union by rank
- [ ] Minimum Spanning Tree (Kruskal + Prim)

### Advanced DP Patterns (Slot 14)
- [ ] Digit DP (count numbers satisfying digit-level constraints)
- [ ] Bitmask DP (subset states, TSP-style)
- [ ] DP on trees (re-rooting technique, subtree DP)
- [ ] DP with stack optimization (monotonic stack)
- [ ] Divide and conquer DP (optimization for convex/concave cost functions)

### DP Hard Patterns (Slot 15)
- [ ] Interval DP with split-point optimization (Burst Balloons, Strange Printer)
- [ ] 2D String DP — regex and wildcard matching
- [ ] Multi-agent grid DP (Cherry Pickup — 3D state compression)
- [ ] Inverted combinatorial DP (Super Egg Drop)
- [ ] State machine DP on circular/graph structure (Freedom Trail)
- [ ] Interval scheduling DP with binary search (Weighted Job Scheduling)
- [ ] Palindrome interval DP (LPS, min insertions)
- [ ] Day-partitioned job schedule DP (Min Difficulty)

### Backtracking Hard Patterns (Slot 15)
- [ ] Constraint satisfaction backtracking (N-Queens, Sudoku)
- [ ] BFS backtracking with validity pruning (Remove Invalid Parentheses)
- [ ] Expression backtracking with running values (Expression Add Operators, 24 Game)
- [ ] Trie-augmented grid backtracking (Word Search II)
- [ ] Grid backtracking with must-visit constraint (Unique Paths III)
- [ ] Trie-constrained partition backtracking (Word Squares)
- [ ] Backtracking on unknown environment (Robot Room Cleaner)

---

## Patterns Needing Drill

List patterns where your comfort is ≤ 3. Schedule a targeted 90-minute session per pattern before your next real interview.

| Pattern | Comfort (1–5) | Target retry date | Notes |
|---------|--------------|------------------|-------|
| (learner fills in) | | | |
| | | | |
| | | | |

---

## System Design Topics Mastered

### Twitter / X (Slot 13, Days 121–130)
**Core architecture:** API Gateway → Tweet Service → Fanout Service (Redis sorted set per user for timeline) → Cassandra (tweet store). Hybrid fanout: push for users with < 1M followers, pull for celebrities. Twitter uses Snowflake IDs for globally unique, time-sortable tweet IDs. Search: Elasticsearch with real-time index. Trending: Redis ZINCRBY sliding window. Rate limiting: token bucket per user_id.

**Key numbers:** 300M DAU, 500M tweets/day, 28K read QPS average / 100K peak, 5K write QPS. Timeline read:write = 20:1. Fanout: avg 100 followers → 50K fanout writes/second.

### Uber / Lyft (Slot 14, Days 131–140)
**Core architecture:** Location Service (1-second heartbeat from drivers → Redis geospatial index) → Matching Service (geohash-based radius search → scoring → ETA from routing graph) → Ride Service (state machine: REQUESTED → ACCEPTED → ARRIVED → IN_RIDE → COMPLETED). Surge pricing computed via supply/demand ratio per geohash cell. ETA: precomputed road graph with Dijkstra; updated for real-time conditions.

**Key numbers:** 5M trips/day, 1M active drivers, driver location updates 1/second = 1M writes/sec, matching latency target < 5 seconds, ETA accuracy ± 30 seconds. Geohash precision 6 = ~1.2km cells.

### YouTube / Netflix (Slot 15, Days 141–150)
**Core architecture:** Upload → chunked S3 + async Kafka → transcode workers (FFmpeg, 15-20 renditions) → CDN origin. Streaming → signed CDN URL → HLS/DASH segments from edge POP → ABR algorithm on client. Discovery → Elasticsearch for search + two-tower NN recommendation with Faiss ANN. Comments → Cassandra (partition: video_id). Likes → Kafka → Redis INCR → Cassandra batch flush. Notifications → Kafka fan-out → FCM/APNs. Auth → JWT (15min) + Redis refresh tokens.

**Key numbers:** 2B YouTube users, 500 hrs uploaded/min, 1B hrs watched/day, 15-20 transcode renditions, Netflix 250M subscribers, 17,000+ OCAs, 95%+ CDN hit rate, recommendations drive 70-80% of watch time, TTFF < 2s.

---

## OA / Mock Interview Stats
- Contests attempted: ___ (fill in)
- Mock interviews done: ___ (fill in)
- Average completion rate under time pressure: ___%
- DSA: Average problems solved per 45-min session: ___/2 hard
- System Design: Average components covered per 45-min session: ___/5 major components

### Hard Problems by Slot

**Slot 13 (Graph Hard) — Days 121–130:**
Covered graph algorithms including Dijkstra, Bellman-Ford, Floyd-Warshall, Kahn's topological sort, Kosaraju SCC, Union-Find, and MST. Problems included LC 743, 787, 207, 210, 1192, 399, 1168, 1489, 1059, 310 (non-exhaustive).

**Slot 14 (Advanced DP) — Days 131–140:**
Covered Digit DP, Bitmask DP, Tree DP, stack-optimized DP, divide-and-conquer DP. Problems included LC 233, 902, 1397, 847, 943, 1681, 337, 543, 1130, 1000 (non-exhaustive).

**Slot 15 (DP Hard + Backtracking Hard) — Days 141–150:**
- LC 312 Burst Balloons, LC 664 Strange Printer, LC 10 Regex Matching, LC 44 Wildcard Matching
- LC 741 Cherry Pickup, LC 887 Super Egg Drop, LC 514 Freedom Trail, LC 1235 Job Scheduling
- LC 1312 Min Insertions Palindrome, LC 1335 Min Difficulty Job Schedule
- LC 51 N-Queens, LC 37 Sudoku Solver, LC 301 Remove Invalid Parentheses, LC 282 Expression Add Operators
- LC 980 Unique Paths III, LC 212 Word Search II, LC 679 24 Game, LC 425 Word Squares
- LC 546 Remove Boxes, LC 489 Robot Room Cleaner

---

## Month Flashcard Master Deck

Complete flashcard sets from each week's review. Use spaced repetition (Anki or physical cards) on these before any interview.

- **Week 19 flashcards:** `knowledge-base/week-19/weekly-review.md` — 35 cards (Graph Hard + Twitter/X architecture)
- **Week 20 flashcards:** `knowledge-base/week-20/weekly-review.md` — 35 cards (Advanced DP + Uber/Lyft architecture)
- **Week 21 flashcards:** `knowledge-base/week-21/weekly-review.md` — 35 cards (DP Hard + Backtracking + YouTube/Netflix early architecture)

**Highest-yield cards to review daily:**

| Q | A |
|---|---|
| What is the Split-Point Interval DP recurrence for Burst Balloons? | `dp[i][j] = max(dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j])` for k in (i,j). Pad nums with 1s. k is the LAST balloon burst. |
| How does the regex `*` DP transition differ from wildcard `*`? | Regex: `dp[i][j] = dp[i][j-2]` (zero) OR `dp[i-1][j]` if p[j-2] matches s[i-1]. Wildcard: `dp[i][j] = dp[i-1][j] OR dp[i][j-1]`. |
| State the Cherry Pickup 3D DP state compression. | Step t shared; agent 1 at row r1 → col c1=t-r1; agent 2 at row r2 → col c2=t-r2. State: dp[t][r1][r2]. |
| How does the Weighted Job Scheduling DP work? | Sort by end time. `dp[i] = max(dp[i-1], profit[i] + dp[p(i)])`. p(i) = binary search for latest job ending ≤ startTime[i]. |
| What three sets enable O(1) constraint checking in N-Queens? | Set<Integer> for columns, diagonals (row-col), anti-diagonals (row+col). |
| What is the `last_operand` trick for Expression Add Operators `*`? | `new_total = running_total - last_operand + last_operand * curr`. Undoes last add, applies multiplication. |
| How does Trie augmentation speed up Word Search II? | All words share one DFS. Prune immediately if current prefix has no Trie child. No per-word DFS restarts. |
| YouTube: how many hours are uploaded per minute? | 500 hours per minute. |
| Netflix: what % of watch time comes from recommendations? | 75–80%. |
| What is Netflix Open Connect and its scale? | CDN appliances embedded in ISP data centers worldwide. 17,000+ appliances; 95%+ of Netflix bytes served from OCAs. |

---

## Next Month Goals

### Month 6 Objectives (Phase 4 — Final Sprint & Real Applications)

1. **Apply to target companies.** Submit applications to all target FAANG/FAANG-adjacent companies. Tailor each resume bullet to match the company's engineering blog language.

2. **Interview scheduling and logistics.** Schedule OA → phone screen → virtual onsite for at least 3 target companies. Build a tracking spreadsheet: company, role, recruiter contact, interview dates, status.

3. **Targeted gap closure.** Run full mock interviews weekly. After each mock, identify the 1–2 patterns or system design areas that stalled you and schedule a focused 90-minute session on that specific topic using the relevant knowledge-base day files.

4. **Behavioral story bank.** Finalize 8–10 STAR stories mapped to all major leadership principles. Practice each story aloud to under 2 minutes. Cover: ownership, scaling under pressure, data-driven decisions, cross-team collaboration, failed project + recovery, mentorship.

5. **System design fluency.** Deliver all three full designs (Twitter/X, Uber/Lyft, YouTube/Netflix) cold in 45 minutes without notes. Time yourself. Target: complete requirements → scale → architecture → deep dive → trade-offs in 45 minutes with 5 minutes to spare.

6. **Maintain DSA edge.** Solve 2 Hard problems per week on LeetCode contest to stay sharp under timed pressure. Review the revision-log weekly and re-solve any problem marked as "not solved within 25 min."
