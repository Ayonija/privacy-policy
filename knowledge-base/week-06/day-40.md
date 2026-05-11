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
- STAR prompt: Walk through a performance problem you diagnosed end-to-end — from noticing the symptom to identifying the root cause to applying the fix — mirroring the full index decision framework from symptom to solution.
- Leadership principle: Dive Deep

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
