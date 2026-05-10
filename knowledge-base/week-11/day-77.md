# Day 77 — Tries: Longest Dictionary Word, Wildcard Search, and Prefix+Suffix Search
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Longest Word in Dictionary finds the deepest Trie node reachable only through words (every prefix is also a word). Design Add and Search Words extends the Trie with '.' wildcard matching — triggering DFS over all children at a wildcard node. Prefix and Suffix Search builds a specialised Trie by inserting all suffix-wrapped words, enabling combined prefix AND suffix queries in O(L).

---

## DSA (2 hours)
### Pattern: Trie BFS/DFS on Word-Only Paths + Wildcard Trie DFS + Wrapped Suffix Trie

**Longest Word in Dictionary (LC 720):**
Find the longest word in `words` such that every prefix of it is also a word in `words`. Insert all words into a Trie, marking each word's endpoint. BFS/DFS from the root, but only traverse edges where the child node has `is_end = True` (meaning every prefix up to that point is a valid word). The longest path found this way is the answer; break ties by lexicographic order.

**Design Add and Search Words Data Structure (LC 211):**
Trie with `addWord(word)` and `search(word)`. The '.' character in a search pattern matches any letter. Implementation: for a literal char → traverse normally; for '.' → recursively try all children. DFS handles nested wildcards. If the pattern is all '.' → DFS over all children at each level.

**Prefix and Suffix Search (LC 745):**
Given a list of words, design a data structure that answers `f(prefix, suffix)` → index of the word in the list with that prefix AND suffix (latest index if multiple).

Technique: For each word `w` at index `i`, insert all strings of the form `"{suffix}#{w}"` into a Trie for all suffixes of `w`, including the empty suffix `"#w"`. To answer `f(prefix, suffix)`: look up `"{suffix}#{prefix}"` in the Trie. Each node stores the latest index of a word that passes through it. The node at the end of the query string holds the answer.

Alternative: HashMap from `(prefix, suffix)` to index — O(n × L²) space for precomputation; O(1) query.

**Trigger condition:**
- "longest word where every prefix is also a valid word" → Trie; BFS only on is_end nodes; or sort + check prefixes with HashSet
- "pattern matching with '.' as single-character wildcard" → Trie DFS; branch on all children at '.' node
- "find word matching both prefix AND suffix simultaneously" → suffix-wrapped Trie with `"{suffix}#{prefix}"` lookup

**Time complexity:** LC 720: O(total_chars) | LC 211: O(L) addWord; O(26^L) worst search | LC 745: O(n×L²) build; O(L) query
**Space complexity:** O(total_chars) / O(total_chars) / O(n × L²)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Longest Word in Dictionary | 720 | Medium | Trie BFS on is_end-only paths | Only traverse edges where child.is_end = True; longest path = answer |
| 2 | Design Add and Search Words | 211 | Medium | Trie DFS with wildcard '.' | Literal → normal traverse; '.' → try all children recursively |
| 3 | Prefix and Suffix Search | 745 | Hard | Suffix-wrapped Trie with index propagation | Insert `"{suffix}#{word}"` for all suffixes; query `"{suffix}#{prefix}"` |

---

### Code Skeleton
```python
from collections import deque

# Longest Word in Dictionary (LC 720)
def longestWord(words):
    trie = Trie()   # using the Trie class from Day 76
    for word in words:
        trie.insert(word)
    best = ""
    # BFS: only traverse children where is_end = True
    queue = deque([(trie.root, "")])
    while queue:
        node, current_word = queue.popleft()
        for ch, child in node.children.items():
            if child.is_end:
                new_word = current_word + ch
                if len(new_word) > len(best) or (len(new_word) == len(best) and new_word < best):
                    best = new_word
                queue.append((child, new_word))
    return best

# Design Add and Search Words (LC 211)
class WordDictionary:
    def __init__(self):
        self.root = TrieNode()

    def addWord(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        def dfs(node, i):
            if i == len(word): return node.is_end
            ch = word[i]
            if ch == '.':
                return any(dfs(child, i + 1) for child in node.children.values())
            if ch not in node.children: return False
            return dfs(node.children[ch], i + 1)
        return dfs(self.root, 0)

# Prefix and Suffix Search (LC 745)
class WordFilter:
    def __init__(self, words):
        # For each word, insert "{suffix}#{word}" for all suffixes
        # Each Trie node stores the latest word index passing through it
        self.trie = {}   # nested dict for simplicity; or TrieNode with index
        for idx, word in enumerate(words):
            n = len(word)
            for i in range(n + 1):   # all suffixes including empty ""
                key = word[i:] + '#' + word   # suffix + '#' + full word
                node = self.trie
                for ch in key:
                    if ch not in node:
                        node[ch] = {}
                    node = node[ch]
                    node['#idx'] = idx   # store latest index at each node

    def f(self, prefix, suffix):
        query = suffix + '#' + prefix
        node = self.trie
        for ch in query:
            if ch not in node: return -1
            node = node[ch]
        return node.get('#idx', -1)
```

---

### Edge Cases to Trace Before Coding
- LC 720: single character words only → return the lexicographically smallest (all are valid); empty words list → return ""; word "a" in list but "ab" not → "a" is valid, "ab" is not
- LC 211: search(".") → True if any single-char word exists; search("...") → True if any 3-char word exists; no words added → all searches False
- LC 745: same prefix and suffix → still works (e.g., f("a", "a") for word "aba"); empty prefix or suffix → "#prefix" or "suffix#" lookup; word index is latest match → if "apple" appears twice, return the later index

