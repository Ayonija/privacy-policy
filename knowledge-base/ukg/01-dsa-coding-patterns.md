# 01 — DSA & Coding Patterns (Java)

> At P5 coding is a **floor, not the bar** — clear it cleanly and move on. Strategy for a 2-day cram: learn the **patterns** (so any problem maps to one) + drill the **exact problems they've asked**. Always narrate: brute force → insight → optimized → complexity → edge cases.

> **The narration habit that scores:** state the approach and Big-O *before* you type, talk while you code, then test on an example and call out edge cases. Interviewers grade your process as much as your answer.

---

## Big-O quick reference
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!). Hashing → O(1) lookup. Sorting → O(n log n). Binary search → O(log n). Nested loops → O(n²). Recursion → branches^depth.

---

## ⭐ THE EXPLICITLY-ASKED PROBLEMS (drill these first)

### Primes 1→200, minimal time complexity (asked, Round 2 — they wanted it optimized)
- **S (Situation):** Return (or print) all primes up to `n` (here 200), with the **best possible time complexity** — the interviewer explicitly pushed for optimization.
- **Brute force:** Test each number `k` for divisibility by every `d` up to `√k` → O(n√n).
- **Optimised + key insight:** **Sieve of Eratosthenes**, O(n log log n) — assume all numbers prime, then for each prime cross out its multiples; start crossing at **p²** because every smaller multiple of `p` was already crossed out by a smaller prime.
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
- **Complexity:** Time O(n log log n), Space O(n).
- 🎤 *"Trial division is order n times root n. The optimal approach is the Sieve of Eratosthenes at n log log n: I assume all numbers prime, then for each prime cross out its multiples starting at p-squared, because every smaller multiple was already crossed out by a smaller prime. That start-at-p-squared detail is the optimization they're listening for."*
- ⚠️ **Edge cases & traps:** `n < 2` → no primes (return empty). `p*p` overflows `int` for large `n` → cast to `long` (done above). 0 and 1 are **not** prime — loop starts at 2. Micro-opt: the outer loop only needs to sieve while `p*p ≤ n` (primes above √n have no multiples to mark). For a huge `n`, mention a **segmented sieve** to bound memory.

