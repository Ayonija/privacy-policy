# Day 36 — Hashing: Prefix Sum Patterns II & In-Place Hashing
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Extend prefix-sum HashMap to XOR-balance (Contiguous Array) and modular arithmetic, then master the in-place hashing trick for First Missing Positive.

---

## DSA (2 hours)
### Pattern: Prefix XOR/Sum Mod + In-Place Index as Hash

**Contiguous Array (balance trick):**
Replace every 0 with −1; then a subarray has equal 0s and 1s iff its sum = 0. With prefix sums: subarray `[l+1, r]` has sum 0 iff `prefix[r] == prefix[l]`. Store the *first index* where each prefix sum appears; the longest subarray ending at r is `r − first_seen[prefix[r]]`.

**Subarray Sums Divisible by K:**
Two prefix sums having the same value mod k means the subarray between them is divisible by k. Normalise: `(prefix % k + k) % k` handles negative numbers. Count pairs with equal normalised remainder.

**First Missing Positive (in-place hashing):**
The answer must be in `[1, n+1]`. Use the array *itself* as a hash table: for each valid value v ∈ [1, n], mark `nums[v-1]` negative. After marking, the first index i where `nums[i] > 0` means `i+1` is missing.

**Trigger condition:**
- "longest subarray with equal count of two values" → replace one with −1, prefix sum + first-seen map
- "subarrays divisible by k" → prefix mod map with normalised negative mod
- "first missing positive in O(n) time O(1) space" → in-place sign-marking

**Time complexity:** O(n) for all | Space complexity: O(n) for prefix maps, O(1) for First Missing Positive

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Contiguous Array | 525 | Medium | Prefix sum + first-seen index map | Replace 0→−1; find longest subarray summing to 0 using prefix sum first-seen indices |
| 2 | Subarray Sums Divisible by K | 974 | Medium | Prefix mod map | Count pairs with same `(prefix % k + k) % k`; init `mod_map[0] = 1` |
| 3 | First Missing Positive | 41 | Hard | In-place sign-marking (index as hash) | Mark `nums[v-1]` negative for each v ∈ [1,n]; first positive index i → answer is i+1 |

---

