# Day 73 — Heaps: K Closest Points, Reorganize String, Sliding Window Median
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
K Closest Points shows that "closest" is just a distance comparator — the same min-heap-size-K template applies. Reorganize String is a greedy max-heap problem: always place the most frequent available character that differs from the last placed. Sliding Window Median extends the two-heap median structure with lazy deletion to handle window evictions.

---

## DSA (2 hours)
### Pattern: Max-Heap Top-K by Distance + Greedy Max-Heap Interleaving + Two-Heap Sliding Window

**K Closest Points to Origin (LC 973):**
Distance from origin = `x² + y²` (no need for sqrt). Use a max-heap of size K: push `(distance, point)`; when size > K, pop the maximum. The heap retains the K smallest distances. Alternatively, use `heapq.nsmallest(k, points, key=lambda p: p[0]**2 + p[1]**2)`.

**Reorganize String (LC 767):**
If any character has frequency > `(len(s) + 1) // 2`, it's impossible (return ""). Otherwise: max-heap of `(-freq, char)`. Greedily pick the most frequent char; if it equals the last placed char, temporarily use the second-most-frequent char instead, then put the first back.

Alternative (simpler): pop the top two chars from the heap each step, place them, decrement, push back if freq > 0. This naturally alternates.

**Sliding Window Median (LC 480):**
Maintain a two-heap median structure (lo = max-heap, hi = min-heap) as a sliding window of size k moves through the array. On each step:
1. Add the new element (same as MedianFinder.addNum)
2. Remove the outgoing element (lazy deletion): record it in a `delayed` counter; when the heap's top is a delayed element, discard and re-balance

Lazy deletion: instead of removing from the middle of the heap (O(n)), mark it as "to delete" and skip it when it rises to the top during future pops.

**Trigger condition:**
- "K points/items closest to a reference (Euclidean or other distance)" → max-heap size K; push (dist, item), pop when > K
- "rearrange string so no two adjacent chars are equal" → max-heap greedy; alternate two most-frequent
- "median of a sliding window of size k" → two heaps + lazy deletion for outgoing elements

**Time complexity:** LC 973: O(n log K) | LC 767: O(n log n) | LC 480: O(n log k)
**Space complexity:** O(K) / O(n) / O(k)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | K Closest Points to Origin | 973 | Medium | Max-heap size K by distance squared | No sqrt needed; push (-dist², x, y); pop when size > K |
| 2 | Reorganize String | 767 | Medium | Greedy max-heap; alternate two most frequent | Pop top two; place both; push back with decremented counts |
| 3 | Sliding Window Median | 480 | Hard | Two heaps + lazy deletion for evictions | `delayed` dict tracks elements to skip; re-balance after lazy pops |

---

