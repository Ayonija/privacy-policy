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
```java
// Daily Temperatures (LC 739)
class Solution {
    public static int[] dailyTemperatures(int[] temperatures) {
        int[] result = new int[temperatures.length];
        Deque<Integer> stack = new ArrayDeque<>(); // stores indices, decreasing temperatures
        for (int i = 0; i < temperatures.length; i++) {
            while (!stack.isEmpty() && temperatures[stack.peekLast()] < temperatures[i]) {
                int idx = stack.pollLast();
                result[idx] = i - idx;
            }
            stack.addLast(i);
        }
        return result;
    }

    // Next Greater Element II — circular (LC 503)
    public static int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];
        Arrays.fill(result, -1);
        Deque<Integer> stack = new ArrayDeque<>(); // stores indices; values are in decreasing order
        for (int i = 0; i < 2 * n; i++) {   // iterate twice to simulate circular array
            while (!stack.isEmpty() && nums[stack.peekLast()] < nums[i % n]) {
                int idx = stack.pollLast();
                result[idx] = nums[i % n];   // first greater element found
            }
            if (i < n) {
                stack.addLast(i);   // only push indices from first pass
            }
        }
        return result;
    }
}
```

### Interview Tips

- **Monotonic stack amortised O(n):** "Each element is pushed at most once and popped at most once — total work is O(n) even though the while loop is nested." State this proof; interviewers at FAANG will probe for it.
- **Always store indices, not values:** "I store indices in the stack so I can compute the distance (e.g., `i - idx` in Daily Temperatures) and to handle circular wrapping with `i % n`." Storing values instead of indices is the single most common monotonic stack mistake.
- **Circular array trick:** "I iterate `range(2 * n)` and use `i % n` to wrap around. In the second pass (`i >= n`), I don't push new indices — I only resolve pending elements whose NGE comes from 'wrapping around'."
- **Brute force baseline:** O(n²) for each element, scan right (and wrap) until a greater element is found → monotonic stack is O(n).
- **Common mistake:** in LC 496 (NGE I), building the NGE map from `nums1` instead of `nums2` — you must precompute NGEs for *all* elements of `nums2`, then look up `nums1` elements in the map.

### Edge Cases to Trace Before Coding
- LC 739 (Daily Temperatures): monotonically decreasing temperatures → stack fills up, nothing gets popped during the loop; all results stay 0 ✓
- LC 739: monotonically increasing → each new temperature pops all previous; results are all 1 except the last (which stays 0) ✓
- LC 503: single element array `[5]` → nothing is greater than itself; result is `[-1]`
- LC 503: all same elements `[3, 3, 3]` → no element has a greater neighbor; result is `[-1, -1, -1]`
- LC 496: element in `nums1` that doesn't appear in `nums2` → per constraints this won't happen, but the HashMap default of -1 handles it

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
