# Day 2 — DE Shaw Specific Drill + 30-Min Strategy
**DE Shaw Round 1 Prep | Day 2 of 2**

## Goal
Solve 10 problems that have been directly reported from DE Shaw coding rounds. Focus on the **first 5 minutes** — classification speed is what separates candidates here.

---

## The 30-Minute Strategy (Internalize This)

```
STEP 1 [0-2 min]  Read + restate in your own words. Write 1 example by hand.
STEP 2 [2-4 min]  Apply the 5-question trigger → name the pattern.
STEP 3 [4-5 min]  State brute force + why it's slow. State optimal approach.
STEP 4 [5-13 min] Code optimal. Don't over-engineer. Clean variable names.
STEP 5 [13-15 min] Dry-run 1 example. Fix bugs. State complexity.

Repeat for problem 2.
```

**If you can't classify in 4 minutes:** write brute force, state "I'd optimize this to O(n) using [pattern] by [key insight]" — and code it. Partial credit + explaining the optimization often scores as well as clean code.

---

## DE Shaw High-Frequency Problems

### Problem 1 — Maximum Product Subarray (LC 152) | Medium
**Reported in:** DE Shaw 2023, 2024 online rounds

**Trigger:** "maximum product subarray" — negative numbers can flip sign, so track both max AND min at each step.

```
Input:  [2, 3, -2, 4]
Output: 6   ([2,3])

Input:  [-2, 0, -1]
Output: 0
```

```java
public int maxProduct(int[] nums) {
    int maxProd = nums[0], minProd = nums[0], result = nums[0];
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] < 0) {
            int tmp = maxProd;
            maxProd = minProd;
            minProd = tmp;
        }
        maxProd = Math.max(nums[i], maxProd * nums[i]);
        minProd = Math.min(nums[i], minProd * nums[i]);
        result = Math.max(result, maxProd);
    }
    return result;
}
```
**Complexity:** O(n) time, O(1) space

**Key insight:** swap max/min when current element is negative — a large negative × negative becomes the new max.

**STAR Walkthrough — Maximum Product Subarray (LC 152):**
- **S:** "Given an integer array with negatives and zeros; find the contiguous subarray with the largest product. Naive approach: O(n²) — enumerate all subarrays and track max product."
- **T:** "Achieve O(n) by tracking both running max AND min at each step — key insight: multiplying by a negative number flips the sign, so today's minimum could become tomorrow's maximum."
- **A:** "(1) Initialize `maxProd = minProd = result = nums[0]`. (2) For each subsequent element: if it is negative, swap maxProd and minProd before multiplying (since negatives invert the relationship). (3) `maxProd = max(nums[i], maxProd * nums[i])`, `minProd = min(nums[i], minProd * nums[i])`. Critical detail: the swap must happen before the multiply — not after — otherwise you compute with already-updated values."
- **R:** "O(n) time, O(1) space. At n=10^5: single pass vs 10^10 operations for brute force."
- **Alt:** Instead of tracking min and max simultaneously, you could reset on zero and handle negatives separately — but the swap trick handles all cases (negatives, zeros, mixed) in one unified pass, reducing implementation complexity.

---

### Problem 2 — Longest Common Prefix (LC 14) | Easy → Watch the edge cases
**Reported in:** DE Shaw 2022, 2023 (often as warm-up problem 1)

```
Input:  ["flower","flow","flight"]
Output: "fl"
```

```java
public String longestCommonPrefix(String[] strs) {
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
```
**Complexity:** O(S) where S = sum of all character lengths

**Edge cases:** single string → return it. All empty strings → return "". One empty string → return "".

**STAR Walkthrough — Longest Common Prefix (LC 14):**
- **S:** "Given an array of strings; find the longest common prefix shared by all strings. Naive approach: O(S×n) — compare each character column-by-column across all strings."
- **T:** "Achieve O(S) using iterative prefix shrinking — key insight: use the first string as a candidate prefix and progressively trim it until every string starts with it."
- **A:** "(1) Initialize `prefix = strs[0]`. (2) For each subsequent string, shrink `prefix` by removing one character from the end while `strs[i]` does not start with `prefix`. (3) If prefix becomes empty, return immediately. Critical detail: the `startsWith` check in a while-loop guarantees correctness but can take O(L) per check — overall still O(S) total since each character is discarded at most once."
- **R:** "O(S) time where S = total characters, O(1) extra space. The shrinking terminates at the true common prefix without ever overshooting."
- **Alt:** Instead of iterative shrinking, you could sort the array and compare only the first and last strings lexicographically — O(S log n) due to sorting. The iterative approach is O(S) without sorting and handles the empty-string edge case naturally.

