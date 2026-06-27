# 01 — DSA & Coding Patterns (Java)

> At P5 coding is a **floor, not the bar** — clear it cleanly and move on. Strategy for a 2-day cram: learn the **patterns** (so any problem maps to one) + drill the **exact problems they've asked**. Always narrate: brute force → insight → optimized → complexity → edge cases.

> **The narration habit that scores:** state the approach and Big-O *before* you type, talk while you code, then test on an example and call out edge cases. Interviewers grade your process as much as your answer.

---

## Big-O quick reference
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!). Hashing → O(1) lookup. Sorting → O(n log n). Binary search → O(log n). Nested loops → O(n²). Recursion → branches^depth.

---

## ⭐ THE EXPLICITLY-ASKED PROBLEMS (drill these first)

### Primes 1→200, minimal time complexity (asked, Round 2 — they wanted it optimized)
**Insight:** trial division is O(n√n); the **Sieve of Eratosthenes** is **O(n log log n)** — that's the "minimal complexity" they wanted.
```java
// Sieve: mark multiples of each prime as composite. O(n log log n) time, O(n) space.
List<Integer> primesUpTo(int n){
  boolean[] composite = new boolean[n+1];
  List<Integer> primes = new ArrayList<>();
  for (int p = 2; p <= n; p++){
    if (!composite[p]){                 // p is prime
      primes.add(p);
      for (long m = (long)p*p; m <= n; m += p)  // start at p*p; smaller multiples already marked
        composite[(int)m] = true;
    }
  }
  return primes;
}
```
🎤 *"Trial division is order n times root n. The optimal approach is the Sieve of Eratosthenes at n log log n: I assume all numbers prime, then for each prime cross out its multiples starting at p-squared, because every smaller multiple was already crossed out by a smaller prime. That start-at-p-squared detail is the optimization they're listening for."*

### Root-to-leaf path sum equals target (asked, Round 3 — "verify sum of nodes root→leaf equals input")
**Insight:** DFS, subtract node value as you descend; at a **leaf** check the remainder == 0.
```java
boolean hasPathSum(TreeNode root, int target){
  if (root == null) return false;                       // empty / fell off
  if (root.left == null && root.right == null)          // leaf
      return target == root.val;
  int rem = target - root.val;
  return hasPathSum(root.left, rem) || hasPathSum(root.right, rem);
}   // Time O(n) visit each node once; Space O(h) recursion stack, h = tree height
```
🎤 *"I walk down with DFS, subtracting each node's value from the target as I go. The key is the check happens only at a leaf — a node with no children — where I test whether the remaining target equals the leaf's value. Order n time since I touch each node once, order h space for the recursion stack."* (Variant: collect the actual paths → backtracking, add node to a list, recurse, remove on the way up.)

### Two Sum (#1, asked)
**Brute:** O(n²) pairs. **Optimal:** one-pass hashmap, store `value→index`, look for `target-x`. O(n)/O(n).
```java
int[] twoSum(int[] a, int t){
  Map<Integer,Integer> seen = new HashMap<>();          // value -> index
  for (int i = 0; i < a.length; i++){
    int need = t - a[i];
    if (seen.containsKey(need)) return new int[]{seen.get(need), i};
    seen.put(a[i], i);
  }
  return new int[]{-1,-1};
}
```
🎤 *"Brute force checks every pair at n-squared. The insight is to trade space for time: as I scan, I ask 'have I already seen the complement target-minus-current?' using a hashmap for O(1) lookup, which makes it one pass, order n."*

### Valid Palindrome (#125, asked)
Two pointers from both ends, skip non-alphanumerics, compare lowercased. O(n)/O(1).
```java
boolean isPalindrome(String s){
  int i = 0, j = s.length()-1;
  while (i < j){
    while (i < j && !Character.isLetterOrDigit(s.charAt(i))) i++;
    while (i < j && !Character.isLetterOrDigit(s.charAt(j))) j--;
    if (Character.toLowerCase(s.charAt(i++)) != Character.toLowerCase(s.charAt(j--)))
      return false;
  }
  return true;
}
```

### LRU Cache (#146, asked — design)
**Insight:** O(1) get+put needs **HashMap (lookup) + doubly-linked list (recency order)**. Most-recent at head, evict from tail. (Java shortcut: `LinkedHashMap` with access-order + `removeEldestEntry`.)
```java
class LRUCache extends LinkedHashMap<Integer,Integer> {
  private final int cap;
  LRUCache(int cap){ super(cap, 0.75f, true); this.cap = cap; } // true = access-order
  public int get(int k){ return super.getOrDefault(k, -1); }
  public void put(int k,int v){ super.put(k,v); }
  protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e){ return size() > cap; }
}
```
🎤 *"The trick is combining two structures: a hashmap for O(1) key lookup and a doubly-linked list to track recency. On access I move the node to the head; when I exceed capacity I evict the tail, the least-recently-used. Both operations stay O(1) because list splicing is constant time. In Java I can express it concisely with an access-ordered LinkedHashMap, but I can explain the hashmap-plus-DLL underneath."*

