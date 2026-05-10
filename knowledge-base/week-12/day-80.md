# Day 80 — Heaps & Tries: Slot 8 Full Synthesis
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Slot 8 close-out: synthesise Heap and Trie patterns with Hand of Straights (greedy sorted map), Camelcase Matching (two-pointer string matching against a pattern), and The Skyline Problem (max-heap with lazy deletion for event-based skyline construction). Synthesise the complete Newsfeed / Social Network decision framework.

---

## DSA (2 hours)
### Pattern: Greedy Sorted Counter + Two-Pointer Pattern Matching + Max-Heap Lazy-Deletion Event Sweep

**Hand of Straights (LC 846):**
Divide `hand` cards into groups of `groupSize` consecutive values. Use a sorted counter (SortedDict or sorted keys + Counter). Repeatedly process the smallest card: try to take `groupSize` consecutive cards starting from it. If any card in the group is missing → return False.

**Camelcase Matching (LC 1023):**
A query matches a pattern if you can insert lowercase letters into the pattern to get the query. Two-pointer check: pointer `p` on pattern, pointer `q` on query. At each query character:
- If `query[q] == pattern[p]` → advance both
- Else if `query[q]` is lowercase → allowed insertion, advance `q` only
- Else `query[q]` is uppercase and ≠ `pattern[p]` → no match

Query matches iff `p == len(pattern)` at the end.

**The Skyline Problem (LC 218):**
Given buildings with `[left, right, height]`, return the skyline as a list of `[x, height]` critical points where the max height changes.

