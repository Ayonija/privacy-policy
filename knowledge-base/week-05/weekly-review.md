# Week 05 Review — Days 29–35

## Phase
Phase 1: DSA Mastery | Month 2 (Days 29–30 are Month 1 close-out; Days 31–35 are Month 2)

## Patterns Covered This Week

**Days 29–30 (Slot 3 close-out):**
- Monotonic Stack — Sum of Subarray Minimums (contribution technique)
- Monotonic Stack — Remove K Digits (greedy increasing stack)
- Stack — Baseball Game simulation
- Monotonic Stack — Remove Duplicate Letters (greedy + last-occurrence guard)
- Monotonic Stack — 132 Pattern (right-to-left scan, third variable)
- Stack — Remove Outermost Parentheses (depth counter)

**Days 31–35 (Slot 4 — Hashing begins, M/M/H difficulty):**
- HashMap — Canonical key grouping (Group Anagrams)
- HashMap + Set — Streak-start detection (Longest Consecutive Sequence)
- Sliding Window + 2 freq maps — Minimum Window Substring
- Frequency Map → Bucket Sort (Top K Frequent Elements)
- Frequency Map → Heap with custom comparator (Top K Frequent Words)
- Frequency Map + Stacked groups (Maximum Frequency Stack)
- Prefix Sum + Count Map (Subarray Sum Equals K)
- Prefix Mod Map (Continuous Subarray Sum)
- 2D Prefix Sum (fix row pair + 1D scan) — Number of Submatrices
- HashMap + Doubly Linked List — LRU Cache
- HashMap + Array (swap-with-last) — Insert Delete GetRandom O(1)
- DLL of freq nodes + dual HashMaps — All O'one Data Structure
- Exact-K via at_most(k) − at_most(k−1) — Count Nice Subarrays
- Fix unique-char count + sliding window — Longest Substring ≥ K Repeating
- Word-level sliding window × L offsets — Substring Concatenation of Words

## System Design Topics Covered

**DB Indexing (Days 31–35):**
- Why indexes exist: full table scan O(n) vs B+Tree O(log n); read-write trade-off
- B+Tree structure: branching factor ~200, height 3–4 for billion rows, leaf linked list
- B+Tree operations: search, insert + split, range scan via leaf chain, delete + merge
- Clustered index: rows physically sorted by PK; only one per table; InnoDB PK as clustered index
- Non-clustered (secondary) index: separate B+Tree, double lookup unless covering
- Covering index: all query columns in the index — eliminates main table read
- Composite index prefix rule: equality columns first, range column last, ORDER BY column if possible

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| HashMap — canonical key grouping | | |
| Set + streak detection (Consecutive) | | |
| Minimum Window Substring (have/need) | | |
| Bucket Sort for Top-K | | |
| Maximum Frequency Stack (dual maps) | | |
| Prefix Sum + Count Map | | |
| Prefix Mod Map | | |
| 2D → 1D prefix sum reduction | | |
| LRU Cache (DLL + HashMap) | | |
| RandomizedSet (array + HashMap swap) | | |
| All O'one (DLL freq nodes) | | |
| Exact-K → at_most trick | | |
| Word-level sliding window | | |
| B+Tree structure + operations | | |
| Composite index prefix rule | | |

## Weekly Flashcard Deck
All 35 cards for Days 29–35:

