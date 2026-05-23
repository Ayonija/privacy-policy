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
```java
class Solution {
    // Top K Frequent Elements — bucket sort (LC 347)
    public static int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int n : nums) {
            freq.put(n, freq.getOrDefault(n, 0) + 1);
        }
        List<List<Integer>> buckets = new ArrayList<>();
        for (int i = 0; i <= nums.length; i++) buckets.add(new ArrayList<>());
        for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
            buckets.get(e.getValue()).add(e.getKey());
        }
        List<Integer> result = new ArrayList<>();
        for (int i = buckets.size() - 1; i > 0; i--) {
            result.addAll(buckets.get(i));
            if (result.size() >= k) {
                int[] arr = new int[k];
                for (int j = 0; j < k; j++) arr[j] = result.get(j);
                return arr;
            }
        }
        int[] arr = new int[result.size()];
        for (int j = 0; j < arr.length; j++) arr[j] = result.get(j);
        return arr;
    }
}

// Maximum Frequency Stack (LC 895)
class FreqStack {
    private Map<Integer, Integer> freq = new HashMap<>();          // val → current frequency
    private Map<Integer, Deque<Integer>> group = new HashMap<>();  // freq → stack of values with that freq
    private int maxFreq = 0;

    public void push(int val) {
        freq.put(val, freq.getOrDefault(val, 0) + 1);
        int f = freq.get(val);
        maxFreq = Math.max(maxFreq, f);
        group.computeIfAbsent(f, k -> new ArrayDeque<>()).addLast(val);
    }

    public int pop() {
        int val = group.get(maxFreq).pollLast();
        freq.put(val, freq.get(val) - 1);
        if (group.get(maxFreq).isEmpty()) {
            maxFreq--;
        }
        return val;
    }
}
```

---

### Edge Cases to Trace Before Coding
- Top K: k equals the number of distinct elements — return all
- Top K Words: equal frequency — heap `(-count, word)` automatically sorts lexicographically
- FreqStack: push same element multiple times — `freq[val]` can grow to n; `group[f]` is a stack not a set

---

### STAR Interview Framework

> **Frequency Map → Top-K via Bucket Sort or Heap:** brute-force O(n log n) full sort → this approach O(n) bucket sort or O(n log k) heap, O(n) space

**S:** "Given an array of n integers, return the k most frequent elements. Sorting all n elements and slicing the top k is O(n log n) — for n=10⁶ and k=5 that's paying for the full sort when we need a tiny fraction."
**T:** "Need O(n) via bucket sort (when input is bounded) or O(n log k) via a min-heap of size k."
**A (60% of answer time):**
1. *Classify:* "'k most/least frequent elements with bounded input' → bucket sort O(n). 'Top-k with unbounded or string keys' → min-heap of size k, O(n log k). 'Pop most recent max-frequency element' → Maximum Frequency Stack (two HashMaps)."
2. *Init:* "Build `freq` HashMap in one pass. For bucket sort: `buckets[i]` = list of elements with frequency i (size n+1). For heap: min-heap capped at size k."
3. *Loop/Step:* "Bucket sort: place each element in `buckets[freq[elem]]`; scan from right to collect k elements. Heap: push `(freq, elem)`; if `heap.size() > k`, poll min — smallest-frequency element is evicted."
4. *Termination:* "Bucket sort: O(n) single scan. Heap: O(n log k) with k≪n being essentially O(n) in practice."
5. *Gotcha:* "For Top K Frequent Words with lexicographic tie-breaking, push `(-count, word)` as a tuple — Python/Java tuple comparison breaks ties on the second element automatically. Forgetting `-count` inverts the frequency order. State this before coding."
**R:** "O(n) bucket sort vs O(n log n) full sort. At n=10⁶: ~10ms vs ~60ms. Maximum Frequency Stack: O(1) push and pop via freq-indexed stacks — the critical insight is that `max_freq` can only decrease by 1 after a pop."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Full sort + slice | n is small, k is close to n | O(n log n) — unnecessary for small k |
| QuickSelect (nth element) | k-th largest by value | Doesn't handle frequency; wrong problem shape |
| TreeMap ordered by frequency | Need dynamic updates with ordered traversal | O(log n) per op vs O(1) for bucket/stack approach |

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
- Leadership principle: Deliver Results

**Full STAR Story — "Surfacing Top-K Impactful Issues Without a Full Sort":**
**S (20%):** "At an e-commerce company, customer support filed 3,000+ tickets per week. Leadership wanted the top-10 most frequently reported issues surfaced in a weekly summary — the existing approach sorted all 3,000 tickets alphabetically, taking 4 minutes, and was replaced weekly by manual review anyway."
**T:** "I was tasked with building an automated top-10 surfacing tool that ran in under 10 seconds and required no manual intervention."
**A (60% — 'I' not 'we'):** "(1) I profiled the pipeline: the bottleneck was sorting all tickets before slicing — we paid O(n log n) when we only needed the top 10. (2) I replaced the full sort with a frequency map + min-heap of size 10: I counted occurrences for each issue category in O(n), then maintained a heap that kept only the top-10 — O(n log 10) = O(n). (3) I added a bucket-sort fallback for when category count ≤ 50, reducing that path to O(n). (4) I deployed the tool with a scheduled job and added a Slack digest so leadership saw results each Monday morning without opening a dashboard."
**R (20%):** "Runtime dropped from 4 minutes to 6 seconds. The tool surfaced actionable insights 48 hours earlier than the previous manual review cycle. Leadership used the output to prioritise two product fixes that reduced ticket volume by 23% over the next quarter."
*Works for: Deliver Results, Customer Obsession, Invent and Simplify.*

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
