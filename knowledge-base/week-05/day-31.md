# Day 31 — Hashing: HashMap Fundamentals
**Week 05 | Phase 1: DSA Mastery | Month 2**

## Focus
Master the two foundational HashMap patterns — canonical key grouping and set-based streak detection — plus the hardest sliding window problem.

---

## DSA (2 hours)
### Pattern: HashMap — Canonical Key Grouping & Set Membership

**Core idea:**
A HashMap transforms a recognition problem into an O(1) lookup. The key insight is choosing the *right key* — for anagram grouping the key is a sorted string or a 26-tuple character count; for consecutive sequences the key is "is this element the start of a streak?".

**Trigger condition:**
- "Group items that are equivalent under some transformation" → canonical key HashMap
- "Longest streak / consecutive range" → build a set, only process streak starts
- "Find any X in O(1)" → HashMap lookup

**Time complexity:** O(n·k) for grouping (k = key size) | O(n) for consecutive sequences
**Space complexity:** O(n)

---

**Interview pattern drill — before reading solutions, classify each:**

| Problem Signal | Pattern |
|---------------|---------|
| "group strings that are anagrams" | Sort/tuple as map key |
| "longest consecutive sequence" | Set + only process streak starts |
| "smallest window containing all of t" | Sliding window + frequency map |

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Group Anagrams | 49 | Medium | Sort/tuple as canonical key | Two anagrams produce the same sorted string; use it as the HashMap key |
| 2 | Longest Consecutive Sequence | 128 | Medium | Set + streak-start detection | Only start counting when `num-1` is NOT in the set — avoids O(n²) by ensuring each streak is counted once |
| 3 | Minimum Window Substring | 76 | Hard | Sliding Window + 2 frequency maps | Expand right until all chars covered; contract left while still valid; track `have` vs `need` counter |

---

### Code Skeleton
```python
# Group Anagrams (LC 49)
from collections import defaultdict
def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))   # or: count array as tuple
        groups[key].append(s)
    return list(groups.values())

# Longest Consecutive Sequence (LC 128)
def longest_consecutive(nums):
    num_set = set(nums)
    best = 0
    for n in num_set:
        if n - 1 not in num_set:          # start of a streak
            length = 1
            while n + length in num_set:
                length += 1
            best = max(best, length)
    return best

# Minimum Window Substring (LC 76) — skeleton
def min_window(s, t):
    need = {}   # freq map of t
    have = {}   # freq map of current window
    formed = 0  # how many unique chars in t are satisfied
    required = len(set(t))
    left = 0
    result = (float('inf'), 0, 0)  # (length, left, right)
    for right, ch in enumerate(s):
        # 1. expand: add s[right] to have
        # 2. if have[ch] == need[ch]: formed += 1
        # 3. while formed == required: try to contract left
        pass
    return "" if result[0] == float('inf') else s[result[1]:result[2]+1]
```

---

### Interview Tips

- **State the key-choice insight for Group Anagrams:** "The insight is choosing the canonical key — sorting the characters gives the same key for all anagrams. An alternative is a 26-tuple character count (avoids O(k log k) sorting, gives O(k) per string)." Offer both; use tuple for speed.
- **Streak-start optimization for LC 128:** "I check `num - 1 not in set` to ensure I only start a streak at its true beginning. This prevents revisiting the same streak from every member, keeping the algorithm O(n) instead of O(n²)."
- **LC 76 (Min Window) — state the `have vs need` invariant:** "I maintain `need` (target char frequencies) and a `have` counter (how many distinct chars in the window meet their required frequency). The window is valid when `have == len(need)`."
- **Common mistake in LC 76:** using `len(set(t))` as `required` correctly handles repeated chars in t (e.g., `t = "aa"` needs frequency 2 of 'a', not just presence). But using `len(t)` instead of `len(set(t))` overcounts — always use `len(need)` (the frequency map length).
- **Brute force for all three:** LC 49 O(n·k·log k) sorting all strings; LC 128 O(n²) nested loops; LC 76 O(n²) all substrings — HashMaps reduce all three significantly.

### Edge Cases to Trace Before Coding
- Group Anagrams: empty string `""` is its own group (sorted `""` = `""`)
- Longest Consecutive: duplicates in input — `set()` removes them before iteration
- Min Window: `t` has duplicate chars (e.g., `t="aa"`) — need count 2, not 1; `have` only increments when window count exactly reaches need count

---

## System Design (1 hour)
### Topic: Why Database Indexes Exist — Full Table Scan vs. B+Tree Lookup

**The problem without an index:**
- A table with 100M rows; query: `SELECT * FROM users WHERE email = 'alice@x.com'`
- Without an index: full table scan — read every row, O(n) I/O operations
- At 100 bytes/row on spinning disk (~100 IOPS): 100M × 100B = 10GB → ~100 seconds
- With a B+Tree index on `email`: O(log n) I/O — ~27 disk reads → milliseconds

**What an index is:**
A separate data structure that maps a column value (key) to the row's disk location (pointer). The DB maintains this structure alongside the table; you pay write cost (update index on INSERT/UPDATE/DELETE) to get O(log n) read cost.

**Core trade-off:**
- **Read-heavy workload:** indexes are pure win — every query is faster
- **Write-heavy workload:** every write must also update the index — can halve write throughput for wide tables with many indexes

**B+Tree overview (teaser for tomorrow):**
- All data in leaf nodes; internal nodes hold only routing keys
- Leaf nodes are linked in a sorted doubly-linked list — enables efficient range scans
- Branching factor ~100–1000 (a node holds 100+ keys) → height of 3–4 for billion-row tables

**Interview talking point:** "If asked why a full table scan is slow, answer: databases are I/O-bound, not CPU-bound. A full scan reads every page on disk; at 8KB page size a 10GB table is ~1.25M page reads. An index reduces this to 3–4 page reads by following a tree path. The cost of indexing is paid on every write."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you reorganised a large dataset or codebase to make a specific type of lookup dramatically faster — analogous to building an index to avoid full-table scans.
- Leadership principle: Invent and Simplify

---

## Flashcards

| Q | A |
|---|---|
| How do you group anagrams in O(n·k) time? | Build a defaultdict; key = `tuple(sorted(s))` for each string; append string to that key's list |
| Why does Longest Consecutive Sequence only process streak starts? | Checking `num-1 not in set` ensures each streak is counted exactly once — prevents O(n²) re-counting of the same streak from every member |
| How do the `have` and `need` counters work in Minimum Window Substring? | `need` = freq map of t; `have` = freq map of current window; `formed` increments when `have[ch] == need[ch]`; window is valid when `formed == required` |
| What is the read/write trade-off of a DB index? | Reads become O(log n) instead of O(n); every write (INSERT/UPDATE/DELETE) must also update the index — penalty scales with index count and table width |
| Why is a B+Tree preferred over a B-Tree for DB range scans? | B+Tree stores all data in leaf nodes linked in a sorted list — range scan just traverses the leaf list; B-Tree stores data in internal nodes requiring tree traversal to collect a range |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