| Q | A |
|---|---|
| How do you handle duplicate minimum values in Sum of Subarray Minimums? | Left boundary: `>=` (strict); right boundary: `>` (non-strict) — prevents double-counting |
| What are the three post-loop steps in Remove K Digits? | 1. Truncate k chars from right if k > 0. 2. Strip leading zeros. 3. Return "0" if empty |
| Why does Zookeeper choose CP over AP? | Coordination data (locks, leader election) must be correct — stale data causes split-brain; unavailability is safer |
| How does Cassandra achieve high availability without a single master? | Gossip protocol for node discovery; any node handles any request (token ring); anti-entropy repair after partition |
| What is the cost difference between DynamoDB eventually vs. strongly consistent reads? | Strongly consistent reads cost 2× read capacity units — must hit the leader shard |
| What is the key guard condition in Remove Duplicate Letters? | `last[stack[-1]] > i` — the character at stack top must appear again later before popping it |
| How do you avoid re-processing chars already in stack in Remove Duplicate Letters? | Maintain an `in_stack` set; skip any char that is already in the set |
| In 132 Pattern, what does the `third` variable represent? | Best candidate for nums[k] (the "2") — the largest value popped because a bigger value arrived |
| What is the interview order for system design? | 1. Clarify requirements + scale. 2. High-level components. 3. Deep-dive hardest piece. 4. Consistency model. 5. Failure handling |
| What system design questions should you always end with? | "What happens when a node crashes?" and "What happens during a network partition?" |
| How do you group anagrams in O(n·k)? | `defaultdict(list)` with key = `tuple(sorted(s))` |
| Why does Longest Consecutive only process streak starts? | `num-1 not in set` ensures each streak counted once; prevents O(n²) re-counting |
| How do `have` and `need` counters work in Minimum Window Substring? | `need` = freq of t; `have` = freq in window; `formed` increments when `have[ch] == need[ch]`; valid when `formed == required` |
| What is the read/write trade-off of a DB index? | Reads: O(log n) instead of O(n); every write must update the index |
| Why is B+Tree preferred over B-Tree for range scans? | B+Tree leaf nodes are linked — range scan traverses the leaf list; B-Tree needs full tree traversal |
| How does bucket sort achieve O(n) for Top K Frequent? | `bucket[freq]` lists; scan right-to-left collecting until k elements gathered |
| How do you break ties in Top K Frequent Words? | Push `(-count, word)` — tuple comparison breaks ties lexicographically on the word |
| How does FreqStack's max_freq stay correct after pop? | If `group[max_freq]` is empty after pop, decrement `max_freq` by 1 |
| What is the branching factor of a B+Tree and why does it matter? | Typically 100–200; higher m → shorter tree → fewer disk I/Os per lookup |
| How does a B+Tree range scan work after finding the start key? | Follow leaf-level `next` pointers — O(k) additional reads for k results |
| Why initialise `prefix_map = {0: 1}` in Subarray Sum Equals K? | Accounts for subarrays starting at index 0 — when `prefix == k` you need 0 already in the map |
| Why store the first-seen index in Continuous Subarray Sum? | You must verify subarray length ≥ 2 using `current_index − stored_index ≥ 2` |
| How do you reduce Number of Submatrices to 1D? | Fix row pair r1,r2; accumulate `col_sum[c]`; run Subarray Sum Equals K on col_sum |
| What triggers a B+Tree leaf split? | Inserting would exceed m−1 entries; split at median and push median key up to parent |
| Why are B+Tree range scans cache-friendly? | Sorted linked leaf list → sequential memory page reads — ideal for CPU prefetcher |
| How does LRU Cache achieve O(1) eviction? | DLL — LRU node is always at the tail (before dummy tail); O(1) removal with prev/next pointers |
| How does RandomizedSet delete in O(1) without shifting? | Swap target with last element; update last element's index in HashMap; pop last |
| What is a covering index? | Includes all columns in the query — DB answers entirely from index, never touches main table |
| Why should InnoDB PKs be small sequential integers, not UUIDs? | UUID is 16 bytes — all secondary indexes bloat; random inserts fragment the B+Tree |
| What is the maximum number of clustered indexes per table? | Exactly one — rows can only be physically sorted one way |
| How does the exact-K → at_most trick work? | `exactly(k) = at_most(k) − at_most(k−1)` |
| Why does LC 395 iterate over unique-char counts 1 to 26? | Fixing unique count creates a valid shrink condition — without it the window has no clean contraction rule |
| What is the composite index prefix rule? | Query must use leftmost columns; a gap in the prefix stops the index from helping |
| How do you choose column order in a composite index? | Equality columns first (highest selectivity first); range column last; ORDER BY column if it follows |
| What makes a word-level sliding window different from char-level? | Steps are `word_len` bytes wide; there are `word_len` different offset starting points to cover all positions |

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| — | — | — | — | — |