### Code Skeleton
```python
import heapq
from collections import Counter

# K Closest Points to Origin (LC 973)
def kClosest(points, k):
    heap = []   # max-heap via negation: (-dist², x, y)
    for x, y in points:
        dist = x*x + y*y
        heapq.heappush(heap, (-dist, x, y))
        if len(heap) > k:
            heapq.heappop(heap)
    return [[x, y] for _, x, y in heap]

# Reorganize String (LC 767)
def reorganizeString(s):
    count = Counter(s)
    max_freq = max(count.values())
    if max_freq > (len(s) + 1) // 2: return ""
    heap = [(-freq, char) for char, freq in count.items()]
    heapq.heapify(heap)
    result = []
    while len(heap) >= 2:
        freq1, c1 = heapq.heappop(heap)
        freq2, c2 = heapq.heappop(heap)
        result.extend([c1, c2])
        if freq1 + 1 < 0: heapq.heappush(heap, (freq1 + 1, c1))
        if freq2 + 1 < 0: heapq.heappush(heap, (freq2 + 1, c2))
    if heap:
        result.append(heap[0][1])   # remaining char (freq must be 1)
    return ''.join(result)

# Sliding Window Median (LC 480)
def medianSlidingWindow(nums, k):
    lo = []   # max-heap (lower half, negated)
    hi = []   # min-heap (upper half)
    delayed = Counter()   # elements marked for lazy deletion
    lo_size = hi_size = 0

    def add(num):
        nonlocal lo_size, hi_size
        if not lo or num <= -lo[0]:
            heapq.heappush(lo, -num); lo_size += 1
        else:
            heapq.heappush(hi, num); hi_size += 1
        balance()

    def remove(num):
        nonlocal lo_size, hi_size
        delayed[num] += 1
        if num <= -lo[0]:
            lo_size -= 1
        else:
            hi_size -= 1
        balance()

    def balance():
        nonlocal lo_size, hi_size
        while lo_size > hi_size + 1:
            heapq.heappush(hi, -heapq.heappop(lo)); lo_size -= 1; hi_size += 1
            prune(lo)
        while hi_size > lo_size:
            heapq.heappush(lo, -heapq.heappop(hi)); hi_size -= 1; lo_size += 1
            prune(hi)

    def prune(heap):
        while heap and (heap[0] in delayed if heap is hi else -heap[0] in delayed):
            val = heapq.heappop(heap) if heap is hi else -heapq.heappop(heap)
            delayed[val] -= 1
            if delayed[val] == 0: del delayed[val]

    def get_median():
        if k % 2 == 1: return float(-lo[0])
        return (-lo[0] + hi[0]) / 2.0

    result = []
    for i, num in enumerate(nums):
        add(num)
        if i >= k:
            remove(nums[i - k])
        if i >= k - 1:
            result.append(get_median())
    return result
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Max-Heap Top-K by Distance / Greedy Max-Heap Interleaving / Two-Heap Sliding Window Median in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three heap extension problems: finding the K closest points to the origin among millions of coordinates, reorganizing a character string so no two adjacent characters are identical, and computing the running median over a sliding window of size k. The naive approaches — sorting all points, checking every arrangement, or re-sorting the window on each slide — all give O(n log n) or worse per operation."

**Task:** "My goal was to recognize that distance comparison, greedy character selection, and sliding-window median all reduce to heap operations: the same data structure solves all three with appropriate configuration."

**Action:** Walk the interviewer through these steps:
1. *Classify the pattern:* "K closest to origin → max-heap of size K (pop the farthest when size exceeds K). Reorganize String → max-heap greedy (always place the two most frequent chars, alternating). Sliding Window Median → two heaps with lazy deletion for outgoing elements."
2. *Initialize:* "For K closest: push `(-x²-y², x, y)` — negating distance makes Python's min-heap behave as a max-heap. For Reorganize String: push `(-freq, char)` pairs. For sliding window median: initialize the two-heap structure plus a `delayed` counter for lazy deletion."
3. *Core loop logic:* "For K closest: pop when size > K. For Reorganize String: pop the two most-frequent chars simultaneously, append both, push back if count > 0 — this guarantees no triple run without checking the last placed char. For sliding window median: on each slide, add the new element (standard two-heap add) and mark the outgoing element in `delayed` — only truly remove it when it reaches the heap top."
4. *Convergence guarantee:* "The lazy deletion for sliding window median is O(log k) amortized because each element is pushed and popped at most once — even if it stays 'marked' for several steps before reaching the top."
5. *Duplicate handling / edge case proactivity:* "For Reorganize String, the key edge case is when max_freq > (n+1)//2 — then no valid arrangement exists. I check this immediately before entering the heap loop to fail fast."

**Result:** "K Closest Points: O(n log K) vs O(n log n) for full sort — for n = 10^6 points with K = 50, that's 3× fewer comparisons. Reorganize String: O(n log n) but with small constant (max 3 chars in heap). Sliding Window Median: O(n log k) vs O(n×k) for naive sliding sort — for n = 10^5, k = 1000, that's 100× faster."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer heap here |
|-------------|---------------------------|----------------------|
| `heapq.nsmallest(k, points)` | n is small (n ≤ 10^4) | Built-in convenience; same O(n log K) but no learning signal |
| Interleave at even positions (LC 767) | Guaranteed feasibility known | Simpler O(n) but less generalizable to variable limits |
| Re-sort window each slide (LC 480) | k is tiny (k ≤ 10) | O(k log k) per slide = O(nk log k) total; lazy deletion wins for large k |

**Why NOT re-sort for sliding window median:** O(nk log k) vs O(n log k) — for k=1000, n=10^5, that's 1000× slower.
**Why NOT just `sorted()` for K closest:** O(n log n) every time vs O(n log K) with heap — when K = 10 and n = 10^6, the heap is 100× fewer comparisons.

---

### Edge Cases to Trace Before Coding
- LC 973: k = len(points) → return all points; all points at same distance → any K valid; k = 1 → return single closest
- LC 767: single unique character with count > 1 → impossible; "aab" → "aba" (valid); "aaab" → impossible (3 a's in length 4 → max = 2 → impossible)
- LC 480: k = 1 → median = each element; k = len(nums) → median of full array; all same elements → median constant

---

### Interview Pattern Drill

| Heap problem type | Heap type | Size maintained | Key operation |
|-----------------|-----------|----------------|--------------|
| K closest to reference | Max-heap (by distance) | Exactly K | Pop when size > K |
| Rearrange no-repeat | Max-heap (by freq) | Unbounded | Pop 2 at once; push back |
| Running median | Max lo + min hi | `lo = hi or lo+1` | Add + rebalance |
| Sliding window median | Max lo + min hi + delayed | `lo_size = hi_size or +1` | Add new + lazy remove old |

**Lazy deletion pattern:** When removing an element from the middle of a heap (O(n) naively), instead mark it in a `delayed` counter. When that element reaches the top during future pops, skip it and decrement the counter. Amortized O(log n) removal — each element is pushed and popped at most once.

---

## System Design (1 hour)
### Topic: Newsfeed — Feed Storage, DB Schema, and Media Pipeline

**Post storage schema:**
```sql
-- Post DB (MySQL, sharded by user_id)
CREATE TABLE posts (
    post_id     BIGINT PRIMARY KEY,        -- Snowflake ID
    user_id     BIGINT NOT NULL,
    content     TEXT,
    media_keys  JSON,                      -- S3 object keys for images/videos
    created_at  TIMESTAMP DEFAULT NOW(),
    likes_count BIGINT DEFAULT 0,
    INDEX idx_user_created (user_id, created_at DESC)
);

