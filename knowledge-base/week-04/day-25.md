# Day 25 — Queue & Deque: Design & Simulation
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Implement queue/stack with opposite data structures and apply deque-based simulation to card and senate problems.

## DSA (2 hours)
### Pattern: Queue Simulation & Deque Ordering
- Implement Queue using Stacks: two stacks (in-stack and out-stack); push to in-stack; pop from out-stack; only transfer when out-stack is empty — amortised O(1) per operation.
- Reveal Cards In Increasing Order: simulate the card-reveal process by working backward — use a deque to determine which position each card occupies.
- Dota2 Senate: greedy queue simulation — each senator bans the next senator from the opposing party; use two queues of indices and always ban the nearest opponent.
- Trigger condition: "implement one ADT using another" OR "simulate a round-robin or turn-based process with a queue."
- Time complexity: O(n) amortised | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Implement Queue using Stacks | 232 | Easy | Two stacks | In-stack for push; out-stack for pop; lazy transfer only when out-stack is empty |
| 2 | Reveal Cards In Increasing Order | 950 | Medium | Deque simulation | Sort cards; simulate reveal-and-move-to-bottom backward using a deque of indices to find deck order |
| 3 | Dota2 Senate | 649 | Medium | Greedy queue simulation | Two index queues; smaller index bans the other; winner re-queues at index + n to maintain round order |

### Code Skeleton
```python
# Implement Queue using Stacks (LC 232)
class MyQueue:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []

    def push(self, x):
        self.in_stack.append(x)

    def _transfer(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

    def pop(self):
        self._transfer()
        return self.out_stack.pop()

    def peek(self):
        self._transfer()
        return self.out_stack[-1]

    def empty(self):
        return not self.in_stack and not self.out_stack

# Dota2 Senate (LC 649) — skeleton
from collections import deque
def predict_party_victory(senate):
    radiant = deque()
    dire = deque()
    # populate queues with indices
    # while both non-empty: smaller index bans other, re-queues at index + n
    pass
```

## System Design (1 hour)
### Topic: Consistency Models — Strong, Eventual, and the Spectrum Between
- **Strong consistency (linearisability):** operations appear instantaneous; all clients see the same order. Used by: etcd, Zookeeper, Google Spanner.
- **Sequential consistency:** all operations appear in some global sequential order, but not necessarily in real time. Used by: some distributed databases in cluster mode.
- **Causal consistency:** causally related operations are seen in order by all clients; unrelated operations may be seen in different orders. Used by: MongoDB (causal sessions), some geo-distributed systems.
- **Eventual consistency:** given no new updates, all replicas converge to the same value eventually. Used by: DynamoDB (default), Cassandra (default), DNS.
- Trade-off: as you relax consistency, latency and availability improve but conflict resolution complexity increases.
- Interview talking point: "If asked to choose a consistency model for a collaborative document editor (like Google Docs), answer: causal consistency — you need to see your own edits immediately and in causal order, but don't need global linearisability; eventual consistency alone would cause unresolvable merge conflicts."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you managed a backlog queue of requests under high load — determining which items to process first based on recency and priority, analogous to the Dota2 senate index-based priority queuing.
- Leadership principle: Customer Obsession

## Flashcards

| Q | A |
|---|---|
| Why is the two-stack queue implementation amortised O(1)? | Each element is pushed to in-stack once and popped to out-stack once — at most 2 operations total regardless of how many peek/pop calls are made |
| How does the Dota2 senate greedy work? | Both senators are in queues with their indices; the one with the smaller index acts first and bans the other; the winner re-queues at `index + n` to participate in the next round |
| How do you reconstruct the deck order in Reveal Cards In Increasing Order? | Sort cards descending; simulate the process backward using a deque of positions — repeatedly move the back position to front, then assign the current card to front |
| What is the difference between eventual consistency and causal consistency? | Eventual: all replicas converge eventually — no ordering guarantee. Causal: causally related writes are seen in order by all clients — unrelated writes may appear in any order |
| Name one real system for each consistency model: strong, causal, eventual. | Strong: Zookeeper / Google Spanner. Causal: MongoDB (causal sessions). Eventual: DynamoDB (default), DNS |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
