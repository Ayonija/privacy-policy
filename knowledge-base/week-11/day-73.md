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
- STAR prompt: Describe a time you optimised a data pipeline that was processing media or large files — what bottleneck did you resolve?
- Leadership principle: Frugality

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
