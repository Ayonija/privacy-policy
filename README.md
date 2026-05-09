# DSA Mastery Atlas — FAANG L5/L6

An interactive, single-file DSA reference designed to take you from zero to FAANG L5/L6 readiness. No install, no build step — open `dsa-atlas-ultimate.html` in any browser and start learning.

---

## How to Open

```
open dsa-atlas-ultimate.html       # macOS
start dsa-atlas-ultimate.html      # Windows
xdg-open dsa-atlas-ultimate.html   # Linux
```

Or serve locally to avoid any cross-origin quirks:

```
npx serve .
python3 -m http.server 8080
```

`index.html` auto-redirects to `dsa-atlas-ultimate.html`, so any local server will work immediately.

---

## What's Inside

### Tab Navigation

The top bar has 16 tabs. Click any tab to jump to that section — no page reload.

| Tab | ID | Topics covered |
|---|---|---|
| Pattern Map | t0 | The meta-skill: classify any problem before writing code |
| Arrays | t1 | Two Pointers, Sliding Window, Prefix Sum, Kadane, Monotonic Stack |
| Linked Lists | t2 | Fast+Slow, Reversal, Merge, Cycle Detection, LRU Cache |
| Trees | t3 | DFS Pre/In/Post, BFS Level-Order, LCA, Serialize, Morris Traversal |
| Graphs | t4 | BFS, DFS, Union-Find, Toposort, Dijkstra, Bellman-Ford, Floyd-Warshall |
| Binary Search | t5 | Exact, Left Bound, Right Bound, Answer Space, Rotated Array |
| Sorting | t6 | Merge, Quick, Counting, Radix, Custom Comparators, Count Inversions |
| Heap + Trie | t7 | Top-K, Kth Largest, Median Stream, Merge K Lists, Prefix Tree |
| Backtracking | t8 | Subsets, Permutations, Combinations, N-Queens, Word Search, Sudoku |
| DP | t9 | 1D, Knapsack, LCS, LIS, Interval, State Machine, Grid, Tree DP, Bitmask |
| Greedy | t10 | Intervals, Jump Game, Task Scheduler, Gas Station, Activity Selection |
| Big-O | t11 | Data structure complexities, algorithm table, L5/L6 checklist |
| Bit Manipulation | t12 | XOR tricks, bit masks, powers of 2, common patterns |
| D&C + Hashing | t13 | Divide & Conquer, Suffix Arrays, Rolling Hash, Rabin-Karp |
| Design | t14 | System design patterns: LRU, LFU, Rate Limiter, consistent hashing |
| Math + Strings | t15 | Primes, GCD, modular arithmetic, string matching, tricky edge cases |
| Problem List | t16 | Curated master list mapped to patterns — use this to test yourself |

---

## Features

### Checkboxes
Every topic card has a checkbox in the top-right corner. Click it to mark a topic done. State is saved in `localStorage` — your progress persists across sessions. Use this to track which patterns you have truly internalized.

### Code Blocks
Each code block has a **copy** button (top-right of the block). Click to copy the template to clipboard.

### Flowcharts
Decision-tree flowcharts show you how to navigate edge cases step-by-step. Follow the boxes — each one is a real decision you make during an interview.

### Dry Runs
Step-through dry runs show pointer/state changes at each iteration. Read these before writing code; they build the mental model that lets you spot bugs in your own solution.

### Difficulty Badges
- `E` (green) — LeetCode Easy, warm-up
- `M` (yellow) — LeetCode Medium, core interview level
- `H` (red) — LeetCode Hard, L6 signal
- `L6` (purple) — senior/staff-level depth, system or algorithm design

### Filter Chips
The hero section has clickable filter chips. Click a pattern name to highlight only cards matching that pattern across the current tab.

---

## Recommended Learning Path

Work through the tabs in this order. Do not skip to DP before you have the foundations — DP problems are just combinations of simpler patterns.

```
1. Pattern Map      — learn to classify first, always
2. Arrays           — highest frequency; master sliding window + two pointers
3. Binary Search    — template matters more than intuition here
4. Trees            — recursive DFS is the most reused pattern in interviews
5. Linked Lists     — fast+slow pointer; practice until it's automatic
6. Graphs           — BFS/DFS + Union-Find cover ~80% of graph questions
7. Backtracking     — template-driven; one solid skeleton handles most cases
8. Sorting          — know merge sort cold; quick select for Kth element
9. Heap + Trie      — high signal at L5+; required for Top-K and autocomplete
10. DP              — do only after steps 1–9 are solid
11. Greedy          — validate with exchange argument; do after DP
12. Bit Manipulation — quick wins; ~2 problems per real interview
13. D&C + Hashing   — niche but appears in hard problems
14. Design          — for L5+; pair with system design prep
15. Math + Strings  — polish; covers edge cases interviewers love
16. Problem List    — final validation: can you identify the pattern cold?
```

---

## Rules for Getting the Most Out of This

1. **Classify before you code.** Open the Pattern Map (t0) and ask: which row describes this problem? Name the pattern out loud before touching the keyboard.

2. **Read the dry run, then close it and reconstruct it.** Do not just read — reproduce the step-by-step trace on paper. If you can't, you don't know it yet.

3. **Use the checkboxes honestly.** A topic is done only when you can solve a medium-difficulty problem using it in under 20 minutes, clean, with correct complexity analysis. Not when you've read the card.

4. **One pattern per session.** Go deep on one tab per sitting rather than skimming all tabs. Breadth feels productive; depth is what the interview measures.

5. **Big-O before you move on.** Before leaving any card, state the time and space complexity aloud. If you hesitate, check the Big-O tab (t11) and come back.

6. **Do the Problem List last.** Tab t16 is a diagnostic, not a tutorial. Use it after you have worked through the other tabs to find gaps, not to learn patterns for the first time.

7. **Copy templates, then delete them.** Use the copy button to paste a skeleton into your editor, then delete it and write it from memory. Repeat until the template is automatic.

8. **Revisit Pattern Map weekly.** As you learn more, the map becomes more useful — you will spot connections you missed earlier.

---

## Files

| File | Purpose |
|---|---|
| `dsa-atlas-ultimate.html` | The complete interactive study tool — open this in your browser |
| `index.html` | Redirects to `dsa-atlas-ultimate.html` |
| `privacy-policy.md` | Quick-reference cheat sheet: pattern table, complexity table, code skeletons, L5/L6 checklist |
| `README.md` | This file — navigation guide and learning rules |

---

## Tips for Interview Day

- The first 2 minutes: classify the problem (Pattern Map habit pays off here).
- State brute force complexity before optimizing — interviewers reward self-awareness.
- Talk through edge cases before writing: empty input, single element, duplicates, negatives, overflow.
- If you're stuck, say which pattern you think applies and why — partial credit is real.
- For L5+: be ready to discuss alternative approaches and their trade-offs unprompted.
