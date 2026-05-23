# Day 79 — Tries: XOR Trie, Word Encoding, and Palindrome Pairs
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Maximum XOR of Two Numbers uses a binary Trie to greedily find the number that maximises XOR bit-by-bit. Short Encoding of Words determines which words need their own encoded slot — equivalently, which reversed words aren't suffixes of any other. Palindrome Pairs enumerates all pairs (i, j) where `words[i] + words[j]` is a palindrome using a HashMap (or Trie) plus palindrome checking.

---

## DSA (2 hours)
### Pattern: Binary XOR Trie + Suffix Trie Deduplication + Palindrome Pair HashMap

**Maximum XOR of Two Numbers in an Array (LC 421):**
For each pair (a, b), XOR is maximised by choosing opposite bits at every position. Binary Trie: insert each number as a 30-bit binary string (MSB first). For each number `a`, traverse the Trie choosing the OPPOSITE bit at each level — if that child exists, we can achieve a 1-bit at that position. This greedily maximises XOR from MSB to LSB.

**Short Encoding of Words (LC 820):**
Encoding: words joined by `#`, terminated by `#`. A word that IS a suffix of another word doesn't need its own segment — it fits inside the longer word's segment. Find words that are NOT suffixes of any other word; sum their lengths + 1 (for `#`).

Approach: insert all REVERSED words into a Trie. A word that is a suffix of another → its reversed form is a prefix of another reversed form → it won't be a leaf in the Trie. Count the total length contributed by leaf nodes only.

Simpler approach: put all words in a set; for each word, try removing it if it's a suffix of another word. Words remaining = independent encodings.

**Palindrome Pairs (LC 336):**
For words list, find all pairs (i, j) where `words[i] + words[j]` is a palindrome. For `concat = words[i] + words[j]` to be a palindrome:
- Case 1: `len(words[i]) == len(words[j])` and `words[j] == reverse(words[i])`
- Case 2: `len(words[i]) > len(words[j])` — prefix of words[i] matches reverse(words[j]) and the remaining suffix of words[i] is a palindrome
- Case 3: `len(words[i]) < len(words[j])` — symmetric case

Algorithm: build HashMap `{word: index}`. For each word w at index i, split w at every position:
- Split `w = prefix + suffix`: if `prefix` is palindrome and `reverse(suffix)` is in HashMap (and not same index) → valid pair (HashMap index, i)
- Split `w = prefix + suffix`: if `suffix` is palindrome and `reverse(prefix)` is in HashMap (and not same index) → valid pair (i, HashMap index)
Also: if `reverse(w)` is in HashMap and `reverse(w) != w` → pair (i, rev_idx) and (rev_idx, i)

**Trigger condition:**
- "maximum XOR of any two numbers in an array" → binary Trie; for each number, traverse choosing opposite bit
- "minimum encoding length for a list of words using suffix sharing" → reversed-word Trie leaf count; or suffix set deduplication
- "all pairs whose concatenation is a palindrome" → HashMap of reversed words; check all splits of each word for palindrome prefix/suffix

**Time complexity:** LC 421: O(n × 30) = O(n) | LC 820: O(n × L) | LC 336: O(n × L²)
**Space complexity:** O(n × 30) / O(n × L) / O(n × L)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Maximum XOR of Two Numbers | 421 | Medium | Binary Trie; greedy opposite-bit traversal | Insert all numbers; for each, traverse choosing opposite bit; XOR = bits chosen |
| 2 | Short Encoding of Words | 820 | Medium | Reversed-word Trie leaves; or suffix dedup | A word not a suffix of any other → independent; reversed Trie: leaves contribute length+1 |
| 3 | Palindrome Pairs | 336 | Hard | HashMap of reversed words + split enumeration | For each split: check if prefix/suffix is palindrome + if complement exists in HashMap |

---

