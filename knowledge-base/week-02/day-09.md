# Day 09 — Sliding Window: Advanced & Real-World Variants
**Week 02 | Phase 1: DSA Mastery | Month 1**

## Focus
Tackle sliding window problems where the optimal window is non-obvious — including "boost window" and card-selection patterns.

## DSA (2 hours)
### Pattern: Sliding Window — Complement & Flip Thinking
- Grumpy Bookstore Owner (boost window): instead of maximising the window directly, compute the baseline and maximise the *gain* from the boost window.
- Maximum Points from Cards: taking k cards from the ends is equivalent to leaving n-k cards in the middle; minimise the middle window to maximise the ends.
- Trigger condition: "you have a one-time modifier/boost" OR "select from both ends" — both reduce to a fixed sliding window on a transformed problem.
- Time complexity: O(n) | Space complexity: O(1)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Max Consecutive Ones | 485 | Easy | Simple scan | No window needed — just track current streak and max streak; reset on 0 |
| 2 | Grumpy Bookstore Owner | 1052 | Medium | Fixed Sliding Window (gain maximisation) | Always count non-grumpy minutes; then slide window of size `minutes` to find max extra from suppressing grumpiness |
| 3 | Maximum Points You Can Obtain from Cards | 1423 | Medium | Fixed Sliding Window (complement) | Taking k cards from ends = leaving n-k in the middle; minimise the sum of the middle window of size n-k |

### Code Skeleton
```python
# Grumpy Bookstore Owner (LC 1052)
def max_satisfied(customers, grumpy, minutes):
    # Step 1: baseline — always-satisfied customers (where grumpy[i] == 0)
    base = sum(c for c, g in zip(customers, grumpy) if g == 0)
    # Step 2: slide a window of size `minutes` to maximise extra gained from suppression
    gain = window = sum(customers[i] * grumpy[i] for i in range(minutes))
    for i in range(minutes, len(customers)):
        window += customers[i] * grumpy[i]
        window -= customers[i - minutes] * grumpy[i - minutes]
        gain = max(gain, window)
    return base + gain

# Maximum Points from Cards (LC 1423)
def max_score(cardPoints, k):
    n = len(cardPoints)
    window_size = n - k    # cards we are NOT taking = middle window
    if window_size == 0:   # taking all cards
        return sum(cardPoints)
    window_sum = sum(cardPoints[:window_size])
    min_middle = window_sum
    for i in range(window_size, n):
        window_sum += cardPoints[i] - cardPoints[i - window_size]
        min_middle = min(min_middle, window_sum)
    return sum(cardPoints) - min_middle
```

### Interview Tips

- **The complement/reframe trick:** when you can't directly optimise the selection you want (k cards from both ends), reframe it as optimising the complement (leave n-k cards in the middle). Mention this reframing explicitly — it's the insight the interviewer is testing.
- **Two-step structure for LC 1052:** "Step 1: compute baseline (sum of non-grumpy minutes). Step 2: slide a window to find maximum extra from the boost." Narrate both steps before coding — the interviewer wants to see structured thinking.
- **Brute force for LC 1423:** O(k²) trying all splits between left-end count (0 to k) and right-end count (k-0 to 0) → complement window is O(n).
- **Common mistake:** in LC 1423, including `window_size = 0` without a guard — `sum(cardPoints[:0])` is 0, which gives the right answer, but `range(0, n)` would re-process all cards and corrupt `min_middle`. Add the guard or trace carefully.
- **Alternative for LC 1052:** instead of tracking `customers[i] * grumpy[i]`, directly subtract satisfied-during-boost from the full-satisfied count — same result, different framing.

### Edge Cases to Trace Before Coding
- LC 1052: `minutes >= len(customers)` → boost covers everything; answer = total customers (no baseline subtraction needed)
- LC 1423: `k = n` → take all cards; return `sum(cardPoints)` (window_size = 0)
- LC 1423: `k = 0` → take no cards; return 0 (window is the entire array, min_middle = total)
- LC 485: all 1s → single scan returns `len(nums)`; all 0s → return 0

## System Design (1 hour)
### Topic: Sliding Window in Real Systems — Rate Limiters & Monitoring
- Fixed-window rate limiter: count requests per fixed time bucket (e.g., per minute); simple but has boundary burst problem.
- Sliding-window rate limiter: maintain a deque of timestamps; evict expired entries; allows exactly k requests per any rolling window of size T.
- Token bucket: replenish tokens at a fixed rate; allows bursts up to bucket capacity. Used by AWS API Gateway.
- Monitoring: moving averages on metrics (CPU %, error rate) are sliding window sums — same algorithm, applied to time-series data.
- Interview talking point: "If asked the difference between fixed-window and sliding-window rate limiters, answer: fixed-window can allow 2x the rate limit at bucket boundaries (e.g., 100 at 0:59 + 100 at 1:00); sliding-window guarantees at most k requests in any T-second period."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Give an example where you reframed a problem — instead of directly maximising what you wanted, you minimised what you didn't want — analogous to the complement window in LC 1423.
- Leadership principle: Think Big

## Flashcards

| Q | A |
|---|---|
| How do you solve Maximum Points from Cards without simulating taking from ends? | Compute total sum; find minimum subarray sum of length n-k (the cards you leave); answer = total − min_middle |
| What is the "gain window" in Grumpy Bookstore Owner? | The extra customers you gain if grumpiness is suppressed during a window of size `minutes`; you slide this window to find where it covers the most grumpy-lost customers |
| What is the sliding-window rate limiter invariant? | At any moment, the deque contains only timestamps within the last T seconds; size of deque ≤ k |
| How does the complement technique apply to sliding window problems? | When taking from both ends, equivalently minimise the middle segment; converts an irregular two-end selection into a simple single window |
| What is the boundary burst problem in fixed-window rate limiters? | A client can send k requests just before a window boundary and k more just after, achieving 2k requests in a very short real-time span |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
