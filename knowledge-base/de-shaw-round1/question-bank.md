# DE Shaw Round 1 — Question Bank
**Recently reported problems (2022–2025) | Sorted by frequency**

> Use this as your mock problem pool. Each entry includes the trigger cue so you can practice classification before looking at the solution.

---

## Tier 1 — Seen Multiple Times (Very High Probability)

| # | Problem | LC # | Pattern | Difficulty | Last Seen |
|---|---------|------|---------|------------|-----------|
| 1 | Maximum Subarray | 53 | Kadane's | Medium | 2024 |
| 2 | Two Sum | 1 | HashMap | Easy | 2024 |
| 3 | Number of Islands | 200 | BFS/DFS Grid | Medium | 2024 |
| 4 | 3Sum | 15 | Sort + Two Pointers | Medium | 2024 |
| 5 | Longest Substring Without Repeating Characters | 3 | Sliding Window | Medium | 2024 |
| 6 | Merge Intervals | 56 | Sort + Greedy | Medium | 2024 |
| 7 | Jump Game | 55 | Greedy | Medium | 2024 |
| 8 | Maximum Product Subarray | 152 | DP/Kadane variant | Medium | 2023 |
| 9 | Find All Anagrams in a String | 438 | Sliding Window | Medium | 2023 |
| 10 | Subarray Sum Equals K | 560 | Prefix Sum + HashMap | Medium | 2023 |

**STAR framing tip for Tier 1 problems:** In your Situation, name the array size and state the naive O(n²) or worse approach explicitly — DE Shaw interviewers want to see you know the brute force before you optimize. In your Action, narrate the single key decision that enables the optimal approach (e.g., "I store the complement in a HashMap so the lookup is O(1)" for Two Sum, or "I track both max and min products because a negative flips the optimal substructure" for Max Product). Your Result must include the complexity improvement as a ratio: "O(n) vs O(n²) — at n=10^5 that's 10^5 operations vs 10^10." The most common mistake is jumping straight to the optimal solution without first stating the brute force — interviewers cannot evaluate your reasoning if they don't hear the baseline you're optimizing from.

---

## Tier 2 — Reported 1-2 Times (Medium Probability)

| # | Problem | LC # | Pattern | Difficulty | Last Seen |
|---|---------|------|---------|------------|-----------|
| 11 | Rotate Image | 48 | Matrix In-place | Medium | 2024 |
| 12 | Spiral Matrix | 54 | Matrix Simulation | Medium | 2024 |
| 13 | Validate Binary Search Tree | 98 | DFS with bounds | Medium | 2023 |
| 14 | Daily Temperatures | 739 | Monotonic Stack | Medium | 2023 |
| 15 | Largest Rectangle in Histogram | 84 | Monotonic Stack | Hard | 2024 |
| 16 | Trapping Rain Water | 42 | Two Pointers | Hard | 2023 |
| 17 | Word Search | 79 | DFS Backtracking | Medium | 2023 |
| 18 | Longest Common Prefix | 14 | String | Easy | 2023 |
| 19 | Container With Most Water | 11 | Two Pointers | Medium | 2023 |
| 20 | Group Anagrams | 49 | HashMap + Sorting | Medium | 2023 |
| 21 | Kth Largest Element in Array | 215 | Heap / QuickSelect | Medium | 2023 |
| 22 | Binary Tree Level Order Traversal | 102 | BFS | Medium | 2022 |
| 23 | Lowest Common Ancestor of BST | 235 | DFS | Medium | 2022 |
| 24 | Course Schedule | 207 | Topological Sort | Medium | 2022 |
| 25 | Product of Array Except Self | 238 | Prefix/Suffix Arrays | Medium | 2022 |

**STAR framing tip for Tier 2 problems:** These questions skew toward pattern combinations and non-obvious optimizations — your Situation must name the constraint that rules out simpler approaches (e.g., "in-place" for Rotate Image, "no division" for Product of Array Except Self, "integer overflow" for BST validation). In your Action, call out the key insight that unlocks the efficient solution: for matrix problems, state the geometric equivalence (transpose + reverse = 90° rotation); for monotonic stack problems, explain the invariant the stack maintains (each element pushed/popped at most once). Your Result should address a concrete follow-up DE Shaw commonly asks: for Largest Rectangle in Histogram, be ready to explain why `width = i - stack.top() - 1` instead of `i - idx`. The most common mistake on Tier 2 problems is under-explaining the non-obvious step — interviewers use these questions specifically to test depth of understanding, so spend 70% of your Action time on the one tricky decision.