---

## THE PATTERNS (recognize → apply)

**1. Hashing / frequency map** — *trigger:* "seen before?", counts, pairs, dedupe, anagrams. O(n) time, O(n) space. (Two Sum, group anagrams, Valid Anagram, Continuous Subarray Sum with prefix-sum%k).

**2. Two pointers** — *trigger:* sorted array, pair/triplet, palindrome, in-place. Opposite-ends or fast/slow. O(n). (Valid Palindrome, Sort Colors (Dutch flag), Move Zeroes, Trapping Rain Water).

**3. Sliding window** — *trigger:* "contiguous subarray/substring" with a constraint (longest/shortest/at-most-k). Expand right, shrink left. O(n). (Longest substring without repeat, min window).

**4. Stack** — *trigger:* matching/nesting, "next greater," undo, expression parsing. (Valid Parentheses, Daily Temperatures (monotonic stack), Decode String, Implement Queue using Stacks).

**5. Binary search** — *trigger:* sorted, or "minimize/maximize a value that's monotonic" (search on answer). O(log n). Template carefully on `lo/hi/mid` to avoid off-by-one. (Kth Smallest in sorted matrix, search rotated).

**6. Linked list** — *trigger:* reverse, cycle, middle, merge. Use dummy head + fast/slow pointers. (Reverse Linked List, cycle detection (Floyd's)).

**7. Trees / BST** — *trigger:* hierarchical, path, ancestor. DFS (recursion) or BFS (queue). **BST search** = go left if smaller, right if larger → O(h). (Path Sum, level-order, LCA, validate BST).
```java
// BST search (asked: "how does searching in a binary tree work")
TreeNode search(TreeNode root, int key){
  while (root != null && root.val != key)
    root = key < root.val ? root.left : root.right;  // halve the search each step
  return root;                                        // O(h): O(log n) balanced, O(n) skewed
}
```

**8. Graphs (BFS/DFS)** — *trigger:* grid, connectivity, shortest path (unweighted→BFS), topological order. BFS = queue (shortest hops); DFS = stack/recursion (paths, cycles). O(V+E). (Number of islands, course schedule).

**9. Heap / priority queue** — *trigger:* "top K," "k-th largest," merge k, streaming median. O(n log k). (Kth Largest — min-heap of size k).

**10. Backtracking** — *trigger:* "all combinations/permutations/subsets," constraint satisfaction. Choose → recurse → un-choose. Exponential but pruned. (Permutations, Combination Sum, Letter Combinations, N-Queens).

**11. Intervals** — *trigger:* meetings, overlaps, merge. Sort by start, then sweep. O(n log n). (Merge intervals, Two Best Non-Overlapping Events).

**12. Dynamic programming** — *trigger:* optimal substructure + overlapping subproblems; "min/max ways/cost," "can you reach." Define state, recurrence, base case; memoize (top-down) or tabulate (bottom-up). (Climbing Stairs, Unique Paths, coin change, LCS, Valid Palindrome III).
```java
int climbStairs(int n){            // ways = fib; each step from i-1 or i-2
  int a=1,b=1;                     // O(n) time, O(1) space
  for(int i=2;i<=n;i++){ int c=a+b; a=b; b=c; }
  return b;
}
```

**13. Greedy** — *trigger:* "maximum/minimum" where a locally-optimal choice is provably globally optimal. Sort + take. (Maximize Greatness, Maximum Swap, interval scheduling). *Risk:* prove greed works or it silently fails.

---

## ⚠️ Edge cases & traps (run through these every problem)
- Empty / null / single element; all-same; already-sorted / reverse-sorted.
- Integer overflow (use `long` for sums/products — e.g. sieve `p*p`).
- Off-by-one in binary search (`lo<=hi` vs `lo<hi`, `mid=lo+(hi-lo)/2` to avoid overflow).
- Duplicates (sets vs counts); negative numbers.
- Recursion depth → stack overflow on huge inputs (consider iterative).
- **State Big-O before coding; test on an example after.**

**Seniority signal in a coding round:** clean naming, narrate the trade-off, handle edges unprompted, and if you finish fast, mention "in production I'd also consider…" (validation, concurrency, the data-structure choice). Don't gold-plate — clear the floor, show judgement, move on.

---

*Next topic, or drill deeper on this one?*