---

### Problem 3 — Spiral Matrix (LC 54) | Medium
**Reported in:** DE Shaw 2023, 2024 on-site and online

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [1,2,3,6,9,8,7,4,5]
```

```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) result.add(matrix[top][c]);
        top++;
        for (int r = top; r <= bottom; r++) result.add(matrix[r][right]);
        right--;
        if (top <= bottom) {
            for (int c = right; c >= left; c--) result.add(matrix[bottom][c]);
            bottom--;
        }
        if (left <= right) {
            for (int r = bottom; r >= top; r--) result.add(matrix[r][left]);
            left++;
        }
    }
    return result;
}
```
**Complexity:** O(m×n) time, O(1) space (excluding output)

**Pattern:** shrink boundaries after each direction. The guards `if (top <= bottom)` and `if (left <= right)` prevent double-counting middle rows/cols in non-square matrices.

**STAR Walkthrough — Spiral Matrix (LC 54):**
- **S:** "Given an m×n matrix; return all elements in spiral order (clockwise from top-left). Naive approach: O(m×n) with a visited array and direction-change logic — correct but requires O(m×n) extra space."
- **T:** "Achieve O(m×n) time, O(1) space using four shrinking boundary pointers — key insight: after traversing each edge of the current layer, shrink that boundary inward and continue."
- **A:** "(1) Initialize `top, bottom, left, right` boundaries. (2) Traverse right across top row, then increment top. (3) Traverse down right column, then decrement right. (4) Guard `if (top <= bottom)`: traverse left across bottom row, decrement bottom. (5) Guard `if (left <= right)`: traverse up left column, increment left. Critical detail: the two guards prevent double-reading the center row or column of non-square matrices when the spiral reduces to a single row or column."
- **R:** "O(m×n) time, O(1) extra space. Every cell visited exactly once."
- **Alt:** Instead of boundary pointers, you could use direction arrays [right, down, left, up] with a visited boolean matrix — cleaner to write but requires O(m×n) space. Boundary pointers solve it in O(1) space.

---

### Problem 4 — Rotate Image (LC 48) | Medium
**Reported in:** DE Shaw 2023, multiple times

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [[7,4,1],[8,5,2],[9,6,3]]
```

**Two-step trick:** transpose + reverse each row (no extra space)

```java
public void rotate(int[][] matrix) {
    int n = matrix.length;
    // Step 1: Transpose (swap across diagonal)
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int tmp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = tmp;
        }
    // Step 2: Reverse each row
    for (int[] row : matrix) {
        int l = 0, r = row.length - 1;
        while (l < r) { int tmp = row[l]; row[l++] = row[r]; row[r--] = tmp; }
    }
}
```
**Complexity:** O(n²) time, O(1) space

**STAR Walkthrough — Rotate Image (LC 48):**
- **S:** "Given an n×n matrix; rotate it 90° clockwise in-place. Naive approach: O(n²) time with O(n²) extra space — copy to new matrix with `new[j][n-1-i] = old[i][j]`."
- **T:** "Achieve O(n²) time, O(1) space using the transpose-then-reverse trick — key insight: a 90° clockwise rotation = transpose across the main diagonal, then reverse each row."
- **A:** "(1) Transpose: for all i < j, swap `matrix[i][j]` and `matrix[j][i]`. (2) Reverse each row: two-pointer swap from each row's ends inward. Critical detail: the transpose only swaps above-diagonal pairs (i < j, not i ≤ j) — swapping i == j (diagonal) is a no-op but swapping i > j would undo the first pass."
- **R:** "O(n²) time (visit every cell twice), O(1) space. At n=1000: 2×10^6 operations with no auxiliary allocation vs 10^6 × 4 bytes = 4MB for the copy approach."
- **Alt:** Instead of transpose + reverse, you could perform a 4-way cyclic swap layer by layer — correct but harder to reason about in interview conditions. Transpose + reverse is the standard, easiest-to-verify approach.