---

## Tier 3 — Occasionally Reported

| # | Problem | LC # | Pattern | Difficulty |
|---|---------|------|---------|------------|
| 26 | Set Matrix Zeroes | 73 | Matrix In-place | Medium |
| 27 | Decode Ways | 91 | DP | Medium |
| 28 | Minimum Path Sum | 64 | DP Grid | Medium |
| 29 | Coin Change | 322 | DP | Medium |
| 30 | House Robber | 198 | DP | Medium |
| 31 | Pascal's Triangle | 118 | Simulation | Easy |
| 32 | Search in Rotated Sorted Array | 33 | Binary Search | Medium |
| 33 | Find Minimum in Rotated Sorted Array | 153 | Binary Search | Medium |
| 34 | Clone Graph | 133 | BFS/DFS | Medium |
| 35 | Pacific Atlantic Water Flow | 417 | Multi-source BFS | Medium |

**STAR framing tip for Tier 3 problems:** These are lower-probability but often harder to explain quickly — your Situation should include the DP subproblem definition stated as one sentence before writing any recurrence (e.g., "dp[i] = minimum coins to make amount i" for Coin Change; "dp[i] = number of ways to decode s[0..i-1]" for Decode Ways). In your Action, state the recurrence, the fill direction (left-to-right for Coin Change unbounded, right-to-left for 0/1 knapsack), and one edge case that breaks naive attempts (e.g., the leading-zero check in Decode Ways, or the first/last element special case in House Robber). Your Result metric for DP problems should compare state space: "O(n) states × O(1) transition = O(n) total, vs O(2^n) for brute-force recursion without memoization." The most common mistake is confusing unbounded knapsack (inner loop forward) with 0/1 knapsack (inner loop backward) — explicitly state which variant applies and why.

---

## Pattern Frequency Summary

```
HashMap / Frequency Map        ████████████  (12 problems)
Sliding Window                 ████████      (8 problems)
BFS / DFS (Grid + Tree)        ████████      (8 problems)
Two Pointers                   ███████       (7 problems)
Dynamic Programming            ███████       (7 problems)
Monotonic Stack                █████         (5 problems)
Binary Search                  ████          (4 problems)
Matrix Simulation              ████          (4 problems)
Sorting + Greedy               ████          (4 problems)
Prefix Sum                     ███           (3 problems)
```

**Conclusion:** Master HashMap + Sliding Window + BFS/DFS first. These three patterns alone cover ~40% of what DE Shaw asks.

---

## Quick Classification Practice

Cover the "Pattern" column. Read problem name → say pattern aloud → reveal.

| Problem | Pattern |
|---------|---------|
| Maximum Subarray | Kadane's (local max reset) |
| Two Sum | HashMap (complement lookup) |
| Longest Substring Without Repeating | Sliding window (shrink on duplicate) |
| Number of Islands | DFS / BFS on grid (sink visited cells) |
| Subarray Sum Equals K | Prefix sum + HashMap |
| Merge Intervals | Sort by start + greedy merge |
| Daily Temperatures | Monotonic decreasing stack |
| Trapping Rain Water | Two pointers (leftMax, rightMax) |
| Jump Game | Greedy (track maxReach) |
| Maximum Product Subarray | Track both max AND min (negatives flip sign) |
| Largest Rectangle in Histogram | Monotonic increasing stack |
| Group Anagrams | HashMap (sorted string as key) |
| Product of Array Except Self | Left-pass prefix + right-pass suffix |
| Search in Rotated Sorted Array | Binary Search (identify sorted half) |
| Course Schedule | Topological Sort / cycle detection |

---

## DE Shaw Interview Tips (From Candidates)

1. **They want you to talk:** narrate your classification before coding. "This looks like a sliding window because I need the longest substring with a constraint."

2. **Clean code matters:** they explicitly note code quality — use meaningful variable names (`left`, `right`, not `i`, `j` for pointers).

3. **State complexity unprompted:** before saying "done", say "this is O(n) time and O(1) space."

4. **Common follow-ups:** "Can you do it without extra space?" / "What if the array has duplicates?" / "What's the space complexity of the recursion stack?" — prepare these for every solution.

5. **Two problems, 30 minutes:** they know 15 min per problem is tight. A correct brute force + stated optimization scores better than an incomplete optimal solution.

6. **Don't skip edge cases in-code:** an empty array/null root check at the top of every function signals maturity. It takes 1 line and impresses consistently.
