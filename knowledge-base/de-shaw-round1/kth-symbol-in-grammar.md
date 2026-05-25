# K-th Symbol in Grammar

**Source:** DE Shaw Round 1 (actual asked question)
**LeetCode:** 779 — K-th Symbol in Grammar
**Difficulty:** Medium
**Pattern:** Recursion + Tree thinking / Bit manipulation

---

## Problem Statement

We build a table of `n` rows (1-indexed).

We start by writing `0` in the first row. In every subsequent row, we look at the previous row and replace:
- each `0` with `01`
- each `1` with `10`

**Example for n = 3:**
```
Row 1: 0
Row 2: 01
Row 3: 0110
Row 4: 01101001
```

**Given** two integers `n` and `k`, return the **k-th (1-indexed) symbol** in the **n-th row**.

**Constraints:**
- `1 <= n <= 30`
- `1 <= k <= 2^(n-1)`

---

## STAR Interview Walkthrough

### S — Situation (Understand the Problem, ~1 min)

> "Let me read this carefully. We have a recursive string construction: every row is built by expanding each character of the previous row. Row 1 is `0`, and each subsequent row doubles in length. I need to find the k-th character of row n."

**Write out a few rows manually first — always do this in the interview:**

```
Row 1:  0
Row 2:  0  1
Row 3:  01 10
Row 4:  0110 1001
```

Now ask yourself: **"What pattern do I see?"**

- Row n has length `2^(n-1)`.
- Row n is literally `Row(n-1)` + `complement(Row(n-1))`.
  - The **left half** of row n = row n-1 exactly.
  - The **right half** of row n = bitwise complement of row n-1.

This is the key insight. Write it down explicitly before coding.

---

### T — Task (State Your Approach, ~2 min)

> "I need to return the k-th symbol in row n. Let me think about what determines that symbol."

**Brute Force — O(2^n) time, O(2^n) space:**
Build every row as a string. For n=30, row 30 has ~500 million characters. **This will MLE/TLE. Say it out loud and immediately pivot.**

> "Brute force builds all rows — that's exponential in space. I can't store a billion characters. I need to figure out the answer without building the rows."

**Optimal Insight — Navigate up the tree:**

Think of it as a **binary tree**:
- The root is position (n=1, k=1) = `0`.
- Each node at position `(row, k)` has two children at `(row+1, 2k-1)` and `(row+1, 2k)`.
- The **left child** (odd position) has the **same value** as its parent.
- The **right child** (even position) has the **opposite value** of its parent.

So to find `(n, k)`: climb UP to the parent `(n-1, ceil(k/2))` and apply the rule:
- If k is **odd** → same as parent.
- If k is **even** → opposite of parent.

Base case: `(1, 1) = 0`.

**This gives O(n) time recursion and O(n) stack space.**

**Bonus insight — Bit manipulation O(1) space:**

Watch what happens as you repeatedly halve k:
- Each time k is even, you flip the result.
- Each time k is odd, you keep it.
- "k is even" ↔ "the last bit of (k-1) in binary is 1".
- The total number of flips = number of `1` bits in `(k-1)`.
- If that count is even → result is `0`; if odd → result is `1`.

**Formula:** `Integer.bitCount(k - 1) % 2`

---

### A — Action (Code the Solution, ~8 min)

#### Approach 1: Recursion (Cleanest for Interview)

```java
class Solution {
    public int kthGrammar(int n, int k) {
        // Base case: first row always starts with 0
        if (n == 1) return 0;

        // Parent is at position ceil(k/2) in the row above
        int parent = kthGrammar(n - 1, (k + 1) / 2);

        // Left child (odd k) = same as parent
        // Right child (even k) = complement of parent
        if (k % 2 == 1) {
            return parent;
        } else {
            return 1 - parent;
        }
    }
}
```

