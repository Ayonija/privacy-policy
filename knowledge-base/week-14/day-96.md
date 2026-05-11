# Day 96 — 1D DP: Palindrome DP & Interval DP Introduction
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master palindrome-based DP: precompute the 2D `is_pal` table for O(1) checks, apply it to Palindrome Partitioning II (Hard), and introduce interval DP through Minimum Insertion Steps — the pattern that underlies "transform string X into string Y with minimum cost" problems.

---

## DSA (2 hours)

### Pattern: 1D DP with 2D Precomputation — Palindrome DP

**Core idea:**  
Precompute `is_pal[i][j]` in O(n²). Then for any problem requiring palindrome checks during DP, each check is O(1). Classic use: `dp[i]` = minimum cuts to partition `s[0:i]` into palindromes → `dp[i] = min(dp[j] + 1)` for all `j < i` where `s[j:i]` is a palindrome.

**Trigger condition:**  
- "Minimum cuts to partition string into palindromes"
- "Minimum operations to make a string a palindrome"
- Any problem combining "is a substring a palindrome?" as a repeated subquery

**Complexity:**  
Precomputation: O(n²) | Main DP: O(n²) | Space: O(n²) for `is_pal` table

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Palindrome Partitioning II | 132 | Hard | 1D DP + 2D palindrome table | `dp[i]` = min cuts for `s[0:i]`; precompute `is_pal` first; `dp[i] = min(dp[j] + 1)` for palindromic `s[j:i]` |
| 2 | Minimum Insertion Steps to Make a String Palindrome | 1312 | Hard | Interval DP | `dp[i][j]` = min insertions to make `s[i..j]` palindrome; reduces to `n - LCS(s, reverse(s))` |
| 3 | Palindromic Substrings (revision) | 647 | Medium | 2D palindrome table expansion | Expand from each center; count palindromes rather than partition |

---

### Full Solution Walkthrough — Palindromic Substrings (LC 647) — Revision

Two approaches; know both:

**Approach 1: Expand around center** (preferred — O(1) space)
```python
def countSubstrings(s: str) -> int:
    count = 0
    n = len(s)

    def expand(l, r):
        nonlocal count
        while l >= 0 and r < n and s[l] == s[r]:
            count += 1
            l -= 1
            r += 1

    for i in range(n):
        expand(i, i)      # odd-length
        expand(i, i + 1)  # even-length

    return count
```

**Approach 2: Precompute is_pal table** (useful when you need O(1) lookups later)
```python
def build_is_pal(s: str) -> list[list[bool]]:
    n = len(s)
    is_pal = [[False] * n for _ in range(n)]
    for i in range(n):
        is_pal[i][i] = True
    for i in range(n - 1):
        is_pal[i][i+1] = (s[i] == s[i+1])
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            is_pal[i][j] = (s[i] == s[j]) and is_pal[i+1][j-1]
    return is_pal
```

---

### Full Solution Walkthrough — Palindrome Partitioning II (LC 132) — Hard

**Goal:** Minimum number of cuts to partition `s` into substrings where every part is a palindrome.

**Step 1:** Precompute `is_pal[i][j]`.  
**Step 2:** `dp[i]` = min cuts for `s[0:i-1]` (1-indexed for clarity).

```python
def minCut(s: str) -> int:
    n = len(s)

    # Step 1: build palindrome table
    is_pal = [[False] * n for _ in range(n)]
    for i in range(n - 1, -1, -1):
        for j in range(i, n):
            if s[i] == s[j] and (j - i <= 2 or is_pal[i+1][j-1]):
                is_pal[i][j] = True

    # Step 2: dp[i] = min cuts for s[0:i]
    dp = list(range(-1, n))   # dp[0] = -1; dp[i] = i-1 (worst: cut every char)

    for i in range(1, n + 1):
        if is_pal[0][i-1]:    # whole prefix is a palindrome — 0 cuts
            dp[i] = 0
            continue
        for j in range(1, i):
            if is_pal[j][i-1]:
                dp[i] = min(dp[i], dp[j] + 1)

    return dp[n]
```

**Trace on `"aab"`:**  
`is_pal: [0][0]=T, [1][1]=T, [2][2]=T, [0][1]=T (aa), [1][2]=F, [0][2]=F`  
`dp[1]=0 (a), dp[2]=0 (aa), dp[3]=1 (aa|b)`  
Answer: 1 ✓

**Alternative cleaner O(n²) with bottom-up `is_pal` fill from bottom-right:**
```python
for i in range(n - 1, -1, -1):
    for j in range(i, n):
        is_pal[i][j] = s[i] == s[j] and (j - i < 2 or is_pal[i+1][j-1])
```

---

### Full Solution Walkthrough — Minimum Insertion Steps to Make Palindrome (LC 1312) — Hard

**Problem:** Minimum insertions to make `s` a palindrome.

**Key insight:** `min_insertions(s) = len(s) - LCS(s, reverse(s))`  
Why? The LCS of `s` and `reverse(s)` is the longest palindromic subsequence already in `s`. Every character NOT in the LPS needs to be inserted (mirrored).

```python
def minInsertions(s: str) -> int:
    t = s[::-1]
    n = len(s)
    # Standard LCS DP with O(n) rolling array
    prev = [0] * (n + 1)
    for i in range(1, n + 1):
        curr = [0] * (n + 1)
        for j in range(1, n + 1):
            if s[i-1] == t[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(curr[j-1], prev[j])
        prev = curr
    lps = prev[n]   # longest palindromic subsequence length
    return n - lps
```

