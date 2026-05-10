# Day 24 — Stack: String Processing & Path Simplification
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Use the stack to process strings with undo/cancel semantics and hierarchical path rules — common in both interviews and real system tooling.

## DSA (2 hours)
### Pattern: Stack — String Processing with Undo/Cancel Semantics
- Remove Adjacent Duplicates: push characters; before pushing, if top equals current character, pop instead — the stack holds the de-duplicated result.
- Simplify Path: split on `/`, process tokens — skip `.` and empty strings, pop stack on `..`, otherwise push.
- Minimum Remove for Valid Parentheses: first pass uses a stack of indices to find unmatched open brackets; second pass marks unmatched indices; third pass builds result skipping marked positions.
- Trigger condition: "characters cancel each other (adjacent duplicates, backspace)" OR "navigate a hierarchy (file paths, nested directories)."
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove All Adjacent Duplicates In String | 1047 | Easy | Stack (cancel semantics) | Push char; if top == current char, pop instead; join stack at end |
| 2 | Simplify Path | 71 | Medium | Stack (hierarchy navigation) | Split by `/`; push directory names; pop on `..`; skip `.` and empty; join with `/` |
| 3 | Minimum Remove to Make Valid Parentheses | 1249 | Medium | Stack of indices + set | Collect unmatched `(` indices in stack; unmatched `)` indices in set; rebuild string skipping both |

### Code Skeleton
```python
# Remove Adjacent Duplicates (LC 1047)
def remove_duplicates(s):
    stack = []
    for ch in s:
        if stack and stack[-1] == ch:
            stack.pop()
        else:
            stack.append(ch)
    return "".join(stack)

# Simplify Path (LC 71)
def simplify_path(path):
    stack = []
    for part in path.split("/"):
        if part == "..":
            if stack: stack.pop()
        elif part and part != ".":
            stack.append(part)
    return "/" + "/".join(stack)

# Minimum Remove for Valid Parentheses (LC 1249) — skeleton
def min_remove_to_make_valid(s):
    stack = []      # indices of unmatched '('
    remove = set()  # all indices to remove
    # pass 1: find unmatched ')' and collect unmatched '(' indices
    # pass 2: add remaining stack indices to remove
    # pass 3: rebuild string
    pass
```

## System Design (1 hour)
### Topic: CAP — Partition Tolerance & the C vs A Trade-off
- **Partition Tolerance** is non-negotiable in any real distributed system — network delays and packet drops happen; you cannot build a system that assumes a perfect network.
- Therefore, the real design question is: **during a network partition, do you favour C or A?**
- **CP design:** reject writes (or reads) from minority partitions to prevent split-brain; guarantees data correctness at the cost of availability. Example: a bank balance must never go negative — a CP design is correct here.
- **AP design:** allow all partitions to accept reads and writes; reconcile conflicts after the partition heals. Example: a shopping cart or social media like count — slightly stale data is acceptable; unavailability is not.
- Interview talking point: "If asked whether a payment system should be CP or AP, answer: CP — a payment must be atomic and consistent across all nodes; returning an availability error is better than charging a user twice or allowing an overdraft; use a consensus protocol (Raft/Paxos) to ensure all replicas agree before committing."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Tell me about a time you had to choose between being correct and being responsive under pressure — analogous to the CP vs AP trade-off where you cannot have both at once during a failure.
- Leadership principle: Have Backbone; Disagree and Commit

## Flashcards

| Q | A |
|---|---|
| How do you simplify a Unix file path using a stack? | Split by `/`; skip empty strings and `.`; pop stack on `..`; push valid directory names; join with `/` prefixed |
| How do you find all indices to remove in Minimum Remove for Valid Parentheses? | Stack collects unmatched `(` indices; a set collects unmatched `)` indices on the fly; add remaining stack contents to the set; rebuild skipping all set indices |
| Why is Partition Tolerance effectively mandatory in real distributed systems? | Network splits are inevitable (hardware failure, packet loss); a system that cannot handle them cannot operate in production |
| What is a split-brain scenario in a CP system? | When a partition isolates a minority shard that continues accepting writes — its data diverges from the majority and cannot be safely merged; CP systems avoid this by refusing minority writes |
| Give one example each of a system that should be CP and one that should be AP. | CP: bank ledger / payment processing (incorrect data is worse than unavailability). AP: shopping cart / social media like count (availability matters more than perfect consistency) |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
