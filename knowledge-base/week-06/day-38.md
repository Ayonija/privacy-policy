# Day 38 — Hashing: String Analysis & LFU Cache Design
**Week 06 | Phase 1: DSA Mastery | Month 2**

## Focus
Characterise string equivalence via frequency multisets, then implement LFU Cache — the most complex data structure design problem in the HashMap canon.

---

## DSA (2 hours)
### Pattern: Frequency Multiset Equivalence + Multi-Level Frequency Data Structure

**Determine if Two Strings Are Close (LC 1657):**
Two strings are "close" if they have: (1) the same set of unique characters, AND (2) the same multiset of character frequencies. You can sort both strings' frequency lists and compare. The character identity check prevents, e.g., `"aab"` and `"bbc"` from being considered close (different character sets).

**Sort Characters By Frequency (LC 451):**
Build a frequency map; sort keys by frequency descending; reconstruct the string by repeating each char by its frequency. Optional: bucket sort for O(n).

**LFU Cache (LC 460):**
Evict the Least Frequently Used key; on tie, evict the Least Recently Used among equal-frequency keys. Requires O(1) for all operations.
- `key → (value, freq)` HashMap for O(1) get/set
- `freq → OrderedDict(key → value)` for O(1) access to all keys at a given frequency (OrderedDict maintains insertion order for LRU tie-breaking)
- `min_freq` variable tracking the current minimum frequency

On `get(key)`: increment freq, move from `freq_map[old_freq]` to `freq_map[old_freq+1]`, update `min_freq` if old bucket empty.
On `put(key)`: if exists, update value + increment freq like `get`. If new: add to `freq_map[1]`, set `min_freq = 1`. If over capacity: evict the oldest key in `freq_map[min_freq]`.

**Trigger condition:**
- "same characters, same frequency distribution" → compare sorted freq lists + char sets
- "evict least frequently used, LRU tie-break" → freq→OrderedDict + min_freq variable

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Determine if Two Strings Are Close | 1657 | Medium | Frequency multiset comparison | Same char set (same keys) AND same sorted frequency values; two checks, not one |
| 2 | Sort Characters By Frequency | 451 | Medium | Frequency map + sort/bucket | Build freq map; sort by freq desc; join `char * freq` for each char in order |
| 3 | LFU Cache | 460 | Hard | freq→OrderedDict + min_freq variable | OrderedDict preserves insertion order (LRU tie-break); `min_freq` resets to 1 only on new key insert |

---

