# Day 72 — Heaps: Kth Largest, Frequency Sort, and Two-Heap Median
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Kth Largest introduces quickselect as an O(n) alternative to the O(n log k) heap approach. Sort by frequency is a straightforward max-heap drain. Find Median from Data Stream is the archetypal two-heap problem: a max-heap for the lower half and a min-heap for the upper half, balanced so median extraction is O(1).

---

## DSA (2 hours)
### Pattern: Min-Heap Kth Largest + Max-Heap Frequency Sort + Two-Heap Running Median

**Kth Largest Element in an Array (LC 215):**
Two approaches:
1. **Min-heap of size K**: maintain a min-heap; push each element; if size > K, pop the min. Top of heap = Kth largest. O(n log K) time, O(K) space.
2. **Quickselect**: partition array around a pivot; if pivot index == n-K, found; if > n-K, recurse left; else recurse right. Average O(n), worst case O(n²). Randomise pivot to get O(n) expected.

**Sort Characters By Frequency (LC 451):**
Count character frequencies with Counter. Use a max-heap (negate frequencies for Python's min-heap). Drain the heap: each pop gives `(−freq, char)`; append `char * freq` to result.

**Find Median from Data Stream (LC 295):**
Two heaps:
- `lo`: max-heap (lower half) — store negated values for Python
- `hi`: min-heap (upper half)

Invariant: all elements in `lo` ≤ all elements in `hi`, and `len(lo) == len(hi)` or `len(lo) == len(hi) + 1`.

Add num:
1. Push to `lo` (as negative); then push `−heappop(lo)` to `hi` (ensures max(lo) ≤ min(hi))
2. Rebalance: if `len(hi) > len(lo)`, move `hi`'s min back to `lo`

Median: if equal size → `(−lo[0] + hi[0]) / 2.0`; else → `−lo[0]` (lo has the extra element).

**Trigger condition:**
- "Kth largest/smallest in array — fast retrieval" → min-heap size K; or quickselect for O(n) avg
- "sort/rebuild string or array by element frequency" → Counter + max-heap drain
- "running median as numbers arrive one by one" → two heaps (max-heap lo + min-heap hi); O(log n) add, O(1) median

**Time complexity:** LC 215: O(n log K) heap / O(n) quickselect avg | LC 451: O(n log n) | LC 295: O(log n) add, O(1) median
**Space complexity:** O(K) / O(n) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Kth Largest Element in an Array | 215 | Medium | Min-heap size K OR quickselect | Min-heap: top is Kth largest; quickselect: partition around pivot index n-K |
| 2 | Sort Characters By Frequency | 451 | Medium | Counter + max-heap drain | Negate freq for max-heap; drain heap appending `char * freq` |
| 3 | Find Median from Data Stream | 295 | Hard | Two heaps: max-heap lo + min-heap hi | Push to lo, rebalance to hi, rebalance sizes; median in O(1) |

---

### Code Skeleton
```python
import heapq
from collections import Counter
import random

# Kth Largest Element in Array (LC 215) — two approaches

# Approach 1: Min-heap of size K
def findKthLargest_heap(nums, k):
    heap = []
    for n in nums:
        heapq.heappush(heap, n)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]

# Approach 2: Quickselect (average O(n))
def findKthLargest(nums, k):
    target = len(nums) - k   # 0-indexed target position in sorted order

    def quickselect(lo, hi):
        pivot = nums[random.randint(lo, hi)]
        left = lo
        right = hi
        mid = lo
        while mid <= right:
            if nums[mid] < pivot:
                nums[left], nums[mid] = nums[mid], nums[left]
                left += 1; mid += 1
            elif nums[mid] > pivot:
                nums[mid], nums[right] = nums[right], nums[mid]
                right -= 1
            else:
                mid += 1
        if left <= target <= right: return nums[target]
        if target < left: return quickselect(lo, left - 1)
        return quickselect(right + 1, hi)

    return quickselect(0, len(nums) - 1)

# Sort Characters By Frequency (LC 451)
def frequencySort(s):
    count = Counter(s)
    heap = [(-freq, char) for char, freq in count.items()]
    heapq.heapify(heap)
    result = []
    while heap:
        freq, char = heapq.heappop(heap)
        result.append(char * (-freq))
    return ''.join(result)

# Find Median from Data Stream (LC 295)
class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (negated) — lower half
        self.hi = []   # min-heap — upper half

    def addNum(self, num):
        heapq.heappush(self.lo, -num)
        # Ensure max(lo) <= min(hi)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        # Rebalance sizes: lo can be equal or one larger
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2.0
```

---

### Edge Cases to Trace Before Coding
- LC 215: k = 1 → return the maximum; k = len(nums) → return the minimum; all elements equal → return that element; n = 1 → return nums[0]
- LC 451: all characters same → one block; characters with same frequency → any valid order (heapq may produce different tie-breaks; both are correct)
- LC 295: single element added → median = that element; two elements → average; even count → float average; after many adds, sizes stay balanced to within 1

---

### Interview Pattern Drill

| Problem | Heap type | Size constraint | Result extraction |
|---------|-----------|----------------|-------------------|
| Top-K largest | Min-heap | Exactly K | `heap[0]` or all elements |
| Kth largest specifically | Min-heap | Exactly K | `heap[0]` |
| Running median | Max-heap lo + min-heap hi | `len(lo) = len(hi)` or `+1` | `−lo[0]` or avg |
| Sort by frequency (desc) | Max-heap | Unbounded (drain all) | Drain in frequency order |

**Quickselect three-way partition (Dutch National Flag):**
Partition into `< pivot`, `== pivot`, `> pivot`. Target falls in the `==` range → found. Otherwise recurse left or right. Three-way partition avoids worst-case O(n²) on arrays with many duplicates.

---

## System Design (1 hour)
### Topic: Newsfeed — Fanout on Write vs Fanout on Read (Deep Dive)

**Problem statement:**
When Taylor Swift (120 M followers) posts a photo, how do we update 120 M users' feeds without bringing down the system?

**Fanout on Write (push model) — detailed flow:**
```
Taylor posts photo
    │
[Post Service] → INSERT post into Post DB
    │
[Message Queue: Kafka topic "new_posts"]
    │
[Fanout Workers] (horizontally scaled)
    │
    For each follower_id in Taylor's follower list:
        ZADD feed:{follower_id} {timestamp} {post_id}
```

- Follower list fetch: paginate from Social Graph DB, 1000 followers/batch
- At 120 M followers × 1000 followers/batch = 120,000 DB reads → expensive but async
- Rate-limited to 1000 writes/sec → 120,000 seconds = 33 hours for Taylor's single post ❌

**Celebrity threshold detection:**
Any user with > 1 M followers is flagged as a "celebrity" in the User Service. Their posts are NOT fanned out.

**Hybrid fanout flow:**
```
User A (normal, 200 followers) posts:
    → Fanout workers write post_id to 200 feed: sorted sets → done in < 1 sec

Taylor (120 M followers) posts:
    → Post stored in Post DB; NO fanout
    → Feed marked as "has celebrity posts" for her followers

User B views feed:
    1. Fetch pre-computed feed from Redis: feed:{user_B} (contains posts from normal followees)
    2. Fetch Taylor's last 20 posts from Post DB (or from Taylor's own Redis cache: user_posts:{taylor})
    3. Merge and sort by timestamp in Feed Service
    4. Return top N posts
```

**Storage for celebrity posts:**
`user_posts:{celebrity_id}` — sorted set of (timestamp, post_id) for the celebrity's posts; capped at last 1000 posts. Maintained by the Post Service on every celebrity post. Cheap to maintain; cheap to read.

**Feed cold start (new user or cache miss):**
1. Fetch the IDs of all accounts the user follows
2. For each: fetch last 20 posts from their `user_posts:{}` sorted set
3. Merge, sort, pick top N
4. Store in `feed:{user_id}` with 7-day TTL

**Why not fanout on read always?**
A user following 500 accounts → 500 DB queries per feed view × 57,000 reads/sec = 28.5 M queries/sec. That's 4 orders of magnitude beyond what a single DB cluster can handle.

**Interview talking point:** "The celebrity problem is why pure fanout on write breaks at scale. The hybrid model uses write-time fanout for normal users (fast, bounded cost) and read-time merge for celebrities (avoids quadratic write amplification). The threshold is typically 1–5 M followers, tuned based on infrastructure capacity. At Instagram, this threshold is configurable per account."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 215 Kth Largest Element (target: 15 min — try both heap and quickselect)
- **Medium 2:** LC 451 Sort Characters By Frequency (target: 12 min)
- **Hard:** LC 295 Find Median from Data Stream (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to handle a high-volume event (a spike, a viral moment) — how did you prevent it from overwhelming your system?
- Leadership principle: Bias for Action

---

## Flashcards

| Q | A |
|---|---|
| How does quickselect find the Kth largest in O(n) average? | Partition array around a random pivot into <, ==, > groups. If pivot index == n-K → found. If > n-K → recurse left; else → recurse right. Average O(n) because each partition halves the search space. |
| How does Sort Characters By Frequency use a max-heap? | Build a Counter; push (-freq, char) pairs and heapify. Drain heap: each pop gives the char with highest remaining frequency; append `char * freq` to the result string. |
| State the two invariants of the two-heap MedianFinder. | (1) max(lo) ≤ min(hi) — all lower-half elements are ≤ all upper-half elements. (2) `len(lo) == len(hi)` or `len(lo) == len(hi) + 1` — lo holds the extra element for odd-length streams. |
| How does the hybrid fanout model handle celebrity accounts? | Celebrity posts (> threshold followers) are NOT fanned out. Instead, at feed-read time, the Feed Service fetches the celebrity's last N posts from a dedicated sorted set and merges them with the pre-computed Redis feed. |
| What is feed cold start and how is it resolved? | Cold start = a user's feed Redis cache is empty (new user or expired). Resolution: fetch all followee IDs → fetch each followee's last 20 posts from `user_posts:` sorted sets → merge + sort → populate `feed:{user_id}` Redis cache. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/12/25 min, no hints)
- [ ] Rewrote two-heap MedianFinder from memory (both addNum and findMedian)
- [ ] Stated time + space complexity aloud for each solution (including quickselect vs heap comparison)
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain celebrity threshold and hybrid fanout flow cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