### Code Skeleton
```python
# Maximum XOR of Two Numbers in an Array (LC 421)
class BinaryTrieNode:
    def __init__(self):
        self.children = [None, None]   # children[0] = bit 0, children[1] = bit 1

def findMaximumXOR(nums):
    root = BinaryTrieNode()
    # Insert all numbers into binary Trie (30 bits, MSB first)
    for num in nums:
        node = root
        for bit in range(29, -1, -1):
            b = (num >> bit) & 1
            if not node.children[b]:
                node.children[b] = BinaryTrieNode()
            node = node.children[b]
    # For each number, traverse choosing opposite bit to maximise XOR
    max_xor = 0
    for num in nums:
        node = root
        xor = 0
        for bit in range(29, -1, -1):
            b = (num >> bit) & 1
            opposite = 1 - b
            if node.children[opposite]:
                xor |= (1 << bit)   # we can achieve a 1-bit here
                node = node.children[opposite]
            else:
                node = node.children[b]
        max_xor = max(max_xor, xor)
    return max_xor

# Short Encoding of Words (LC 820) — set-based approach
def minimumLengthEncoding(words):
    word_set = set(words)
    # Remove any word that is a suffix of another word in the set
    for word in words:
        for i in range(1, len(word)):
            word_set.discard(word[i:])   # remove suffix from set
    return sum(len(w) + 1 for w in word_set)   # +1 for '#'

# Palindrome Pairs (LC 336)
def palindromePairs(words):
    def is_palindrome(s): return s == s[::-1]
    word_map = {word: i for i, word in enumerate(words)}
    result = []
    for i, word in enumerate(words):
        n = len(word)
        for j in range(n + 1):
            prefix, suffix = word[:j], word[j:]
            # Case 1: prefix is palindrome; reverse(suffix) exists as another word
            rev_suffix = suffix[::-1]
            if is_palindrome(prefix) and rev_suffix in word_map and word_map[rev_suffix] != i:
                result.append([word_map[rev_suffix], i])
            # Case 2: suffix is palindrome; reverse(prefix) exists; j > 0 avoids duplicates
            if j > 0 and is_palindrome(suffix):
                rev_prefix = prefix[::-1]
                if rev_prefix in word_map and word_map[rev_prefix] != i:
                    result.append([i, word_map[rev_prefix]])
    return result
```

---

### STAR Interview Framework

> **How to use the STAR method when explaining Binary XOR Trie / Suffix Trie Deduplication / Palindrome Pair HashMap in an interview.**
> *Time allocation: 20% on S+T, 60-70% on A, 10-20% on R.*

**Situation:** "I was given three problems that extend Trie thinking to bit-level operations and string structural analysis: Maximum XOR of Two Numbers (finding the pair with the largest XOR), Short Encoding of Words (minimum encoding length via suffix sharing), and Palindrome Pairs (all pairs whose concatenation is a palindrome). Brute-force XOR checking all pairs is O(n²), scanning all suffix relationships is O(n² × L), and palindrome checking all pairs is O(n² × L)."

**Task:** "My goal was to reduce each to O(n) or O(n × L) by exploiting structure: bit-level greedy traversal in a binary Trie for XOR, suffix set deduplication for encoding, and HashMap lookup with split enumeration for palindrome pairs."

