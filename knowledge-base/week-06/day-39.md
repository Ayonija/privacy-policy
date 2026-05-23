# Day 39 — Hashing: Advanced Sliding Window & Randomised Design
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Tackle sliding window with recency tracking, fixed-window distinct constraint, and the hardest randomised data structure problem — RandomizedSet with duplicates.

---

## DSA (2 hours)
### Pattern: HashMap Last-Seen Index + Fixed-Window Distinct Constraint + Randomised Structures with Duplicates

**Minimum Consecutive Cards to Pick Up (LC 2260):**
For each card value, track the last index it was seen. When a duplicate is found, the window length = `current_index - last_seen[card] + 1`. Take the minimum over all such windows.

**Maximum Sum of Distinct Subarrays With Length K (LC 2461):**
Fixed-window of size k; maintain a HashMap of `{value: count}`; the window is "distinct" when all counts equal 1 (alternatively track `distinct_count`). Slide: add right element, remove left element; record window sum only if `distinct_count == k`.

**Insert Delete GetRandom O(1) — Duplicates Allowed (LC 381):**
Extension of LC 380: a value can appear multiple times, but `getRandom` must weight by frequency. Solution: store `val → set_of_indices` in HashMap; array holds values. On delete: pick one index from the set, swap with last, update the moved element's set, remove deleted index from its set.

**Trigger condition:**
- "minimum window enclosing two identical values" → HashMap last-seen index
- "maximum sum of k-length window with all distinct" → fixed sliding window + distinct count
- "O(1) insert/delete/getRandom with duplicates" → val→set_of_indices + array

**Time complexity:** O(n) for all | Space complexity: O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Consecutive Cards to Pick Up | 2260 | Medium | HashMap last-seen index | On duplicate: window = `i - last_seen[card] + 1`; take minimum; update last_seen for every card |
| 2 | Maximum Sum of Distinct Subarrays With Length K | 2461 | Medium | Fixed window + distinct tracking | Slide fixed window; maintain `{val: count}` map and `distinct_count`; record sum only when `distinct_count == k` |
| 3 | Insert Delete GetRandom O(1) - Duplicates Allowed | 381 | Hard | `val → set_of_indices` + array | On delete: pick any index from set; swap with array tail; update tail's index set; clean up deleted entry |

---

