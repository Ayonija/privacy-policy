# Day 74 — Heaps: Task Scheduler, Greedy Heap Allocation, and K-List Range
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Task Scheduler is a frequency-based greedy problem with a heap simulation. Furthest Building teaches a key greedy insight: delay the decision of which jumps use ladders vs bricks by using a min-heap to swap out smaller jumps when a ladder is needed. Smallest Range Covering Elements from K Lists extends K-way merge: instead of building a full sorted sequence, track the active minimum and current maximum to find the smallest range containing one element per list.

---

## DSA (2 hours)
### Pattern: Frequency Greedy + Min-Heap Deferred Swap + K-way Merge Range Tracking

**Task Scheduler (LC 621):**
CPU must execute tasks with a cooldown of `n` between identical tasks. Minimize total CPU time.

**Math approach (O(n)):** Let `max_freq` = highest task frequency, and `count_max` = number of tasks with that frequency.
- Result = `max((max_freq - 1) * (n + 1) + count_max, len(tasks))`
- Intuition: the most frequent task determines the minimum frame structure. `(max_freq - 1)` full "frames" of size `(n + 1)`, plus the final occurrence(s) of the most frequent task. The final `len(tasks)` ensures the result is at least the number of tasks.

**Heap simulation (for interview explanation):** Max-heap of frequencies. Each "round" of `n + 1` slots: pop up to `n + 1` tasks from the heap, execute them, decrement and push back non-zero counts.

