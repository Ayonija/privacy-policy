# Day 04 — Sliding Window: Frequency Map Variant
**Week 01 | Phase 1: DSA Mastery | Month 1**

## Focus
Master the sliding window + frequency map combination — critical for substring matching problems at Medium/Hard level.

## DSA (2 hours)
### Pattern: Sliding Window + Character Frequency Map
- Maintain a frequency map of the current window; shrink left when the window violates the constraint (e.g., has more distinct chars than allowed or a char count is wrong).
- Trigger condition: "find substring/subarray where character distribution matches or fits within a budget."
- Time complexity: O(n) | Space complexity: O(alphabet size) — O(26) for lowercase English

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Maximum Average Subarray I | 643 | Easy | Fixed Sliding Window | Fixed window k; slide by adding arr[right] and subtracting arr[left] without recomputing sum |
| 2 | Longest Repeating Character Replacement | 424 | Medium | Sliding Window + Freq Map | Window is valid when `window_size − max_freq ≤ k`; max_freq only needs to increase |
| 3 | Permutation in String | 567 | Medium | Fixed Sliding Window + Freq Map | Fixed window size = len(s1); compare freq maps; use a `matches` counter instead of full map comparison |

### Code Skeleton
```java
class Solution {
    public static int longestRepeatingReplacement(String s, int k) {
        Map<Character, Integer> count = new HashMap<>();
        int left = 0;
        int maxFreq = 0;   // max frequency of any single character in the window
        int result = 0;
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            count.put(c, count.getOrDefault(c, 0) + 1);
            maxFreq = Math.max(maxFreq, count.get(c));
            // window is invalid when (window_size - maxFreq) > k replacements needed
            while ((right - left + 1) - maxFreq > k) {
                char leftChar = s.charAt(left);
                count.put(leftChar, count.get(leftChar) - 1);
                left++;
            }
            result = Math.max(result, right - left + 1);
        }
        return result;
    }
}
```

### Interview Tips

- **Explain the validity condition first:** "A window is valid if we need at most k replacements — that's `window_size - max_frequency_char ≤ k`." State this in one sentence before touching code; it's the key insight.
- **The `maxFreq` trick:** `maxFreq` can only increase (it's a running maximum) — we never need to recalculate it on shrink because we're looking for the *longest* valid window. Explain this reasoning to the interviewer; it sounds like a bug but isn't.
- **`matches` counter for LC 567:** instead of comparing two 26-element arrays every step (O(26) per slide), maintain a `matches` counter that tracks how many characters have matching frequencies — O(1) per slide.
- **Brute force baseline:** "O(n² · 26) checking all substrings and counting character frequencies" → sliding window + freq map reduces to O(n).
- **Common mistake:** for LC 567 (Permutation in String), forgetting that the window size is *fixed* at `len(s1)` — you always slide by one, never grow or shrink; it's not a variable window.

### STAR Interview Framework

> **How to use the STAR method when explaining Sliding Window + Character Frequency Map in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given a string and an integer k, and asked to find the length of the longest substring that can be made into a single repeating character by replacing at most k characters. The brute-force approach of checking all O(n²) substrings and computing character frequencies for each is O(n²·26), which for n = 10⁵ means ~2.6×10¹¹ operations — clearly infeasible."

**Task:** "My goal was to solve this in O(n) time and O(26) = O(1) space by recognising that the 'valid window' condition has a clean mathematical form I can check in O(1) per step by maintaining a running frequency map."

**Action:** Walk the interviewer through these steps (this is where you spend most time):
1. *Classify the pattern:* "I noticed I need the longest substring satisfying an internal constraint — the number of characters to replace. Internal state (frequency counts) → sliding window + frequency map."
2. *Initialize:* "I set `left = 0`, `maxFreq = 0`, and a HashMap `count`. The window is valid when `window_size - maxFreq <= k` — the number of non-dominant characters must be within our replacement budget."
3. *Core loop logic:* "For each `right`, I add `s[right]` to `count` and update `maxFreq = max(maxFreq, count[s[right]])`. If `(right - left + 1) - maxFreq > k`, I shrink by removing `s[left]` from `count` and `left++`. Then I record `right - left + 1` as a candidate."
4. *Convergence guarantee:* "`maxFreq` only ever increases (we want the longest window, so a smaller maxFreq can't extend our best result). This means we shrink by at most 1 per iteration — the window size never decreases, just slides forward."
5. *Duplicate handling / edge case proactivity:* "The `maxFreq` trick is the key subtlety: I never decrease `maxFreq` after a shrink. This is intentional — not a bug — because the window can only get longer if it finds a higher `maxFreq` in the future."

**Result:** "This reduces time complexity from O(n²·26) to O(n). For n = 10⁵, brute force takes ~2.6×10¹¹ operations (~4+ minutes); the sliding window completes in ~2×10⁵ operations (~0.2ms). The `maxFreq` monotonicity argument is the senior-level insight that separates this from a naive sliding window solution."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer Sliding Window + Freq Map here |
|-------------|---------------------------|-------------------------------|
| Brute force — all substrings O(n²·26) | When n ≤ 200 | Times out at n = 10⁵; sliding window is O(n) |
| Binary search on answer length O(n log n) | When you can validate a given length efficiently | Works but adds O(log n) factor; sliding window achieves O(n) directly and is standard |

**Why NOT brute force:** Checking all O(n²) substrings and counting chars in each is O(n²·26) — for n = 10⁵ that's ~2.6×10¹¹ operations, far too slow.
**Why NOT binary search on length:** While binary search on the answer works here (check if a window of size L is valid in O(n)), it's O(n log n) vs O(n) for sliding window — no advantage and more code.

### Edge Cases to Trace Before Coding
- LC 424: `k = 0` → any window with more than one distinct character is invalid; answer = longest run of one character
- LC 424: all same characters → `maxFreq = window_size`, condition never violated; answer = `s.length()`
- LC 567: `len(s1) > len(s2)` → impossible, return false immediately
- LC 643 (Average Subarray): `k > len(nums)` → invalid; check constraints say this won't happen but good to mention

## System Design (1 hour)
### Topic: Big-O Applied — Space vs. Time Trade-offs
- Trading space for time is a core system design move: precomputed indexes, caches, and hash maps all pay upfront space to reduce query time.
- A frequency map turns O(n·k) substring comparison into O(1) comparison by maintaining state incrementally.
- In distributed systems the same principle applies: a denormalised read replica pays write cost to serve reads in O(1) vs. a normalised DB that requires joins.
- Beware hidden space costs: a 26-bucket array is O(1) space; a map keyed by arbitrary Unicode strings is O(n) space.
- Interview talking point: "If asked how to check two strings are anagrams in O(1) space, answer: use a fixed 26-element integer array as a frequency diff counter; it's O(1) because the array size is bounded by the alphabet, not the input."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)

