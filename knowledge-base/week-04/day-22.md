# Day 22 — Monotonic Stack: Next Greater Element
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Master the monotonic decreasing stack — the single pattern behind every "next greater/smaller element" problem family.

## DSA (2 hours)
### Pattern: Monotonic Stack (Decreasing)
- Maintain a stack of elements in strictly decreasing order; when a new element is greater than the top, the top has found its "next greater element" — pop and record.
- Trigger condition: "next greater / smaller element to the right (or left)", "span / distance to next larger", "temperatures / stock price lookback" — any problem asking for the first element that breaks a monotone relationship.
- Time complexity: O(n) — each element pushed and popped at most once | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Next Greater Element I | 496 | Easy | Monotonic Stack + HashMap | Precompute NGE for nums2 using monotonic stack; look up each nums1 element in the map |
| 2 | Daily Temperatures | 739 | Medium | Monotonic Stack (indices) | Store indices; when today is warmer than stack top, answer for top = today − top_index |
| 3 | Next Greater Element II | 503 | Medium | Monotonic Stack (circular) | Iterate 2n times (index mod n); second pass resolves elements that wrap around |

### Code Skeleton
```python
# Daily Temperatures (LC 739)
def daily_temperatures(temperatures):
    result = [0] * len(temperatures)
    stack = []  # stores indices, decreasing temperatures
    for i, temp in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < temp:
            idx = stack.pop()
            result[idx] = i - idx
        stack.append(i)
    return result

# Next Greater Element II — circular (LC 503) skeleton
def next_greater_elements(nums):
    n = len(nums)
    result = [-1] * n
    stack = []  # stores indices
    for i in range(2 * n):
        # use i % n to wrap around
        pass
    return result
```

## System Design (1 hour)
### Topic: CAP — Consistency Deep Dive
- **Strong consistency:** after a write completes, all subsequent reads return that value — as if there is only one copy of the data. Achieved via synchronous replication or consensus protocols (Paxos, Raft).
- **Linearisability:** the strictest form of consistency — operations appear instantaneous and globally ordered; expensive in latency.
- **Read-your-writes consistency:** a user always sees their own writes; weaker than strong consistency — sufficient for many user-facing applications.
- CP systems: HBase, Zookeeper, etcd — refuse to serve stale reads during a partition; may return errors.
- Trade-off: strong consistency requires coordination between nodes → higher latency, lower throughput.
- Interview talking point: "If asked how a distributed database achieves strong consistency, answer: via a consensus protocol like Raft — one node is the leader; all writes go through the leader and are only acknowledged once a majority of nodes confirm they have persisted the write; reads also go to the leader to guarantee freshness."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you had to wait for confirmation from multiple stakeholders before declaring a decision final — analogous to a Raft leader waiting for a quorum before acknowledging a write.
- Leadership principle: Are Right, A Lot

## Flashcards

| Q | A |
|---|---|
| How does a monotonic decreasing stack find the Next Greater Element? | Maintain stack of pending indices; when current element > top element, the top has found its NGE — pop and record `result[top] = current` |
| How do you handle the circular array in Next Greater Element II? | Iterate from 0 to 2n−1 using `index = i % n`; the second pass resolves elements that had no greater element in the first pass |
| What does the stack store in Daily Temperatures — values or indices? | Indices — you need the index to compute the number of days elapsed (`current_index − stack_top_index`) |
| How does Raft achieve strong consistency in a distributed database? | All writes go through the elected leader; the write is acknowledged only after a majority (quorum) of nodes persist it; reads also go to the leader |
| What is the difference between strong consistency and read-your-writes consistency? | Strong: all clients see the latest write immediately. Read-your-writes: only the same client that wrote sees their own write immediately — other clients may see stale data |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
