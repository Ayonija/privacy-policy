# Day 35 — Hashing: Advanced Sliding Window + HashMap
**Week 05 | Phase 1: DSA Mastery | Month 2**

## Focus
Push sliding window fluency to its limits: count subarrays with exactly-k constraint, divide-and-conquer on substrings, and the hardest sliding window problem on LeetCode.

---

## DSA (2 hours)
### Pattern: Sliding Window — Exact-K via "At-Most-K Minus At-Most-(K-1)" + Divide & Conquer

**Core idea — Exact-K trick:**
Counting subarrays with *exactly k* odd numbers is hard directly. But: `exactly(k) = at_most(k) − at_most(k−1)`. The `at_most(k)` function is a standard variable sliding window. This technique works whenever "exactly k" of something is required.

**Divide & Conquer for Least K Repeating:**
The window is invalid when any character appears fewer than k times. Key insight: fix the *number of unique characters* in the window (1 to 26). For each unique-count target, find the longest window where all chars appear ≥ k times AND the window has exactly that many unique chars. This converts to a standard sliding window.

**Concatenation of All Words (LC 30):**
All words have the same length `L`. Slide a window of size `n×L`; instead of char-by-char, slide `L` bytes at a time. There are `L` possible starting offsets (0, 1, …, L−1). For each offset, run a word-frequency window.

**Trigger condition:**
- "count subarrays with *exactly* k of something" → `at_most(k) - at_most(k-1)`
- "longest substring where all chars appear ≥ k times" → fix unique-char count, standard window
- "find all substrings = concatenation of all words" → word-level sliding window with L-spaced offsets

**Time complexity:** O(n) for exact-k | O(26n) = O(n) for least-k-repeating | O(n·L) for concatenation

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Count Number of Nice Subarrays | 1248 | Medium | Exact-K → at_most trick | Convert odd numbers to 1, even to 0; `exactly(k) = at_most(k) - at_most(k-1)` |
| 2 | Longest Substring with At Least K Repeating Characters | 395 | Medium | Fix unique-char count (1..26) + sliding window | For each target unique count u (1 to 26): find longest window with exactly u unique chars AND all appearing ≥ k times |
| 3 | Substring with Concatenation of All Words | 30 | Hard | Word-level sliding window × L offsets | Slide at word-length granularity; L starting offsets; word freq map with `need` and `have` counts |

---

### Code Skeleton
```java
class Solution {
    // Count Number of Nice Subarrays (LC 1248)
    public static int numberOfSubarrays(int[] nums, int k) {
        return atMost(nums, k) - atMost(nums, k - 1);
    }

    private static int atMost(int[] nums, int goal) {
        if (goal < 0) return 0;
        int count = 0, left = 0, curr = 0;
        for (int right = 0; right < nums.length; right++) {
            curr += nums[right] % 2;   // 1 if odd, 0 if even
            while (curr > goal) {
                curr -= nums[left] % 2;
                left++;
            }
            count += right - left + 1;
        }
        return count;
    }

    // Longest Substring with At Least K Repeating Characters (LC 395)
    public static int longestSubstring(String s, int k) {
        int result = 0;
        for (int uniqueTarget = 1; uniqueTarget <= 26; uniqueTarget++) {  // fix number of unique chars
            Map<Character, Integer> freq = new HashMap<>();
            int left = 0, atLeastKCount = 0, uniqueCount = 0;
            for (int right = 0; right < s.length(); right++) {
                char rc = s.charAt(right);
                freq.put(rc, freq.getOrDefault(rc, 0) + 1);
                if (freq.get(rc) == 1) uniqueCount++;
                if (freq.get(rc) == k) atLeastKCount++;
                while (uniqueCount > uniqueTarget) {
                    char lc = s.charAt(left);
                    if (freq.get(lc) == k) atLeastKCount--;
                    if (freq.get(lc) == 1) uniqueCount--;
                    freq.put(lc, freq.get(lc) - 1);
                    left++;
                }
                if (uniqueCount == atLeastKCount) {
                    result = Math.max(result, right - left + 1);
                }
            }
        }
        return result;
    }

    // Substring with Concatenation of All Words (LC 30) — skeleton
    public static List<Integer> findSubstring(String s, String[] words) {
        List<Integer> result = new ArrayList<>();
        if (s == null || s.isEmpty() || words == null || words.length == 0) return result;
        int wordLen = words[0].length(), nWords = words.length;
        int windowLen = wordLen * nWords;
        Map<String, Integer> need = new HashMap<>();
        for (String w : words) need.put(w, need.getOrDefault(w, 0) + 1);
        for (int offset = 0; offset < wordLen; offset++) {
            // slide a window of n_words words, stepping by word_len
            // have = counter for current window words
            // use left/right pointers at word granularity
            // TODO: implement
        }
        return result;
    }
}
```

