# Pattern Recognition Cheatsheet — DE Shaw Round 1

> Read this on Day 1 morning and Day 2 morning. Write the trigger table from memory before each mock.

---

## The 5-Question Trigger Framework

Ask these in order when you see a new problem. First match = your pattern.

```
Q1. Is the input sorted (array, string, linked list)?
    → YES + find pair/triplet/range  →  TWO POINTERS
    → YES + find min/max/kth element →  BINARY SEARCH

Q2. Does the problem involve a contiguous subarray/substring with a constraint?
    → Fixed size k               →  SLIDING WINDOW (fixed)
    → "longest/shortest" + "at most k distinct / sum ≤ target"  →  SLIDING WINDOW (variable)

Q3. Does it involve a sequence with "next greater/smaller" or stack of pending decisions?
    → YES  →  MONOTONIC STACK

Q4. Is the input a tree or graph?
    → Level order / shortest path     →  BFS
    → Path sum / DFS traversal / LCA  →  DFS (recursive or iterative)
    → Cycle detection / ordering      →  TOPOLOGICAL SORT or UNION-FIND

Q5. Is the problem asking for count/existence of subarrays with a sum target?
    → YES  →  PREFIX SUM + HASHMAP
    → Group/count by some property    →  HASHMAP / FREQUENCY MAP
```

---

## Pattern → Time Complexity Quick Table

| Pattern | Time | Space | DE Shaw Frequency |
|---------|------|-------|-------------------|
| Two Pointers | O(n) | O(1) | ★★★★★ |
| Sliding Window | O(n) | O(k) | ★★★★★ |
| Binary Search | O(log n) | O(1) | ★★★★☆ |
| Prefix Sum + HashMap | O(n) | O(n) | ★★★★☆ |
| Monotonic Stack | O(n) | O(n) | ★★★★☆ |
| BFS (graph/grid) | O(V+E) | O(V) | ★★★★☆ |
| DFS (tree/graph) | O(V+E) | O(h) | ★★★★☆ |
| HashMap / Frequency | O(n) | O(n) | ★★★★★ |
| Sorting + Greedy | O(n log n) | O(1) | ★★★☆☆ |
| DP (1D/2D) | O(n²) | O(n) | ★★★☆☆ |

---

## Code Skeletons (Write From Memory)

### Two Pointers
```java
int l = 0, r = arr.length - 1;
while (l < r) {
    if (condition) l++;
    else if (otherCondition) r--;
    else { /* found */ l++; r--; }
}
```

### Sliding Window (Variable)
```java
int l = 0, maxLen = 0;
Map<Character, Integer> freq = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    freq.merge(s.charAt(r), 1, Integer::sum);
    while (windowInvalid(freq)) {          // shrink
        freq.merge(s.charAt(l), -1, Integer::sum);
        if (freq.get(s.charAt(l)) == 0) freq.remove(s.charAt(l));
        l++;
    }
    maxLen = Math.max(maxLen, r - l + 1);  // window = [l..r]
}
```

### Binary Search (on answer)
```java
int lo = min, hi = max;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (canAchieve(mid)) hi = mid;   // find minimum valid
    else lo = mid + 1;
}
return lo;
```

### Prefix Sum + HashMap (subarray sum = k)
```java
Map<Integer, Integer> seen = new HashMap<>();
seen.put(0, 1);
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += seen.getOrDefault(sum - k, 0);
    seen.merge(sum, 1, Integer::sum);
}
```

### Monotonic Stack (Next Greater Element)
```java
int[] result = new int[n];
Arrays.fill(result, -1);
Deque<Integer> stack = new ArrayDeque<>(); // stores indices
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
        result[stack.pop()] = arr[i];
    }
    stack.push(i);
}
```

### BFS (shortest path on grid)
```java
Queue<int[]> q = new LinkedList<>();
q.offer(new int[]{startR, startC, 0});
boolean[][] visited = new boolean[rows][cols];
visited[startR][startC] = true;
int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
while (!q.isEmpty()) {
    int[] cur = q.poll();
    if (cur[0] == endR && cur[1] == endC) return cur[2];
    for (int[] d : dirs) {
        int nr = cur[0]+d[0], nc = cur[1]+d[1];
        if (nr>=0 && nr<rows && nc>=0 && nc<cols && !visited[nr][nc] && grid[nr][nc]!=1) {
            visited[nr][nc] = true;
            q.offer(new int[]{nr, nc, cur[2]+1});
        }
    }
}
```

### DFS on Tree (post-order result collection)
```java
private int dfs(TreeNode node) {
    if (node == null) return BASE_CASE;
    int left  = dfs(node.left);
    int right = dfs(node.right);
    // update global answer here if needed
    return /* value to return to parent */;
}
```

---

## DE Shaw Problem "Types" Seen Most Often

1. **Subarray with max sum / product** → Kadane's / Sliding Window
2. **Count subarrays with sum = k** → Prefix Sum + HashMap
3. **Merge overlapping intervals** → Sort + greedy scan
4. **Next greater element / stock problems** → Monotonic Stack
5. **Valid parentheses / path simplification** → Stack
6. **Word search / number of islands** → BFS or DFS on grid
7. **Find kth largest / top k** → Heap or QuickSelect
8. **Anagram / substring with all chars** → Sliding Window + freq map
9. **Rotate/reverse array or string** → Two pointers in-place
10. **LCA / path in binary tree** → DFS with recursion

---

## 60-Second Classification Drill

Read the problem title only. Guess the pattern before reading the body.

| Title | Pattern |
|-------|---------|
| Longest Substring Without Repeating Characters | Sliding Window |
| Maximum Subarray | Kadane's (DP / Greedy) |
| Two Sum | HashMap |
| Trapping Rain Water | Two Pointers OR Monotonic Stack |
| Number of Islands | BFS / DFS |
| Merge Intervals | Sort + Greedy |
| Subarray Sum Equals K | Prefix Sum + HashMap |
| Kth Largest Element | Heap / QuickSelect |
| Valid Parentheses | Stack |
| Word Search | DFS Backtracking |