---

### Interview Pattern Drill

| Trie variant | What makes it different | Query time |
|-------------|------------------------|-----------|
| Standard Trie | Exact match + prefix | O(L) |
| Wildcard Trie | '.' matches any char → DFS all children | O(26^L) worst |
| Suffix-wrapped Trie | Stores `suffix#word` → enables prefix+suffix lookup | O(n×L²) build, O(L) query |
| BFS on is_end paths | Explores only word-valid paths | O(total_chars) |

**WordDictionary iterative alternative:**
Instead of recursion for wildcard DFS, maintain a list of currently active Trie nodes. For each character in the pattern: if literal → filter nodes that have the child; if '.' → expand to all children of all current nodes. Active nodes at end → check is_end on any. This handles large patterns without stack overflow.

---

## System Design (1 hour)
### Topic: Newsfeed — Activity Feed Timeline and Ranked vs Chronological Feed

**Two feed modes:**

**Chronological feed:**
Posts ordered strictly by time. Simple to implement: `ZREVRANGEBYSCORE feed:{user_id}` by timestamp. No ML needed. Drawback: if you follow 500 people, the feed fills with posts from whoever posts most frequently — high-frequency posters dominate; low-frequency posters' posts are buried.

**Ranked / algorithmic feed:**
Each post assigned a relevance score. Posts ordered by score, not time. Score factors:
```
score = (recency_weight × recency_score)
      + (affinity_weight × affinity_score)
      + (engagement_weight × engagement_score)
      + (diversity_penalty × same_author_penalty)
```

- **Recency score**: decays over time (exponential decay: `score × e^(−λ×hours_old)`)
- **Affinity score**: how often the viewer interacts with this poster (comments, likes, DMs)
- **Engagement score**: likes + comments + shares on the post, normalised by post age
- **Diversity penalty**: reduce score if viewer has already seen many posts from the same author

**Score storage:**
The `feed:{user_id}` sorted set uses the computed score (not timestamp) as the Redis ZADD score. Re-scoring happens when:
1. A post receives new engagement (likes, comments) → async score update
2. Time passes → batch score decay job every 15 minutes (updates scores of top 1000 feed items per active user)

**A/B testing feed ranking:**
- Control group (10%): chronological feed
- Treatment groups (each 10%): different ML models or weight configurations
- Metric: engagement rate (likes + comments per session), session length, negative feedback (hide/report)
- Winning model promoted to 100% after statistical significance (2 weeks minimum)

**Feed diversity:**
Problem: if the user mostly interacts with Cat Content, the algorithm over-indexes and shows only Cat Content → filter bubble. Mitigation:
- Hard cap: max 3 consecutive posts from the same author
- Topic diversity: ensure at least 20% of posts are from topics the user interacts with infrequently
- "Why am I seeing this?" — show users the ranking signal that surfaced a post

**Interview talking point:** "At Instagram, the switch from chronological to ranked feed doubled average session time — users saw content they were more likely to engage with, not just content from the most prolific posters they follow. The ranking model uses affinity (how often you DM or comment on a creator's posts) as the strongest signal. The main risk is filter bubbles — we mitigate with diversity constraints and regular injection of content from topics the user rarely engages with."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 720 Longest Word in Dictionary (target: 15 min)
- **Medium 2:** LC 211 Design Add and Search Words (target: 18 min)
- **Hard:** LC 745 Prefix and Suffix Search (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to choose between a simple chronological ordering and a more complex ranking algorithm — what drove the decision?
- Leadership principle: Think Big

---

## Flashcards

| Q | A |
|---|---|
| How does Longest Word in Dictionary use Trie BFS to guarantee every prefix is valid? | BFS from root, but only traverse edges where `child.is_end = True`. This ensures every node visited on the path represents a complete word — so every prefix of the found word is itself a valid word in the dictionary. |
| How does WordDictionary.search handle the '.' wildcard? | DFS on the Trie. At a literal character: traverse the matching child. At '.': recursively try DFS on ALL children. Return True if any DFS branch reaches the end with `is_end = True`. |
| How does WordFilter build the suffix-wrapped Trie? | For each word w at index i, for each suffix s of w (including empty string): insert `"{s}#{w}"` into the Trie, storing `latest_index` at each node. Query `f(prefix, suffix)` looks up `"{suffix}#{prefix}"` and returns the index stored at the final node. |
| What is the key difference between a chronological and a ranked newsfeed? | Chronological: posts ordered strictly by timestamp. Simple, transparent, favours high-frequency posters. Ranked: posts ordered by a computed relevance score (recency × affinity × engagement). More engaging but risks filter bubbles. |
| What is affinity score in feed ranking and how is it computed? | Affinity = how often the viewer has interacted with a poster (liked, commented, DM'd, replied). Computed from the interaction history stored in a graph DB or time-series store; decays over time so recent interactions weigh more than old ones. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/18/25 min, no hints)
- [ ] Rewrote WordDictionary with wildcard DFS and suffix-wrapped Trie from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can articulate ranked vs chronological feed trade-offs cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