---

### Edge Cases to Trace Before Coding
- LC 1248: k=0 — `atMost(-1)` should return 0; handle in `atMost` by returning 0 for negative goal
- LC 395: k=1 — every substring is valid; answer = len(s)
- LC 30: words with duplicates; s shorter than one word; words of length 1

---

## System Design (1 hour)
### Topic: Composite Indexes, Prefix Rule & Index-Only Scans

**Composite Index (multi-column):**
A B+Tree index on `(col_a, col_b, col_c)` sorts first by col_a, then by col_b within the same col_a value, then col_c. Queries benefit from a composite index *only if they use a prefix of the column list*.

**The Prefix Rule:**
| Query | Uses index `(a, b, c)`? |
|-------|------------------------|
| `WHERE a = 1` | ✅ Yes (uses prefix a) |
| `WHERE a = 1 AND b = 2` | ✅ Yes (uses prefix a, b) |
| `WHERE a = 1 AND b > 2 AND c = 3` | ⚠️ Partial — a and b (range stops prefix) |
| `WHERE b = 2` | ❌ No — a must be in the WHERE for the index to help |
| `WHERE a = 1 AND c = 3` | ⚠️ Only a — c is not reachable without b |

**Column order in composite index:**
1. Equality conditions first (most selective columns first among them)
2. Range condition last (e.g., `BETWEEN`, `>`, `<`)
3. Columns needed for ORDER BY / GROUP BY if possible

**Index-only scan (covering index):**
If the index contains all columns referenced in the query, the DB never reads the main table. Example: `SELECT name FROM users WHERE city = 'London'` with index `(city, name)` — fully answered from the index.

**Interview talking point:** "If asked how to optimise `SELECT name FROM orders WHERE user_id = 5 AND status = 'pending' ORDER BY created_at`, answer: create a composite index on `(user_id, status, created_at, name)` — user_id and status are equality conditions (prefix), created_at satisfies ORDER BY without a sort step, and including name makes it a covering index eliminating the table lookup entirely."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a situation where the order in which you addressed sub-problems mattered — solving the wrong one first wasted significant effort — analogous to column ordering in a composite index where the wrong order makes the index useless.
- Leadership principle: Are Right, A Lot

---

## Flashcards

| Q | A |
|---|---|
| How does the exact-K → at_most trick work? | `exactly(k) = at_most(k) − at_most(k−1)`; implement a reusable `at_most(goal)` variable window and call it twice |
| Why does LC 395 iterate over unique-char counts 1 to 26 instead of solving directly? | Directly maximising length with the "all chars ≥ k" constraint has no clean shrink condition; fixing the unique-char count turns it into a standard two-constraint window that CAN be shrunk |
| What is the prefix rule for composite indexes? | A query uses the index only if it filters on a *prefix* of the indexed columns — the leftmost columns must appear in the WHERE clause; a gap in the prefix stops the index from helping further |
| What makes an index a covering index? | The index contains all columns referenced in the query (both WHERE conditions and SELECT list) — the DB can answer entirely from the index without reading the main table |
| How do you choose column order in a composite index? | Equality-condition columns first (highest selectivity first among equals); range-condition column last; include ORDER BY column if it follows the equality columns |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
