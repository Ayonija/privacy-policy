# Day 95 — 1D DP: Word Break & String Segmentation
**Week 14 | Phase 2: Exposure & Assessment | Month 4**

## Focus
Master string segmentation DP — problems where `dp[i]` asks "can the prefix of length i be formed from a dictionary?" — and extend to enumerate all valid segmentations and detect concatenated words, which are the Hard variants that appear in FAANG interviews.

---

## DSA (2 hours)

### Pattern: 1D DP — String Segmentation (Bottom-Up)

**Core idea:**  
`dp[i]` = True if `s[0:i]` can be formed from the word dictionary. For each position `i`, check all valid word endings `j < i`: if `dp[j]` is True and `s[j:i]` is in the dictionary, set `dp[i] = True`. Alternatively, iterate words: for each word at position `j`, if `dp[j]` and word matches → mark `dp[j + len(word)] = True`.

**Trigger condition:**  
- "Can a string be segmented into words from a dictionary?"
- "Return all valid segmentations"
- "Is this string formed by concatenating shorter strings from a list?"

**Complexity:**  
Time: O(n² × L) where L = avg word length for dict lookup | Space: O(n) for dp array  
With a Trie: O(n²) — scan from each position, follow Trie branches

---

### Problems

| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Word Break II | 140 | Hard | DP + Backtracking (or memoized DFS) | Build dp[i] first; then backtrack only from valid positions to enumerate paths |
| 2 | Concatenated Words | 472 | Hard | Word Break applied per word | For each word, run Word Break where the dictionary excludes the word itself |
| 3 | Word Break (revision) | 139 | Medium | 1D DP | `dp[i] = any(dp[j] and s[j:i] in word_set for j in range(i))` |

---

### Full Solution Walkthrough — Word Break (LC 139) — Revision

```python
def wordBreak(s: str, wordDict: list[str]) -> bool:
    word_set = set(wordDict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True  # empty prefix always valid

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break  # early exit once dp[i] is True

    return dp[n]
```

**Optimization:** Iterate words instead of substrings when dictionary is small:
```python
for i in range(n + 1):
    if not dp[i]: continue
    for word in wordDict:
        end = i + len(word)
        if end <= n and s[i:end] == word:
            dp[end] = True
```

---

### Full Solution Walkthrough — Word Break II (LC 140) — Hard

**Problem:** Return all possible sentence segmentations of `s` using words from `wordDict`.

**Approach:** Memoized top-down DFS. `memo[i]` = list of all valid segmentations of `s[i:]`.

```python
def wordBreak(s: str, wordDict: list[str]) -> list[str]:
    word_set = set(wordDict)
    memo = {}

    def dfs(start: int) -> list[str]:
        if start == len(s):
            return [""]   # one valid segmentation: empty suffix
        if start in memo:
            return memo[start]

        result = []
        for end in range(start + 1, len(s) + 1):
            word = s[start:end]
            if word in word_set:
                suffixes = dfs(end)
                for suffix in suffixes:
                    sentence = word + (" " + suffix if suffix else "")
                    result.append(sentence)

        memo[start] = result
        return result

    return dfs(0)
```

**Complexity:** O(n² × 2^n) worst case for output (exponential segmentations possible); with memoization, subproblem reuse makes it practical.

**Bottom-up alternative:** Build `dp[i]` = list of words ending at position `i` that start from a valid position. Then backtrack from `dp[n]` to reconstruct paths. Use this if you're more comfortable with bottom-up:

```python
def wordBreak_bottomup(s: str, wordDict: list[str]) -> list[str]:
    word_set = set(wordDict)
    n = len(s)
    # dp[i] = list of last words that brought us to position i
    dp = [[] for _ in range(n + 1)]
    dp[0] = [""]  # sentinel

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i].append(s[j:i])

    # Reconstruct paths from dp[n] back to dp[0]
    def reconstruct(pos: int) -> list[str]:
        if pos == 0:
            return [""]
        results = []
        for word in dp[pos]:
            for prev in reconstruct(pos - len(word)):
                results.append((prev + " " + word).strip())
        return results

    return reconstruct(n)
```

---

### Full Solution Walkthrough — Concatenated Words (LC 472) — Hard

**Problem:** Given a list of words, find all words that are formed by concatenating 2+ other words from the list.

**Insight:** For each word `w`, run Word Break on `w` using the dictionary *excluding w itself*. If `wordBreak(w, dict - {w})` returns True → `w` is a concatenated word.

```python
def findAllConcatenatedWordsInADict(words: list[str]) -> list[str]:
    word_set = set(words)
    result = []

    def can_form(word: str) -> bool:
        if not word:
            return False
        n = len(word)
        dp = [False] * (n + 1)
        dp[0] = True
        for i in range(1, n + 1):
            for j in range(i):
                if dp[j] and s[j:i] in word_set and (j > 0 or i < n):
                    # The condition (j>0 or i<n) ensures we use ≥2 words
                    # i.e., the word itself isn't just matched as one piece
                    dp[i] = True
                    break
        return dp[n]

    for word in words:
        word_set.discard(word)       # temporarily remove word from dict
        if can_form(word):
            result.append(word)
        word_set.add(word)           # restore

    return result
```

