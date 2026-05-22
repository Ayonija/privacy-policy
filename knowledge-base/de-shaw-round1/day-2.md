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
