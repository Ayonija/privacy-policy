# Day 21 — Stacks: Basics & Expression Evaluation
**Week 03 | Phase 1: DSA Mastery | Month 1**

## Focus
Introduce the stack data structure through its two most fundamental interview applications: balanced parentheses and expression evaluation.

## DSA (2 hours)
### Pattern: Stack — LIFO & Matching
- A stack tracks "open" or "pending" items; each new item is either matched against the top or pushed. When the stack is empty at the end, all items were matched.
- Trigger condition: "balanced brackets/parentheses", "evaluate a postfix expression", "decode a nested structure" — any problem where the most-recently-seen item must be resolved first.
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Valid Parentheses | 20 | Easy | Stack matching | Push open brackets; on closing bracket check stack top for matching open; return stack empty at end |
| 2 | Evaluate Reverse Polish Notation | 150 | Medium | Stack arithmetic | Push numbers; on operator pop two operands, compute, push result; final answer is the one remaining element |
| 3 | Decode String | 394 | Medium | Stack (nested state) | Push current string and count onto stack when `[` is seen; on `]` pop and repeat the string `count` times |

### Code Skeleton
```python
# Valid Parentheses (LC 20)
def is_valid(s):
    stack = []
    match = {')': '(', '}': '{', ']': '['}
    for ch in s:
        if ch in match:
            if not stack or stack[-1] != match[ch]:
                return False
            stack.pop()
        else:
            stack.append(ch)
    return len(stack) == 0

# Decode String (LC 394) — skeleton
def decode_string(s):
    stack = []   # stores (current_string, repeat_count)
    curr_str, curr_num = "", 0
    for ch in s:
        # handle digit, '[', ']', letter
        pass
    return curr_str
```

## System Design (1 hour)
### Topic: CAP Theorem — Introduction
- **CAP** stands for Consistency, Availability, and Partition Tolerance — three properties a distributed system can aim for.
- **Consistency (C):** every read receives the most recent write or an error — all nodes see the same data at the same time.
- **Availability (A):** every request receives a (non-error) response — though it may not be the most recent data.
- **Partition Tolerance (P):** the system continues to operate despite network partitions (message loss between nodes).
- Brewer's theorem (2000): a distributed system can guarantee at most two of the three simultaneously; since network partitions are unavoidable in practice, real systems choose between C and A when a partition occurs.
- Interview talking point: "If asked what CAP means in practice, answer: you must always tolerate partitions in a real distributed system, so the real choice is between Consistency and Availability during a partition — either refuse requests (CP) or serve potentially stale data (AP)."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a situation where you had to maintain strict ordering — nothing could be processed out of sequence — analogous to a stack enforcing LIFO order.
- Leadership principle: Insist on the Highest Standards

## Flashcards

| Q | A |
|---|---|
| How do you check that parentheses are balanced using a stack? | Push open brackets; on each closing bracket, if stack is empty or top doesn't match the closing bracket return false; return `stack == []` at end |
| How do you evaluate a Reverse Polish Notation expression? | Push numbers; on operator pop two values (second operand first), apply operator, push result; the single remaining element is the answer |
| How do you decode a nested encoded string like `3[a2[bc]]`? | Push `(curr_string, curr_num)` when `[` is seen; on `]` pop and append `curr_string * count` to the restored string |
| What are the three properties of the CAP theorem? | Consistency (all nodes see the same data), Availability (every request gets a response), Partition Tolerance (system works despite network splits) |
| Why can't a distributed system guarantee all three of CAP simultaneously? | During a network partition, you must choose: either refuse requests to stay consistent (CP) or return possibly stale data to stay available (AP) — you cannot do both |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