### Code Skeleton
```python
# Determine if Two Strings Are Close (LC 1657)
def close_strings(word1, word2):
    from collections import Counter
    c1, c2 = Counter(word1), Counter(word2)
    return (set(c1.keys()) == set(c2.keys()) and
            sorted(c1.values()) == sorted(c2.values()))

# Sort Characters By Frequency (LC 451)
def frequency_sort(s):
    from collections import Counter
    freq = Counter(s)
    return "".join(ch * count for ch, count in sorted(freq.items(), key=lambda x: -x[1]))

# LFU Cache (LC 460)
from collections import defaultdict, OrderedDict
class LFUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.min_freq = 0
        self.key_map = {}                          # key → [value, freq]
        self.freq_map = defaultdict(OrderedDict)   # freq → {key: value}

    def _update(self, key):
        val, freq = self.key_map[key]
        del self.freq_map[freq][key]
        if not self.freq_map[freq] and freq == self.min_freq:
            self.min_freq += 1
        self.key_map[key] = [val, freq + 1]
        self.freq_map[freq + 1][key] = val

    def get(self, key):
        if key not in self.key_map: return -1
        self._update(key)
        return self.key_map[key][0]

    def put(self, key, value):
        if self.cap == 0: return
        if key in self.key_map:
            self.key_map[key][0] = value
            self._update(key)
        else:
            if len(self.key_map) == self.cap:
                # evict LRU key at min_freq
                evict_key, _ = self.freq_map[self.min_freq].popitem(last=False)
                del self.key_map[evict_key]
            self.key_map[key] = [value, 1]
            self.freq_map[1][key] = value
            self.min_freq = 1
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining LFU Cache design in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was asked to design a cache that evicts the least frequently used key and, on a tie, evicts the least recently used among tied keys — all get, put, and evict operations must be O(1). The obvious approach of sorting keys by frequency on every eviction is O(k log k) where k is the cache size — on a cache of 100,000 entries, that's 1.7 million operations per eviction, which would cripple a high-throughput API serving 50,000 requests per second."

**Task:** "My goal was to design a data structure that achieves O(1) for all operations by maintaining the frequency ordering incrementally rather than recomputing it."

**Action:** Walk the interviewer through these steps:
1. *Identify the three invariants:* "I need: (1) O(1) lookup of a key's value and current frequency. (2) O(1) promotion of a key from frequency f to f+1. (3) O(1) eviction of the LRU key at the minimum frequency."
2. *Choose the data structures:* "I use three data structures: `key_map = {key: [value, freq]}` for O(1) key lookup. `freq_map = {freq: OrderedDict(key → value)}` — an OrderedDict at each frequency level to maintain LRU order within that level. A `min_freq` integer tracking the current minimum."
3. *Implement `get`:* "On get: look up key in `key_map`. Remove it from `freq_map[freq]`. Add it to `freq_map[freq + 1]`. If `freq_map[freq]` is now empty and `freq == min_freq`, increment `min_freq`. All O(1) amortised."
4. *Implement `put`:* "On put of a new key: if at capacity, evict with `freq_map[min_freq].popitem(last=False)` — the oldest insertion at the minimum frequency. Insert the new key at `freq_map[1]` and reset `min_freq = 1`. On update of an existing key: same as get, then update the value."
5. *Why OrderedDict for LRU tie-breaking:* "OrderedDict maintains insertion order. `popitem(last=False)` removes the oldest-inserted key in O(1) — that's the LRU key among equal-frequency keys."

**Result:** "All three operations are O(1). For a cache of 100,000 entries at 50,000 req/sec, eviction takes microseconds instead of 1.7 million comparisons. The design scales to any cache size without degrading."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer freq→OrderedDict + min_freq |
|-------------|---------------------------|----------------------------------------|
| Sort by (freq, timestamp) on eviction | Cache is tiny (< 100 entries) | O(k log k) eviction — catastrophic at scale |
| LRU only (LC 146) | Access pattern is temporal, not frequency-based | Doesn't track frequency — wrong eviction policy |
| MinHeap of (freq, timestamp, key) | Acceptable O(log k) eviction | Lazy deletion complicates `put`; O(log k) not O(1) |

**Why NOT a heap:** A min-heap of `(freq, timestamp, key)` gives O(log k) eviction, not O(1). It also requires lazy deletion to handle key updates, adding complexity. The OrderedDict approach achieves true O(1) for all operations.

---

### Edge Cases to Trace Before Coding
- LC 1657: strings of different lengths → always False (Counter values will differ)
- LC 451: single char string; all same chars (one bucket)
- LC 460: capacity = 0 → return immediately; get on missing key; put updates value of existing key

---

## System Design (1 hour)
### Topic: Full-Text Search Indexes — Inverted Index

**The problem B+Tree can't solve:**
`SELECT * FROM articles WHERE body LIKE '%database%'` — a B+Tree index on `body` cannot help; LIKE with a leading wildcard forces a full table scan.

**Inverted Index:**
Maps each word (token) to the list of document IDs (and positions) containing that word. Structure:

```
"database" → [(doc_1, pos: 5, 42), (doc_3, pos: 12), ...]
"index"    → [(doc_1, pos: 7), (doc_2, pos: 3), ...]
```

**Construction:**
1. Tokenise and normalise (lowercase, remove stop words, stem: "running" → "run")
2. Build a posting list for each token
3. Store on disk sorted by token (B+Tree or sorted file for merge efficiency)

**Query execution:**
- Single term: lookup token → posting list O(log V) where V = vocabulary size
- AND query: intersect two sorted posting lists — O(n₁ + n₂) merge
- Phrase query: require consecutive positions

**Used by:** Elasticsearch (Lucene), PostgreSQL `tsvector`, MySQL FULLTEXT index.

**B+Tree vs. Inverted Index:**
| Use case | B+Tree | Inverted Index |
|---------|--------|---------------|
| Exact/range column match | ✅ | ❌ |
| Full-text word search | ❌ | ✅ |
| Prefix match `LIKE 'abc%'` | ✅ | ✅ (with prefix token) |
| Fuzzy match (typo tolerance) | ❌ | ✅ (edit-distance index) |

**Interview talking point:** "If asked how Google indexes web pages for keyword search, answer: inverted index — for each word, store the list of documents containing it (posting list). Queries intersect posting lists. Modern systems add TF-IDF or BM25 scoring to rank results. The index is so large it's partitioned across thousands of shards with replication for availability."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)

**Leadership principle: Customer Obsession**

**STAR Story — Prioritising the most-used resources under a capacity constraint**

**Situation:** I was working on a recommendation engine that precomputed similarity scores for 8 million product pairs and cached them in memory for sub-millisecond serving. The cache had a hard 2 GB memory budget, but the naive LRU policy we were using kept warm cache entries for popular winter products long after the season ended — because they had been recently accessed during the previous week's spike. Meanwhile, evergreen products that were queried consistently throughout the year kept getting evicted. Customer-facing recommendation latency was spiking to 400 ms for those evergreen products because every cache miss triggered a cold database scan.

**Task:** I was asked to replace the LRU eviction policy with one that better matched our access patterns, with the goal of reducing P95 recommendation latency from 400 ms to under 50 ms for evergreen products — without increasing the memory budget.

**Action:** I analysed the access logs and confirmed that 15% of products accounted for 80% of queries (classic power-law distribution). I proposed switching the cache eviction policy to LFU: products queried more frequently would survive evictions while seasonal spikes wouldn't permanently displace high-frequency items. I implemented the LFU cache using the freq→OrderedDict + min_freq pattern to maintain O(1) eviction — critical because the service handled 30,000 lookups per second and couldn't afford O(log n) overhead. I shadow-tested both policies simultaneously on 5% of traffic for a week, measuring hit rate, P95 latency, and eviction rate per product category. LFU achieved a 91% hit rate versus LRU's 74% for the evergreen segment, while seasonal products' hit rate dropped from 68% to 41% — an acceptable trade-off since they had fallback logic.

**Result:** After full rollout, P95 recommendation latency for evergreen products dropped from 400 ms to 31 ms, exceeding the 50 ms target. Overall cache hit rate improved from 78% to 88%, reducing database read load by 45% and saving roughly $12,000 per month in DB compute costs. Customer click-through rate on recommendations increased by 6% over the following month, which the product team attributed to more relevant, lower-latency results.

---

## Flashcards

| Q | A |
|---|---|
| What are the two conditions for "Two Strings Are Close"? | (1) Same set of unique characters. (2) Same sorted multiset of character frequencies — both conditions must hold |
| Why must `min_freq` only reset to 1 on a NEW key insert (not on `get`)? | On a `get` or `put` update, the min_freq might increase (if its bucket becomes empty), but it never drops below 1. Only when a brand-new key is inserted does min_freq reset to 1 |
| How does LFU use OrderedDict for LRU tie-breaking? | OrderedDict preserves insertion order; `popitem(last=False)` removes the oldest-inserted key in the bucket — the one that has been at this frequency the longest (LRU among equal-frequency keys) |
| What is an inverted index and what problem does it solve? | Maps each word (token) to the list of documents containing it; enables O(log V) word search — impossible with a B+Tree which can't handle `LIKE '%word%'` queries |
| How do you intersect two posting lists in an AND query? | Merge two sorted posting lists using two pointers — O(n₁ + n₂); this is why posting lists are kept sorted by document ID |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
