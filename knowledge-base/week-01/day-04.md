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
```python
def longest_repeating_replacement(s, k):
    count = {}
    left = 0
    max_freq = 0   # max frequency of any single character in the window
    result = 0
    for right in range(len(s)):
        count[s[right]] = count.get(s[right], 0) + 1
        max_freq = max(max_freq, count[s[right]])
        # window is invalid when (window_size - max_freq) > k replacements needed
        while (right - left + 1) - max_freq > k:
            count[s[left]] -= 1
            left += 1
        result = max(result, right - left + 1)
    return result
```

### Interview Tips

- **Explain the validity condition first:** "A window is valid if we need at most k replacements — that's `window_size - max_frequency_char ≤ k`." State this in one sentence before touching code; it's the key insight.
- **The `max_freq` trick:** `max_freq` can only increase (it's a running maximum) — we never need to recalculate it on shrink because we're looking for the *longest* valid window. Explain this reasoning to the interviewer; it sounds like a bug but isn't.
- **`matches` counter for LC 567:** instead of comparing two 26-element arrays every step (O(26) per slide), maintain a `matches` counter that tracks how many characters have matching frequencies — O(1) per slide.
- **Brute force baseline:** "O(n² · 26) checking all substrings and counting character frequencies" → sliding window + freq map reduces to O(n).
- **Common mistake:** for LC 567 (Permutation in String), forgetting that the window size is *fixed* at `len(s1)` — you always slide by one, never grow or shrink; it's not a variable window.

### Edge Cases to Trace Before Coding
- LC 424: `k = 0` → any window with more than one distinct character is invalid; answer = longest run of one character
- LC 424: all same characters → `max_freq = window_size`, condition never violated; answer = `len(s)`
- LC 567: `len(s1) > len(s2)` → impossible, return False immediately
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
- STAR prompt: Tell me about a time you optimised a repeated computation by caching intermediate results — analogous to maintaining a running frequency map instead of recomputing from scratch.
- Leadership principle: Deliver Results

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
