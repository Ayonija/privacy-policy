# Week 04 Review — Days 22–28

## Phase
Phase 1: DSA Mastery | Month 1

## Patterns Covered This Week
- Monotonic Stack (decreasing) — Next Greater Element I, II
- Monotonic Stack — Daily Temperatures (index-based)
- Monotonic Stack — Online Stock Span (span accumulation)
- Monotonic Stack — Asteroid Collision (collision simulation)
- Stack — String processing: adjacent duplicate cancellation, backspace compare
- Stack — Path simplification (Simplify Path)
- Stack — Index-based validation (Minimum Remove for Valid Parentheses)
- Queue — Two-stack implementation (amortised O(1))
- Deque — Reveal Cards In Increasing Order (simulation)
- Queue — Greedy simulation (Dota2 Senate)
- Stack — Parallel min tracking (Min Stack)
- Stack — Balance counting O(1) space (Minimum Add to Make Valid)
- Stack — Depth-based score (Score of Parentheses)
- Stack — Fleet merging (Car Fleet)
- Monotonic Stack — Contribution technique (Sum of Subarray Ranges)
- Single-queue stack (Implement Stack using Queues)
- Queue — Sliding window (Number of Recent Calls)
- Stack — Case-pair cancellation (Make The String Great)
- Stack — Star-pop (Removing Stars From a String)

