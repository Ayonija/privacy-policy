# Day 76 — Tries: Implement Trie, Replace Words, Word Search II
**Week 11 | Phase 1: DSA Mastery | Month 3**

## Focus
Implement Trie is the foundational template for all Trie problems. Replace Words demonstrates the Trie's prefix-matching superpower: find the shortest matching prefix in O(L) where L is the word length. Word Search II combines Trie with DFS on a grid — the Trie prunes dead-end DFS branches early and avoids re-finding the same word.

---

## DSA (2 hours)
### Pattern: Trie Template + Prefix Replacement + Grid DFS with Trie Pruning

**Implement Trie (Prefix Tree) (LC 208):**
A Trie is a tree where each node represents one character, and each root-to-leaf path represents a word. Each TrieNode has a `children` dict (or array of 26) and an `is_end` flag.

- `insert(word)`: traverse from root, creating nodes as needed; set `is_end = True` at the last character
- `search(word)`: traverse; if any character is missing → False; check `is_end` at last node
- `startsWith(prefix)`: same traversal as search, but return True if the path exists regardless of `is_end`

**Replace Words (LC 648):**
Replace words in a sentence with their shortest root from a dictionary. Insert all roots into a Trie. For each word: traverse the Trie character by character; if `is_end` is reached before the word ends → replace with the root so far (shortest prefix match). If no root matches → keep the original word.

**Word Search II (LC 212):**
Given a board of characters and a list of words, find all words that can be formed by sequentially adjacent cells (4-directional, no revisit). Build a Trie of all target words. For each cell on the board, launch DFS while simultaneously traversing the Trie. Prune if the current cell's character is not in the Trie node's children. When `is_end` is found → word located.

Optimisation: after finding a word, set `is_end = False` on that node to avoid duplicates. After DFS from a cell, if a Trie node has no children and no `is_end`, prune it (remove from parent) to reduce future DFS branching.

**Trigger condition:**
- "store and search strings with prefix queries efficiently" → Trie; O(L) insert/search/startsWith
- "replace each word in a sentence with its shortest root from a dictionary" → Trie; stop at first is_end
- "find all words from a list that exist in a character grid" → Trie + grid DFS; Trie prunes dead branches

**Time complexity:** LC 208: O(L) per op | LC 648: O(total_chars) | LC 212: O(m×n×4^L) with Trie pruning
**Space complexity:** O(total_chars_in_dict) for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Implement Trie (Prefix Tree) | 208 | Medium | Trie with children dict + is_end flag | `search` checks is_end; `startsWith` only checks path exists |
| 2 | Replace Words | 648 | Medium | Trie prefix match; stop at first is_end | Insert all roots; traverse each word; return prefix if is_end hit |
| 3 | Word Search II | 212 | Hard | Trie + grid DFS; prune Trie after finding | Build Trie of all words; DFS on grid tracks Trie node; prune found words |

---

### Code Skeleton
```python
# Implement Trie (LC 208)
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children: return False
            node = node.children[ch]
        return node.is_end

    def startsWith(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node.children: return False
            node = node.children[ch]
        return True

# Replace Words (LC 648)
def replaceWords(dictionary, sentence):
    trie = Trie()
    for root in dictionary:
        trie.insert(root)
    words = sentence.split()
    result = []
    for word in words:
        node = trie.root
        replaced = False
        for i, ch in enumerate(word):
            if ch not in node.children: break
            node = node.children[ch]
            if node.is_end:
                result.append(word[:i + 1])   # shortest root match
                replaced = True
                break
        if not replaced:
            result.append(word)
    return ' '.join(result)

# Word Search II (LC 212)
def findWords(board, words):
    trie = Trie()
    for word in words:
        trie.insert(word)
    rows, cols = len(board), len(board[0])
    found = []
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    def dfs(r, c, node, path):
        ch = board[r][c]
        if ch not in node.children: return
        next_node = node.children[ch]
        path.append(ch)
        if next_node.is_end:
            found.append(''.join(path))
            next_node.is_end = False   # mark to avoid duplicates
        board[r][c] = '#'              # mark visited
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != '#':
                dfs(nr, nc, next_node, path)
        board[r][c] = ch               # restore
        path.pop()
        # Pruning: remove exhausted Trie nodes
        if not next_node.children and not next_node.is_end:
            del node.children[ch]

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, trie.root, [])
    return found
```

---

### Edge Cases to Trace Before Coding
- LC 208: search empty string → depends on implementation (usually False); insert then search same word → True; startsWith full word → True
- LC 648: no root matches a word → keep original; multiple roots match → return shortest (Trie guarantees this by returning at first is_end); root == word itself → replace with itself
- LC 212: same word appears at multiple board positions → set is_end=False after first find prevents duplicates; 1×1 board → if board[0][0] is in words → return it; word longer than board → naturally not found

---

### Interview Pattern Drill

| Trie operation | Traversal | Return condition | Notes |
|---------------|-----------|-----------------|-------|
| insert | Root → each char | Set is_end at last | Create nodes as needed |
| search | Root → each char | `is_end` at last char | False if path broken |
| startsWith | Root → each char | `True` if path complete | Ignore is_end |
| Replace words | Root → chars of word | Return prefix at first is_end | Break early |
| Word Search II DFS | Board + Trie simultaneously | Collect when is_end | Prune exhausted nodes |