**Action:** Walk the interviewer through these steps:
1. *Classify the pattern:* "Maximum XOR → binary Trie (30-bit depth, children indexed 0/1); for each number, greedily traverse choosing the OPPOSITE bit to maximise XOR. Short Encoding → reversed-word suffix deduplication: a word needing its own encoding slot is one not a suffix of any other word. Palindrome Pairs → HashMap of `{word: index}`; for each word, split at every position and check if the complement exists."
2. *Initialize:* "Binary XOR Trie: insert all numbers as 30-bit binary strings (MSB first). Suffix dedup: build a set of all words, then remove all non-empty suffixes of each word. Palindrome Pairs HashMap: `word_map = {word: idx for idx, word in enumerate(words)}`."
3. *Core loop logic:* "XOR Trie: for each number `a`, traverse 30 bits from MSB; at each level, try `opposite = 1 - b`; if that child exists, set that bit in XOR result and follow it. Palindrome split: for word `w`, split at position `j`; if `prefix` is palindrome and `reverse(suffix)` is in HashMap → valid pair `(HashMap[rev_suffix], i)`."
4. *Convergence guarantee:* "XOR Trie terminates after exactly 30 levels per number. The palindrome split enumerates O(L) positions per word, and each check is O(L) for palindrome verification — total O(n × L²) which is the theoretical lower bound for this problem."
5. *Duplicate handling / edge case proactivity:* "Palindrome Pairs: the split at position 0 (empty prefix, full word as suffix) duplicates the same word pair from the split at position n (full word as prefix, empty suffix). Adding `j > 0` to the second case avoids duplicate pairs — this is the most common bug in this problem."

**Result:** "XOR Trie: O(n × 30) = O(n) vs O(n²) brute force — for n = 10^5 numbers, that's 3 × 10^6 vs 10^10 operations. Suffix encoding: O(n × L²) worst case but O(n × L) in practice when word lengths are bounded. Palindrome Pairs: O(n × L²) vs O(n² × L) naive — for n = 10^3 words of average length L = 10, that's 10^5 vs 10^7 operations."

---

**Alternative Approaches & Trade-offs**

| Alternative | When you might consider it | Why prefer this approach |
|-------------|---------------------------|--------------------------|
| Brute force all pairs for XOR | n ≤ 1000 | O(n²) = 10^6 — fine; Trie gives O(n) — use for n ≥ 10^4 |
| Reversed-word Trie for suffix dedup | Very large dictionary with many shared suffixes | Trie leaf counting vs set approach is equivalent O(n × L); set approach is simpler to implement |
| Check all O(n²) pairs for palindromes | n ≤ 50 | O(n² × L) fine for small n; HashMap split is needed for n ≥ 500 |

**Why NOT brute force XOR at scale:** O(n²) for n = 10^5 is 10^10 operations — will time out. Binary Trie is O(30n) = 3 × 10^6 operations.
**Why NOT brute force palindrome pairs:** O(n² × L) for n = 10^3, L = 10 is 10^7 — borderline. For n = 5000, it's 2.5 × 10^8 — times out. HashMap split enumeration is O(n × L²) = 5 × 10^5 — comfortably within limits.

---

### Edge Cases to Trace Before Coding
- LC 421: all same number → XOR with itself = 0; two numbers → result = XOR of the two
- LC 820: words list has duplicates → set deduplication; single word → length + 1; word is suffix of itself (impossible since no word is suffix of itself with strict suffix `i > 0`)
- LC 336: empty string in words → "" is palindrome; concatenation "a" + "" = "a" is palindrome → ["a", ""] is a valid pair; self-pair not allowed (i != j); duplicate words → word_map maps to ONE index, use enumerate carefully

---

### Interview Pattern Drill

| XOR Trie | Standard Trie | Difference |
|----------|--------------|-----------|
| Children indexed by bits (0 and 1) | Children indexed by chars (a-z or dict) | Bit-level traversal |
| Depth = 30 (bit width) | Depth = max word length | Depth = numeric precision |
| Greedy: prefer opposite bit | Exact match or prefix match | Goal: maximise XOR |

**Palindrome pair enumeration — why split at every position:**
`words[i] + words[j]` is palindrome iff the combination reads the same forward and backward. Instead of checking all O(n²) pairs, we fix `words[i]` and look up the complement in the HashMap. A split of `words[i]` into prefix/suffix lets us handle different length cases: if the suffix is palindrome and `reverse(prefix)` is in the HashMap, then `words[i] + words[j]` = `prefix + suffix + reverse(prefix)` = palindrome.

