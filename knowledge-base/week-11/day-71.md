# Day 71 — Heaps & Tries: Top-K Frequency, K Closest Elements, Merge K Sorted Lists
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Three canonical heap patterns: maintaining a min-heap of size K for Top-K problems, binary search for K closest elements, and min-heap merge for K sorted sequences. These three templates cover the majority of heap interview questions.

---

## DSA (2 hours)
### Pattern: Min-Heap Top-K + Binary Search K Window + K-way Merge Min-Heap

**Top K Frequent Elements (LC 347):**
Count frequencies with a HashMap. Then use a min-heap of size K: push `(count, element)` pairs; when heap size exceeds K, pop the minimum. The K remaining elements in the heap are the K most frequent. Alternative: bucket sort — array of size `n+1` where index = frequency; scan from right for top K. Bucket sort achieves O(n).

**Find K Closest Elements (LC 658):**
Find the K-element subarray of `arr` (sorted) closest to value `x`. Binary search on the left boundary `lo` of the window. Invariant: `x - arr[mid] <= arr[mid+k] - x` → `mid` is a valid left boundary (left side is closer or equal). Binary search until `lo == hi`. Return `arr[lo:lo+k]`.

**Merge K Sorted Lists (LC 23):**
Min-heap stores `(node.val, list_index, node)`. Push the head of each non-null list. Repeatedly pop the minimum, append to result, and push the popped node's `.next`. The `list_index` tiebreaker ensures stable comparison without comparing ListNode objects.

**Trigger condition:**
- "K most/least frequent elements" → min-heap of size K; or bucket sort O(n)
- "K closest elements to x in a sorted array" → binary search on window left boundary; O(log(n-k))
- "merge K sorted streams/lists into one" → min-heap with one element per stream; O(n log K)

**Time complexity:** LC 347: O(n log k) / O(n) bucket | LC 658: O(log(n-k) + k) | LC 23: O(n log k)
**Space complexity:** O(k) / O(1) / O(k)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Top K Frequent Elements | 347 | Medium | Min-heap of size K by frequency | Push (count, elem); pop when size > K; heap = top K |
| 2 | Find K Closest Elements | 658 | Medium | Binary search on window left boundary | `x - arr[mid] <= arr[mid+k] - x` → left is valid; binary search until lo==hi |
| 3 | Merge K Sorted Lists | 23 | Hard | Min-heap one-from-each-list | Push `(val, idx, node)`; pop min; push node.next; idx breaks ties |

---

### Code Skeleton
```python
import heapq
from collections import Counter

# Top K Frequent Elements (LC 347)
def topKFrequent(nums, k):
    count = Counter(nums)
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
    return [num for freq, num in heap]

# Find K Closest Elements (LC 658)
def findClosestElements(arr, k, x):
    lo, hi = 0, len(arr) - k
    while lo < hi:
        mid = (lo + hi) // 2
        # Compare left side (arr[mid]) vs right side (arr[mid+k]) distance to x
        if x - arr[mid] <= arr[mid + k] - x:
            hi = mid        # left side is closer or equal → valid left boundary; try smaller
        else:
            lo = mid + 1    # right side is closer → left boundary must move right
    return arr[lo:lo + k]

# Merge K Sorted Lists (LC 23)
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val; self.next = next

def mergeKLists(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    dummy = ListNode()
    curr = dummy
    while heap:
        val, i, node = heapq.heappop(heap)
        curr.next = node
        curr = curr.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

---

### Edge Cases to Trace Before Coding
- LC 347: all elements have the same frequency → any K are valid; k = len(unique elements) → return all
- LC 658: x is less than arr[0] → closest window is arr[0:k]; x is greater than arr[-1] → arr[n-k:n]; ties: prefer smaller elements (left side is chosen when equal → binary search naturally handles this)
- LC 23: empty `lists` → return None; all lists empty → return None; single list → return as-is

---

### Interview Pattern Drill

| Pattern | Heap content | When to pop | Result |
|---------|-------------|------------|--------|
| Top-K max | `(freq, elem)` min-heap | When `len > k` | Remaining heap elements |
| K closest (sorted) | Binary search, no heap | Never | `arr[lo:lo+k]` |
| K-way merge | `(val, list_idx, node)` min-heap | Process each | Linked list in order |

**Min-heap for max problems:** To keep the K LARGEST elements, use a min-heap — the minimum element in the heap is always the "weakest" candidate; popping it removes elements that aren't in the top K.

---

## System Design (1 hour)
### Topic: Newsfeed — Requirements, Architecture Overview, and Fanout Strategies

**What is a newsfeed?**
A continuously updated stream of posts from accounts a user follows — e.g., Instagram feed, Twitter home timeline, Facebook feed.

**Functional requirements:**
1. `POST /post` — create a post (text, image, video)
2. `GET /feed` — retrieve the home feed (latest N posts from followees)
3. `POST /follow` — follow a user
4. `GET /user/{id}/posts` — view a user's profile posts

**Non-functional requirements:**
- High read-to-write ratio: ~100:1 (browsing >> posting)
- Low latency feed reads: < 200 ms p99
- Eventual consistency acceptable: a 5-second delay before a post appears in followers' feeds is fine
- High availability: Instagram-level 99.99 % uptime

**Capacity estimation (Instagram-scale):**
- 1 B daily active users; each views feed 5×/day → 5 B feed reads/day = ~57,000 reads/sec
- 1 M new posts/day = ~12 writes/sec
- Read-heavy ratio: ~4,700:1 (reads >> writes)

**Core components:**
```
[Client]
    │