**Clean implementation with length-≥2 enforcement:**
```python
def findAllConcatenatedWordsInADict(words: list[str]) -> list[str]:
    word_set = set(words)
    result = []

    def word_break(s: str, min_parts: int) -> bool:
        # True if s can be formed from word_set using ≥ min_parts words
        n = len(s)
        # dp[i] = min number of words to form s[0:i], or inf if impossible
        dp = [float('inf')] * (n + 1)
        dp[0] = 0
        for i in range(1, n + 1):
            for j in range(i):
                if s[j:i] in word_set and s[j:i] != s:  # don't use full word
                    if dp[j] + 1 < dp[i]:
                        dp[i] = dp[j] + 1
        return dp[n] >= min_parts

    for word in words:
        if word_break(word, 2):
            result.append(word)

    return result
```

**Complexity:** O(W × L²) where W = number of words, L = max word length.

---

## System Design (30 min)

### Topic: Chat Service — Message Delivery Guarantees & Receipts

**Core components:**
1. **Message states:** `SENT` (server received) → `DELIVERED` (target device received) → `READ` (user opened)
2. **ACK protocol:** Client sends message → server assigns sequence ID, stores in Cassandra → returns ACK to sender
3. **Delivery receipt:** Target device receives message over WebSocket → sends delivery ACK to server → server updates state + notifies sender via WebSocket
4. **Read receipt:** User opens conversation → client sends READ event → server updates + fans out to sender
5. **Retry logic:** If WebSocket disconnects mid-send, client resends using idempotency key (message UUID); server deduplicates by checking Cassandra

**Key trade-offs:**
- **At-least-once vs. exactly-once delivery:** At-least-once is standard (resend on timeout + dedup by message ID at consumer). Exactly-once requires distributed transactions — too expensive for a chat service.
- **Receipt fan-out cost:** For a 1:1 chat, receipts are cheap (1 update → 1 notification). For group chats, "X people read" receipts would generate O(members) events per message — batch them (send receipt summary every 5s).
- **Sequence gaps:** If message sequence IDs jump (e.g., 5 → 7), client should request the gap from the server rather than stall.

**Interview talking point:**  
*"If asked how WhatsApp shows double-tick vs. blue-tick, answer: single tick = sent to server; double tick = delivered to recipient device (delivery ACK); blue tick = opened by recipient (read event). Each state triggers a server write and a WebSocket notification to the sender. Read receipts are opt-in (privacy setting) — if disabled, the server doesn't send the read event."*

---

## Assessment / Mock (1 hour)

### Activity: LeetCode — Word Break Variants Sprint

**Goal:** Solve LC 139 + LC 140 in sequence without hints.

**Session structure:**
- 0:00 — LC 139 (Medium): implement bottom-up dp, submit (target: 8 min)
- 8:00 — LC 140 (Hard): first write `dfs(start)` skeleton, trace on `"catsanddog"` with `["cat","cats","and","sand","dog"]` before coding (target: 25 min)
- 33:00 — Review: trace `memo[0]` → `memo[3]` → `memo[7]` → `memo[10]` manually
- 50:00 — Describe Concatenated Words in 3 sentences without looking at code

**Debrief prompt:** What's the difference between the memoized DFS approach and the bottom-up + backtrack approach for LC 140? Which did you find clearer? Write one sentence explaining the trade-off.

---

## Behavioral (30 min)

**STAR prompt:**  
Tell me about a time you had to debug a subtle bug in production. How did you isolate the root cause?

**Target LP:** *Dive Deep* (Amazon) — get to root cause; don't treat symptoms.

**Tip:** Be specific about your debugging *method*, not just the outcome. "I added logging and found it was X" is weak. "I formed a hypothesis, tested it by isolating variable Y, and confirmed by checking Z" shows systematic thinking.

---

## Flashcards

| Q | A |
|---|---|
| Write the Word Break DP recurrence. | `dp[0] = True`. For `i` in `1..n`: `dp[i] = True` if `∃j < i: dp[j] is True AND s[j:i] ∈ word_set`. O(n²) time with set lookup. |
| What is the base case for Word Break II's memoized DFS? | `dfs(len(s))` returns `[""]` — one valid path (empty suffix). This allows callers to prepend the word that led to the end without special-casing. |
| How does Concatenated Words differ from Word Break? | In Concatenated Words, the word being tested must be excluded from the dictionary (don't match itself) and must use ≥ 2 sub-words. Run Word Break with those constraints for each word in the input list. |
| When would you use a Trie instead of a set for Word Break? | When words are long and you want to avoid slicing substrings O(L) per check. A Trie lets you scan from position `i` character by character, matching all words simultaneously. Reduces O(n²L) to O(n²) in practice. |
| In Chat, what is the difference between SENT, DELIVERED, and READ states? | SENT = server acknowledged receipt; DELIVERED = target device acknowledged download (double tick); READ = user opened the conversation (blue tick). Each triggers a separate server update + WebSocket notification to sender. |

---

## Checklist

- [ ] Word Break (LC 139): wrote bottom-up DP from memory, traced `"leetcode"` with `["leet","code"]`
- [ ] Word Break II (LC 140): implemented memoized DFS, traced `"catsanddog"` step by step
- [ ] Concatenated Words: explained the "exclude self, use ≥ 2 parts" constraint from memory
- [ ] Described message delivery states (SENT / DELIVERED / READ) and their triggers
- [ ] Completed 1-hour Word Break sprint with debrief
- [ ] Behavioral STAR delivered aloud with specific debugging method described
- [ ] Reviewed all 5 flashcards
- [ ] Logged any uncertain problems to `knowledge-base/revision-log.md`