Algorithm: convert buildings to events: `(left, -height)` for "building starts" and `(right, height)` for "building ends". Sort events (ties: start before end; taller before shorter for same start). Process events with a max-heap (negate for Python's min-heap):
- Start event: push height to heap
- End event: lazy delete height from heap (mark in a `removed` counter)
- After each event: if current max height changed → record critical point

**Trigger condition:**
- "partition array into groups of K consecutive values" → sort + Counter; process smallest first; decrease counts
- "check if a string can be formed by inserting lowercase letters into a pattern" → two-pointer matching; uppercase mismatch = fail; lowercase = allowed insertion
- "compute the outer boundary of a set of overlapping rectangles" → event sweep with max-heap; lazy deletion for building ends

**Time complexity:** LC 846: O(n log n) | LC 1023: O(Q × (|query| + |pattern|)) | LC 218: O(n log n)
**Space complexity:** O(n) / O(Q) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Hand of Straights | 846 | Medium | Greedy sorted counter; consume smallest first | Sort keys; consume groupSize consecutive from smallest; fail if any missing |
| 2 | Camelcase Matching | 1023 | Medium | Two-pointer against pattern | Uppercase mismatch = fail; lowercase in query not in pattern = allowed skip |
| 3 | The Skyline Problem | 218 | Hard | Max-heap event sweep with lazy deletion | Events sorted; push height on start, lazy-delete on end; record when max changes |

---

### Code Skeleton
```python
import heapq
from collections import Counter, SortedDict   # SortedDict from sortedcontainers

# Hand of Straights (LC 846)
def isNStraightHand(hand, groupSize):
    if len(hand) % groupSize != 0: return False
    count = Counter(hand)
    for card in sorted(count):
        if count[card] == 0: continue
        n = count[card]   # we need n complete groups starting here
        for k in range(groupSize):
            count[card + k] -= n
            if count[card + k] < 0: return False
    return True

# Camelcase Matching (LC 1023)
def camelMatch(queries, pattern):
    def matches(query, pattern):
        p = 0
        for q_char in query:
            if p < len(pattern) and q_char == pattern[p]:
                p += 1
            elif q_char.isupper():
                return False   # uppercase in query must match pattern
        return p == len(pattern)
    return [matches(q, pattern) for q in queries]

# The Skyline Problem (LC 218)
def getSkyline(buildings):
    # Create events: (x, height) where height < 0 = start, height > 0 = end
    events = []
    for left, right, h in buildings:
        events.append((left, -h))    # start: negative height (processed first on tie)
        events.append((right, h))    # end: positive height
    events.sort()   # sort by x; on same x: start (-h) < end (+h) ← natural sort

    result = []
    heap = [0]   # max-heap (negated), always has 0 as ground level
    removed = Counter()   # lazy deletion tracker
    prev_max = 0

    for x, h in events:
        if h < 0:   # start event: add building height
            heapq.heappush(heap, h)   # h is already negative
        else:       # end event: mark height for lazy deletion
            removed[h] += 1

        # Lazy-clean the top of the heap
        while heap and removed[-heap[0]] > 0:
            removed[-heap[0]] -= 1
            heapq.heappop(heap)

        curr_max = -heap[0]
        if curr_max != prev_max:
            result.append([x, curr_max])
            prev_max = curr_max

    return result
```

---

### Slot 8 Complete Algorithm Reference

| Pattern | Data structure | Core operation | Complexity | Use when |
|---------|---------------|---------------|-----------|---------|
| Top-K frequent | Min-heap size K | Push + pop when > K | O(n log K) | Find K largest/smallest by value |
| K-way merge | Min-heap one-per-stream | Pop min, push next | O(n log K) | Merge K sorted streams |
| Two-heap median | Max-heap lo + min-heap hi | Balance sizes | O(log n) add, O(1) query | Running median |
| Sliding window median | Two heaps + lazy deletion | Add + lazy remove | O(n log k) | Median over a window |
| Greedy two-heap | Min-heap threshold + max-heap value | Move affordable, pop max | O(n log n) | IPO, task scheduling |
| Trie insert/search | TrieNode children + is_end | Char-by-char traversal | O(L) | Prefix queries |
| Trie + DFS wildcard | Trie + recursion | Branch on '.' | O(26^L) worst | Pattern matching |
| Trie + grid DFS | Trie + backtracking | Prune dead nodes | O(m×n×4^L) | Word Search II |
| Binary XOR Trie | Bit-indexed trie | Greedy opposite-bit | O(n × 30) | Max XOR pair |
| Reversed-word Trie | Trie of reversed words | Active search list | O(stream × active) | Stream matching |
| Suffix-wrapped Trie | `suffix#word` trie | Latest index per node | O(n×L²) build, O(L) query | Prefix + suffix search |
| Max-heap skyline | Heap + lazy deletion | Event sweep | O(n log n) | Geometric sweep |

---

### Edge Cases to Trace Before Coding
- LC 846: `groupSize = 1` → always True; hand size not divisible by groupSize → False immediately; cards form groups but not from smallest → sorted approach handles naturally
- LC 1023: pattern is empty → all queries match (all chars are "insertions"); query shorter than pattern → impossible (can't insert to get extra pattern chars); all uppercase pattern → query must match exactly
- LC 218: single building → two critical points: `[left, height]` and `[right, 0]`; buildings at same x → sort ensures starts before ends; all same height → only two critical points

---

## System Design (1 hour)
### Topic: Newsfeed / Social Network — Full Decision Framework

**Master decision tree:**
```
Design a social network newsfeed?
  │
  ├─ Post storage
  │    ├─ Post DB: MySQL sharded by user_id; (user_id, created_at) composite index
  │    ├─ Media: S3 + CloudFront CDN (1-year cache + URL versioning)
  │    └─ Post cache: Redis `post:{post_id}` JSON (1-hour TTL)
  │
  ├─ Social graph
  │    ├─ Follow graph: Cassandra, dual table (by followee_id + by follower_id)
  │    ├─ Celebrity detection: user.follower_count > threshold (1M)
  │    └─ Mutual friends: graph DB for specialised queries; skip for hot path
  │
  ├─ Feed generation (fanout)
  │    ├─ Normal users (< 1M followers): fanout on write via Kafka + workers
  │    │    → ZADD feed:{follower_id} {timestamp} {post_id} in Redis pipelines
  │    ├─ Celebrities (≥ 1M followers): NO fanout → merge at read time
  │    │    → ZADD user_posts:{celebrity_id} {timestamp} {post_id}
  │    └─ Feed store: Redis sorted set, cap 1000 entries, 7-day TTL, active users only
  │
  ├─ Feed read path
  │    ├─ ZREVRANGE feed:{user_id} → regular post_ids
  │    ├─ ZREVRANGE user_posts:{celebrity_id} → celebrity posts (merged at read)
  │    ├─ Batch fetch post details from Redis/DB
  │    └─ Rank by timestamp (chronological) or ML score (algorithmic)
  │
  ├─ Ranking
  │    ├─ Chronological: simple; ZADD score = timestamp
  │    ├─ Algorithmic: score = recency × affinity × engagement; async re-scoring
  │    └─ A/B test: 10% control (chronological) vs treatments (ML models)
  │
  ├─ Notifications
  │    ├─ Action events → Kafka "user_actions" → Notification Fanout Worker
  │    ├─ Deduplication: Redis counter with TTL; batch: "X others liked your post"
  │    └─ Delivery: FCM/APNS (push), SendGrid (email), Redis inbox (in-app)
  │
  └─ Analytics
       ├─ Events: Kafka "user_events" → Flink → ClickHouse
       ├─ Trending: sliding window count + min-heap top-K → Redis sorted set
       └─ Recommendations: 2-hop collaborative filtering (Spark nightly batch)
```

**The 5 questions to answer cold for any social newsfeed:**
1. **What is the fanout model?** Hybrid (write for normal, read for celebrities). Threshold = configurable (typically 1M followers).
2. **Where is the feed stored?** Redis sorted set per user. Score = timestamp or ML score. Capped at 1000 entries.
3. **How is ranking implemented?** Chronological (simple MVP) or algorithmic (recency × affinity × engagement). A/B test to validate.
4. **How are notifications managed?** Event-driven with deduplication. Kafka pipeline. Per-channel delivery workers.
5. **How does the system scale?** Fanout workers scale with post volume. Redis cluster scales with user count. Cassandra scales the social graph.

**Trade-off summary:**

| Design choice | Trade-off |
|--------------|-----------|
| Fanout on write | Fast reads; write amplification for celebrities |
| Redis sorted set | Sub-ms feed reads; memory cost; inactive users need cold start |
| Algorithmic ranking | Higher engagement; filter bubbles; ML complexity |
| Eventual consistency | Fast writes; 1–10s delay before followers see post |
| Cassandra social graph | Fast follower list; no complex queries (mutual friends need graph DB) |

**Interview synthesis talking point:** "Instagram's feed at a billion users runs a hybrid fanout model. For a product manager, I'd lead with: every new post triggers async fanout via Kafka; followers see the post within 1–10 seconds (eventual consistency). For a celebrity with 100M followers, we merge their posts at read time — this trades a slightly slower feed read (one extra Redis lookup per celebrity followed) for the ability to avoid 100M writes per post. The ranked feed uses a model trained on engagement signals; we A/B test all changes against a chronological baseline."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — full Slot 8 synthesis
- **Medium 1:** LC 846 Hand of Straights (target: 15 min)
- **Medium 2:** LC 1023 Camelcase Matching (target: 12 min)
- **Hard:** LC 218 The Skyline Problem (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Walk through a system you built or improved that had to serve personalised content at massive scale — describe the key architectural decisions.
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| How does Hand of Straights greedily form consecutive groups? | Sort keys. For each smallest key with count > 0: take `count` groups starting there. Decrement count for `card, card+1, ..., card+groupSize-1`. If any becomes negative → impossible → return False. |
| How does Camelcase Matching use two pointers? | Pointer `p` on pattern, scan query left to right. If `query[q] == pattern[p]` → advance both. If `query[q]` is lowercase → allowed insertion, advance `q` only. If `query[q]` is uppercase and ≠ `pattern[p]` → return False. Match = `p == len(pattern)`. |
| How does The Skyline Problem handle building ends lazily? | Building ends push the height to a `removed` counter (lazy deletion). Before reading the current max, clean the heap top: while `removed[-heap[0]] > 0`, decrement `removed` and pop. The true current max is then `-heap[0]`. |
| State the 5 questions for a newsfeed system design interview. | (1) Fanout model? Hybrid (write for normal, read for celebrities). (2) Feed store? Redis sorted set. (3) Ranking? Chronological or ML score. (4) Notifications? Kafka + dedup + per-channel workers. (5) Scaling? Fanout workers + Redis cluster + Cassandra. |
| What is the trade-off of algorithmic vs chronological feed ranking? | Algorithmic: higher engagement, longer sessions, personalised. Risks: filter bubbles (over-indexing user interests), ML complexity, less transparency. Chronological: simple, transparent, no filter bubble, but favours high-frequency posters over quality content. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/12/30 min, no hints)
- [ ] Rewrote The Skyline Problem event-sweep and lazy-deletion from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Recited the Slot 8 algorithm reference table from memory
- [ ] Completed system design — recited the 5-question newsfeed decision framework cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
