# Day 40 — Hashing: Slot 4 Synthesis
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Slot 4 close-out: apply HashMap to multi-pass counting (Bulls & Cows), custom comparison sorting, and geometry (Max Points on a Line) — then synthesise the full DB Indexing unit.

---

## DSA (2 hours)
### Pattern: HashMap for Multi-Pass Counting & Slope Normalisation

**Bulls & Cows:**
Two-pass approach: Pass 1 — count exact matches (bulls). Pass 2 — for non-bull positions, count cows using the overlap between guess's non-bull freq map and secret's non-bull freq map.

**Custom Sort String:**
Given an order string `order`, rearrange `s` so chars in `order` come first (in order order), then remaining chars. Build a priority map `{char: index}`; sort `s` using this as the sort key.

**Max Points on a Line:**
For each "origin" point, compute the slope to every other point; the answer is the maximum number of points sharing the same slope from any origin. Key challenge: representing slope exactly without floating-point errors — store as `(dy/gcd, dx/gcd)` with sign normalisation.

**Trigger condition:**
- "count partial matches after counting exact matches" → two-pass HashMap
- "sort by custom priority" → HashMap priority + sort key lambda
- "maximum collinear points" → nested slope HashMap per origin point

**Time complexity:** O(n) for Bulls & Cows | O(n log n) for Custom Sort | O(n²) for Max Points
**Space complexity:** O(1) to O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Bulls and Cows | 299 | Medium | Two-pass frequency map | Pass 1: count bulls (exact position match). Pass 2: cows = sum of min(secret_freq[ch], guess_freq[ch]) for non-bull chars |
| 2 | Custom Sort String | 791 | Medium | Priority HashMap + sort | Build `{char: rank}`; sort `s` with `key=lambda c: priority.get(c, len(order))`; chars not in order go last |
| 3 | Max Points on a Line | 149 | Hard | Nested HashMap + GCD slope normalisation | For each point i, map slope → count; slope = `(dy//gcd, dx//gcd)` with canonical sign; answer = max count + 1 |

---

### Code Skeleton
```python
# Bulls and Cows (LC 299)
def get_hint(secret, guess):
    bulls = sum(s == g for s, g in zip(secret, guess))
    # For cows: count overlap in non-bull positions
    s_freq, g_freq = {}, {}
    for s, g in zip(secret, guess):
        if s != g:
            s_freq[s] = s_freq.get(s, 0) + 1
            g_freq[g] = g_freq.get(g, 0) + 1
    cows = sum(min(s_freq.get(c, 0), g_freq.get(c, 0)) for c in g_freq)
    return f"{bulls}A{cows}B"

# Custom Sort String (LC 791)
def custom_sort_string(order, s):
    priority = {ch: i for i, ch in enumerate(order)}
    return "".join(sorted(s, key=lambda c: priority.get(c, len(order))))

# Max Points on a Line (LC 149)
from math import gcd
def max_points(points):
    if len(points) <= 2: return len(points)
    result = 2
    for i in range(len(points)):
        slopes = {}
        for j in range(i + 1, len(points)):
            dy = points[j][1] - points[i][1]
            dx = points[j][0] - points[i][0]
            if dx == 0:
                slope = (1, 0)        # vertical line
            else:
                g = gcd(abs(dy), abs(dx))
                # normalise sign: dx always positive
                if dx < 0: dy, dx = -dy, -dx
                slope = (dy // g, dx // g)
            slopes[slope] = slopes.get(slope, 0) + 1
            result = max(result, slopes[slope] + 1)  # +1 for the origin point i
    return result
```

---

### Interview Tips

- **LC 299 (Bulls & Cows) — narrate two passes:** "Pass 1 counts exact matches (bulls). Pass 2 counts positional mismatches that share a character — cows = overlap between secret's non-bull freq map and guess's non-bull freq map." Interviewers test that you don't accidentally double-count bulls as cows.
- **LC 149 (Max Points) — why GCD normalisation:** "Two slopes are the same if and only if `(dy/dx)` is equal as a fraction. I represent the slope as `(dy // gcd, dx // gcd)` with `dx` normalised to positive, so identical slopes hash to identical tuples — no floating-point precision loss."
- **LC 149 — the `+1` for the origin:** "The slope map counts *other* points collinear with origin `i`. The answer for that origin is `max_count + 1` (include `i` itself)." Forgetting `+1` is the most common LC 149 bug.
- **Custom Sort String alternative:** instead of `sorted`, use a counting approach — count chars in `order` that appear in `s`, output in order order, then append remaining chars. O(n + |order|) vs O(n log n) for sort — offer both.
- **Common mistake in LC 299:** counting a bull position's character in the cow frequency maps — the two-pass approach avoids this by only adding non-bull positions to the freq maps.

