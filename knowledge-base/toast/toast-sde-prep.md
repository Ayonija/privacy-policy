# Toast — Round 1 DSA Interview Prep
**Role:** Senior Software Engineer — Backend (Java/Kotlin)  
**Round:** R1 — Algorithmic / DSA (~45 min)  
**Study time:** 2–3 hours of focused practice

---

## How to Use This Guide

Each problem follows the **STAR interview framework** adapted for coding:

- **S — Situation:** What is the problem really asking?
- **T — Task:** What is the core constraint or challenge?
- **A — Approach:** Walk from brute force → optimal (think out loud this way in the interview)
- **R — Result:** Final code + complexity you state before moving on

> **Rule for Round 1:** Always clarify constraints first, name your approach's complexity before coding, and say edge cases before the interviewer asks.

---

## Problem 1 — Toast Original: Stock Profit with 5-Second Gap

### S — Situation
Given a `List<Long>` of stock prices where each element represents the price at a 1-second interval (in order), find the maximum profit from a single buy + sell. The sell must happen **at least 5 seconds** after the buy. Return `0` if no profit is possible.

### T — Task
This is LC 121 (Buy and Sell Stock) with an extra constraint: `sell_index - buy_index >= 5`. The challenge is not letting a recent minimum price be used for a sell that's too close in time.

### A — Approach

**Step 1 — Brute Force O(n²): state this first, then improve**
```java
public long maxProfit(List<Long> prices) {
    long maxProfit = 0;
    int n = prices.size();
    for (int buy = 0; buy <= n - 6; buy++) {
        for (int sell = buy + 5; sell < n; sell++) {
            maxProfit = Math.max(maxProfit, prices.get(sell) - prices.get(buy));
        }
    }
    return maxProfit;
}
```
Say: *"This is O(n²) — for large price lists this is too slow. Let me optimize."*

**Step 2 — Optimal O(n): Sliding minimum window**

Key insight: as `sell` moves forward, the earliest valid `buy` index is `sell - 5`. So maintain a running minimum, but only update it using the price at `sell - 5` (not `sell - 1` like standard LC 121).

```java
public long maxProfit(List<Long> prices) {
    int n = prices.size();
    if (n < 6) return 0L;

    long maxProfit = 0L;
    long minPrice = prices.get(0);

    for (int sell = 5; sell < n; sell++) {
        // Only consider buy prices that satisfy the >= 5s gap
        minPrice = Math.min(minPrice, prices.get(sell - 5));
        long profit = prices.get(sell) - minPrice;
        maxProfit = Math.max(maxProfit, profit);
    }
    return maxProfit;
}
```

**Why `sell - 5` and not `sell - 1`?**
At `sell = 5`: earliest valid buy is index `0` → `sell - 5 = 0` ✓  
At `sell = 6`: earliest valid buy is index `1` → `sell - 5 = 1` ✓  
Each iteration we unlock one new valid buy candidate and fold it into `minPrice`.

### R — Result
| Approach | Time | Space |
|----------|------|-------|
| Brute force | O(n²) | O(1) |
| Sliding min | O(n) | O(1) |

**Edge cases to state:**
- `n < 6` → impossible to have a 5-second gap → return `0`
- All prices strictly descending → every profit is negative → `maxProfit` stays `0` ✓
- All prices equal → profit is 0 ✓

---

## Problem 2 — LC 121: Best Time to Buy and Sell Stock

### S — Situation
Given `int[] prices` where `prices[i]` is the stock price on day `i`. Buy once, sell once. Maximize profit. Return `0` if no profit.

### T — Task
Find two indices `i < j` such that `prices[j] - prices[i]` is maximized. Classic single-pass min-tracking problem.

### A — Approach

**Brute force O(n²):** Try every pair — too slow, mention and skip.

**Optimal O(n): Track minimum so far**

