# Week 18 Review — Days 120–126
**Phase 3 — Mock Interviews | Month 5**

> **Note:** Day 120 content and flashcards are in Slot 12 (`week-18/day-120.md`). This review covers Days 121–126 from Slot 13 plus Day 120 from Slot 12. Add Day 120 flashcards to the deck below after retrieving them from that file.

---

## Phase
Phase 3 — Mock Interviews, Month 5. Focus shifts from pattern learning to interview-simulation fluency: solve hard problems cold, explain every step aloud, and handle full mock interview loops.

---

## Patterns Covered This Week

- Variable Sliding Window + Frequency Map (Minimum Window Substring)
- Monotonic Deque Sliding Window (Window Maximum)
- Exactly-K via AtMost-K Subtraction (Subarrays with K Distinct)
- Fixed-Length Window with Multi-Word Frequency Matching
- Two Pointer with Running Max (Trapping Rain Water)
- Binary Search on Partition (Median of Two Sorted Arrays)
- Heap-Guided Multi-List Two Pointer (Smallest Range Covering K Lists)
- Binary Search on Answer + Sliding Window Count (K-th Smallest Pair Distance)
- Two-Heap Sliding Window with Lazy Deletion (Sliding Window Median)
- Forward-Backward Two Pointer (Minimum Window Subsequence)
- Deferred Effect Tracking via Difference Array (K Consecutive Bit Flips)
- Contribution Counting per Index (Unique Character Substrings)

---

## System Design Topics Covered

**Twitter / X — Days 121–126:**
- **Day 121 — Overview & Scale:** 400M DAU, 500M tweets/day, 1B+ reads/day; hybrid fan-out model (push for normal users, pull for celebrities).
- **Day 122 — Write Path & Storage:** Tweet table schema, Snowflake IDs for time-sortable globally unique IDs, async fan-out via Kafka to Redis timeline caches.
- **Day 123 — Timeline Feed Generation:** Fan-out-on-write vs fan-out-on-read; k-way merge of celebrity tweets with cached timeline; Redis timeline list capped at 800 entries.
- **Day 124 — User Graph & Follow Relationships:** Two separate follow tables sharded by follower_id and followee_id; denormalized follower counters in Redis; offline suggestion computation.
- **Day 125 — Search & Trending Topics:** Elasticsearch for full-text tweet search; Flink streaming for trending hashtag computation with time-decayed sliding window; Redis for trend result cache.
- **Day 126 — Notification Service:** Kafka event ingestion → per-channel workers (push/email/in-app); time-window deduplication in Redis; at-least-once delivery with retry + dead-letter queue.

---

## Problems to Revisit

| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Variable SW + Frequency Map | | |
| Monotonic Deque | | |
| Exactly-K = AtMost-K − AtMost-(K-1) | | |
| Fixed-Length Multi-Word Window | | |
| Two Pointer + Running Max | | |
| Binary Search on Partition (Median) | | |
| Heap + Multi-List Two Pointer | | |
| Binary Search on Answer + SW Count | | |
| Two-Heap Lazy Deletion (Median SW) | | |
| Forward-Backward Two Pointer | | |
| Difference Array + Deferred Effect | | |
| Contribution Counting | | |

---

## Weekly Flashcard Deck
*(Days 121–126 = 30 cards; add 5 cards from Day 120 in Slot 12 to reach 35 total)*