### Code Skeleton
```python
# Minimum Consecutive Cards to Pick Up (LC 2260)
def minimum_card_pickup(cards):
    last_seen = {}
    result = float('inf')
    for i, card in enumerate(cards):
        if card in last_seen:
            result = min(result, i - last_seen[card] + 1)
        last_seen[card] = i
    return result if result != float('inf') else -1

# Maximum Sum of Distinct Subarrays Length K (LC 2461)
def maximumSubarraySum(nums, k):
    freq = {}
    window_sum = distinct = result = 0
    for right in range(len(nums)):
        freq[nums[right]] = freq.get(nums[right], 0) + 1
        if freq[nums[right]] == 1: distinct += 1
        window_sum += nums[right]
        if right >= k:
            left_val = nums[right - k]
            freq[left_val] -= 1
            if freq[left_val] == 0:
                distinct -= 1
                del freq[left_val]
            window_sum -= left_val
        if right >= k - 1 and distinct == k:
            result = max(result, window_sum)
    return result

# Insert Delete GetRandom O(1) Duplicates Allowed (LC 381)
import random
from collections import defaultdict
class RandomizedCollection:
    def __init__(self):
        self.val_to_indices = defaultdict(set)  # val → set of indices in array
        self.arr = []

    def insert(self, val):
        self.arr.append(val)
        self.val_to_indices[val].add(len(self.arr) - 1)
        return len(self.val_to_indices[val]) == 1  # True if first occurrence

    def remove(self, val):
        if not self.val_to_indices[val]: return False
        remove_idx = next(iter(self.val_to_indices[val]))   # any index of val
        last_val = self.arr[-1]
        # swap remove_idx with last
        self.arr[remove_idx] = last_val
        self.val_to_indices[last_val].add(remove_idx)
        self.val_to_indices[last_val].discard(len(self.arr) - 1)
        self.val_to_indices[val].discard(remove_idx)
        self.arr.pop()
        return True

    def getRandom(self):
        return random.choice(self.arr)
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Sliding Window + HashMap Last-Seen patterns in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a deck of cards and asked to find the minimum number of consecutive cards to pick up that contains at least one matching pair. The brute-force approach of checking every contiguous subarray for a duplicate is O(n²) — for a deck of 10⁵ cards that's 10¹⁰ comparisons, which times out."

**Task:** "My goal was to solve this in O(n) by maintaining a last-seen index for each card value so I could detect the most recent duplicate in a single pass."

**Action:** Walk the interviewer through these steps:
1. *Key insight:* "When I see card value `c` at index `i`, if I've seen `c` before at index `j`, then the subarray `cards[j..i]` contains exactly one duplicate pair. Its length is `i - j + 1`. I want the minimum such length."
2. *Data structure:* "I use a `last_seen = {}` HashMap mapping each card value to the most recent index where it appeared. Initialised empty."
3. *Single-pass logic:* "For each card at index `i`: if `last_seen[card]` exists, candidate window = `i - last_seen[card] + 1`; update the running minimum. Then always set `last_seen[card] = i` — even for the first occurrence — so the next appearance of this card checks against the most recent position."
4. *Why update even on first occurrence:* "If I only wrote to `last_seen` on duplicates, I'd miss the chance to tighten the window when the same card appears a third time at an even closer position."
5. *Return value:* "If no duplicate was found, `result` stays at `inf` — return -1. Otherwise return the minimum window length."

**Result:** "Single pass, O(n) time, O(n) space for the HashMap. For n = 10⁵ cards that's 100,000 operations instead of 10 billion — the algorithm runs in under 1 ms."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer last-seen HashMap here |
|-------------|---------------------------|----------------------------------|
| Brute force (nested loops) | n ≤ 1,000 | O(n²) — times out for n ≥ 10⁴ |
| Sliding window with set | Variable-size window up to first duplicate | Correct but requires shrink logic; last-seen is simpler and faster |
| Sort + scan | Find any duplicate, not minimum window | Sorting destroys positional information — can't recover window length |

**Why NOT sliding window with a set:** a set-based variable-window approach works for "does a duplicate exist" but requires a shrink loop that in the worst case processes each element twice. The last-seen HashMap is one clean forward pass.

---

### Edge Cases to Trace Before Coding
- LC 2260: no duplicates at all → return -1
- LC 2461: k = 1 → every single element is a valid subarray; answer = max(nums) when all distinct
- LC 381: removing a value that equals the last element — `remove_idx == len(arr)-1`; the swap is a no-op but must still remove from the set correctly

---

## System Design (1 hour)
### Topic: When NOT to Index & Index Maintenance Overhead

**When NOT to index:**
1. **Low-cardinality columns:** `status` (3 values), `gender` (2 values), `is_deleted` — query planner ignores low-selectivity indexes; they waste storage and slow writes
2. **Write-heavy tables:** a table receiving 100K inserts/sec cannot afford 5 indexes — each insert requires 5 B+Tree updates; consider a single primary key index only and use a read replica or materialised view for queries
3. **Small tables:** for a table with < 1000 rows, a sequential scan is as fast as an index scan (same page reads) — no index needed
4. **Columns never queried in WHERE/JOIN:** indexes on columns only used in SELECT have zero benefit
5. **Columns that are always null:** nulls are typically not indexed; a query `WHERE col = 1` on a 99%-null column might still scan most of the table

**Index maintenance overhead — quantified:**
- Each index adds ~10–15% write overhead per index on typical workloads
- A table with 5 indexes on a 10K insert/sec workload: each insert does 6 B+Tree writes (1 table + 5 indexes) — effective throughput drops by up to 50% vs. un-indexed
- Index bloat: deleted rows leave "dead" index entries until VACUUM (PostgreSQL) or purge runs — monitor for index bloat in long-running tables

**Partial indexes:**
Index only rows matching a condition: `CREATE INDEX ON orders(user_id) WHERE status = 'pending'`. Much smaller index; much faster scans for the filtered case.

**Interview talking point:** "If asked how to improve write throughput on a table with many indexes, answer: audit index usage with `pg_stat_user_indexes` (PostgreSQL) or `sys.dm_db_index_usage_stats` (SQL Server) — remove indexes with zero or very low reads. Consider partial indexes (index only active/pending rows). For extremely write-heavy loads, bulk-insert without indexes, then rebuild them — PostgreSQL's `CREATE INDEX CONCURRENTLY` does this without locking."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Insist on the Highest Standards**

**STAR Story — Removing overhead that was adding cost without benefit**

**Situation:** Our data pipeline had an audit-logging middleware that wrote a structured JSON log entry for every API request — including internal health-check pings from our load balancer. Load balancers were hitting each of our 40 service instances 60 times per minute for health checks, generating 2,400 log writes per minute — 3.4 million per day — to a shared PostgreSQL audit table. The table had grown to 120 GB over eight months and its two broad indexes (one on `timestamp`, one on `service_name`) were consuming an additional 18 GB. Write throughput on the audit table had degraded to the point where the audit insert was adding 80 ms of latency to every real user request, because the connection pool was frequently saturated by health-check writes.

**Task:** I was assigned to reduce the overhead of the audit logging system to below 5 ms per request without losing any audit data that compliance required.

**Action:** I first mapped which log entries were actually read during audits by querying `pg_stat_user_indexes` and interviewing the compliance team. I found: (1) health-check entries had never been queried in any audit — they were pure noise. (2) The `timestamp` index was used in every audit query, so it stayed. (3) The `service_name` index was used in only 3 of 200 queries per month — compliance agreed it could be dropped and they'd filter in application code for those 3 cases. I implemented a middleware filter that skipped audit logging for requests matching the health-check User-Agent, then dropped the `service_name` index using `DROP INDEX CONCURRENTLY` to avoid table locks. I also added a 90-day partition-expiry policy to prune old rows automatically, capping the table at 15 GB steady-state.

**Result:** The health-check filter eliminated 3.4 million unnecessary writes per day — write load on the audit table dropped by 78%. Dropping the `service_name` index reduced write overhead per remaining log entry from 80 ms to under 4 ms, meeting the 5 ms target. The table shrank from 120 GB to 15 GB over the following quarter with no compliance gaps, freeing up 120 GB of disk and reducing monthly storage costs by $340.

---

## Flashcards

| Q | A |
|---|---|
| How does Minimum Consecutive Cards to Pick Up find the minimum window? | For each card: if seen before, record `i - last_seen[card] + 1`; always update `last_seen[card] = i`; return the minimum recorded window |
| Why must you also update `last_seen` for cards without duplicates? | The same card may form a shorter window with a later occurrence; always updating ensures you always compare against the most recent previous position |
| What is the critical edge case when removing an element equal to the last array element in LC 381? | `remove_idx == len(arr) - 1` — the "swap" is with itself; you must still remove the index from `val_to_indices[val]` and NOT accidentally add it back |
| When should you NOT add an index to a column? | When the column has low cardinality (few distinct values), the table is small (< ~1000 rows), the column is never used in WHERE/JOIN, or the table is extremely write-heavy and can't afford the index maintenance overhead |
| What is a partial index and when is it most useful? | An index with a WHERE clause — only indexes rows matching the condition; ideal for filtering active/pending/non-null rows where most rows don't qualify (e.g., 5% of rows are "active") |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