---

### Problem 5 — Jump Game (LC 55) | Medium | Greedy
**Reported in:** DE Shaw 2022, 2023, 2024

```
Input:  [2,3,1,1,4]  → true
Input:  [3,2,1,0,4]  → false
```

```java
public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false;
        maxReach = Math.max(maxReach, i + nums[i]);
    }
    return true;
}
```
**Complexity:** O(n) time, O(1) space

**Mental model:** `maxReach` is how far you can possibly get. If the current index exceeds it, you're stranded.

**STAR Walkthrough — Jump Game (LC 55):**
- **S:** "Given an array where each element is the max jump length from that index; determine if you can reach the last index. Naive approach: O(2^n) — recursive exploration of all jump combinations."
- **T:** "Achieve O(n) using a greedy maxReach tracker — key insight: if you track the furthest index reachable at any point, you can detect stranding in a single pass without backtracking."
- **A:** "(1) Initialize `maxReach = 0`. (2) For each index i: if `i > maxReach`, return false — you cannot reach this index. (3) Update `maxReach = max(maxReach, i + nums[i])`. Critical detail: the early return `if (i > maxReach) return false` must come before the update — otherwise you update maxReach from an unreachable index, which is logically invalid."
- **R:** "O(n) time, O(1) space. At n=10^5: single scan vs exponential search for brute force."
- **Alt:** Instead of greedy, you could use DP where `dp[i] = true` if index i is reachable — O(n²) because each cell might be updated by all preceding cells. Greedy achieves O(n) because `maxReach` summarizes all reachability information in one variable.

---

### Problem 6 — Find All Anagrams in a String (LC 438) | Medium | Sliding Window
**Reported in:** DE Shaw 2023, 2024

```
Input:  s = "cbaebabacd", p = "abc"
Output: [0, 6]
```

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    if (s.length() < p.length()) return result;
    int[] pFreq = new int[26], wFreq = new int[26];
    for (char c : p.toCharArray()) pFreq[c - 'a']++;
    int k = p.length();
    for (int i = 0; i < s.length(); i++) {
        wFreq[s.charAt(i) - 'a']++;
        if (i >= k) wFreq[s.charAt(i - k) - 'a']--;
        if (Arrays.equals(pFreq, wFreq)) result.add(i - k + 1);
    }
    return result;
}
```
**Complexity:** O(n) time (comparing two 26-element arrays is O(1))

**STAR Walkthrough — Find All Anagrams in a String (LC 438):**
- **S:** "Given strings s and p; find all start indices in s where a substring of length |p| is an anagram of p. Naive approach: O(n×|p|) — slide a window and sort or count each window."
- **T:** "Achieve O(n) using fixed sliding window with frequency arrays — key insight: two strings are anagrams iff their character frequency arrays are equal; maintain the window's freq array incrementally rather than recomputing."
- **A:** "(1) Build `pFreq[26]` from p; initialize `wFreq[26]` from the first |p| characters of s. (2) Slide one character at a time: add `s[r]` to wFreq, remove `s[r - |p|]` from wFreq. (3) Compare `Arrays.equals(pFreq, wFreq)` — this is O(26) = O(1). Critical detail: comparing 26-element int arrays is O(1) since the alphabet size is fixed — this is what makes the overall algorithm O(n)."
- **R:** "O(n) time, O(1) space (two fixed 26-element arrays). At n=10^5, |p|=100: O(n) vs O(n×|p|) = O(10^7) for the sort-per-window approach."
- **Alt:** Instead of int arrays, you could use a HashMap and a `formed` counter (like Minimum Window Substring) — this generalizes to Unicode but is more complex to implement. The 26-element array is the fastest constant for lowercase ASCII problems like this one.

---

### Problem 7 — Validate Binary Search Tree (LC 98) | Medium | DFS
**Reported in:** DE Shaw 2022, 2023

```
Input:  [2,1,3]  → true
Input:  [5,1,4,null,null,3,6]  → false
```

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
```
**Complexity:** O(n) time, O(h) space

**Common mistake:** checking only `node.left.val < node.val` — this misses cases where a right-subtree node is smaller than an ancestor.

