# Day 1 — Pattern Engine: Build Your Classification Reflex
**DE Shaw Round 1 Prep | Day 1 of 2**

## Goal
Wire 6 core patterns into muscle memory. Each problem below is solvable in 15 minutes once you know the pattern. Time yourself — if you exceed 15 min, note it and move on.

---

## Block 1: Arrays & Hashing (Most Frequent in DE Shaw)

### Problem 1 — Two Sum (LC 1) | Easy | HashMap
**Trigger:** "Find pair with target" + unsorted → HashMap, not Two Pointers

```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
```

**Pattern:** store complement → look up on each step

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) return new int[]{map.get(complement), i};
        map.put(nums[i], i);
    }
    return new int[]{};
}
```
**Complexity:** O(n) time, O(n) space

**Edge cases:** duplicate values (map stores latest index, but problem guarantees one answer), negative numbers (works fine)

**STAR Walkthrough — Two Sum (LC 1):**
- **S:** "Given an unsorted integer array and a target. Naive approach: O(n²) — check every pair."
- **T:** "Achieve O(n) using HashMap — key insight: for each element, the complement we need is already determined, so we can look it up in O(1)."
- **A:** "(1) Initialize empty HashMap. (2) At each index, compute `complement = target - nums[i]`. (3) If complement exists in map, return its stored index. Critical detail: store `nums[i]` AFTER the lookup, not before — this prevents a single element matching itself when target is double that element."
- **R:** "O(n) time, O(n) space. At n=10^6: one linear pass vs 10^12 iterations for brute force — effectively real-time vs minutes."
- **Alt:** Instead of HashMap, you could sort + two pointers — but sorting destroys the original indices, which this problem requires returning. HashMap wins here.

---

### Problem 2 — Subarray Sum Equals K (LC 560) | Medium | Prefix Sum + HashMap
**Trigger:** "count subarrays" + "sum = target" → prefix sum

```
Input:  nums = [1, 1, 1], k = 2
Output: 2
```

**Pattern:** `count += seen[prefixSum - k]` at each step

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> seen = new HashMap<>();
    seen.put(0, 1);
    int sum = 0, count = 0;
    for (int n : nums) {
        sum += n;
        count += seen.getOrDefault(sum - k, 0);
        seen.merge(sum, 1, Integer::sum);
    }
    return count;
}
```
**Complexity:** O(n) time, O(n) space

**Why `seen.put(0,1)`?** Handles subarrays starting at index 0 whose sum equals k directly.

**STAR Walkthrough — Subarray Sum Equals K (LC 560):**
- **S:** "Given an integer array (may contain negatives) and integer k; count contiguous subarrays that sum to k. Naive approach: O(n²) — enumerate all start/end pairs."
- **T:** "Achieve O(n) using Prefix Sum + HashMap — key insight: if `prefixSum[j] - prefixSum[i] = k`, the subarray `[i+1..j]` sums to k, so we need how many prior prefix sums equal `currentSum - k`."
- **A:** "(1) Seed `seen = {0: 1}` to handle subarrays starting at index 0. (2) Accumulate running sum. (3) Add `seen.get(sum - k, 0)` to count. Critical detail: update the map AFTER the count lookup — otherwise a subarray of length 0 could be counted when `sum == k`."
- **R:** "O(n) time, O(n) space. At n=10^5: single pass vs 10^10 pair-checks for brute force."
- **Alt:** Instead of prefix sum, you could use a nested loop — but negative numbers prevent the two-pointer shrink optimization, so HashMap is the only O(n) solution here.

---

### Problem 3 — Maximum Subarray (LC 53) | Medium | Kadane's
**Trigger:** "maximum sum subarray" → Kadane's (local vs global max)