---

## System Design (1 hour)
### Topic: Newsfeed — Data Pipeline, Trending Topics, and Recommendation Engine

**Event pipeline for analytics:**
Every user action (view, like, share, click, scroll-past) emits an event:
```
{user_id, post_id, action, timestamp, session_id}
    │
Kafka topic "user_events"
    │
[Flink Stream Processor]
    ├─ Real-time aggregation (last 5 min)
    └─ Write to ClickHouse (OLAP) for analytics
```

**Trending topics computation:**
Trending = topics/hashtags with the most mentions in a sliding time window.

Algorithm: min-heap of top-K trending topics.
- Count hashtag occurrences in a 1-hour sliding window using Flink
- Maintain a top-K min-heap (Slot 8 Day 71 pattern!): push (count, hashtag); pop when heap size > K
- Push result to Trending Topics Redis sorted set every 5 minutes
- Feed Service reads trending topics for "Explore" and "Discover" feeds

```python
# Flink-like pseudocode (applies Day 71 Top-K pattern)
trending_heap = []
for hashtag, count in window_counts.items():
    heapq.heappush(trending_heap, (count, hashtag))
    if len(trending_heap) > K:
        heapq.heappop(trending_heap)
top_k_trending = [(count, tag) for count, tag in trending_heap]
```