| Q | A |
|---|---|
| How does the `have` counter avoid over-counting in Minimum Window Substring? | `have` only increments when `window[c]` exactly reaches `need[c]`; extra occurrences beyond `need[c]` don't trigger an increment |
| What invariant does the monotonic deque maintain in Sliding Window Maximum? | Indices stored in decreasing order of values; deque front always holds the index of the current window's maximum |
| When do you pop from the back of the monotonic deque? | When `nums[new_i] >= nums[dq[-1]]`; the back element can never be the window max while `nums[new_i]` is present |
| What is the time complexity of Minimum Window Substring and why is it O(n + m)? | Right pointer traverses `s` once; left pointer also moves at most n times; building `need` from `t` is O(m) |
| What causes the `left` pointer to stop moving in Minimum Window Substring? | When `have < required` again — window is no longer valid so we expand right |
| What is the "exactly K" sliding window trick? | `count(exactly K) = atMost(K) - atMost(K - 1)` |
| In the `atMost(k)` helper, why does `right - left + 1` count subarrays correctly? | Every subarray ending at `right` with left boundary from `left` to `right` satisfies the constraint; there are `right - left + 1` such subarrays |
| Why does LC 30 require iterating over `word_len` different starting offsets? | Words only align at multiples of `word_len` from a given start; without all offsets, you miss non-zero-aligned concatenations |
| What is a Snowflake ID and why does Twitter use it for tweet IDs? | 64-bit ID = timestamp + datacenter + machine + sequence; globally unique, time-sortable, no central coordination needed |
| Why does Twitter use async fan-out via Kafka instead of synchronous fan-out? | Sync fan-out ties API write latency to follower count; async decouples them — user sees tweet immediately while fan-out completes in background |
| In the two-pointer Trapping Rain Water solution, why is it safe to process the side with the smaller running max? | That side's water is bounded by its own max (the smaller one); we don't need to know the other side's value |
| What is the partition invariant in Median of Two Sorted Arrays? | `left_a ≤ right_b` AND `left_b ≤ right_a`; all left-half elements ≤ all right-half elements; total left size = ⌊(m+n)/2⌋ |
| How do you handle `partition_a = 0` in LC 4? | `left_a = -infinity`; similarly `right_a = +infinity` when `partition_a = m` |
| Why binary search on the smaller array in LC 4? | Keeps time complexity O(log(min(m, n))); `partition_b` is derived from `partition_a` |
| In Trapping Rain Water (two-pointer), when do you update `left_max` vs add water? | Update `left_max` when `height[left] >= left_max`; add `left_max - height[left]` to water otherwise |
| In LC 632, why do you advance the pointer of the list containing the minimum? | To potentially raise the range's lower bound; minimum determines the range floor |
| What stops the loop in LC 632? | When the list with the current minimum has no more elements — no smaller valid range covering all lists exists |
| What is the answer space for binary search in LC 719? | `[0, nums[-1] - nums[0]]` after sorting the array |
| In `count_pairs_le(D)` for LC 719, why does `right - left` count correctly? | For each `right`, indices `left` through `right - 1` form valid pairs; there are `right - left` of them |
| What query pattern is avoided by denormalizing follower counts? | `SELECT COUNT(*) FROM follows WHERE followee_id = X` — full index scan replaced by a maintained counter |
| How does lazy deletion work in the two-heap sliding window median? | Mark exiting elements in a stale map; pop stale entries from heap tops before reading the median |
| What is the balancing invariant for the two heaps in Sliding Window Median? | `len(lo) == len(hi)` (even k) or `len(lo) == len(hi) + 1` (odd k); and `max(lo) ≤ min(hi)` |
| In LC 727, what does the backward pass accomplish? | Finds the minimum-length window ending at the position identified by the forward pass |
| How is LC 727 different from LC 76? | LC 76: all chars of t present in any order/frequency; LC 727: t must appear as a subsequence (relative order preserved) |
| What data structure powers Twitter's real-time trending topics? | Flink/Spark Streaming with time-decayed hashtag frequency count; results stored in Redis |
| In LC 995, why use a difference array instead of actually flipping? | Flipping updates O(k) elements per op = O(nk) total; difference array marks only start/end of flip effect = O(n) |
| How do you determine the effective bit value at index `i` in LC 995? | `effective = nums[i] XOR (running_flip_count % 2)` |
| In LC 828, what does `(i - prev[i]) * (next_[i] - i)` represent? | Number of substrings where `s[i]` is the only occurrence of its character: left choices × right choices |
| What prevents `s[i]` from being unique in a substring? | Another occurrence of the same character within the substring boundaries |
| In Twitter's notification service, what is the dead-letter queue used for? | Push notifications that failed all retries; used for debugging and potential re-routing to fallback channel |

---

## Mock / Contest Log

| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

---

## Week 18 → Week 19 Transition

**Consolidate before moving on:**
- Any pattern with comfort < 3/5 above: schedule a targeted re-drill day in Week 19
- Review the Twitter system design components covered this week (Days 121–126) — you should be able to draw the full write path + timeline read path + notification flow from memory
- Week 19 continues with advanced sliding window, two-pointer hard problems, and completes the Twitter system design coverage (media, caching, rate limiting)