As you scan right, every price is a potential sell. The best profit at any sell point is `price - min_so_far`. Update min after computing profit.

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;
    int maxProfit = 0;

    for (int price : prices) {
        if (price < minPrice) {
            minPrice = price;           // new cheapest buy day
        } else {
            maxProfit = Math.max(maxProfit, price - minPrice); // best sell today
        }
    }
    return maxProfit;
}
```

**Mental model:** Imagine you're scanning a graph left to right, tracking the lowest valley seen so far. Every peak above that valley is a profit candidate.

### R — Result
Time: O(n) | Space: O(1)

**Edge cases:**
- Single element → return `0`
- `[7,6,4,3,1]` → always falling → return `0` (maxProfit never updates above 0)

---

## Problem 3 — LC 122: Best Time to Buy and Sell Stock II

### S — Situation
Same price array, but now you can buy and sell **multiple times** (hold at most 1 share at a time). Maximize total profit.

### T — Task
Capture every upward movement. Unlike LC 121, there's no single transaction limit.

### A — Approach

**Key insight (greedy):** If tomorrow is higher than today, buy today and sell tomorrow. You don't need to simulate actual holdings — just sum every positive day-to-day difference.

Why this works: `prices[4] - prices[1]` equals `(prices[2]-prices[1]) + (prices[3]-prices[2]) + (prices[4]-prices[3])`. So capturing every upward step is equivalent to finding the perfect buy/sell windows.

```java
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++) {
        if (prices[i] > prices[i - 1]) {
            profit += prices[i] - prices[i - 1];
        }
    }
    return profit;
}
```

### R — Result
Time: O(n) | Space: O(1)

**Distinguish from LC 121 in interview:** *"In 121 we track a running minimum because we only get one transaction. In 122 we just take every positive increment — unlimited transactions means greedy is optimal."*

---

## Problem 4 — LC 1: Two Sum

### S — Situation
Given `int[] nums` and `int target`, return indices of two numbers that add up to `target`. Exactly one solution exists. Can't use the same element twice.

### T — Task
For each element, find its complement (`target - nums[i]`) in the array. Avoid O(n²) nested loops.

### A — Approach

**Brute force O(n²):** Nested loops — mention only.

**Optimal O(n): HashMap for complement lookup**

As you scan, store each value and its index. For each new element, check if its complement already exists in the map.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);  // store AFTER checking to avoid using same element twice
    }
    return new int[]{};  // guaranteed solution exists per problem statement
}
```

**Why store after checking?** If `target = 6` and `nums[i] = 3`, we don't want to use index `i` as both buy and sell. Storing after the check prevents `seen` from returning `i` for its own complement.

### R — Result
Time: O(n) | Space: O(n)

**Follow-up the interviewer might ask:** *"What if the array is sorted?"* → Use two pointers: left + right, move inward based on sum vs target. O(n) time, O(1) space.

---

## Problem 5 — LC 20: Valid Parentheses

### S — Situation
Given a string containing only `()[]{}`, return `true` if every open bracket is closed by the same type in correct order.

### T — Task
Track unmatched open brackets. When a close bracket appears, it must match the most recently opened bracket (LIFO → Stack).

### A — Approach

**Stack-based:**
- Open bracket → push onto stack
- Close bracket → stack must be non-empty AND top must be the matching open bracket
- End: stack must be empty

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> matchMap = Map.of(')', '(', ']', '[', '}', '{');

    for (char c : s.toCharArray()) {
        if (matchMap.containsValue(c)) {         // it's an open bracket
            stack.push(c);
        } else {                                  // it's a close bracket
            if (stack.isEmpty() || stack.peek() != matchMap.get(c)) {
                return false;
            }
            stack.pop();
        }
    }
    return stack.isEmpty();
}
```

**Simpler version (explicit matching):**
```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
}
```

### R — Result
Time: O(n) | Space: O(n)

**Edge cases:**
- `"("` → stack not empty at end → `false`
- `")("` → stack empty when `)` arrives → `false`
- Empty string → stack empty → `true`

---

## Problem 6 — LC 125: Valid Palindrome

### S — Situation
Given string `s`, return `true` if it reads the same forward and backward after keeping only alphanumeric characters and ignoring case.

### T — Task
Filter non-alphanumeric characters and case, then check palindrome. Do it without extra space if possible.

### A — Approach

**Extra space O(n):** Build cleaned string, compare with reverse — mention, then optimize.

**Optimal O(n) time O(1) space: Two pointers**

Left pointer starts at 0, right at end. Skip non-alphanumeric from both sides, then compare lowercased characters.

```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;

    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left)))  left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;

        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