**Recommendation engine — "Who to Follow":**
Collaborative filtering: if A follows B and B follows C, C might be interesting to A.
- Graph traversal: 2-hop neighbourhood of A (A's followees' followees)
- Filter: exclude users A already follows or has dismissed
- Rank: by mutual follow count (how many of A's followees also follow C)
- Compute offline in batch (Spark job, nightly); store recommendations in Redis

**Recommendation engine — "You Might Like" (content recommendations):**
- TF-IDF vectors for each post (keyword importance)
- Cosine similarity between posts and user's engagement history
- Served via approximate nearest neighbour (ANN) search using FAISS or Pinecone
- Updated every hour for active users

**A/B testing infrastructure:**
- Feature flags in LaunchDarkly or custom flag service
- Each user assigned to an experiment bucket (hash of user_id % 100)
- Metrics: engagement rate, CTR, session length, unfollow rate
- Experiment results in ClickHouse; significance computed by data science team

**Interview talking point:** "Trending topics is the Top-K streaming problem at scale. We maintain a 1-hour sliding window count of hashtag mentions using Flink's event-time processing. Every 5 minutes, we update the Trending sorted set in Redis. The min-heap pattern (size K) efficiently maintains top-K without sorting all hashtags on each update. The challenge is normalising for trending: a hashtag with 1M mentions from bots is different from one with 100K organic mentions — we apply spam filters before counting."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 421 Maximum XOR of Two Numbers (target: 20 min)
- **Medium 2:** LC 820 Short Encoding of Words (target: 15 min)
- **Hard:** LC 336 Palindrome Pairs (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)

**Leadership Principle:** Customer Obsession

**STAR Story: Building a Personalized Content Recommendation System That Customers Could Trust**

**Situation:** At my previous company, we operated a developer documentation platform where users regularly struggled to find relevant guides among 40,000+ articles. Search was keyword-only — users had to know the exact terms. Our NPS scores for content discovery sat at 22, and exit surveys consistently cited "couldn't find what I needed" as the top frustration. Engagement with documentation beyond the first result was below 8% — users landed on one page and left.

**Task:** I was asked to design and ship a content recommendation system — a "Related Articles" module that would appear at the bottom of each page. The business goal was to increase documentation engagement depth (pages per session) by at least 30%. My constraint: the system had to be interpretable. Developers distrust black-box recommendations and will ignore them if they can't understand why something was surfaced.

**Action:**

*First,* I audited the data we had: article text, user session logs (page sequences), and explicit signals (time on page, copy-code events). I prototyped three approaches: (1) co-occurrence counting (articles read in the same session), (2) TF-IDF cosine similarity between article texts, and (3) a hybrid scoring combining both signals with a recency decay. I rejected neural embedding approaches — we had no GPU inference infrastructure and they would have been uninterpretable to users.

*Then,* I implemented the co-occurrence matrix using a sliding window over sessions: for each pair of articles (A, B) appearing in the same session within 5 hops of each other, I incremented `co_occurrence[A][B]`. I normalised by the Jaccard coefficient to avoid high-traffic pages dominating every recommendation. I ran this as a nightly Spark job on our session logs.

*Next,* I combined co-occurrence with TF-IDF similarity: `score(A, B) = 0.6 × co_occurrence_jaccard(A, B) + 0.4 × cosine_similarity(A, B)`. The split was determined by a grid search on a holdout session set, optimising for click-through on the "Related Articles" panel. I stored pre-computed top-10 recommendations per article in a Redis hash (article_id → JSON list), refreshed nightly.

*Finally,* I addressed interpretability explicitly. Each recommendation card displayed a one-line reason: "Developers also read this after [current article title]" (co-occurrence dominant) or "This covers a related concept: [top shared TF-IDF term]" (similarity dominant). I A/B tested the module with and without the reason labels — the reason labels increased click-through by 34% by themselves, even before the recommendation quality improvements.

**Result:** After 6 weeks of rollout, documentation pages-per-session increased from 1.3 to 2.1 (62% improvement — well above the 30% target). Click-through on the Related Articles panel reached 19% (from 0% baseline). NPS for content discovery improved from 22 to 41. Critically, the interpretability labels drove a 34% higher CTR than recommendations without labels, validating the developer trust hypothesis. The system cost $0 incremental in compute — the nightly Spark job ran on existing infrastructure.

*In an interview, say:* "I'd describe it this way: The customer feedback was clear — developers were leaving frustrated because they couldn't find what they needed. I chose an interpretable hybrid recommendation approach over a black-box model specifically because developer users are skeptical of opaque systems. The 'why am I seeing this' label ended up being as important as the algorithm itself." Use this for "Tell me about a time you obsessed over the customer experience," "Tell me about a data-driven decision," or "Tell me about a time you made a technical trade-off to serve the user better."

---

## Flashcards

| Q | A |
|---|---|
| How does the XOR Trie greedily maximise XOR? | For each number `a`, traverse the Trie. At each bit level, try to go to the OPPOSITE bit child (to get a 1-bit in the XOR). If opposite child exists, take it and set that bit in the result; otherwise follow the same bit (0 in the result). |
| How does the set-based Short Encoding approach identify independent words? | Put all words in a set. For each word, remove all its non-empty suffixes from the set. Remaining words in the set are NOT suffixes of any other word → each needs its own `word#` encoding. Sum `len(w) + 1` for all. |
| How does Palindrome Pairs enumerate all pairs without O(n²) brute force? | For each word w[i], split at every position j: if `prefix` is palindrome and `reverse(suffix)` is in HashMap → pair `(rev_suffix_idx, i)`; if `suffix` is palindrome and `reverse(prefix)` is in HashMap → pair `(i, rev_prefix_idx)`. O(n × L²) total. |
| How does the newsfeed compute trending topics? | Sliding window hashtag counts in Flink; min-heap of size K (Top-K pattern from Day 71); push (count, hashtag), pop when size > K; write top-K to Redis sorted set every 5 min. |
| What is collaborative filtering for "Who to Follow" recommendations? | 2-hop graph traversal from user A: find A's followees' followees; exclude already-followed users; rank by mutual follow count; serve from Redis. Computed offline (Spark nightly batch); stored as pre-ranked suggestions. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 20/15/30 min, no hints)
- [ ] Rewrote XOR Trie greedy traversal and palindrome pair split enumeration from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain trending topics using Top-K heap pattern cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