-- Follow graph (Cassandra, optimised for "get all followers of user X")
-- Partition key = followee_id so we can list all followers of a user efficiently
CREATE TABLE followers (
    followee_id BIGINT,
    follower_id BIGINT,
    created_at  TIMESTAMP,
    PRIMARY KEY (followee_id, follower_id)
);
```

**Why Cassandra for the follow graph?**
- The follow graph is write-heavy (follows/unfollows happen constantly)
- The access pattern is always "give me all followers of user X" — a partition scan by `followee_id`
- No joins needed; Cassandra's wide-row model fits perfectly
- Read: `SELECT follower_id FROM followers WHERE followee_id = ?` → fast partition scan

**Media pipeline:**
```
User uploads image/video
    │
[Upload Service] → direct upload to S3 (presigned URL — no proxy needed)
    │
[Media Processing Lambda]
    ├─ Generate thumbnails (3 sizes: 150px, 600px, 1080px)
    ├─ Transcode video to HLS (adaptive bitrate)
    └─ Store processed media at deterministic S3 keys
    │
[CDN] — origin = S3; cache all media with 1-year TTL + URL versioning
```

**Feed Redis schema:**
```
feed:{user_id}   → Sorted Set
  Score: Unix timestamp (float)
  Member: post_id (string)

Commands:
  ZADD feed:{user_id} {timestamp} {post_id}   → add post to feed
  ZREVRANGE feed:{user_id} 0 19               → get latest 20 posts
  ZREMRANGEBYSCORE feed:{user_id} 0 {cutoff}  → prune old posts (TTL-like cleanup)