### STAR Interview Framework

> **How to use the STAR method when explaining Multi-Pass HashMap Counting & Slope Normalisation patterns in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a secret number and a guess — both 4-digit strings — and had to compute bulls (exact position matches) and cows (right digit, wrong position). The tricky constraint is that each digit can only be counted once: if the secret is `'1122'` and the guess is `'1111'`, the answer is `2A0B` — not `2A2B` — because the second pair of 1s in the guess doesn't correspond to a real cow in the secret."

**Task:** "My goal was to count bulls and cows correctly in O(n) without double-counting, using two separate frequency passes."

**Action:** Walk the interviewer through these steps:
1. *Pass 1 — exact matches:* "I iterate over both strings simultaneously. Wherever `secret[i] == guess[i]`, I increment `bulls` and skip those characters — they're fully resolved."
2. *Pass 2 — build frequency maps:* "For each non-bull position, I add `secret[i]` to `s_freq` and `guess[i]` to `g_freq`. This isolates the pool of 'available' characters for cow counting."
3. *Count cows:* "For each digit `d` in `g_freq`, the number of valid cows is `min(s_freq.get(d, 0), g_freq[d])` — the overlap between what the secret has left and what the guess has left. Sum over all digits."
4. *Why two passes, not one:* "A single-pass approach risks including bulls in the cow count if you're not careful about when you decrement. Separating the passes eliminates that entire class of bug."
5. *Return:* "`f'{bulls}A{cows}B'` — format and return."

**Result:** "O(n) time, O(1) space (digit alphabet is fixed at 10 characters). For secret and guess strings of length n, this handles all duplicates and position overlaps correctly with zero risk of double-counting."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer two-pass HashMap |
|-------------|---------------------------|------------------------------|
| Single-pass with inline cow tracking | Only if you're very careful about bull exclusion | Error-prone — easy to count bulls twice; two-pass is safer and equally fast |
| Character counting array (size 10) | When alphabet is known and fixed | Slightly more cache-friendly than HashMap, but functionally identical |

**Why two passes:** Separating exact-match detection (pass 1) from frequency overlap counting (pass 2) eliminates the most common bug (double-counting bulls as cows). The cost is negligible — still O(n) with very low constant.

---

### Edge Cases to Trace Before Coding
- LC 299: `secret = "1122"`, `guess = "2211"` — bulls = 0, cows = 4; trace pass 1 and pass 2 explicitly
- LC 791: chars in `order` that don't appear in `s` — safe to include in priority map; `priority.get(c, len(order))` defaults to end position ✓
- LC 149: all points identical → every pair has slope (0, 0) (or vertical); must handle: `gcd(0, 0)` is 0 — guard: if `dy == dx == 0`, skip (same point) or handle duplicates separately
- LC 149: exactly 1 or 2 points → early return avoids division edge cases; any 2 points always define a line

---

## System Design (1 hour)
### Topic: DB Indexing — Slot 4 Full Synthesis

**Complete index decision framework:**

```
New query is slow?
  ├─ Is it a full-text search? → Inverted index (Elasticsearch / pg tsvector)
  ├─ Is it an equality-only lookup on high-cardinality column? → Hash index (in-memory) or B+Tree
  ├─ Is it a range / ORDER BY / prefix LIKE? → B+Tree index
  │    ├─ Single column? → Single-column B+Tree
  │    └─ Multi-column WHERE? → Composite index (equality cols first, range col last)
  ├─ Is it frequently paired with certain SELECT columns? → Consider covering index
  ├─ Is it only on a subset of rows? → Consider partial index
  └─ Is the table write-heavy? → Re-evaluate; remove unused indexes first
```

**All index types covered this slot:**
| Index Type | Data Structure | Best For | Limitation |
|-----------|---------------|---------|-----------|
| B+Tree | Balanced tree, sorted leaf list | Ranges, ORDER BY, prefix LIKE | Overhead on write |
| Hash | Hash table | Equality only (in-memory) | No ranges, no ORDER BY |
| Clustered | B+Tree, row data in leaves | PK lookups, ranges on PK | Only 1 per table |
| Covering | B+Tree (includes extra cols) | High-frequency queries | More storage, more write cost |
| Composite | B+Tree on tuple key | Multi-column filters | Prefix rule limits usage |
| Partial | B+Tree with filter | Sparse active rows | Only useful for the specific filter |
| Inverted | Posting lists | Full-text search | Complex to build/maintain |