**Direct interval DP (shows the structure more clearly):**
```python
def minInsertions_interval(s: str) -> int:
    n = len(s)
    # dp[i][j] = min insertions to make s[i..j] a palindrome
    dp = [[0] * n for _ in range(n)]

    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1]   # chars match: extend inward
            else:
                # Insert a char matching s[j] before s[i], OR matching s[i] after s[j]
                dp[i][j] = 1 + min(dp[i+1][j], dp[i][j-1])

    return dp[0][n-1]
```

**O(n²) space** — can reduce to O(n) with careful rolling.

---

## System Design (30 min)

### Topic: Notification System — Architecture Overview

**Core components:**
1. **Notification Service** — receives trigger events from Chat Service, Auth Service, etc. via an internal message queue (Kafka)
2. **User Preference Store** — Redis/MySQL: `user_id → {push_enabled, email_enabled, sms_enabled, quiet_hours}`
3. **Push Provider Adapters** — APNs (iOS), FCM (Android), Web Push; each adapter handles provider-specific formatting and token management
4. **Device Token Registry** — Cassandra: `(user_id, device_id, push_token, platform, updated_at)` — users may have multiple devices
5. **Delivery log** — Kafka + S3: log every notification attempt with status (sent/failed/bounced) for analytics and retry

**Key trade-offs:**
- **Fan-out service vs. per-event processing:** For a chat notification, one event → fan-out to all the sender's contacts' devices. Use a Kafka consumer group with a fan-out worker to avoid blocking the Chat Service.
- **Push token staleness:** Tokens expire when users reinstall apps or revoke permissions. APNs/FCM return error codes on stale tokens — the adapter must delete the token on failure codes (e.g., `Unregistered`).
- **Rate limiting:** Mobile platforms (APNs, FCM) have per-app rate limits. The notification service must throttle sends per user (e.g., max 5 push per minute) and respect platform quotas.

**Interview talking point:**  
*"If asked how to design a notification system that handles 1M sends per second, answer: decouple ingestion (Kafka topic per notification type) from delivery (worker pool per platform). APNs/FCM natively batch up to 5000 tokens per HTTP/2 request. At 1M/s: 200 requests/s per platform to FCM — well within their limits. Scale worker pods horizontally; the platform adapter is stateless."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode — Hard DP Timed Set

**Goal:** Attempt Palindrome Partitioning II (LC 132) under 30-min time limit without looking at today's notes.

**Session structure:**
- 0:00 — Read problem; write `is_pal` construction from memory (5 min)
- 5:00 — Write DP recurrence in comments before coding (2 min)
- 7:00 — Code and submit (20 min cap)
- 27:00 — If not passing: review your `is_pal` fill order (are you filling short lengths first?)
- 40:00 — Attempt LC 1312 (Minimum Insertion Steps) — LCS approach only (20 min)
- 60:00 — Debrief: which step cost the most time?

**Debrief prompt:** Did you build `is_pal` correctly the first try? The fill order `(i from n-1 down to 0, j from i to n-1)` is the most common bug. Write it once more from memory before sleeping.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you had to deliver a project with an unclear or changing specification. How did you manage it?

**Target LP:** *Deliver Results* (Amazon) — despite ambiguity, find a path to completion.

**Tip:** Show that you drove clarity rather than waiting for it. "I scheduled a meeting to align on acceptance criteria" is stronger than "I worked with what I had."

---

## Flashcards

| Q | A |
|---|---|
| Write the is_pal DP recurrence (bottom-up). | `is_pal[i][j] = (s[i] == s[j]) AND (j - i < 2 OR is_pal[i+1][j-1])`. Fill: start with `length=1` (diagonal = True), then `length=2`, then `length ≥ 3`. OR fill `i` from `n-1` down to `0`, `j` from `i` to `n-1`. |
| What is the recurrence for Palindrome Partitioning II? | `dp[i] = 0` if `is_pal[0][i-1]`. Otherwise `min(dp[j] + 1)` for all `j ∈ [1,i)` where `is_pal[j][i-1]` is True. Represents: cut before position `j`, where `s[j:i]` is a palindrome. |
| How does Minimum Insertion Steps reduce to LCS? | `min_insertions = n - LPS(s)` where LPS = length of Longest Palindromic Subsequence = `LCS(s, reverse(s))`. Every character not in the LPS must be "mirrored" by an insertion. |
| Write the interval DP recurrence for Minimum Insertions (LC 1312). | If `s[i] == s[j]`: `dp[i][j] = dp[i+1][j-1]`. Else: `dp[i][j] = 1 + min(dp[i+1][j], dp[i][j-1])`. Fill by increasing substring length. Base: `dp[i][i] = 0`. |
| In Notification System design, what happens when APNs returns an "Unregistered" error? | The push token is no longer valid (app uninstalled or notifications disabled). The adapter must immediately delete this token from the Device Token Registry to avoid wasting future sends and stay within platform rate limits. |

---

## Checklist

- [ ] Built `is_pal` table from memory for `"aabbc"` (verify each entry)
- [ ] Solved Palindrome Partitioning II (LC 132) without notes, timed 30 min
- [ ] Explained LPS → min insertions connection in one sentence from memory
- [ ] Drew Notification System architecture: Kafka → Fan-out Worker → APNs/FCM adapters
- [ ] Described device token staleness handling and why it matters at scale
- [ ] Completed 1-hour assessment with debrief recorded
- [ ] Behavioral STAR delivered aloud; focused on how clarity was driven
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved / uncertain problems to `knowledge-base/revision-log.md`