```

**Feed size bounding:**
Store at most 1000 posts in each user's feed sorted set. When a new post arrives and the set is at 1000 entries, remove the oldest. This bounds Redis memory: 1000 posts × 8 bytes/post_id × 1 B users = 8 TB — too large. Only cache feeds for **active users** (last login within 30 days). Inactive users → cold start on next login.

**Post DB sharding:**
Shard `posts` table by `user_id % N_shards`. A user's profile page and their timeline both query by `user_id` → no cross-shard joins. The `(user_id, created_at)` composite index makes "get latest posts by this user" fast.

**Interview talking point:** "The posts table is sharded by user_id so all of a user's posts live on one shard — their profile page and feed-construction queries never need cross-shard joins. The follow graph in Cassandra is partitioned by followee_id — fanout workers just do a partition scan to get all followers, no secondary indexes needed. Media lives in S3 behind CloudFront, never touching the application servers after upload."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 973 K Closest Points to Origin (target: 12 min)
- **Medium 2:** LC 767 Reorganize String (target: 18 min)
- **Hard:** LC 480 Sliding Window Median (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)

**Leadership Principle:** Frugality

**STAR Story: Optimising a Media Transcoding Pipeline Using Lazy Deletion and Priority Queues**

**Situation (20%):** "Our video transcoding service maintained a priority queue of upload jobs sorted by file size, processing smaller files first to maximize throughput. When a user cancelled an upload mid-flight, we were removing their job by scanning the entire queue — an O(n) operation that caused 200–400ms stalls on our priority queue during high-traffic periods when the queue held 50,000+ jobs."

**Task (part of S/T):** "I was asked to eliminate the stalls without introducing a more complex data structure (we didn't have budget for a rewrite). My goal was to reduce cancelled-job removal from O(n) to O(log n) amortized without changing the external API."

**Action (60-70% — be specific about what YOU did):**
"First, I profiled the service and confirmed that O(n) scans during cancellation were causing cascading delays — when 5 cancellations arrived simultaneously, we had 5 serial O(n) scans, each taking 150ms.
Then, I proposed lazy deletion: instead of removing the cancelled job from the heap, I'd mark its ID in a `cancelled_set`. When the job reached the heap top during the processing loop, I'd skip it and pop it in O(log n).
Next, I implemented the `cancelled_set` as a Redis set (to survive server restarts) and modified the dequeue loop to check membership before processing each job. The check was O(1) with Redis.
Finally, I measured that each job could be cancelled and lazily evicted with two Redis operations — one SET write on cancel, one DELETE on eviction — replacing the O(n) heap scan with O(log n) amortized work."

**Result (10-20%):** "Queue stall time on cancellation dropped from 150–400ms to under 5ms. At 50,000 jobs in the queue, we went from O(50,000) = O(n) scans to O(log 50,000) ≈ 16 heap operations per cancellation. Transcoding throughput increased by 18% during peak hours because the queue was no longer blocking on cancellations. The solution cost zero additional infrastructure — just a Redis key per cancelled job, which we already had Redis for."

**Interview tip:** Interviewers want to hear about *your* contribution. Say "I profiled", "I proposed", "I implemented", "I measured" — not "we did". Prepare this story for questions about: Frugality, engineering efficiency, bottleneck identification, and clever use of existing infrastructure.

---

## Flashcards

| Q | A |
|---|---|
| How does K Closest Points avoid computing sqrt in the distance? | Distance squared (`x² + y²`) preserves the ordering — if `a² < b²`, then `a < b`. No need for sqrt; push `(-x²-y², x, y)` to a max-heap of size K. |
| How does Reorganize String handle the case where the most frequent char equals the last placed? | Pop the top two most-frequent chars simultaneously; place both in the result; push back with decremented counts. This guarantees alternation without needing to explicitly check the last character. |
| What is lazy deletion in the sliding window median? | Instead of removing an element from the heap interior (O(n)), mark it in a `delayed` counter. When it rises to the heap top, discard it and re-balance. Amortized O(log n) per removal. |
| Why use Cassandra for the follow graph in a newsfeed? | The access pattern is always "list all followers of user X" — a partition scan by followee_id. Cassandra's wide-row model makes this fast and write-efficient. No joins needed; scales to billions of follow relationships. |
| Why cap the Redis feed sorted set at 1000 entries and only cache active users? | 1000 posts × 8B × 1B users = 8 TB Redis — too large. Capping bounds memory; inactive users (> 30 days no login) get a cold-start rebuild on next login, which is acceptable since they browse infrequently. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/18/30 min, no hints)
- [ ] Rewrote Reorganize String two-at-a-time approach and lazy deletion pattern from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw the feed Redis schema and explain media pipeline cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
