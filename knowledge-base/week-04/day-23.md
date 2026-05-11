# Day 23 — Monotonic Stack: Stock Span & Asteroid Collision
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Apply the monotonic stack to span-counting and simulation problems — two common disguises for the same underlying pattern.

## DSA (2 hours)
### Pattern: Monotonic Stack — Span & Collision Simulation
- Online Stock Span: the span for today = number of consecutive days (including today) where price ≤ today's price. Maintain a decreasing stack of (price, span) pairs; when today's price ≥ top, absorb top's span.
- Asteroid Collision: simulate head-on collisions with a stack; push right-moving (+) asteroids; on a left-moving (−) asteroid, pop all smaller right-movers; handle equal-size destruction.
- Trigger condition: "how far back does this element dominate?" (span) OR "simulate pairwise destruction between elements moving toward each other."
- Time complexity: O(n) amortised | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Backspace String Compare | 844 | Easy | Stack (or two pointers) | Simulate typing with a stack; `#` pops the last character; compare final strings |
| 2 | Online Stock Span | 901 | Medium | Monotonic Stack (span accumulation) | Stack stores (price, span); absorb smaller-or-equal spans into today's span before pushing |
| 3 | Asteroid Collision | 735 | Medium | Stack (collision simulation) | Push positive; on negative: pop positives smaller in absolute value; handle equal destruction; push negative if stack empty or top is negative |

### Code Skeleton
```java
// Online Stock Span (LC 901)
class StockSpanner {
    private Deque<int[]> stack; // {price, span}

    public StockSpanner() {
        this.stack = new ArrayDeque<>();
    }

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peekLast()[0] <= price) {
            span += stack.pollLast()[1];
        }
        stack.addLast(new int[]{price, span});
        return span;
    }
}

// Asteroid Collision (LC 735) — skeleton
class Solution {
    public static int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> stack = new ArrayDeque<>();
        for (int a : asteroids) {
            // positive: push
            // negative: collide with stack top if top > 0
            // handle: top == abs(a) → both destroyed
            //         top > abs(a) → a is destroyed
            //         top < abs(a) or top < 0 → pop/push logic
            // TODO: implement
        }
        int[] result = new int[stack.size()];
        int i = result.length - 1;
        for (int val : stack) result[i--] = val;
        return result;
    }
}
```

## System Design (1 hour)
### Topic: CAP — Availability Deep Dive
- **Availability** in CAP means every non-failing node returns a response to every request — no timeouts, no errors due to coordination.
- AP systems (e.g., Cassandra, DynamoDB, CouchDB): during a partition, nodes keep serving reads and writes locally; when the partition heals, nodes reconcile via eventual consistency.
- Availability ≠ uptime (99.9% SLA): CAP availability is about *what the system does during a partition*, not overall uptime percentage.
- **Last-write-wins (LWW):** one conflict resolution strategy for AP systems — keep the write with the latest timestamp; simple but can lose data if clocks are skewed.
- **Vector clocks:** track causal history of writes per node; allow precise conflict detection but add metadata overhead.
- Interview talking point: "If asked how Cassandra handles a network partition, answer: it stays available — each node accepts writes locally using its replication factor; on partition heal, anti-entropy repairs sync diverged replicas; the consistency level (ONE, QUORUM, ALL) controls how much staleness the caller tolerates per request."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you had to resolve a conflict between two competing updates to the same resource — analogous to asteroid collision where two items moving toward each other must be reconciled.
- Leadership principle: Earn Trust

## Flashcards

| Q | A |
|---|---|
| How does Online Stock Span accumulate span across multiple days? | Stack stores `(price, span)` pairs; when today's price ≥ stack top, pop and add top's span to today's span — spans are merged before pushing |
| How do you handle equal-size asteroids in Asteroid Collision? | When `stack[-1] == -asteroid` (equal magnitudes, opposite directions), both are destroyed — pop the stack and do NOT push the current asteroid |
| How does Backspace String Compare work in O(1) space? | Two-pointer approach: scan both strings backward, counting `#` to skip characters; compare the next valid character from each |
| What is the key difference between CAP Availability and SLA uptime? | CAP availability is about returning a response during a partition (no coordination failures); SLA uptime (e.g., 99.9%) is about total operational hours regardless of partition |
| How does Cassandra's consistency level (ONE vs QUORUM vs ALL) trade off availability and consistency? | ONE: fastest, most available, most stale. QUORUM: balanced. ALL: fully consistent but fails if any replica is unreachable — lowest availability |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
