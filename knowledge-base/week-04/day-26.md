# Day 26 — Stack: Parentheses Variants
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Solve the three main parentheses variants — min additions, scoring, and min stack — building fluency with stack state tracking.

## DSA (2 hours)
### Pattern: Stack — Balance Counting & Score Accumulation
- Minimum Add to Make Valid: track open count (unmatched `(`) and close count (unmatched `)`); increment close on unmatched `)`, increment open on `(`; answer is open + close.
- Score of Parentheses: treat depth as a multiplier; `()` at depth d contributes 2^d; sum all contributions.
- Min Stack: maintain a parallel stack of minimums — push the current minimum alongside each value.
- Trigger condition: "fewest edits to fix parentheses" (balance counting) OR "score/value that doubles with nesting" (depth tracking) OR "O(1) getMin with push/pop" (parallel min stack).
- Time complexity: O(n) | Space complexity: O(n) for min stack, O(1) for balance counting

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Min Stack | 155 | Easy | Parallel min tracking | Store `(value, current_min)` pairs; getMin is always `stack[-1][1]` |
| 2 | Minimum Add to Make Parentheses Valid | 921 | Medium | Balance counting (no stack needed) | Count unmatched open (`open_count`) and unmatched close (`close_count`); answer = open + close |
| 3 | Score of Parentheses | 856 | Medium | Stack depth as multiplier | Each `()` at depth d = 2^d; use a stack to track depth; or: every `()` contributes the current depth value |

### Code Skeleton
```python
# Min Stack (LC 155)
class MinStack:
    def __init__(self):
        self.stack = []  # (val, current_min)

    def push(self, val):
        curr_min = min(val, self.stack[-1][1] if self.stack else val)
        self.stack.append((val, curr_min))

    def pop(self): self.stack.pop()
    def top(self): return self.stack[-1][0]
    def getMin(self): return self.stack[-1][1]

# Minimum Add to Make Valid (LC 921) — O(1) space
def min_add_to_make_valid(s):
    open_count = close_count = 0
    for ch in s:
        if ch == '(':
            open_count += 1
        elif open_count > 0:
            open_count -= 1
        else:
            close_count += 1
    return open_count + close_count

# Score of Parentheses (LC 856) — skeleton
def score_of_parentheses(s):
    stack = [0]   # base score at current depth
    for ch in s:
        # '(' pushes a new 0; ')' pops and either adds 1 or doubles
        pass
    return stack[0]
```

## System Design (1 hour)
### Topic: Eventual Consistency in Practice
- **DNS:** a canonical AP-eventual system — TTL-based caching means a zone update takes minutes to hours to propagate globally; clients may see stale records.
- **Shopping cart (Amazon Dynamo):** designed AP; two users adding to a cart on different partitions produces two versions; on merge, take the union (no item is silently lost); user resolves ambiguity at checkout.
- **Social media likes/view counts:** exact count is not required; an approximate count with eventual consistency is acceptable — MongoDB Atlas or Redis with async replication.
- **Conflict resolution strategies:** last-write-wins (LWW) — simple but may lose data; application-level merge (Dynamo shopping cart) — safe but complex; CRDT (Conflict-free Replicated Data Type) — automatic, mathematically guaranteed convergence.
- Interview talking point: "If asked how Amazon's shopping cart survives a datacenter partition, answer: DynamoDB (the inspiration for Dynamo) keeps the cart available on both sides; on reconnect, conflicting versions are presented to the application layer for merge — the cart takes the union of both item lists so no item is ever silently lost, even at the cost of occasionally showing a removed item."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you had to reconcile two divergent versions of work (e.g., two branches, two document drafts) — analogous to merging conflicting replica states in an eventual consistency system.
- Leadership principle: Invent and Simplify

## Flashcards

| Q | A |
|---|---|
| How does Min Stack achieve O(1) getMin without scanning? | Store `(value, current_min)` pairs; current_min at each level is the minimum of all values pushed so far; it never needs recalculation on pop |
| How do you solve Minimum Add to Make Parentheses Valid in O(1) space? | Count unmatched `)` (when no open to match) and remaining unmatched `(` at the end; answer = their sum |
| How does Score of Parentheses handle nested scoring? | Each `()` at depth d contributes 2^d; the stack accumulates partial scores — on `)`, pop and either add 1 (for `()`) or double the popped score and add to the new top |
| What conflict resolution strategy does Amazon Dynamo use for shopping carts? | Application-level merge — on conflict, present both versions to the client; the cart takes the union of items (nothing silently lost), with user resolution at checkout |
| What is a CRDT and when is it preferable to LWW? | Conflict-free Replicated Data Type — a data structure that automatically merges replicas without conflicts (e.g., G-Counter, OR-Set); preferable when you need automatic resolution without application-level logic |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