**The 5 index interview questions:**
1. Why is a full table scan slow? → I/O bounded; 10GB = 1.25M page reads
2. How does B+Tree achieve O(log n)? → Branching factor 200; height 4 for 1B rows
3. Why only one clustered index? → Rows can only be physically sorted one way
4. Why does the query planner ignore your index? → Low selectivity, stale stats, type coercion
5. When to avoid an index? → Low cardinality, write-heavy, tiny table, columns never filtered

**Interview talking point:** "If asked to design the indexing strategy for a `users` table (10M rows) that supports `login by email`, `admin search by name`, `list users by signup date`, and a GDPR audit query, answer: (1) clustered index on `user_id` (PK). (2) Unique B+Tree on `email` (login). (3) Composite B+Tree on `(last_name, first_name)` (admin search). (4) B+Tree on `created_at` (date range queries). (5) Partial index on `(created_at)` WHERE `deleted_at IS NULL` for GDPR active users — avoids indexing deleted accounts."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Dive Deep**

**STAR Story — Diagnosing a performance problem end-to-end**

**Situation:** About three months after launching a new user-search feature, I noticed that the API's P99 latency for `/search/users` had crept from 120 ms to over 2,000 ms over a span of six weeks. The symptom was gradual — no alarm fired immediately — but users started filing support tickets about the search being "broken." The feature searched a `users` table of 15 million rows by partial name match using a `LIKE '%query%'` clause.

**Task:** I volunteered to own the investigation. My goal was to identify the exact root cause and restore P99 latency to under 200 ms without a schema change that would require a maintenance window.

**Action:** I started by running `EXPLAIN ANALYZE` on the slow query: `SELECT id, name FROM users WHERE LOWER(name) LIKE '%alice%'`. The output showed `Seq Scan` — 15 million rows, 1.8 seconds. I checked `pg_stat_user_indexes` and found a B+Tree index on `name` that looked applicable but was being ignored. The issue: the query applied `LOWER()` to `name`, which broke index use (a B+Tree on `name` cannot answer a query on `LOWER(name)` without a functional index). Additionally, the leading wildcard `%alice%` would preclude a standard B+Tree even with the right index. I walked through the full decision tree: full-text substring search → the right structure is an inverted index or a trigram index. I created a GIN trigram index: `CREATE INDEX CONCURRENTLY ON users USING gin(name gin_trgm_ops)`. This ran without locking the table (took 4 minutes on our 15 M rows). After creation, I re-ran `EXPLAIN ANALYZE` — the planner switched to `Bitmap Index Scan` using the trigram index. I also removed the now-unused B+Tree index on `name` to recover write throughput.

**Result:** P99 latency on `/search/users` dropped from 2,000 ms to 85 ms — a 23× improvement — within 15 minutes of the index creation completing. The old B+Tree index removal recovered 12% write throughput on the users table. I wrote an internal guide summarising the decision tree: "LIKE with leading wildcard → trigram GIN index, not B+Tree" and ran a 30-minute knowledge-share with the team to prevent recurrence.

---

## Flashcards

| Q | A |
|---|---|
| How do you count cows in Bulls and Cows without counting bulls again? | Build freq maps only for non-bull positions; cows = sum of `min(s_freq[c], g_freq[c])` for each char in g_freq |
| How do you represent slopes exactly in Max Points on a Line? | `(dy // gcd, dx // gcd)` with sign normalised so dx > 0 (or dx == 0 for vertical lines) — avoids floating-point errors |
| Why does Max Points on a Line add 1 to the slope count? | The slope map counts *other* points; the origin point itself must be included — answer = `slope_count + 1` |
| What is the complete index decision tree for a slow query? | Full-text → inverted index. Equality-only → hash. Range/ORDER BY/prefix → B+Tree. Multi-col → composite (equality first, range last). Sparse rows → partial. Read-optimised → covering |
| What are the 5 index interview questions you must be able to answer cold? | (1) Why is full scan slow? (2) How does B+Tree achieve O(log n)? (3) Why only one clustered? (4) Why does planner ignore index? (5) When to avoid indexing? |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