[API Gateway / Load Balancer]
    │
    ├─ Post Service: creates posts → stores in Post DB (MySQL) + Media in S3
    ├─ User Service: user profiles, follow relationships → stored in Social Graph DB
    └─ Feed Service: computes + serves the home feed
              │
         [Feed Store: Redis sorted set per user]
              │
         [Fanout Service: populates feed on new posts]
```

**Two fanout strategies:**

**Fanout on Write (push model):**
When a user posts: the fanout service reads all followers, pushes the post_id to each follower's feed in Redis.
- Pro: O(1) feed read (pre-computed); fast user experience
- Con: O(followers) work per post; celebrity problem — Bieber posts → 100 M Redis writes

**Fanout on Read (pull model):**
When a user views feed: query DB for the latest posts from all followees; merge and sort in memory.
- Pro: O(1) write; no celebrity problem
- Con: O(followees) DB queries per feed view; slow for users who follow many people

**Hybrid approach (Instagram/Facebook strategy):**
- Regular users (< 5 K followers): fanout on write → fast reads from Redis
- Celebrity accounts (> 5 K followers): NO fanout; instead, merge celebrity posts into each viewer's feed at read time
- Redis feed format: sorted set `feed:{user_id}` → score = timestamp, member = post_id

**Interview talking point:** "For Instagram-scale, we use a hybrid fanout model. Normal users' posts are fanned out to all followers' Redis feeds immediately. Celebrity posts are NOT pushed (too expensive) — instead, when a user views their feed, we fetch the Redis pre-computed feed for regular followees and merge in the last 20 posts from any celebrity followees they have. This keeps feed read latency low while avoiding the write amplification of celebrity fanout."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 347 Top K Frequent Elements (target: 15 min)
- **Medium 2:** LC 658 Find K Closest Elements (target: 18 min)
- **Hard:** LC 23 Merge K Sorted Lists (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to merge data from multiple sources into a single unified view — how did you handle ordering and consistency?
- Leadership principle: Deliver Results

---

## Flashcards

| Q | A |
|---|---|
| How does a min-heap of size K find the Top-K frequent elements? | Push `(frequency, element)` for each unique element; when heap size > K, pop the minimum. The K remaining elements are the most frequent — the min-heap efficiently evicts the "least frequent" candidate. |
| What is the binary search invariant for Find K Closest Elements? | `x - arr[mid] <= arr[mid+k] - x` means the left side is closer to x than the right boundary of the window — so `mid` is a valid left boundary and we try smaller (hi = mid). Otherwise lo = mid + 1. |
| How does Merge K Sorted Lists handle stable comparison in the heap? | Push `(node.val, list_index, node)` — the `list_index` integer breaks ties when two nodes have equal values, avoiding comparison of ListNode objects which would raise a TypeError. |
| What is the Newsfeed hybrid fanout strategy? | Regular users (< threshold followers): fanout on write → push post_id to all follower Redis sorted sets. Celebrity accounts: no fanout; merge their posts at read time. Balances write cost vs read latency. |
| What data structure stores the pre-computed feed in Redis? | A sorted set (`ZADD feed:{user_id} {timestamp} {post_id}`). Reads use `ZREVRANGEBYSCORE` to get latest posts. Score = timestamp enables chronological ordering; `ZREVRANGE` returns newest first. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/18/25 min, no hints)
- [ ] Rewrote min-heap Top-K and K-way merge templates from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw the hybrid fanout architecture cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