**STAR Walkthrough — Validate Binary Search Tree (LC 98):**
- **S:** "Given a binary tree; determine whether it is a valid BST (left subtree < node < right subtree, recursively). Naive approach: O(n²) — for each node, scan its entire left/right subtree to verify bounds."
- **T:** "Achieve O(n) using DFS with propagated bounds — key insight: pass a valid range [min, max] to each node; a node is invalid if its value falls outside the range inherited from its ancestors."
- **A:** "(1) Recurse with `validate(root, Long.MIN_VALUE, Long.MAX_VALUE)`. (2) At each node: if `node.val <= min || node.val >= max`, return false. (3) Recurse left with `(node, min, node.val)` and right with `(node, node.val, max)`. Critical detail: use `Long.MIN_VALUE` / `Long.MAX_VALUE` (not `Integer`) to handle trees where node values are `Integer.MIN_VALUE` or `Integer.MAX_VALUE` — the boundary comparison would fail with int bounds."
- **R:** "O(n) time, O(h) space for the recursion stack where h = tree height. Each node visited exactly once."
- **Alt:** Instead of bounds propagation, you could do an in-order traversal and check that the sequence is strictly increasing — O(n) time and O(n) space to store values. Bounds propagation uses O(h) space and finds violations earlier without building any list.

---

### Problem 8 — Number of Islands (LC 200) | Medium | BFS/DFS on Grid
**Reported in:** DE Shaw 2022, 2023, 2024 — very common

```
Input:  [["1","1","0","0","0"],
          ["1","1","0","0","0"],
          ["0","0","1","0","0"],
          ["0","0","0","1","1"]]
Output: 3
```

```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == '1') { dfs(grid, r, c); count++; }
    return count;
}
private void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1') return;
    grid[r][c] = '0'; // mark visited by sinking the island
    dfs(grid, r+1, c); dfs(grid, r-1, c);
    dfs(grid, r, c+1); dfs(grid, r, c-1);
}
```
**Complexity:** O(m×n) time, O(m×n) space (recursion stack worst case)

**STAR Walkthrough — Number of Islands (LC 200):**
- **S:** "Given a 2D grid of '1's (land) and '0's (water); count the number of islands (connected components of '1's). Naive approach: O((m×n)²) — for each '1', flood-fill and mark, but with a separate visited array rebuilt each time."
- **T:** "Achieve O(m×n) using DFS with in-place marking — key insight: when you find a '1', immediately sink the entire island (set all connected '1's to '0') before incrementing the counter, ensuring each cell is processed at most once."
- **A:** "(1) Scan every cell. (2) On finding `grid[r][c] == '1'`, call `dfs(grid, r, c)` and increment count. (3) In dfs: boundary check + `grid[r][c] != '1'` → return; set `grid[r][c] = '0'`; recurse in 4 directions. Critical detail: setting the cell to '0' before recursing (not after) prevents infinite loops in cyclic paths and eliminates the need for a separate visited array."
- **R:** "O(m×n) time, O(m×n) space (recursion stack in worst case — a fully-connected grid). At a 1000×1000 grid: 10^6 cells each visited once."
- **Alt:** Instead of DFS, you could use BFS with a queue — same O(m×n) complexity but avoids potential stack overflow on very large fully-connected grids. For most interview settings, DFS with in-place marking is the cleaner, faster-to-write solution.

---

### Problem 9 — Largest Rectangle in Histogram (LC 84) | Hard | Monotonic Stack
**Reported in:** DE Shaw 2023, 2024 — the hardest problem they typically ask

```
Input:  [2,1,5,6,2,3]
Output: 10
```

```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;
    for (int i = 0; i <= heights.length; i++) {
        int h = (i == heights.length) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```
**Complexity:** O(n) time, O(n) space

**Key insight:** maintain an increasing stack of indices. When a bar is shorter than the top, pop and calculate area using the popped bar as the shortest. Width extends left to the new stack top.