**Leadership Principle:** Deliver Results

**STAR Story: Caching Intermediate Frequency Counts to Eliminate Redundant Computation**

**Situation (20%):** "At my previous company, we ran a daily content moderation pipeline that scanned comment threads for anomalous character pattern distributions — a signal for spam bots. The pipeline re-computed character frequency distributions for every sliding 50-message window by re-reading all 50 messages from scratch. For threads with 10K+ messages, this made each thread take 8–12 minutes, and we had 50K threads to process daily."

**Task (part of S/T):** "I owned the moderation pipeline's throughput. My goal was to process all 50K threads within a 2-hour daily window, down from the current 6–8 hours, without increasing server costs."

**Action (60-70% — be specific about what YOU did):**
"First, I benchmarked the pipeline and confirmed that re-computing the frequency map from scratch on each window slide was 97% of the CPU cost.
Then, I redesigned it to maintain an incremental frequency map — on each slide, I added the new message's characters and subtracted the evicted message's characters, exactly like the sliding window + freq map approach. I also introduced a `matches` counter tracking how many character slots met the anomaly threshold, so validation was O(1) per step instead of O(26).
Next, I ran the new pipeline on a 1K-thread sample and validated that it produced identical output to the old pipeline while running 60× faster.
Finally, I replaced the production pipeline with a gradual rollout over 3 days, monitoring output correctness against a shadow run of the old pipeline."

**Result (10-20%):** "Processing time for the full 50K-thread daily run dropped from 6.5 hours to 40 minutes — an 89% reduction. We could now run the pipeline 3× per day instead of once, catching spam patterns 8 hours earlier on average. Server costs stayed flat because we eliminated the redundant computation, not added machines."

**Interview tip:** Ground the story in concrete before/after numbers. "I redesigned the freq map update to be O(1) per slide" is more credible than "we made it faster." Prepare for questions about: delivering results, optimization, pipeline design, or ownership.

## Flashcards

| Q | A |
|---|---|
| How do you check if a fixed-size window is an anagram of s1 in O(1) per step? | Maintain a `matches` counter; increment when a char's freq in window equals s1, decrement when it diverges; answer when matches == 26 |
| Why can max_freq in LC 424 only increase and never need to decrease? | You're looking for the longest valid window; if max_freq would decrease, the window can only stay the same size — it never needs to shrink below the current best |
| What is the validity condition for the window in LC 424? | `window_length − max_freq ≤ k` — the number of non-dominant characters must be ≤ k (replacements available) |
| How does a fixed sliding window differ from a variable one? | Fixed: window size is constant (k elements); slide by removing leftmost and adding new right. Variable: window grows/shrinks based on a constraint |
| Name a real-world system design analogue of the frequency map trick. | A denormalised read replica or precomputed aggregate table: pay extra write cost to make reads O(1) |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
