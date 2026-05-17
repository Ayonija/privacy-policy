# Week 21 Review — Days 141–147

## Phase
Phase 3 — Mock Interviews | Month 5

---

## Patterns Covered This Week

### DSA Patterns
1. **Interval DP (Split-Point Optimization)** — Day 141: `dp[i][j]` over all ranges; choose last/split point k; O(n³).
2. **Longest Common Subsequence** — Day 141 (revision): classic 2D DP; `dp[i][j]` = LCS of prefixes.
3. **2D String DP — Pattern Matching** — Day 142: regex and wildcard matching; `*` transitions differ between LC 10 and LC 44.
4. **Edit Distance (2D String DP)** — Day 142 (revision): three operations → three cell sources in DP table.
5. **Multi-Agent Grid DP** — Day 143: simultaneous path simulation; compress to 3D with step variable t.
6. **Inverted Combinatorial DP** — Day 143: Super Egg Drop; invert question to "max floors with k eggs, t trials."
7. **State Machine DP on Circular Structure** — Day 144: Freedom Trail; cost = CW/CCW rotation + 1 for button press.
8. **Interval Scheduling DP with Binary Search** — Day 144: sort by end time; binary search for last non-overlapping job; O(n log n).
9. **Palindrome Interval DP** — Day 145: `dp[i][j]` for min insertions / longest palindromic subsequence; fill by increasing length.
10. **Day-Partitioned Job Schedule DP** — Day 145: `dp[day][i]` = min difficulty from job i with day days left; iterate split points.
11. **Constraint Satisfaction Backtracking** — Day 146: place → check → recurse → undo; O(1) constraint checking with sets.
12. **Subset Enumeration Backtracking** — Day 146 (revision): include/exclude with index start; add result at every call.
13. **BFS Backtracking with Validity Pruning** — Day 147: level = edit distance; stop at first valid level (Remove Invalid Parentheses).
14. **Expression Backtracking with Running Values** — Day 147: track last_operand for correct multiplication; O(4^n × n).

### System Design Topics Covered This Week
| Day | Topic |
|-----|-------|
| 141 | YouTube/Netflix Requirements, Scale Numbers & Core Architecture Overview |
| 142 | Video Ingestion & Transcoding Pipeline |
| 143 | CDN & Adaptive Bitrate Streaming (ABR) |
| 144 | Recommendation Engine |
| 145 | Search, Trending & Real-time Analytics |
| 146 | Comments, Likes & Social Graph |
| 147 | Auth, Uploads & Notifications |

---

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in after each session) | | | |
| | | | |
| | | | |

---

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Interval DP (split-point) | | |
| 2D String DP — regex/wildcard | | |
| Multi-Agent Grid DP | | |
| Inverted Combinatorial DP (Super Egg Drop) | | |
| State Machine DP (Freedom Trail) | | |
| Interval Scheduling DP + Binary Search | | |
| Palindrome Interval DP | | |
| Day-Partitioned Job Schedule DP | | |
| Constraint Satisfaction Backtracking | | |
| BFS Backtracking with Pruning | | |
| Expression Backtracking (last_operand) | | |
| YouTube/Netflix System Design overall | | |

**Scoring guide:** 5 = can implement cold in 20 min; 4 = minor prompts needed; 3 = need to look up recurrence; 2 = partial understanding; 1 = blocked.

---

## Weekly Flashcard Deck

All 35 cards aggregated from Days 141–147, 5 per day.