**STAR Walkthrough — Largest Rectangle in Histogram (LC 84):**
- **S:** "Given an array of bar heights; find the area of the largest rectangle that fits within the histogram. Naive approach: O(n²) — for each bar, expand left and right to find its max width as the limiting height."
- **T:** "Achieve O(n) using a monotonic increasing stack — key insight: a bar can extend leftward only as far as the nearest shorter bar to its left; maintain an increasing stack so that when a shorter bar arrives, you immediately know the right boundary of all popped bars."
- **A:** "(1) Append a sentinel height 0 at the end to flush all remaining bars. (2) For each index i (including sentinel): while stack non-empty and `heights[stack.top()] > h`, pop `idx`; compute `width = stack.isEmpty() ? i : i - stack.top() - 1`; update maxArea. (3) Push current index. Critical detail: `width = i - stack.top() - 1` (not `i - idx`) — after popping, the new stack top is the nearest bar shorter than the popped bar, so the rectangle extends from just after the new top to just before i."
- **R:** "O(n) time, O(n) space. Each bar pushed and popped at most once. At n=10^5: 2×10^5 stack ops vs 10^10 for brute force."
- **Alt:** Instead of monotonic stack, you could use a divide-and-conquer approach — O(n log n) average but O(n²) worst case on sorted input. The stack solution is O(n) guaranteed and is the canonical interview answer.

---

### Problem 10 — 3Sum (LC 15) | Medium | Two Pointers
**Reported in:** DE Shaw 2022, 2023, 2024 — extremely common

```
Input:  [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]
```

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue; // skip duplicate fixed
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l+1]) l++;
                while (l < r && nums[r] == nums[r-1]) r--;
                l++; r--;
            } else if (sum < 0) l++;
            else r--;
        }
    }
    return result;
}
```
**Complexity:** O(n²) time, O(1) space (excluding output)

**STAR Walkthrough — 3Sum (LC 15):**
- **S:** "Given an integer array; find all unique triplets that sum to zero. Naive approach: O(n³) — check all triplets; O(n²) with a HashSet for the third element."
- **T:** "Achieve O(n²) time, O(1) space using sort + two pointers — key insight: fix one element as the anchor and use two pointers on the rest of the sorted array to find pairs summing to the negative of the anchor in O(n) per anchor."
- **A:** "(1) Sort the array. (2) For each index i (skip if `i > 0 && nums[i] == nums[i-1]` to avoid duplicate triplets), set `l = i+1, r = n-1`. (3) While `l < r`: if sum == 0, record and advance both pointers past duplicates; if sum < 0, advance l; if sum > 0, retreat r. Critical detail: after recording a valid triplet, skip duplicate values for both l and r before incrementing/decrementing — otherwise the same triplet appears multiple times in the output."
- **R:** "O(n²) time, O(1) extra space. At n=3000: ~4.5×10^6 operations vs 2.7×10^10 for O(n³) brute force."
- **Alt:** Instead of sort + two pointers, you could use a HashSet to find the third element in O(n²) — but deduplication becomes significantly harder to implement correctly. Sort + two pointers naturally handles deduplication by skipping adjacent duplicates.

---

## 30-Minute Mock — Do This Today

Set a 30-minute timer. Pick any 2 problems from the list below (mix easy+hard):
- Easy warm-up: Problem 2 (Longest Common Prefix) or Problem 5 (Jump Game)
- Medium: Problem 1 (Max Product), Problem 3 (Spiral Matrix), Problem 6 (Anagrams), Problem 10 (3Sum)
- Hard: Problem 9 (Largest Rectangle)

**Evaluate yourself on:**
- [ ] Classified pattern in <4 minutes
- [ ] Stated brute force before coding
- [ ] Finished both in 30 minutes with correct output
- [ ] Stated time + space complexity for both
- [ ] Code is readable — good variable names, no magic numbers

---

## Day 2 Flashcards

| Q | A |
|---|---|
| Max Product Subarray: why track min AND max? | Negative × negative = positive; today's min could become tomorrow's max |
| How do you rotate a matrix 90° clockwise in-place? | Transpose (swap matrix[i][j] and matrix[j][i]) then reverse each row |
| Spiral matrix: what guards prevent double-counting? | `if (top <= bottom)` before left traverse, `if (left <= right)` before up traverse |
| Largest Rectangle in Histogram: what triggers a pop? | When current bar is shorter than stack top — the popped bar can't extend further right |
| Number of Islands: how to mark visited? | Sink the cell: set `grid[r][c] = '0'` in-place before recursing |
| BST validation: why pass min/max bounds instead of checking children? | Catches violations across ancestor boundaries, not just parent-child |
