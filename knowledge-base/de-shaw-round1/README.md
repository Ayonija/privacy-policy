# DE Shaw — Round 1 DSA Prep (2 Days)
**30-minute coding round | 2 problems | Medium difficulty**

---

## What DE Shaw Round 1 Looks Like

| Metric | Detail |
|--------|--------|
| Duration | 30 minutes |
| Problems | 2 (aim: solve both, get partial on 2nd if stuck) |
| Difficulty | Leetcode Medium, occasionally Easy+hard combo |
| Language | Your choice — Java/C++/Python all fine |
| Platform | HackerEarth or proprietary portal |
| Graded on | Correctness, time complexity, code clarity |

**The real skill tested:** Can you recognize the pattern in <5 minutes and write clean code in <15 minutes per problem?

---

## 2-Day Schedule

### Day 1 — Pattern Recognition Engine
- Morning (2h): Read [pattern-cheatsheet.md](pattern-cheatsheet.md) — memorize the 5-question trigger framework
- Afternoon (3h): Work through [day-1.md](day-1.md) — 8 core problems (timed, 15 min each)
- Evening (1h): Re-read cheatsheet; write skeletons from memory

### Day 2 — DE Shaw Specific Drill
- Morning (2h): Work through [day-2.md](day-2.md) — DE Shaw high-frequency problems
- Afternoon (2h): Full 30-min mock: pick 2 problems from [question-bank.md](question-bank.md), set timer
- Evening (1h): Review all weak spots, re-read flashcards

---

## Files in This Folder

| File | Purpose |
|------|---------|
| [pattern-cheatsheet.md](pattern-cheatsheet.md) | 1-page trigger table — read this first and last |
| [day-1.md](day-1.md) | Core patterns with DE Shaw frequency, problems + solutions |
| [day-2.md](day-2.md) | DE Shaw specific questions with walk-throughs |
| [question-bank.md](question-bank.md) | Full list of recently reported DE Shaw questions |

---

## The 30-Minute Breakdown

```
00:00 – 05:00  Read problem 1. Classify pattern. State brute force aloud/in comments.
05:00 – 15:00  Code optimal solution for problem 1. Test 2 examples mentally.
15:00 – 20:00  Read problem 2. Classify. State approach.
20:00 – 28:00  Code problem 2.
28:00 – 30:00  Final pass: edge cases, variable names, remove debug prints.
```

**If stuck after 8 minutes:** write the brute force, note the optimization, move on — partial credit beats zero.

---

## STAR Method for DE Shaw Round 1

DE Shaw Round 1 is a fast-paced coding screen — typically 2 problems in 30 minutes. Use STAR to structure your verbal reasoning before and after coding.

**Before you write any code (S+T, spend 20% of your time):**
> "I see [trigger condition] which tells me to use [pattern]. The brute-force approach would be O([X]), but I can reduce to O([Y]) by [key insight]. Let me code that up."

For example: "I see 'count subarrays with sum = k' on an unsorted array — that's the prefix sum + HashMap pattern. Brute force is O(n²) by checking all pairs; I can reduce to O(n) by storing prefix sums in a HashMap and looking up `prefixSum - k` at each step."

**While coding (A, spend 60-70% of your time):**
Narrate each step: "I'm initializing `seen = {0: 1}` because [invariant it maintains — handles subarrays starting at index 0]." "At each step I check `seen[sum - k]` because [what it guarantees — any earlier prefix that pairs with current prefix to sum to k]." "I handle [edge case] here because [why it would break otherwise — e.g., empty array, negative numbers, duplicates]."

**After coding (R, spend 10-20% of your time):**
> "Final complexity: O([Y]) time, O([Z]) space. Edge cases I handled: [list 2-3 explicitly]. This beats the O([X]) brute force by [factor] — at DE Shaw's typical constraint of n=10^5, that's the difference between ~10^10 operations and ~10^5 operations."

**DE Shaw-specific STAR tip:** They explicitly grade on communication. A candidate who says "this is O(n) because each element is pushed and popped from the stack at most once" scores higher than one who just writes the code — even if the code is identical. Always close with the complexity sentence before saying "I'm done."