### R — Result
Time: O(n) | Space: O(1)

**Edge cases:**
- `" "` (only spaces) → all skipped → `left >= right` immediately → `true`
- `"a"` → single char → palindrome → `true`
- `"race a car"` → `false`

---

## Problem 7 — LC 277: Find the Celebrity

### S — Situation
In a party of `n` people, a celebrity is known by everyone but knows nobody. Given a helper `boolean knows(a, b)`, find the celebrity in O(n) calls. Return `-1` if none exists.

### T — Task
Naively checking all pairs is O(n²) API calls. Need to eliminate candidates efficiently.

### A — Approach

**Two-phase algorithm:**

**Phase 1 — Elimination (find the only possible candidate):**
Start with `candidate = 0`. For each person `i`:
- If `candidate` knows `i`: `candidate` can't be a celebrity (celebrities know nobody) → set `candidate = i`
- If `candidate` doesn't know `i`: `i` can't be a celebrity (everyone must know celebrity) → keep `candidate`

After this loop, only one person could possibly be the celebrity.

**Phase 2 — Verification (confirm the candidate):**
- Candidate must NOT know anyone
- Everyone must know the candidate

```java
public int findCelebrity(int n) {
    // Phase 1: find candidate
    int candidate = 0;
    for (int i = 1; i < n; i++) {
        if (knows(candidate, i)) {
            candidate = i;
        }
    }

    // Phase 2: verify candidate
    for (int i = 0; i < n; i++) {
        if (i == candidate) continue;
        if (knows(candidate, i) || !knows(i, candidate)) {
            return -1;
        }
    }
    return candidate;
}
```

**Why Phase 1 is safe:** Each step eliminates exactly one person. If a celebrity exists, they can never be eliminated — because no one can say "celebrity knows X" (celebrities know nobody), and "X doesn't know celebrity" is impossible (everyone knows celebrity). So the true celebrity always survives to be the candidate.

### R — Result
Time: O(n) API calls | Space: O(1)

**Common mistake:** Skipping Phase 2. Phase 1 only narrows to one candidate — it doesn't confirm one exists. Always verify.

---

## Patterns Summary Card

| Pattern | Problems | When to use |
|---------|----------|-------------|
| Running min/max | LC 121, Toast stock | Optimize over "best pair" with ordering constraint |
| Greedy increment | LC 122 | Unlimited choices, capture every local gain |
| HashMap lookup | LC 1 | O(1) membership/complement checks |
| Stack (LIFO) | LC 20 | Matching/nesting problems |
| Two pointers | LC 125 | In-place scan from both ends |
| Elimination + verify | LC 277 | Reduce candidate set then confirm |

---

## Interview Execution Checklist (use for every problem)

1. **Restate the problem** — confirm you understood it correctly (30 sec)
2. **Ask 1-2 clarifying questions** — input size? any constraints? return type?
3. **State brute force + complexity** — *"Naively this is O(n²) because..."*
4. **Explain optimization** — *"I can improve to O(n) by..."*
5. **Code the optimized solution** — talk through key lines as you write
6. **State edge cases** before the interviewer asks — empty input, single element, no-profit case
7. **Walk through one example** after writing — pick the tricky case, not the happy path
8. **State final complexity** — time and space

> **Senior-level signal:** Proactively mention trade-offs. *"I'm using a HashMap here which costs O(n) space — if memory is a constraint and the array were sorted, two pointers would be O(1) space."*
