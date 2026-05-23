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

### Interview Tips

- **Span accumulation insight:** "Each call to `next(price)` absorbs all previous spans that are dominated by today's price — the while loop pops in amortised O(1) over all calls because each `(price, span)` pair is pushed and popped at most once."
- **Why store (price, span) pairs, not just prices:** "Storing the accumulated span lets you absorb multiple previous spans in a single pop — if you stored raw prices you'd need a second pass to recount days."
- **Asteroid collision case order matters:** "Check the cases in this order: (1) no collision possible (stack empty or top is negative), (2) right-mover is destroyed (top > |current|), (3) mutual destruction (top == |current|), (4) right-mover is destroyed (pop and continue loop). Getting the order wrong is the most common bug."
- **Brute force baseline for Stock Span:** O(n) per call (scan backward each time) = O(n²) total → monotonic stack is O(n) amortised.

### STAR Interview Framework

> **Monotonic Stack — Span & Collision Simulation:** brute-force O(n²) → this approach O(n) amortised time, O(n) space

**S:** "Given a stream of stock prices and a need to compute each day's span in real time. Naive backward scan per call is O(n) per call = O(n²) total — too slow for n=10^4 daily calls."
**T:** "Need O(1) amortised per call by accumulating spans inside the stack. Goal: for each price, return the number of consecutive prior days (including today) where price was ≤ today's price."
**A (60% of answer time):**
1. *Classify:* "The trigger is 'how far back does today's element dominate?' — span accumulation variant of monotonic stack."
2. *Init:* "Maintain a stack of `(price, span)` pairs. Span starts at 1 (today counts)."
3. *Loop/Step:* "While the stack top's price ≤ today's price: pop and add its span to today's span. Then push `(today_price, accumulated_span)`."
4. *Termination:* "Each pair is pushed once and popped at most once across all calls — O(n) total work, O(1) amortised per call."
5. *Gotcha:* "Use ≤ not < when comparing prices — a day with equal price is still dominated and its span should be absorbed."
**R:** "O(n) amortised time over n calls, O(n) space. For 10,000 trading days, this runs in ~0.5ms vs ~100ms for the naive scan — critical for real-time streaming dashboards."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Backward linear scan per call | n is small (≤ 100 calls) | O(n²) total — too slow for a live data stream |
| Segment tree with range-max | Arbitrary range queries in O(log n) | Overcomplicated — we only need the span ending at today, not arbitrary ranges |

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
- Leadership principle: Earn Trust

**Full STAR Story — "Mediating a Conflicting Schema Migration Between Two Teams":**

**S (20%):** "At a mid-size e-commerce company, two backend teams simultaneously submitted schema migrations to the shared orders database — one adding a `status_v2` column with a new enum, the other renaming the same column from `status` to `order_status`. Applied together they caused a constraint violation that took down the checkout service for 22 minutes during peak traffic, affecting ~$140K in GMV."
**T:** "I was the on-call platform engineer and the only person with write access to the migration pipeline. Goal: restore checkout within 30 minutes and prevent the conflict pattern from recurring."
**A (60% — use 'I' not 'we'):** "(1) I immediately rolled back both migrations using the pipeline's rollback command and confirmed checkout was restored within 8 minutes. (2) I held a joint postmortem with both teams the same day — I presented the conflict timeline neutrally, without assigning blame, to build trust before proposing changes. (3) I implemented a mandatory migration-preview CI check that dry-runs all pending migrations together and fails if any constraint violations are detected. (4) I wrote the postmortem doc and got sign-off from both team leads, then presented it to the CTO as a process improvement."
**R (20%):** "Zero migration-caused outages in the following 8 months. The CI check caught 3 further conflicts before they hit staging. Both team leads cited the postmortem as a model for cross-team incident handling — I was asked to run the next two postmortems as a facilitator."
*Works for LP questions on: Earn Trust; Ownership; Insist on the Highest Standards.*

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
