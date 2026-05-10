# Day 30 — Stack Synthesis: Remove Duplicate Letters & 132 Pattern
**Week 05 | Phase 1: DSA Mastery | Month 1**

## Focus
Month 1 close-out: solve two advanced monotonic stack problems that combine greedy + counting logic, then complete the Month 1 review.

## DSA (2 hours)
### Pattern: Monotonic Stack — Greedy with Last-Occurrence Constraint & Behind-Pattern Detection
- Remove Duplicate Letters: keep the lexicographically smallest result; use a monotonic increasing stack, but only pop when the character to be removed still appears later in the string (check via last-occurrence index).
- 132 Pattern: find indices i < j < k where nums[i] < nums[k] < nums[j]; scan right-to-left, maintain a monotonic decreasing stack to track candidates for nums[k], and a running minimum for nums[i].
- Trigger condition: "lexicographically smallest result with all unique characters" → greedy + last-occurrence; "find a three-element subsequence with a specific ordering" → stack + running min.
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove Outermost Parentheses | 1021 | Easy | Depth counter | Track depth with a counter; output chars only when depth > 0 (exclude outermost open) or depth > 1 (exclude outermost close) |
| 2 | Remove Duplicate Letters | 316 | Medium | Monotonic stack + last-occurrence | Pop stack top only if it appears again later (`last[ch] > i`); mark chars in stack to avoid duplicates |
| 3 | 132 Pattern | 456 | Medium | Monotonic stack (right-to-left) + running min | Scan right-to-left; stack holds candidates for nums[k] (the "2"); pop when top < current (update third); return true if any nums[i] < third |

### Code Skeleton
```python
# Remove Duplicate Letters (LC 316)
def remove_duplicate_letters(s):
    last = {ch: i for i, ch in enumerate(s)}
    stack = []
    in_stack = set()
    for i, ch in enumerate(s):
        if ch in in_stack:
            continue
        while stack and ch < stack[-1] and last[stack[-1]] > i:
            in_stack.discard(stack.pop())
        stack.append(ch)
        in_stack.add(ch)
    return "".join(stack)

# 132 Pattern (LC 456)
def find132pattern(nums):
    stack = []   # candidates for nums[k] (the "2" in 132)
    third = float('-inf')   # current best nums[k]
    for n in reversed(nums):
        if n < third:
            return True
        while stack and stack[-1] < n:
            third = stack.pop()
        stack.append(n)
    return False
```

## System Design (1 hour)
### Topic: Month 1 System Design Synthesis — Big-O → Scaling → CAP
- **Month 1 system design arc:** Big-O + RAM model → Vertical/Horizontal scaling → Load balancers (L4 vs L7, algorithms, health checks) → CAP theorem (CP vs AP) → Consistency models (strong, causal, eventual) → PACELC → Real systems (Zookeeper, Cassandra, DynamoDB, Spanner).
- **The interview flow:** when asked to design any distributed system, the conversation naturally follows: capacity estimates → single server → scale the app tier → scale the data tier → choose consistency model → handle failures.
- The CAP decision is always tied to the data type: financial data → CP; user-facing features where availability matters → AP; real-time collaboration → causal.
- Always end a system design answer with failure modes: "what happens when a node crashes?" and "what happens during a network partition?"
- Interview talking point: "If asked to start a system design for any large-scale service, answer: 1. Clarify requirements and scale (DAU, QPS, storage). 2. High-level components (LB → App servers → Cache → DB). 3. Deep dive on the hardest component. 4. Discuss consistency model choice. 5. Address failures and monitoring."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Reflect on the last 30 days — what was the one pattern you found hardest, and what specific action will you take in Month 2 to close that gap?
- Leadership principle: Learn and Be Curious

## Flashcards

| Q | A |
|---|---|
| What is the key guard condition that prevents Remove Duplicate Letters from removing a character that won't appear again? | `last[stack[-1]] > i` — the character at the top must still appear at a future index for it to be safe to pop |
| How do you avoid re-processing already-added characters in Remove Duplicate Letters? | Maintain an `in_stack` set; skip any character that is already in the stack |
| In the 132 Pattern right-to-left scan, what does the `third` variable represent? | The best candidate for nums[k] (the "2" in 1-3-2) — the largest value seen so far that was popped because a bigger value arrived |
| What is the correct order of steps in a system design interview? | 1. Clarify requirements + scale. 2. High-level components. 3. Deep-dive hardest piece. 4. Consistency model choice. 5. Failure handling + monitoring |
| What is the system design question to always end with? | "What happens when a node crashes?" and "What happens during a network partition?" — this shows you understand distributed systems |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