| Q | A |
|---|---|
| **Day 141** | |
| How do you set up the Interval DP state for Burst Balloons (LC 312)? | Pad array with 1s on both ends. `dp[i][j]` = max coins bursting all balloons strictly between i and j. Iterate over k as the LAST balloon burst. |
| How do you derive the Strange Printer recurrence when `s[i] == s[j]`? | `dp[i][j] = dp[i][j-1]` — the turn that prints position i can extend to cover j at no extra cost. |
| What is the time complexity of interval DP and why? | O(n³): n² states (all pairs i,j) × O(n) to try each split point k. |
| How do you compute LCS for two strings of lengths m and n? | `dp[i][j] = dp[i-1][j-1]+1` if chars match, else `max(dp[i-1][j], dp[i][j-1])`. Return `dp[m][n]`. |
| What YouTube scale number should you state first in a system design interview? | 500 hours of video uploaded per minute and 1B hours watched per day — these two numbers establish upload pipeline and streaming CDN requirements. |
| **Day 142** | |
| How do you handle `p[j-1] == '*'` in the regex matching DP (LC 10)? | Two branches: (1) zero occurrences: `dp[i][j] = dp[i][j-2]`; (2) one more occurrence: if `p[j-2]` matches `s[i-1]`, `dp[i][j] |= dp[i-1][j]`. |
| How does the wildcard `*` differ from regex `*` in the DP transition? | Wildcard `*` independently matches any sequence: `dp[i][j] = dp[i-1][j] OR dp[i][j-1]`. Regex `*` always refers to the preceding character. |
| What are the three operations in Edit Distance and which DP cell does each correspond to? | Insert → `dp[i][j-1]+1`; Delete → `dp[i-1][j]+1`; Replace → `dp[i-1][j-1]+1`. |
| Why does YouTube use chunked upload instead of a single PUT request? | Resume on failure, parallel chunk upload, progress tracking, and no single 4K raw file overwhelming one server connection. |
| What triggers transcoding in the YouTube/Netflix pipeline? | Upload Service writes raw video to S3, then publishes a Kafka/SQS message. Transcode Coordinator consumes it and fans out per-rendition worker tasks. |
| **Day 143** | |
| How do you compress Cherry Pickup from 4D DP to 3D DP? | Use step `t` as a shared variable: at step t, agent 1 at row r1 → column c1 = t-r1; agent 2 at row r2 → c2 = t-r2. State: `dp[t][r1][r2]`. |
| What is the key insight of the inverted formulation for Super Egg Drop? | `dp[k][t]` = max floors checkable with k eggs and t trials. Increment t until `dp[K][t] >= N`. Recurrence: `dp[k][t] = dp[k-1][t-1] + dp[k][t-1] + 1`. |
| What is the HLS manifest format called and what does it contain? | `.m3u8` playlist. Master manifest lists all quality renditions + bandwidth. Sub-manifests list individual segment URLs with durations. |
| How often does an ABR client probe bandwidth and what triggers an immediate downgrade? | Probe every ~30 seconds or per segment. Immediate downgrade if playback buffer drops below ~10 seconds. |
| What is Netflix Open Connect and why does it reduce latency? | OCA = CDN appliances embedded inside ISP networks. Traffic never hits public internet; 95%+ of Netflix bytes served from OCAs during peak hours. |
| **Day 144** | |
| What does `dp[j]` represent in the Freedom Trail DP, and what is the rotation cost formula? | `dp[j]` = min steps to spell key[0..i] with ring at index j. Rotation cost between p and c: `min(abs(c-p), n-abs(c-p)) + 1`. |
| How do you find the latest non-overlapping job in the weighted interval scheduling DP? | Sort jobs by end time. For job i, binary search end time array for rightmost job ending ≤ startTime[i]. Result is index p(i). |
| What is the two-stage recommendation pipeline and what are typical latency targets? | Stage 1: Candidate generation using ANN search over embeddings → top 500 in < 100ms. Stage 2: Ranking model scores 500 → 20 in < 20ms. |
| Why does collaborative filtering fail for new users or new videos? | Cold start problem: no historical interactions exist for new entities, so no row/column in the user-item matrix to factorize. |
| What percentage of Netflix watch time comes from recommendations? | 75–80% of content watched on Netflix is driven by the recommendation engine. |
| **Day 145** | |
| How do you derive minimum insertions to make a palindrome from LPS? | `minInsertions(s) = s.length() - longestPalindromeSubsequence(s)`. Every character NOT in the LPS needs a matching character inserted. |
| What is the base case for Minimum Difficulty of a Job Schedule when d=1? | With 1 day remaining, you must do all remaining jobs. Difficulty = max(jobDifficulty[j..n-1]). |
| How does the sliding window trending counter work in Redis? | `ZINCRBY trending:hour:<H> 1 video_id` for each view. Union last N hour buckets with ZUNIONSTORE. Get top-K with ZREVRANGE. |
| Why use Count-Min Sketch for YouTube trending instead of exact counts? | Exact counts require O(unique_videos) memory; at billions of events, too large. Count-Min Sketch gives approximate counts in fixed O(w×d) space with < 1% error. |
| What is the delay in YouTube's view count display, and why is it acceptable? | View counts are 30–60 seconds behind real-time. Acceptable because users don't need millisecond-accurate counts; exact counts in analytics DB. |
| **Day 146** | |
| What three data structures track constraints in N-Queens backtracking? | `Set<Integer>` for occupied columns, diagonals (row-col), and anti-diagonals (row+col). All O(1) lookup. |
| How do you index the 3×3 box in Sudoku Solver? | `boxIndex = (row / 3) * 3 + (col / 3)`. Gives values 0–8 for the nine 3×3 boxes. |
| Why is keyset pagination better than OFFSET for YouTube comments? | OFFSET requires scanning and discarding rows — O(offset). Keyset uses last seen ID as WHERE clause — O(log n) via index. Stable under concurrent inserts. |
| How does YouTube handle like counting on popular videos without a write bottleneck? | Write clicks to Kafka (async). Redis INCR for fast approximate count. Kafka consumer batch-writes to Cassandra every 30s for durability. |
| What is the hybrid fan-out strategy YouTube uses for subscription feeds? | Push for channels with < 1M subscribers. Pull for channels > 1M subscribers to avoid 100M+ writes per upload. |
| **Day 147** | |
| Why does BFS find minimum removals naturally in Remove Invalid Parentheses? | BFS explores all strings at edit distance 1, then 2, etc. First level with valid strings = minimum removal count. |
| What is the `last_operand` trick in Expression Add Operators and when is it used? | When inserting `*`: `new_total = running_total - last_operand + last_operand * curr_num`. Undoes last addition, replaces with multiplication. |
| Why are JWT access tokens short-lived (15 min) while refresh tokens are long-lived (30 days)? | JWTs cannot be revoked without a blocklist. Short TTL limits damage window. Refresh tokens stored in Redis and can be deleted (true revocation). |
| What is a presigned S3 URL and why does it reduce backend load for uploads? | A time-limited URL allowing bearer to PUT an object directly to S3. Upload bytes never pass through backend servers — reduces bandwidth cost by 100%. |
| How does YouTube deliver notifications to 110M subscribers of a popular channel? | Async Kafka consumer fan-out in background batches. Target: all notifications within 1–5 minutes. Notification also written to per-user Cassandra feed for in-app view. |

---

## Mock / Contest Log
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | LeetCode Weekly | | | |
| | Mock Interview | | | |
| | | | | |
