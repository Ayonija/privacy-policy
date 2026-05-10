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
- STAR prompt: Describe a time you leveraged an existing resource (memory, data already in place) to solve a problem more efficiently rather than allocating new resources — analogous to First Missing Positive using the input array itself as a hash table.
- Leadership principle: Frugality

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