## System Design Topics Covered
**CAP Theorem (Days 22–28):** Full CAP coverage — Consistency (Raft/Paxos quorum, linearisability vs. read-your-writes), Availability (Cassandra consistency levels, LWW, vector clocks), Partition Tolerance (CP vs AP trade-off, split-brain prevention). Extended with PACELC — everyday latency vs. consistency trade-off, DynamoDB/Cassandra as PA/EL, Zookeeper/etcd as PC/EC, Google Spanner as PC/EL hybrid. Practical examples: DNS (eventual), shopping cart (AP + union merge), collaborative docs (causal), payment systems (CP).

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (fill in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Monotonic Stack — Next Greater (basic) | | |
| Monotonic Stack — circular array | | |
| Monotonic Stack — span accumulation | | |
| Monotonic Stack — collision simulation | | |
| Monotonic Stack — contribution technique | | |
| Stack — string cancel/undo semantics | | |
| Stack — path simplification | | |
| Queue — two-stack implementation | | |
| Queue — greedy simulation | | |
| Min Stack (parallel tracking) | | |
| Balance counting (O(1) space) | | |
| CAP theorem — CP vs AP decision | | |
| PACELC — latency vs. consistency | | |

## Weekly Flashcard Deck
All 35 cards for Days 22–28:

| Q | A |
|---|---|
| How does a monotonic decreasing stack find the Next Greater Element? | Maintain pending indices; when current > top, top has found its NGE — pop and record `result[top] = current` |
| How do you handle circular array in Next Greater Element II? | Iterate 0 to 2n−1 using `index = i % n`; second pass resolves wrap-around elements |
| What does the stack store in Daily Temperatures? | Indices — you need the index to compute days elapsed (`current − stack_top`) |
| How does Raft achieve strong consistency? | All writes go through the elected leader; acknowledged only after a quorum of nodes persists it |
| What is the difference between strong consistency and read-your-writes? | Strong: all clients see latest write immediately. Read-your-writes: only the same client sees their own write immediately |
| How does Online Stock Span accumulate span across days? | Stack stores `(price, span)` pairs; absorb smaller-or-equal spans before pushing; return accumulated span |
| How do you handle equal-size asteroids in Asteroid Collision? | Both destroyed — pop the stack top and do NOT push the current asteroid |
| How does Backspace String Compare work in O(1) space? | Two pointers scanning backward, counting `#` to skip characters; compare next valid chars |
| What is the key difference between CAP Availability and SLA uptime? | CAP availability: returns a response during a partition (no coordination errors). SLA uptime: total operational hours percentage |
| How does Cassandra's consistency level (ONE vs ALL) trade off availability? | ONE: fastest, most available, most stale. ALL: fully consistent but fails if any replica unreachable |
| How do you simplify a Unix file path? | Split by `/`; skip empty strings and `.`; pop stack on `..`; push valid names; join with `/` prefix |
| How do you find unmatched indices in Minimum Remove for Valid Parentheses? | Stack collects unmatched `(` indices; set collects unmatched `)` on the fly; add remaining stack to set; rebuild skipping all |
| Why is Partition Tolerance mandatory in real distributed systems? | Network splits are inevitable in production; a system that cannot handle them cannot operate reliably |
| What is a split-brain scenario and how do CP systems avoid it? | Minority partition keeps accepting writes diverging from majority; CP systems refuse minority writes to prevent this |
| Give one CP and one AP example system. | CP: bank ledger / Zookeeper. AP: shopping cart / Cassandra / DynamoDB |
| Why is the two-stack queue amortised O(1)? | Each element is pushed once and popped once — at most 2 operations total regardless of call count |
| How does Dota2 Senate greedy work? | Smaller index acts first; bans the other senator; winner re-queues at `index + n` to participate next round |
| How do you reconstruct deck order in Reveal Cards In Increasing Order? | Sort descending; simulate backward with deque of positions — move back to front, assign current card to front |
| What is the difference between eventual and causal consistency? | Eventual: all replicas converge, no ordering guarantee. Causal: causally related writes seen in order by all clients |
| Name one real system for strong, causal, and eventual consistency. | Strong: Zookeeper. Causal: MongoDB (causal sessions). Eventual: DynamoDB default / DNS |
| How does Min Stack achieve O(1) getMin? | Store `(value, current_min)` pairs; current_min is the minimum of all values pushed so far — never recalculated on pop |
| How do you solve Minimum Add to Make Valid in O(1) space? | Count unmatched `)` and remaining unmatched `(` — answer is their sum |
| How does Score of Parentheses handle nested scoring? | Stack accumulates depth; on `)`, pop and either add 1 (for `()`) or double and add to new top |
| What conflict resolution does Amazon Dynamo use for shopping carts? | Application-level merge — take union of items; user resolves ambiguity at checkout |
| What is a CRDT and when is it preferable to LWW? | Conflict-free Replicated Data Type — auto-merges without conflicts; preferable when you need automatic resolution |
| Why does Car Fleet sort by position descending? | Process cars front-to-back; a rear car can only catch one ahead — sort ensures we check in encounter order |
| When does a car NOT push a new fleet entry? | When its time-to-target ≤ stack top (it catches the fleet ahead) |
| How does the contribution technique compute sum of subarray minima? | For each element: `left[i]` = distance to previous smaller, `right[i]` = distance to next smaller-or-equal; contribution = `nums[i] * left[i] * right[i]` |
| What does the ELC part of PACELC stand for? | Else (normal operation) — choose between Latency and Consistency |
| Why does Spanner achieve low latency despite external consistency? | TrueTime bounds clock skew — waits only ~7ms uncertainty window rather than a full quorum round-trip |
| How does Number of Recent Calls maintain its window? | Append each ping; pop from front while front < `t − 3000`; return `len(deque)` |
| How does Make The String Great detect a bad pair? | `stack[-1].lower() == ch.lower()` and `stack[-1] != ch` — same letter, different case |
| How does Removing Stars differ from Remove Adjacent Duplicates? | Stars explicitly pop the preceding character; adjacent duplicates pop only when matching the exact same character |
| What is the three-step CAP decision guide? | 1. Harmful if incorrect (money/inventory)? → CP. 2. Availability matters more than staleness? → AP. 3. Need causal ordering? → causal consistency |
| How do you design a highly available distributed counter? | Redis INCR per shard (AP, O(1)); async flush to main DB; tolerate seconds of staleness for non-critical counts |

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| — | — | — | — | — |
