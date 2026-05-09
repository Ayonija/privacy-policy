# DSA Mastery Atlas — Quick Reference

A companion cheat-sheet for the **DSA Mastery Atlas** (`dsa-atlas-ultimate.html`), targeting FAANG L5/L6 interview readiness.

---

## Pattern Decision Tree (TL;DR)

| Signal in problem | Pattern to reach for |
|---|---|
| Sorted array / search for value | Binary Search |
| "Subarray / window" | Sliding Window |
| Two indexes walking toward each other | Two Pointers |
| Running total / range query | Prefix Sum |
| "Next greater / smaller" | Monotonic Stack |
| Linked list cycle / midpoint | Fast + Slow Pointers |
| Tree traversal (all nodes) | DFS or BFS |
| Shortest path (unweighted) | BFS |
| Shortest path (weighted, no neg) | Dijkstra |
| Connected components / union | Union-Find |
| Ordering with dependencies | Topological Sort |
| Top-K / Kth element | Heap (Min or Max) |
| Prefix word matching | Trie |
| All subsets / permutations | Backtracking |
| Overlapping subproblems | Dynamic Programming |
| "Optimal local → optimal global" | Greedy |
| XOR / counting bits / flip | Bit Manipulation |
| Split and recombine | Divide & Conquer |

---

## Complexity Cheat Sheet

### Data Structures

| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1) | O(1) |
| Stack / Queue | O(n) | O(n) | O(1) | O(1) |
| Hash Map | O(1) avg | O(1) avg | O(1) avg | O(1) avg |
| Binary Search Tree | O(log n) | O(log n) | O(log n) | O(log n) |
| AVL / Red-Black Tree | O(log n) | O(log n) | O(log n) | O(log n) |
| Min/Max Heap | O(1) peek | O(n) | O(log n) | O(log n) |
| Trie | — | O(m) | O(m) | O(m) |

### Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable |
|---|---|---|---|---|---|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | Yes |

---

## Must-Know Patterns (Code Skeletons)

### Sliding Window (Variable)
```python
l = 0
for r in range(len(s)):
    # expand window
    while invalid(window):
        # shrink from left
        l += 1
    ans = max(ans, r - l + 1)
```

### Binary Search (Answer Space)
```python
lo, hi = min_possible, max_possible
while lo < hi:
    mid = (lo + hi) // 2
    if feasible(mid):
        hi = mid
    else:
        lo = mid + 1
return lo
```

### BFS (Graph / Grid)
```python
from collections import deque
q = deque([start])
visited = {start}
while q:
    node = q.popleft()
    for nb in neighbors(node):
        if nb not in visited:
            visited.add(nb)
            q.append(nb)
```

### DFS Backtracking
```python
def backtrack(start, path):
    if len(path) == k:
        result.append(path[:])
        return
    for i in range(start, len(nums)):
        path.append(nums[i])
        backtrack(i + 1, path)
        path.pop()
```

### 0/1 Knapsack (DP)
```python
dp = [0] * (W + 1)
for item_val, item_wt in items:
    for w in range(W, item_wt - 1, -1):
        dp[w] = max(dp[w], dp[w - item_wt] + item_val)
```

### Union-Find (Path Compression + Rank)
```python
parent = list(range(n))
rank = [0] * n

def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])
    return parent[x]

def union(x, y):
    px, py = find(x), find(y)
    if px == py: return False
    if rank[px] < rank[py]: px, py = py, px
    parent[py] = px
    if rank[px] == rank[py]: rank[px] += 1
    return True
```

### Dijkstra
```python
import heapq
dist = {node: float('inf') for node in graph}
dist[src] = 0
heap = [(0, src)]
while heap:
    d, u = heapq.heappop(heap)
    if d > dist[u]: continue
    for v, w in graph[u]:
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w
            heapq.heappush(heap, (dist[v], v))
```

---

## L5 / L6 Interview Checklist

- [ ] Can recognize the pattern within 2 minutes
- [ ] Can write brute force, then optimize
- [ ] Knows time + space complexity before coding
- [ ] Handles all edge cases (empty, single element, negatives, overflow)
- [ ] Explains trade-offs when asked
- [ ] Iterative DFS (no recursion stack overflow risk)
- [ ] At least 2 approaches for DP problems (top-down + bottom-up)
- [ ] System design: can sketch LRU Cache, LFU Cache from scratch

---

## Suggested Study Order

1. Pattern Map (t0) — understand how to classify problems first
2. Arrays & Strings (t1) — most common interview category
3. Binary Search (t5) — deceptively tricky; master boundaries
4. Trees (t3) — DFS/BFS variants appear everywhere
5. Graphs (t4) — Union-Find + Dijkstra are L5 must-haves
6. Dynamic Programming (t9) — hardest; do last when foundations are solid
7. Backtracking (t8) — template-driven, relatively learnable
8. Heap + Trie (t7) — niche but high-signal at L6
9. Bit Manipulation (t12) — quick wins, 1–2 problems per interview
10. Problem List (t16) — validate readiness across all patterns

---

*Companion to `dsa-atlas-ultimate.html`. Open that file in a browser for interactive flowcharts, dry runs, code templates, and progress tracking.*