**Trie vs HashSet for prefix queries:**
- HashSet: `search(word)` O(L), `startsWith(prefix)` requires O(n × L) scan of all words. Cannot answer prefix queries efficiently.
- Trie: both `search` and `startsWith` are O(L). Additional advantage: shared prefixes share memory — "apple" and "app" share the "app" node.

---

## System Design (1 hour)
### Topic: Newsfeed — Notification System Design

**What notifications does a newsfeed generate?**
- Like on your post
- Comment on your post
- Someone mentions you (`@username`)
- New follower
- Post from a user you enabled "Notify me" for
- Activity digest (weekly "here's what you missed")

**High-level notification architecture:**
```
User action (like/comment/follow/mention)
    │
[Action Service] → Kafka topic "user_actions"
    │
[Notification Fanout Worker]
    ├─ Check user notification preferences (User Prefs DB)
    ├─ Deduplicate (e.g., 5 likes in 1 min → batch "5 people liked your post")
    └─ Emit to Kafka topic "notification_tasks"
    │
[Notification Delivery Workers] (one per channel)
    ├─ Push Notification Worker → FCM (Android) / APNS (iOS)
    ├─ Email Worker → SendGrid / SES
    └─ In-App Worker → write to notification_inbox:{user_id} in Redis/DB
```

**Deduplication and batching:**
- Problem: a viral post gets 10,000 likes in 1 minute → 10,000 individual "X liked your post" push notifications would be spam
- Solution: sliding window aggregation in the Notification Fanout Worker
  - Aggregate likes/comments within a 5-minute window per (post_id, notification_type)
  - Emit one batched notification: "Alice, Bob, and 9,998 others liked your post"
- Implementation: Redis counter `notify:like:{post_id}` with 5-min TTL; atomic increment; send notification only on first increment (and again if count reaches thresholds: 10, 100, 1000)

**Push notification delivery:**
- FCM (Firebase Cloud Messaging) for Android; APNS (Apple Push Notification Service) for iOS
- Token management: each device registers a push token; stored in `device_tokens` table; stale tokens removed after delivery failure
- Retry with exponential backoff on delivery failure

**Notification preferences:**
User can configure:
- Which types of notifications they want (toggle per type)
- Channel preferences (push, email, or both)
- Quiet hours (no push between 11 PM and 8 AM)

Stored in User Prefs DB; cached in Redis with 1-hour TTL per user.

**In-app notification inbox:**
- Redis sorted set `notification_inbox:{user_id}` (score = timestamp, member = notification JSON)
- Cap at 100 most recent; older ones in DB for "load more"
- On app open: client fetches from Redis; marks as read

**Interview talking point:** "The notification pipeline is event-driven: every user action publishes to Kafka; the fanout worker reads from Kafka, checks preferences, deduplicates, and writes delivery tasks to per-channel Kafka topics. Delivery workers are stateless and horizontally scalable. The key design challenge is batching — we must aggregate notifications (e.g., 10K likes → 1 push) to avoid spamming users and overwhelming mobile notification services."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 208 Implement Trie (target: 12 min — write from memory)
- **Medium 2:** LC 648 Replace Words (target: 15 min)
- **Hard:** LC 212 Word Search II (target: 30 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you built or improved a notification or alerting system — how did you handle the trade-off between too many notifications and missing important ones?
- Leadership principle: Customer Obsession

---

## Flashcards

| Q | A |
|---|---|
| What are the three operations of a Trie and their time complexities? | `insert(word)`, `search(word)`, `startsWith(prefix)` — all O(L) where L is the word/prefix length. Each traverses from root one character at a time, creating nodes (insert) or returning early on missing nodes (search/startsWith). |
| How does Replace Words find the shortest matching root? | Insert all roots into a Trie. For each word in the sentence, traverse the Trie character by character. The first time `is_end = True` is encountered → return the prefix built so far (shortest root). If traversal breaks before any is_end → keep original word. |
| What two optimisations make Word Search II efficient? | (1) Set `is_end = False` after finding a word — prevents duplicate results without an extra set. (2) Delete a Trie child node when it has no children and no is_end — prunes dead branches from future DFS calls, reducing redundant work. |
| Why is Trie better than HashSet for autocomplete / prefix queries? | HashSet checks exact membership in O(L) but can't answer "does any word start with X?" without scanning all entries O(n×L). Trie answers both in O(L) by sharing prefix nodes — all words sharing a prefix share those Trie nodes. |
| How does the notification system prevent spam from viral posts? | Redis counter with TTL: on first like → send notification and set counter=1 with 5-min TTL. Subsequent likes increment the counter but don't trigger new notifications. At TTL expiry: if count > 1, send batched "X others liked your post." Thresholds (10, 100, 1000) trigger additional batched notifications. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 12/15/30 min, no hints)
- [ ] Rewrote the full Trie class (TrieNode + insert + search + startsWith) from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw the notification pipeline with deduplication cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