**Furthest Building You Can Reach (LC 1642):**
Move from building 0 to the highest index reachable with `b` bricks and `l` ladders. Greedy: use ladders for the largest height differences; use bricks for smaller ones. Min-heap of size `l`: tracks the `l` largest jumps used so far. When a new jump exceeds the heap's minimum, swap: use bricks for the heap's minimum (it's now the smallest "ladder jump"), free that ladder for the new bigger jump.

**Smallest Range Covering Elements from K Lists (LC 632):**
K sorted lists; find the smallest range [lo, hi] such that each list contributes at least one element to the range.

Algorithm: min-heap with one element from each list. Track `current_max` (updated as we push). The range at any moment is `[heap_min, current_max]`. Pop the minimum; push the next element from the same list. Update range if `current_max − heap_min < current_best`. Stop when any list is exhausted.

**Trigger condition:**
- "minimum CPU time with cooldown between identical tasks" → math formula; or max-heap frequency simulation
- "furthest reachable point choosing between two resource types (ladders vs bricks)" → min-heap of used ladder jumps; swap smallest used ladder for bricks on new larger jump
- "smallest range with one element from each of K sorted lists" → K-way min-heap; track current_max; update range on each pop

**Time complexity:** LC 621: O(n) math | LC 1642: O(n log l) | LC 632: O(n log k) where n = total elements
**Space complexity:** O(26) = O(1) / O(l) / O(k)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Task Scheduler | 621 | Medium | Frequency greedy formula | `(max_freq-1)*(n+1) + count_max`; floor at `len(tasks)` |
| 2 | Furthest Building You Can Reach | 1642 | Medium | Min-heap of ladder jumps; swap when heap full | New jump > heap min → use bricks for heap min, add new jump to heap |
| 3 | Smallest Range from K Lists | 632 | Hard | K-way merge + track (min, max) range | Push one from each list; range = [heap_min, current_max]; pop min, push next; stop when list exhausted |

---

### Code Skeleton
```python
import heapq
from collections import Counter

# Task Scheduler (LC 621) — Math approach
def leastInterval(tasks, n):
    count = Counter(tasks)
    max_freq = max(count.values())
    count_max = sum(1 for v in count.values() if v == max_freq)
    return max((max_freq - 1) * (n + 1) + count_max, len(tasks))

# Task Scheduler (LC 621) — Heap simulation (for explanation)
def leastInterval_heap(tasks, n):
    count = Counter(tasks).values()
    heap = [-c for c in count]
    heapq.heapify(heap)
    time = 0
    while heap:
        cycle = []
        for _ in range(n + 1):
            if heap:
                cycle.append(heapq.heappop(heap))
        for c in cycle:
            if c + 1 < 0: heapq.heappush(heap, c + 1)
        time += n + 1 if heap else len(cycle)
    return time

# Furthest Building You Can Reach (LC 1642)
def furthestBuilding(heights, bricks, ladders):
    heap = []   # min-heap of ladder jump heights (the l smallest jumps use ladders)
    for i in range(len(heights) - 1):
        diff = heights[i + 1] - heights[i]
        if diff <= 0: continue   # going down or flat — free
        heapq.heappush(heap, diff)
        if len(heap) > ladders:
            # Must use bricks for the smallest ladder jump
            bricks -= heapq.heappop(heap)
        if bricks < 0: return i   # can't proceed
    return len(heights) - 1

# Smallest Range Covering Elements from K Lists (LC 632)
def smallestRange(nums):
    # heap: (value, list_index, element_index)
    heap = []
    current_max = float('-inf')
    for i, lst in enumerate(nums):
        heapq.heappush(heap, (lst[0], i, 0))
        current_max = max(current_max, lst[0])
    best = [float('-inf'), float('inf')]
    while heap:
        val, i, j = heapq.heappop(heap)
        if current_max - val < best[1] - best[0]:
            best = [val, current_max]
        if j + 1 == len(nums[i]): break   # list exhausted → stop
        next_val = nums[i][j + 1]
        heapq.heappush(heap, (next_val, i, j + 1))
        current_max = max(current_max, next_val)
    return best
```

---

### Edge Cases to Trace Before Coding
- LC 621: n=0 → no cooldown → result = len(tasks); all tasks unique → result = len(tasks); one task type with very high frequency → formula gives correct idle slots
- LC 1642: no upward jumps → reach end; ladders=0 → use bricks only; bricks=0 → use ladders only; first jump > total bricks → stuck at building 0
- LC 632: K=1 → range = [min_elem, max_elem] in that single list, but actually the smallest range is [lst[0], lst[0]] for a single list; all lists have only one element → range = [global_min, global_max]

---

### Interview Pattern Drill

| Pattern | Heap type | Heap content | When to swap / stop |
|---------|-----------|-------------|---------------------|
| Task Scheduler (simulation) | Max-heap | Task frequencies | Each round: pop up to n+1; push back |
| Furthest Building | Min-heap | Ladder-assigned jump heights | Heap full + new jump > min → swap |
| Smallest K-list Range | Min-heap | (val, list_idx, elem_idx) | Any list runs out → stop |

**Task Scheduler intuition:** The most frequent task is the "bottleneck." Between each pair of its occurrences, we have a frame of `n + 1` slots. Fill frames with other tasks; remaining slots are idle. If we have more tasks than idle slots, no idle time is needed — result = total tasks.

---

## System Design (1 hour)
### Topic: Newsfeed — Social Graph Storage and the Celebrity Problem

**Social graph fundamentals:**
A social network is a directed graph: user A → user B means A follows B. Key queries:
1. "Who does user A follow?" (followees) — for feed construction
2. "Who follows user A?" (followers) — for fanout when A posts

**Storage options:**

| Approach | Technology | Trade-offs |
|----------|-----------|------------|
| Adjacency list in Cassandra | Cassandra wide rows | Fast "get all followers of X"; no complex queries |
| Relational follow table | MySQL sharded by followee_id | ACID; slower for bulk follower lists |
| Graph database | Neo4j, Amazon Neptune | Mutual friends, path queries; complex; hard to scale |

**Cassandra schema (recommended):**
```
-- "Who follows user X?" — for fanout
TABLE follows_by_followee (
    followee_id bigint,
    follower_id bigint,
    created_at timestamp,
    PRIMARY KEY (followee_id, follower_id)
)

-- "Who does user Y follow?" — for feed cold start
TABLE follows_by_follower (
    follower_id bigint,
    followee_id bigint,
    created_at timestamp,
    PRIMARY KEY (follower_id, followee_id)
)
```

Two tables, denormalized — on FOLLOW action, write to both. On UNFOLLOW, delete from both. This is standard Cassandra dual-write for bidirectional queries.

**Celebrity detection:**
```python
# On user profile update or follower count change:
if follower_count > CELEBRITY_THRESHOLD:  # e.g., 1_000_000
    user_flags.add_celebrity_flag(user_id)
```

The celebrity flag is stored in the User Service (Redis + DB). Feed Service checks this flag at read time to decide whether to merge celebrity posts separately.

**Mutual friends query:**
SQL: `SELECT followee_id FROM follows WHERE follower_id IN (set_of_my_followees)`
This is a "friends of friends" query — expensive at scale. Typically solved with:
- Pre-computed mutual friend lists (batch job nightly)
- Graph DB for real-time mutual friend queries (limited to < 1 M users)
- Approximation: show "X others you may know follow this user" using Bloom filter membership

**Follow/unfollow consistency:**
Dual-write to both Cassandra tables with lightweight transactions (LWT) to prevent partial writes. Or use Kafka: on FOLLOW event → Kafka → two consumers writing to each table. The Kafka approach is async — a brief inconsistency window (< 1 sec) is acceptable.

**Interview talking point:** "For a Twitter-scale social graph, I'd use Cassandra with two tables: one partitioned by followee_id (for listing followers during fanout) and one by follower_id (for listing followees during feed cold-start). Dual-write on follow/unfollow events, with Kafka as the event bus to ensure both tables are eventually consistent. Graph databases like Neo4j are only practical for specialized queries like mutual friends, not for the hot path of feed generation."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 621 Task Scheduler (target: 18 min — implement both math and heap)
- **Medium 2:** LC 1642 Furthest Building You Can Reach (target: 18 min)
- **Hard:** LC 632 Smallest Range Covering Elements from K Lists (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to allocate limited resources (time, money, people) across competing demands — how did you prioritise?
- Leadership principle: Frugality

---

## Flashcards

| Q | A |
|---|---|
| State the Task Scheduler formula. | `max((max_freq - 1) * (n + 1) + count_max, len(tasks))`. `max_freq` = highest task count. `count_max` = number of tasks with that count. Floor at `len(tasks)` because we can always fill idle slots with other tasks if enough exist. |
| How does Furthest Building decide when to swap a ladder for bricks? | A min-heap tracks the `l` largest height differences used so far (assigned to ladders). When a new jump must use a ladder but the heap is full: if the new jump > heap's minimum, use bricks for the heap's minimum and add the new jump to the heap. |
| What stops the Smallest Range from K Lists algorithm? | When the list that just provided the current minimum is exhausted (no more elements to push). At that point, no range can include an element from that list, so the minimum range has been found. |
| Why use two Cassandra tables for the social follow graph? | The two access patterns (followers of X, followees of Y) require different partition keys for efficient reads. Denormalizing into two tables — one by followee_id, one by follower_id — gives O(1) partition scans for both patterns. |
| What is the celebrity threshold and how is it used in the fanout decision? | A configurable threshold (e.g., 1 M followers). Users above the threshold are "celebrities": their posts are NOT fanned out (to avoid write amplification). Instead, their posts are merged into viewers' feeds at read time. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/18/25 min, no hints)
- [ ] Rewrote Task Scheduler formula and Furthest Building heap swap from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw dual Cassandra table schema and explain celebrity detection cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