**Walk through verbally with n=3, k=3:**
```
kthGrammar(3, 3)
  → parent = kthGrammar(2, 2)       [k=3 is odd → same as parent]
      → parent = kthGrammar(1, 1)   [k=2 is even → complement of parent]
          → returns 0               [base case]
      → k=2 is even → return 1 - 0 = 1
  → k=3 is odd → return 1
```
Row 3 is `0110`, position 3 = `1`. ✓

---

#### Approach 2: Iterative (No Call Stack)

```java
class Solution {
    public int kthGrammar(int n, int k) {
        int result = 0; // row 1, position 1 is always 0

        // Walk from row n up to row 1, tracking flips
        for (int row = n; row > 1; row--) {
            if (k % 2 == 0) {
                // This position is a right child → flip
                result ^= 1;
            }
            // Move to parent position
            k = (k + 1) / 2;
        }

        return result;
    }
}
```

**Why `^= 1`:** XOR with 1 flips 0↔1 in one operation. Cleaner than `1 - result`.

---

#### Approach 3: Bit Manipulation O(1) Space, O(1) Time

```java
class Solution {
    public int kthGrammar(int n, int k) {
        // The value at position k equals the parity of set bits in (k-1)
        // n is irrelevant as long as k is within bounds (guaranteed by constraints)
        return Integer.bitCount(k - 1) % 2;
    }
}
```

**Why this works — prove it verbally:**

```
k=1: k-1=0 (binary: 0000) → 0 ones → result 0  ✓ (Row any, pos 1 = 0)
k=2: k-1=1 (binary: 0001) → 1 one  → result 1  ✓ (Row 2+, pos 2 = 1)
k=3: k-1=2 (binary: 0010) → 1 one  → result 1  ✓ (Row 3+, pos 3 = 1)
k=4: k-1=3 (binary: 0011) → 2 ones → result 0  ✓ (Row 3+, pos 4 = 0)
```

Each time k is a right child (even), we flip. "k is even" ↔ "last bit of (k-1) is set". Every level either flips or doesn't. Total flips = popcount of (k-1).

> In an interview, **present Approach 1 first** (clear, easy to reason about), then offer Approach 3 as "there's actually an O(1) insight here" to show depth.

---

### R — Result (Close Strongly, ~1 min)

> "Let me summarize. The recursive solution runs in **O(n) time** — one recursive call per row — and uses **O(n) stack space**. The iterative version achieves the same without the call stack overhead. The bit manipulation version is **O(1) time and O(1) space** — it leverages the fact that each right-child hop flips the bit, and right-child hops correspond exactly to set bits in (k-1)."

**Complexity Table:**

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force (build rows) | O(2^n) | O(2^n) | MLE for n=30 |
| Recursion | O(n) | O(n) stack | Clearest for interview |
| Iterative | O(n) | O(1) | No stack overflow risk |
| Bit manipulation | O(1) | O(1) | Impressive but explain it |

**Edge cases to call out:**
1. `n=1, k=1` → always `0` (base case).
2. `k=1` any row → always `0` (leftmost symbol never gets complemented).
3. `k=2^(n-1)` → max valid k; constraints guarantee this is within bounds.

---

## Pattern Recognition Trigger

> If you see: **"each element spawns two children with a deterministic rule"** → think **binary tree + recurse upward to root**.

This problem is isomorphic to:
- "Find the leaf value in a complete binary tree where each node's children follow rule X"
- The trick is always: **don't build down, reason up**.

Similar problems using this same upward-recursion pattern:
- Pascal's Triangle value queries
- Josephus problem variants
- Segment tree single-element queries

---

## What DE Shaw Is Testing Here

1. **Do you see the recursive structure?** Many candidates try to simulate row by row and hit TLE/MLE.
2. **Can you formalize the parent-child relationship?** The even/odd → same/flip rule is the core insight.
3. **Do you know multiple solutions?** Offering the bit manipulation solution after the recursive one signals strong problem-solving depth.
4. **Can you prove your solution verbally?** Trace through n=3, k=3 by hand — interviewers often ask "walk me through your code."