### Code Skeleton
```python
# Contiguous Array (LC 525)
def find_max_length(nums):
    first_seen = {0: -1}   # prefix_sum → first index seen
    prefix = result = 0
    for i, n in enumerate(nums):
        prefix += 1 if n == 1 else -1
        if prefix in first_seen:
            result = max(result, i - first_seen[prefix])
        else:
            first_seen[prefix] = i
    return result

# Subarray Sums Divisible by K (LC 974)
def subarrays_div_by_k(nums, k):
    mod_map = {0: 1}
    prefix = count = 0
    for n in nums:
        prefix = (prefix + n) % k
        if prefix < 0: prefix += k    # normalise
        count += mod_map.get(prefix, 0)
        mod_map[prefix] = mod_map.get(prefix, 0) + 1
    return count

# First Missing Positive (LC 41)
def first_missing_positive(nums):
    n = len(nums)
    # Step 1: clean — replace non-positive and > n with n+1 (out of range)
    for i in range(n):
        if nums[i] <= 0 or nums[i] > n:
            nums[i] = n + 1
    # Step 2: mark — for each value v in [1,n], negate nums[v-1]
    for i in range(n):
        v = abs(nums[i])
        if 1 <= v <= n:
            nums[v - 1] = -abs(nums[v - 1])
    # Step 3: scan — first index i where nums[i] > 0 → answer = i+1
    for i in range(n):
        if nums[i] > 0:
            return i + 1
    return n + 1
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Prefix Sum / In-Place Hashing patterns in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a binary array of 0s and 1s and needed to find the longest contiguous subarray with equal counts of each. The brute-force approach of checking every subarray pair would be O(n²) — roughly 10¹² comparisons for n = 10⁶, which would time out after about 17 minutes."

**Task:** "My goal was to solve this in O(n) time and O(n) space by recognising that equal counts of 0s and 1s is equivalent to a prefix sum reaching zero — a classic complement-lookup pattern."

**Action:** Walk the interviewer through these steps:
1. *Classify the pattern:* "I noticed that replacing each 0 with −1 transforms 'equal counts' into 'prefix sum equals zero', which is exactly the HashMap first-seen pattern."
2. *Initialize:* "I seeded `first_seen = {0: -1}` to handle the case where the entire prefix sums to zero, and set `prefix = 0`."
3. *Core loop logic:* "For each element, I update the prefix sum (+1 for 1, −1 for 0). If this prefix value has been seen before, the subarray between the first occurrence and now sums to zero — record its length. If not, store the current index as the first time we've seen this prefix."
4. *Convergence guarantee:* "Each element is processed exactly once and produces one HashMap lookup and at most one insert — guaranteeing O(n) time and O(n) space."
5. *Duplicate handling / edge case proactivity:* "I store the *first* occurrence of each prefix sum, not the last, because a longer subarray requires the earliest possible starting point."

**Result:** "This reduces time from O(n²) to O(n). For n = 10⁶ elements that's the difference between finishing in 1 ms versus timing out after 17 minutes."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Prefix Sum + HashMap here |
|-------------|---------------------------|--------------------------------------|
| Brute force (nested loops) | When n ≤ 1,000 and code simplicity is paramount | O(n²) vs O(n); for n ≥ 10⁴ brute force times out |
| Sorting by prefix value | Never — destroys index ordering | We need the *first-seen* index, so we cannot sort |

**Why NOT brute force:** O(n²) on n = 10⁶ is ~10¹² ops at 10⁹ ops/sec ≈ 17 minutes; this pattern is O(n).
**Why NOT sorting:** Sorting changes the relative order of prefix values and destroys the index information we need to compute subarray length.

---

### Edge Cases to Trace Before Coding
- LC 525: all zeros, all ones — no equal subarray (return 0)
- LC 974: negative numbers in `nums` — `(prefix + n) % k` can be negative in Python too; normalise with `+ k`
- LC 41: `[1]` → 2; `[1,2,3]` → 4; duplicate values (e.g., `[1,1]`) — second marking of `nums[0]` doesn't change anything since it's already negative

---

## System Design (1 hour)
### Topic: Hash Indexes — When to Use Instead of B+Tree

**Hash Index structure:**
A hash function maps a key to a bucket; each bucket contains (key, row_pointer) pairs. Lookup is O(1) average. Used in PostgreSQL's Hash Index, MySQL MEMORY engine, and in-memory caches (Redis dictionaries).

**When hash indexes beat B+Trees:**
| Operation | Hash Index | B+Tree Index |
|-----------|-----------|-------------|
| Equality lookup (`=`) | O(1) | O(log n) |
| Range query (`BETWEEN`, `>`) | ❌ Not supported | O(log n + k) |
| ORDER BY | ❌ Not supported | ✅ Sorted leaf list |
| Prefix match (`LIKE 'abc%'`) | ❌ Not supported | ✅ |
| Multi-column equality | ✅ Composite hash | ✅ Composite B+Tree |

**Hash index limitations:**
- Cannot support range queries — hash function destroys order
- Must store entire key in bucket — no prefix-compression (B+Tree compresses similar keys)
- Hash collisions degrade performance: a hash bucket with many collisions degrades to O(n) scan
- Resizing (rehashing) when load factor exceeds threshold is expensive

**InnoDB does NOT support persistent hash indexes on disk** — only an adaptive hash index (AHI) in memory as a cache for frequently accessed B+Tree pages. PostgreSQL does support on-disk hash indexes.

**Interview talking point:** "If asked when to use a hash index vs. a B+Tree index, answer: hash indexes are ideal for in-memory hash tables (like Redis) or equality-only lookups on columns that are never ranged. For any production relational DB where you need range queries, ORDER BY, or LIKE prefix matches, B+Trees are the only correct choice — hash indexes literally cannot answer range queries."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Frugality**

**STAR Story — Reusing existing resources instead of allocating new ones**

**Situation:** At my previous role we had a nightly batch job that categorised 50 million user events by type. The original implementation first copied the entire event list into a separate boolean-flag array to mark which event IDs had been "processed", then iterated over the copy to collect uncategorised IDs. For 50 M events, that extra allocation was 400 MB of RAM — enough to push the job into swap on our 4 GB batch servers, which ballooned runtime from 8 minutes to over 40 minutes.

**Task:** I was asked to reduce memory usage of the batch pipeline to fit within a 2 GB working-set budget without slowing it down. My goal was to eliminate the auxiliary allocation entirely while preserving correctness.

**Action:** I recognised that the input array itself was effectively idle after the first read pass — we weren't going back to modify the raw events. I adapted the in-place sign-marking trick: rather than allocating a separate "seen" structure, I used the sign bit of an integer field already present in each event record as a temporary visited flag. Specifically: (1) I validated and normalised IDs in a first pass, replacing out-of-range values with a sentinel. (2) I marked each seen ID by negating the corresponding element at position `id - 1`. (3) I scanned once more to collect any positions still positive — those were the uncategorised events. After the job finished, I restored signs in a final O(n) pass so downstream consumers received clean data. I added a unit test that verified zero net allocations (measured via a custom allocator hook) and ran the suite on 10 random seeds.

**Result:** Memory footprint dropped from 400 MB auxiliary + 400 MB source to zero auxiliary, total working set fell from 850 MB to 420 MB, and runtime went from 40 minutes (swap-bound) to 6 minutes — a 6.7× speedup. The batch server's peak RAM stayed under 512 MB, well within budget, and we avoided the planned hardware upgrade that would have cost $18,000 per year.

---

## Flashcards

| Q | A |
|---|---|
| How do you find the longest subarray with equal 0s and 1s in O(n)? | Replace 0 → −1; store first-seen index for each prefix sum; longest subarray summing to 0 = `current_index − first_seen[prefix]` |
| Why do you store the *first* seen index (not last) for each prefix sum in Contiguous Array? | You want the longest subarray — using the first occurrence of a prefix sum maximises the distance to the current index |
| What are the three steps of the First Missing Positive in-place algorithm? | 1. Clean: replace out-of-range values with n+1. 2. Mark: negate `nums[v-1]` for each v ∈ [1,n]. 3. Scan: first positive index i → answer is i+1 |
| What key operation does a hash index NOT support that a B+Tree does? | Range queries (BETWEEN, >, <) — the hash function destroys key ordering |
| What is InnoDB's Adaptive Hash Index (AHI)? | An automatic in-memory cache that InnoDB builds on top of its B+Tree for frequently accessed pages — not configurable or persistent; transparent to the user |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
