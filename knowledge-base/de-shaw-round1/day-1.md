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