### Root-to-leaf path sum equals target (asked, Round 3 — "verify sum of nodes root→leaf equals input")
- **S (Situation):** Given a binary tree and a target, return true if **some root-to-leaf path** sums exactly to the target.
- **Brute force:** Enumerate every root-to-leaf path, sum each, compare → still O(n) nodes but with extra path storage.
- **Optimised + key insight:** Single **DFS**, subtracting each node's value from the target as you descend; the success check happens **only at a leaf** (`target == node.val`).
```java
boolean hasPathSum(TreeNode root, int target){
  if (root == null) return false;                       // empty / fell off
  if (root.left == null && root.right == null)          // leaf
      return target == root.val;
  int rem = target - root.val;
  return hasPathSum(root.left, rem) || hasPathSum(root.right, rem);
}   // Time O(n) visit each node once; Space O(h) recursion stack, h = tree height
```
- **Complexity:** Time O(n) (each node once), Space O(h) recursion stack (h = height; O(n) if skewed).
- 🎤 *"I walk down with DFS, subtracting each node's value from the target as I go. The key is the check happens only at a leaf — a node with no children — where I test whether the remaining target equals the leaf's value. Order n time since I touch each node once, order h space for the recursion stack."*
- ⚠️ **Edge cases & traps:** Empty tree → false (even if target is 0 — there's no path). A node with **one** child is **not** a leaf, so don't "succeed" on a mid-path remainder of 0. **Negative values** mean you **can't** prune early (a path can dip below target then recover). Distinguish this (root-to-**leaf**, LC 112) from "any node-to-node path" (LC 437, uses prefix-sum + hashmap) — clarify which the interviewer means. **Variant asked as a follow-up:** to return the actual paths, use backtracking — add node to a list, recurse, remove on the way back up.

### Two Sum (#1, asked)
→ **Full STAR solution + edge cases in [Tier-1](#two-sum-1--75-ukg-oa-classic) below.** One-line recall: one-pass hashmap of `value→index`, look for the complement `target − x`; record *after* checking so you don't reuse an element. O(n)/O(n).

### Valid Palindrome (#125, asked)
→ **Full STAR solution + edge cases in [Tier-1](#valid-palindrome-125--75) below.** One-line recall: two pointers from both ends, skip non-alphanumerics in place, compare lowercased. O(n)/O(1).

### LRU Cache (#146, asked — design)
- **S (Situation):** Design a cache with a fixed capacity supporting `get(key)` and `put(key,value)` both in **O(1)**; on overflow, evict the **least-recently-used** entry. (Reading or writing a key counts as "using" it.)
- **Brute force:** Store entries in a list/array and scan to find LRU on each eviction → O(n) per operation.
- **Optimised + key insight:** Combine a **HashMap** (O(1) key→node lookup) with a **doubly-linked list** ordered by recency — most-recent at the head, evict from the tail. Splicing a node out and to the head is O(1) because you have direct node references.
```java
// Idiomatic Java: LinkedHashMap in access-order mode does the DLL bookkeeping for you.
class LRUCache extends LinkedHashMap<Integer,Integer> {
  private final int cap;
  LRUCache(int cap){ super(cap, 0.75f, true); this.cap = cap; } // true = access-order (recency)
  public int get(int k){ return super.getOrDefault(k, -1); }     // miss -> -1
  public void put(int k,int v){ super.put(k,v); }
  protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e){ return size() > cap; } // auto-evict LRU
}
```
- **Complexity:** Time O(1) for both `get` and `put`, Space O(capacity).
- 🎤 *"The trick is combining two structures: a hashmap for O(1) key lookup and a doubly-linked list to track recency. On access I move the node to the head; when I exceed capacity I evict the tail, the least-recently-used. Both operations stay O(1) because list splicing is constant time. In Java I can express it concisely with an access-ordered LinkedHashMap, but I can explain the hashmap-plus-DLL underneath if they ask me to build it from scratch."*
- ⚠️ **Edge cases & traps:** Updating an **existing** key must also mark it most-recently-used (move to head), not just overwrite the value. `get` on a missing key returns -1 *and* must not change order. Capacity 0 → never stores anything. **Interviewers often want the manual hashmap + DLL** (with a dummy head/tail sentinel to avoid null checks) — be ready to hand-roll it, not just lean on `LinkedHashMap`. **Thread-safety:** this is not concurrent; for a thread-safe LRU you'd guard with a lock (a plain `ConcurrentHashMap` won't preserve recency ordering).

---

## 🔥 MOST LIKELY TOMORROW — UKG Pune priority list (drill top-down)

> Built from your provided frequency table **+** UKG's known OA pattern (HackerRank: **2 coding + 1 SQL**, easy→medium, plus MCQs on DSA/SQL/OOP/time-complexity/C-C++-Java output). Full STAR solutions for every Tier-1 + Tier-2 problem are below.

| Tier | Problem | LC# | Why it's likely | Pattern |
|------|---------|-----|-----------------|---------|
| **1** | Find the Celebrity | 277 | 100% freq in your list | Elimination / two-pointer-ish |
| **1** | Best Time to Buy & Sell Stock | 121 | 87.5% + UKG "stock price" staple | One-pass min tracking |
| **1** | Best Time to Buy & Sell Stock II | 122 | 87.5% | Greedy |
| **1** | Two Sum | 1 | 75% + UKG OA classic | Hashing |
| **1** | Valid Parentheses | 20 | 75% | Stack |
| **1** | Valid Palindrome | 125 | 75% | Two pointers |
| **2** | Longest Consecutive Sequence | 128 | confirmed UKG OA | Hash set |
| **2** | Longest Substring Without Repeating | 3 | confirmed UKG OA | Sliding window |
| **2** | Merge Intervals | 56 | confirmed UKG OA | Intervals |
| **2** | Number of Islands | 200 | confirmed UKG OA | Graph BFS/DFS |
| **3** | LRU Cache | 146 | asked | HashMap + DLL (see above) |
| **3** | Primes 1→200 (Sieve) · Root-to-leaf sum · BST search | — | asked in your rounds | (see above) |

---

## ⭐⭐ TIER-1 — FULL STAR SOLUTIONS (your listed problems)

### Find the Celebrity (#277) — 100% frequency, do this FIRST
- **S (Situation):** Among `n` people, a *celebrity* is known by everyone but knows no one. You can only call `knows(a, b)` → does a know b? Return the celebrity's index or `-1`.
- **Brute force:** For each person, ask if everyone else knows them and they know no one → O(n²) calls.
- **Optimised + key insight:** **Each `knows` call eliminates exactly one person.** Phase 1 — find the *one* possible candidate in a single pass; Phase 2 — verify it. O(n).
```java
public int findCelebrity(int n) {
    int cand = 0;
    for (int i = 1; i < n; i++)
        if (knows(cand, i)) cand = i;     // cand knows i -> cand can't be celeb; i might be
    // Phase 2: verify the lone candidate against everyone
    for (int i = 0; i < n; i++) {
        if (i == cand) continue;
        if (knows(cand, i) || !knows(i, cand)) return -1; // cand knows someone, OR someone doesn't know cand
    }
    return cand;
}
```
- **Complexity:** Time O(n) (~3n calls), Space O(1).
- 🎤 *"The naive solution asks everyone about everyone — n squared calls. The insight is that one comparison eliminates one person: if the current candidate knows person i, the candidate can't be the celebrity, so i becomes the new candidate. One linear pass leaves a single possible celebrity, and then I verify it knows nobody and everybody knows it. Linear time, constant space."*
- ⚠️ Edge: the candidate from phase 1 still **must** be verified — phase 1 only proves no one *else* can be the celebrity, not that this one *is*.

### Best Time to Buy and Sell Stock (#121) — 87.5%
- **S:** `prices[i]` = price on day i. **One** buy then one later sell. Max profit (0 if none).
- **Brute force:** Try every (buy, sell) pair → O(n²).
- **Optimised + key insight:** Track the **minimum price seen so far**; best profit selling *today* = `today − minSoFar`. One pass.
```java
int maxProfit(int[] p) {
    int minP = Integer.MAX_VALUE, best = 0;
    for (int x : p) {
        minP = Math.min(minP, x);        // cheapest buy up to here
        best = Math.max(best, x - minP); // best sell if I sell today
    }
    return best;
}
```
- **Complexity:** Time O(n), Space O(1).
- 🎤 *"I keep the cheapest price I've seen so far as I scan left to right, and at each day I ask 'if I sold today, having bought at that minimum, what's my profit?' I keep the best. That collapses the n-squared pair search into a single linear pass with constant space."*
- ⚠️ Edge: prices strictly decreasing → profit 0 (never sell at a loss); empty array → 0.

### Best Time to Buy and Sell Stock II (#122) — 87.5%
- **S:** Same prices, but **unlimited** transactions (buy/sell as many times, one position at a time). Max total profit.
- **Brute force:** Recursion/DP over buy-sell decisions → exponential or O(n²).
- **Optimised + key insight:** **Greedy — sum every positive day-to-day gain.** Capturing each upward step equals buying at every local valley and selling at every local peak.
```java
int maxProfit(int[] p) {
    int profit = 0;
    for (int i = 1; i < p.length; i++)
        if (p[i] > p[i - 1]) profit += p[i] - p[i - 1]; // grab every upward move
    return profit;
}
```
- **Complexity:** Time O(n), Space O(1).
- 🎤 *"With unlimited transactions the optimal strategy is to capture every rise. Summing each positive consecutive difference is mathematically identical to buying at every valley and selling at every peak — I don't need to find the peaks and valleys explicitly. It's a clean greedy, linear time."*
- ⚠️ Trap: this greedy works *because* transactions are unlimited and frictionless — it would be wrong for #121 (one transaction) or with fees.

### Two Sum (#1) — 75%, UKG OA classic
- **S:** Array `nums` + `target`; return indices of the two numbers summing to target (exactly one solution).
- **Brute force:** Check every pair → O(n²).
- **Optimised + key insight:** Trade space for time — a hashmap of `value→index`; for each `x` look for the complement `target − x` already seen. One pass.
```java
int[] twoSum(int[] a, int t) {
    Map<Integer, Integer> seen = new HashMap<>(); // value -> index
    for (int i = 0; i < a.length; i++) {
        int need = t - a[i];
        if (seen.containsKey(need)) return new int[]{seen.get(need), i};
        seen.put(a[i], i);                        // record AFTER the check (avoids using same element twice)
    }
    return new int[]{-1, -1};
}
```
- **Complexity:** Time O(n), Space O(n).
- 🎤 *"Brute force is every pair, n squared. I trade space for time: as I scan, I ask whether I've already seen the complement, target minus the current value, using a hashmap for O(1) lookup. Recording each number only after checking avoids pairing an element with itself. One pass, linear time."*
- ⚠️ Edge: duplicates (e.g. `[3,3], 6`) — works because we check before inserting; if asked for **all** pairs or **sorted** input, switch to two-pointer.

### Valid Parentheses (#20) — 75%
- **S:** String of `()[]{}`; valid if every bracket is closed by the right type in the right order.
- **Brute force:** Repeatedly remove matched pairs `()`/`[]`/`{}` until stable → O(n²).
- **Optimised + key insight:** A **stack** — push openers, and every closer must match the **most recent** opener (LIFO).
```java
boolean isValid(String s) {
    Deque<Character> st = new ArrayDeque<>();
    Map<Character, Character> close = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : s.toCharArray()) {
        if (close.containsKey(c)) {                              // it's a closing bracket
            if (st.isEmpty() || st.pop() != close.get(c)) return false; // nothing to match / wrong type
        } else {
            st.push(c);                                         // opening bracket
        }
    }
    return st.isEmpty();                                        // leftover openers = invalid
}
```
- **Complexity:** Time O(n), Space O(n).
- 🎤 *"Nesting and matching screams stack. I push every opening bracket; when I hit a closing bracket it must match whatever's on top of the stack — the most recent unclosed opener. If the stack is empty or the top is the wrong type, it's invalid, and at the end the stack must be empty, otherwise there are unclosed brackets."*
- ⚠️ Edge: empty string = valid; odd length can't be valid; a closer as the very first char → stack empty → false.

### Valid Palindrome (#125) — 75%
- **S:** Return true if a string is a palindrome considering only alphanumerics, ignoring case.
- **Brute force:** Build a cleaned, lowercased string, reverse it, compare → O(n) time but O(n) extra space.
- **Optimised + key insight:** **Two pointers** from both ends, skipping non-alphanumerics in place → O(1) space.
```java
boolean isPalindrome(String s) {
    int i = 0, j = s.length() - 1;
    while (i < j) {
        while (i < j && !Character.isLetterOrDigit(s.charAt(i))) i++; // skip junk left
        while (i < j && !Character.isLetterOrDigit(s.charAt(j))) j--; // skip junk right
        if (Character.toLowerCase(s.charAt(i++)) != Character.toLowerCase(s.charAt(j--)))
            return false;
    }
    return true;
}
```
- **Complexity:** Time O(n), Space O(1).
- 🎤 *"I converge two pointers from both ends. The only nuance is skipping non-alphanumeric characters in place before comparing, and lowercasing on the fly — that keeps it constant space instead of building a cleaned copy. Mismatch anywhere means not a palindrome."*
- ⚠️ Edge: empty/single char = palindrome; string of only punctuation (e.g. `",."`) → true.

---

## ⭐ TIER-2 — UKG OA STAPLES (confirmed from recent UKG experiences)

### Longest Consecutive Sequence (#128)
- **S:** Unsorted array; length of the longest run of consecutive integers. Must be **O(n)** (sorting is too slow / not allowed as the intended answer).
- **Brute/naive:** Sort then scan → O(n log n).
- **Insight:** Put all in a **HashSet**; only start counting a streak from a number whose predecessor (`x−1`) is **absent** (a true sequence start) → each number visited at most twice.
```java
int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int x : nums) set.add(x);
    int best = 0;
    for (int x : set) {
        if (!set.contains(x - 1)) {           // x is the start of a streak
            int len = 1, cur = x;
            while (set.contains(cur + 1)) { cur++; len++; }
            best = Math.max(best, len);
        }
    }
    return best;
}
```
- **Complexity:** Time O(n), Space O(n).
- 🎤 *"Sorting gives n-log-n, but the intended trick is a hash set for O(1) membership. The key is I only begin counting from a number whose predecessor is missing — a genuine sequence start — so I never recount a streak from the middle, and each element is touched at most twice. That's what makes it linear."*

### Longest Substring Without Repeating Characters (#3)
- **S:** Length of the longest substring with all distinct characters.
- **Brute force:** Check every substring for uniqueness → O(n²)/O(n³).
- **Insight:** **Sliding window** + a map of `char→lastIndex`; on a repeat, jump the left edge past the previous occurrence.
```java
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> last = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (last.containsKey(c) && last.get(c) >= left)
            left = last.get(c) + 1;           // shrink window past the duplicate
        last.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```
- **Complexity:** Time O(n), Space O(min(n, charset)).
- 🎤 *"I keep a window with all-unique characters and slide the right edge out. When I hit a character I've already seen inside the window, I jump the left edge to just past its previous position, so the window stays duplicate-free. Tracking last-seen indices makes that jump O(1), giving a single linear pass."*

### Merge Intervals (#56)
- **S:** Array of `[start,end]` intervals; merge all overlapping ones.
- **Insight:** **Sort by start**, then sweep: if the next interval starts ≤ current end, extend; else push and move on.
```java
int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    List<int[]> out = new ArrayList<>();
    for (int[] iv : intervals) {
        if (out.isEmpty() || iv[0] > out.get(out.size() - 1)[1])
            out.add(iv);                                          // no overlap -> new interval
        else
            out.get(out.size() - 1)[1] = Math.max(out.get(out.size() - 1)[1], iv[1]); // overlap -> extend end
    }
    return out.toArray(new int[0][]);
}
```
- **Complexity:** Time O(n log n) (sort dominates), Space O(n).
- 🎤 *"Sorting by start time is the unlock: once sorted, any overlap must be with the most recently kept interval, so I sweep once and either extend its end or start a fresh interval. The sort dominates at n-log-n."*

### Number of Islands (#200)
- **S:** Grid of `'1'` (land) / `'0'` (water); count connected land groups (4-directional).
- **Insight:** Scan the grid; each unvisited `'1'` starts a **DFS/BFS** that sinks the whole island (mark visited), incrementing the count once.
```java
int numIslands(char[][] g) {
    int count = 0;
    for (int r = 0; r < g.length; r++)
        for (int c = 0; c < g[0].length; c++)
            if (g[r][c] == '1') { count++; sink(g, r, c); }
    return count;
}
void sink(char[][] g, int r, int c) {
    if (r < 0 || c < 0 || r >= g.length || c >= g[0].length || g[r][c] != '1') return;
    g[r][c] = '0';                                   // mark visited (flood)
    sink(g, r + 1, c); sink(g, r - 1, c); sink(g, r, c + 1); sink(g, r, c - 1);
}
```
- **Complexity:** Time O(rows·cols), Space O(rows·cols) worst-case recursion.
- 🎤 *"Each new piece of land I find is a new island, so I increment the count and then flood-fill that entire island to water so I never count it again. It's a connected-components count — DFS or BFS both work; I mutate the grid to mark visited and keep space down."*
- ⚠️ Edge: deep recursion on a huge all-land grid can stack-overflow → use an explicit stack/BFS queue.

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
