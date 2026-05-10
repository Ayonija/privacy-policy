# Day 32 — Hashing: Frequency Maps & Top-K Patterns
**Week 05 | Phase 1: DSA Mastery | Month 2**

## Focus
Frequency maps unlock Top-K problems in O(n log k) — master bucket sort and the max-frequency stack as the two exam-critical frequency patterns.

---

## DSA (2 hours)
### Pattern: Frequency Map → Top-K via Bucket Sort or Heap

**Core idea:**
Count occurrences with a HashMap, then retrieve the k most frequent elements. Two approaches:
1. **Min-heap of size k:** O(n log k) — push all, pop when size > k; works for any comparable key
2. **Bucket sort:** O(n) — bucket index = frequency; bucket[i] holds all elements with frequency i; scan from high to low to collect k elements

Maximum Frequency Stack (LC 895) extends this: maintain a HashMap of `freq → stack_of_elements` and a `max_freq` variable; push/pop maintain both maps.

**Trigger condition:**
- "k most/least frequent elements" → bucket sort (if input is bounded) or heap
- "pop the most recently added element with maximum frequency" → freq-to-stack map
- "sort by frequency descending, tie-break alphabetically" → heap with custom comparator

**Time complexity:** O(n) bucket sort, O(n log k) heap | Space complexity: O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Top K Frequent Elements | 347 | Medium | Bucket sort OR min-heap | Bucket sort is O(n): `bucket[freq]` lists; scan right-to-left collecting until k elements gathered |
| 2 | Top K Frequent Words | 692 | Medium | Min-heap with custom comparator | Heap of `(-count, word)` gives most frequent first; ties broken lexicographically by word string |
| 3 | Maximum Frequency Stack | 895 | Hard | Two HashMaps (freq→stack, elem→freq) | On `push`: increment `freq[val]`, append to `group[freq[val]]`, update `max_freq`. On `pop`: pop from `group[max_freq]`, decrement `freq[val]`, if `group[max_freq]` empty decrement `max_freq` |

---

### Code Skeleton
```python
# Top K Frequent Elements — bucket sort (LC 347)
def top_k_frequent(nums, k):
    freq = {}
    for n in nums:
        freq[n] = freq.get(n, 0) + 1
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, count in freq.items():
        buckets[count].append(num)
    result = []
    for i in range(len(buckets) - 1, 0, -1):
        result.extend(buckets[i])
        if len(result) >= k:
            return result[:k]
    return result

# Maximum Frequency Stack (LC 895)
class FreqStack:
    def __init__(self):
        self.freq = {}          # val → current frequency
        self.group = {}         # freq → stack of values with that freq
        self.max_freq = 0

    def push(self, val):
        self.freq[val] = self.freq.get(val, 0) + 1
        f = self.freq[val]
        self.max_freq = max(self.max_freq, f)
        self.group.setdefault(f, []).append(val)

    def pop(self):
        val = self.group[self.max_freq].pop()
        self.freq[val] -= 1
        if not self.group[self.max_freq]:
            self.max_freq -= 1
        return val
```

---

### Edge Cases to Trace Before Coding
- Top K: k equals the number of distinct elements — return all
- Top K Words: equal frequency — heap `(-count, word)` automatically sorts lexicographically
- FreqStack: push same element multiple times — `freq[val]` can grow to n; `group[f]` is a stack not a set

---

## System Design (1 hour)
### Topic: B+Tree Structure — The Heart of Every Relational DB Index

**Structure:**
```
          [30 | 70]          ← root (internal node)
         /    |    \
    [10|20] [40|60] [80|90]  ← internal nodes
    /  \    /  \    /  \
  [leaf nodes — data + next pointer linked list]
```

**Key properties:**
- **Internal nodes:** store only keys (routing information) — no actual row data
- **Leaf nodes:** store keys + row pointers (or actual row data for clustered index) + a `next` pointer linking all leaves in sorted order
- **Order m (branching factor):** each internal node has between ⌈m/2⌉ and m children; typical m = 100–200 in PostgreSQL/MySQL
- **Height:** with m=200 and 1B rows: height = ⌈log₂₀₀(10⁹)⌉ = 4 levels → 4 disk reads to find any row

**Operations:**
- **Search:** traverse root → leaf following key comparisons — O(log_m n) disk reads
- **Range scan:** find start leaf, then follow `next` pointers — O(log_m n + k) where k = result size
- **Insert:** find leaf, insert in sorted order; if leaf overflows (> m-1 keys), split and push median key up
- **Delete:** find and remove; if leaf underflows (< ⌈m/2⌉ keys), rebalance with sibling or merge

**Interview talking point:** "If asked how a B+Tree handles 1 billion rows with only 4 disk reads, answer: with branching factor 200, height = log₂₀₀(10⁹) ≈ 3.9 ≈ 4 levels. Each internal node fits on one 16KB page (200 × 8-byte keys + 200 × 6-byte pointers ≈ 2.8KB). Each level requires one disk I/O. Four disk reads regardless of table size — that's why B+Trees scale to petabyte-scale tables."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to quickly surface the most impactful items from a large backlog — analogous to extracting the top-k most frequent elements without sorting the entire dataset.
- Leadership principle: Deliver Results

---

## Flashcards

| Q | A |
|---|---|
| How does bucket sort achieve O(n) for Top K Frequent Elements? | Create `n+1` buckets indexed by frequency; put each element in its frequency bucket; scan right-to-left collecting until k elements are gathered |
| How do you handle lexicographic tie-breaking in Top K Frequent Words with a heap? | Push `(-count, word)` tuples; Python's tuple comparison breaks ties by the second element (word string) lexicographically |
| How does FreqStack's `max_freq` stay correct after a pop? | If `group[max_freq]` becomes empty after a pop, decrement `max_freq` by 1 — the next highest frequency is always `max_freq - 1` due to how stacks are built |
| What is the branching factor of a B+Tree and why does it matter? | Branching factor m = number of children per internal node (typically 100–200); higher m → shorter tree → fewer disk I/Os per lookup |
| How does a B+Tree range scan work after finding the start key? | Traverse to the leaf containing the start key (O(log n) disk reads); then follow the leaf-level linked list (`next` pointers) until the end key — O(k) additional reads for k results |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