```
Input:  [-2,1,-3,4,-1,2,1,-5,4]
Output: 6   (subarray [4,-1,2,1])
```

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], currentSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```
**Complexity:** O(n) time, O(1) space

**Mental model:** at each index, decide "am I better starting fresh here, or extending the previous subarray?"

**STAR Walkthrough — Maximum Subarray (LC 53):**
- **S:** "Given an integer array with possible negatives; find the maximum sum of any contiguous subarray. Naive approach: O(n²) — try all subarrays."
- **T:** "Achieve O(n) using Kadane's algorithm — key insight: at each index, the best subarray ending here is either the element alone or the element extended from the best subarray ending at the previous index."
- **A:** "(1) Initialize `currentSum = maxSum = nums[0]`. (2) For each subsequent element, set `currentSum = max(nums[i], currentSum + nums[i])`. (3) Update `maxSum` if `currentSum` is larger. Critical detail: initialize from `nums[0]` not 0 — this handles all-negative arrays correctly."
- **R:** "O(n) time, O(1) space. At n=10^6: constant memory vs O(n) for a DP table, and one pass vs O(n²) for brute force."
- **Alt:** Instead of Kadane's, you could use divide-and-conquer in O(n log n) — but Kadane's is simpler, faster, and uses O(1) space, making it unambiguously better for this problem.

---

## Block 2: Sliding Window

### Problem 4 — Longest Substring Without Repeating Characters (LC 3) | Medium
**Trigger:** "longest substring" + "no repeating" → variable sliding window

```
Input:  "abcabcbb"
Output: 3   ("abc")
```

```java
public int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int l = 0, maxLen = 0;
    for (int r = 0; r < s.length(); r++) {
        while (window.contains(s.charAt(r))) {
            window.remove(s.charAt(l++));
        }
        window.add(s.charAt(r));
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
```
**Complexity:** O(n) time, O(min(m,n)) space where m = charset size

**STAR Walkthrough — Longest Substring Without Repeating Characters (LC 3):**
- **S:** "Given a string; find the length of the longest substring with all unique characters. Naive approach: O(n²) or O(n³) — check all substrings for uniqueness."
- **T:** "Achieve O(n) using variable sliding window — key insight: maintain a window where all characters are unique; when a duplicate enters from the right, shrink from the left until the duplicate is evicted."
- **A:** "(1) Initialize a HashSet for the window and left pointer `l = 0`. (2) Expand right pointer; if `s[r]` is already in the window, remove `s[l]` and advance `l` until the duplicate is gone. (3) Add `s[r]` and update max length. Critical detail: the while-loop removal (not a single removal) is needed because `l` may need to advance past multiple characters before the duplicate is cleared."
- **R:** "O(n) time, O(min(m,n)) space where m = alphabet size. Each character is added and removed from the set at most once — O(n) total operations."
- **Alt:** Instead of a HashSet, you could use a HashMap storing the last seen index — this allows jumping `l` directly to `lastSeen[char] + 1` rather than shrinking one-by-one, but the HashSet version is simpler to reason about and equally O(n).

---

### Problem 5 — Minimum Window Substring (LC 76) | Hard | Sliding Window + Freq Map
**Trigger:** "minimum window containing all chars" → sliding window with two freq maps

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
```

```java
public String minWindow(String s, String t) {
    Map<Character, Integer> need = new HashMap<>(), have = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
    int formed = 0, required = need.size();
    int l = 0, minLen = Integer.MAX_VALUE, minL = 0;
    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        have.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && have.get(c).equals(need.get(c))) formed++;
        while (formed == required) {
            if (r - l + 1 < minLen) { minLen = r - l + 1; minL = l; }
            char lc = s.charAt(l);
            have.merge(lc, -1, Integer::sum);
            if (need.containsKey(lc) && have.get(lc) < need.get(lc)) formed--;
            l++;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minL, minL + minLen);
}
```
**Complexity:** O(|s| + |t|) time

**Key insight:** `formed` tracks how many characters in `t` are currently satisfied — only shrink when `formed == required`.

**STAR Walkthrough — Minimum Window Substring (LC 76):**
- **S:** "Given strings s and t; find the minimum window in s that contains all characters of t. Naive approach: O(|s|²×|t|) — check every substring against t's character requirements."
- **T:** "Achieve O(|s| + |t|) using sliding window with two frequency maps — key insight: use a `formed` counter to know in O(1) whether the window is currently valid, avoiding a full map comparison on every step."
- **A:** "(1) Build `need` freq map from t; set `required = need.size()`. (2) Expand right: add character to `have`; if `have[c] == need[c]`, increment `formed`. (3) While `formed == required`, record window, then shrink left: if removing `s[l]` drops `have[lc]` below `need[lc]`, decrement `formed`. Critical detail: compare with `.equals()` not `==` in Java for Integer objects — autoboxing with `==` fails for values outside the cached range [-128, 127]."
- **R:** "O(|s| + |t|) time, O(|t|) space. At |s|=10^5, |t|=100: two-pointer scans each char at most twice vs O(|s|²) substring enumeration."
- **Alt:** Instead of sliding window, you could use binary search on window size — but this gives O(|s| log |s| × |t|), strictly worse. The sliding window's amortized two-pointer scan is optimal.

---

## Block 3: Stack & Two Pointers

### Problem 6 — Trapping Rain Water (LC 42) | Hard | Two Pointers
**Trigger:** "trap water" → two pointers (simpler than stack for interviews)

```
Input:  [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

```java
public int trap(int[] height) {
    int l = 0, r = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] < height[r]) {
            if (height[l] >= leftMax) leftMax = height[l];
            else water += leftMax - height[l];
            l++;
        } else {
            if (height[r] >= rightMax) rightMax = height[r];
            else water += rightMax - height[r];
            r--;
        }
    }
    return water;
}
```
**Complexity:** O(n) time, O(1) space

**Mental model:** water at index i = min(maxLeft, maxRight) - height[i]. The two-pointer moves from the side with the smaller max, guaranteeing the min is already known.

**STAR Walkthrough — Trapping Rain Water (LC 42):**
- **S:** "Given a height map array; compute total water trapped between bars. Naive approach: O(n²) — for each index, scan left and right to find maxLeft and maxRight."
- **T:** "Achieve O(n) time, O(1) space using two pointers — key insight: water at any index is determined by the minimum of its left and right maximums; if we process from the side with the smaller maximum, that maximum is already finalized."
- **A:** "(1) Initialize `l=0, r=n-1, leftMax=0, rightMax=0`. (2) Whichever side has the smaller max, process that side: if `height[l] >= leftMax`, update leftMax; else add `leftMax - height[l]` to water. (3) Advance that pointer inward. Critical detail: the invariant is that we only compute water for an index when we're certain the limiting max (the smaller side) won't change — this is only true when processing from the shorter side."
- **R:** "O(n) time, O(1) space. At n=10^6: no auxiliary arrays vs O(n) for the precomputed left/right max arrays approach."
- **Alt:** Instead of two pointers, you could precompute leftMax[] and rightMax[] arrays — O(n) time but O(n) space. The two-pointer approach gives the same time with constant space, making it strictly better for this problem.

---

### Problem 7 — Daily Temperatures (LC 739) | Medium | Monotonic Stack
**Trigger:** "next warmer day" / "next greater element" → decreasing monotonic stack

```
Input:  [73,74,75,71,69,72,76,73]
Output: [1,1,4,2,1,1,0,0]
```

```java
public int[] dailyTemperatures(int[] temps) {
    int[] result = new int[temps.length];
    Deque<Integer> stack = new ArrayDeque<>(); // stores indices
    for (int i = 0; i < temps.length; i++) {
        while (!stack.isEmpty() && temps[stack.peek()] < temps[i]) {
            int idx = stack.pop();
            result[idx] = i - idx;
        }
        stack.push(i);
    }
    return result;
}
```
**Complexity:** O(n) time, O(n) space

**STAR Walkthrough — Daily Temperatures (LC 739):**
- **S:** "Given a temperatures array; for each day, find how many days until a warmer temperature. Naive approach: O(n²) — for each index, scan right until a higher temperature is found."
- **T:** "Achieve O(n) using a decreasing monotonic stack — key insight: maintain a stack of indices with unresolved 'waiting for warmer' status; when a warmer day arrives, pop and resolve all waiting indices."
- **A:** "(1) Initialize result array and index stack. (2) For each index i: while stack is non-empty and `temps[stack.top()] < temps[i]`, pop index `idx` and set `result[idx] = i - idx`. (3) Push current index. Critical detail: store indices, not values — you need the distance `i - idx`, which requires knowing the original position."
- **R:** "O(n) time, O(n) space. Each index is pushed and popped at most once — O(n) total stack operations vs O(n²) for brute force."
- **Alt:** Instead of monotonic stack, you could scan right from each index — O(n²). There is no O(n) solution without the stack; this is exactly the pattern for 'next greater element' problems.

---

### Problem 8 — Merge Intervals (LC 56) | Medium | Sort + Greedy
**Trigger:** "merge overlapping intervals" → sort by start, then scan

```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> result = new ArrayList<>();
    int[] current = intervals[0];
    for (int[] interval : intervals) {
        if (interval[0] <= current[1]) {
            current[1] = Math.max(current[1], interval[1]);
        } else {
            result.add(current);
            current = interval;
        }
    }
    result.add(current);
    return result.toArray(new int[0][]);
}
```
**Complexity:** O(n log n) time (dominated by sort)

**STAR Walkthrough — Merge Intervals (LC 56):**
- **S:** "Given a list of intervals; merge all overlapping intervals and return the result. Naive approach: O(n²) — for each interval, check every other interval for overlap."
- **T:** "Achieve O(n log n) using sort + greedy scan — key insight: after sorting by start time, any overlapping interval must have its start ≤ the current merged interval's end, allowing a single left-to-right pass."
- **A:** "(1) Sort intervals by start time. (2) Initialize `current = intervals[0]`. (3) For each subsequent interval: if `interval[0] <= current[1]`, merge by updating `current[1] = max(current[1], interval[1])`; else, add current to result and advance current. Critical detail: take `max` of end times when merging — a fully contained interval would incorrectly shrink the end if you just take the new interval's end."
- **R:** "O(n log n) time (dominated by sort), O(1) extra space (excluding output). No O(n) solution exists without sorted input."
- **Alt:** Instead of sorting by start, you could sort by end — but then the merge condition becomes more complex and you'd need to handle non-overlapping detection differently. Start-time sort gives the simplest invariant.

---

## Day 1 Checklist

- [ ] Read pattern-cheatsheet.md — 5-question trigger framework memorized
- [ ] Solved all 8 problems timed (≤15 min each)
- [ ] Wrote code skeletons from memory for: Two Pointers, Sliding Window, Monotonic Stack, Prefix Sum
- [ ] Can state time + space complexity instantly for each problem
- [ ] Listed which problems took >15 min for Day 2 review

## Flashcards

| Q | A |
|---|---|
| What pattern handles "count subarrays with sum = k"? | Prefix Sum + HashMap. Key: `count += seen[prefixSum - k]` |
| Why does Kadane's work? | At each index, reset if current element alone beats extending: `max(nums[i], cur + nums[i])` |
| When does sliding window shrink? | When the window constraint is violated — shrink from left until valid again |
| Monotonic stack stores indices or values? | Indices — so you can compute distances and access original values |
| What does `seen.put(0,1)` do in prefix sum? | Handles subarrays from index 0 whose total sum equals k |
