# Day 78 — Tries: Map Sum Pairs, Search Suggestions, and Stream of Characters
**Week 12 | Phase 1: DSA Mastery | Month 3**

## Focus
Map Sum Pairs extends Trie nodes to carry aggregated values — each node stores the sum of all key values that pass through it. Search Suggestions System is an autocomplete problem solvable with a sorted Trie (or binary search on a sorted word list). Stream of Characters uses a reversed-word Trie to match streaming characters against a dictionary of words in real time.

---

## DSA (2 hours)
### Pattern: Value-Accumulating Trie + Sorted Trie Autocomplete + Reverse-Word Trie Streaming

**Map Sum Pairs (LC 677):**
Insert key-value pairs where the key is a string. `sum(prefix)` returns the sum of all values whose keys start with the given prefix. Each Trie node stores the running sum of values for all keys that pass through it. On insert: compute `delta = val - old_val` (if key was inserted before); add `delta` to every node along the key's path.

**Search Suggestions System (LC 1268):**
Given a list of `products` and a `searchWord`, return up to 3 lexicographically smallest products that share the prefix of each incrementally longer prefix of `searchWord`.

Approach 1 (Trie): insert all products; at each Trie node, store the top-3 lexicographically sorted words passing through it (maintained on insert). Query each prefix of searchWord by traversing the Trie.

Approach 2 (Sort + Binary Search — simpler): Sort products. For each prefix of searchWord, binary search for the first product with that prefix and return the next 3.

**Stream of Characters (LC 1032):**
Given a list of words, process a stream of characters one at a time. After each character, report if any suffix of the stream matches a word in the list.

Key insight: build a Trie of REVERSED words. Maintain a list of "active searches" (Trie nodes advanced so far). When character `c` arrives: start a new search from the root (new candidate beginning at this character); advance all active searches by `c`. An active search that hits a word (is_end) means the word ending at this stream position matches. Remove dead searches (no matching child).

**Trigger condition:**
- "sum of values for all keys with a given prefix" → Trie nodes accumulate sum; delta-update on re-insert
- "autocomplete top-3 suggestions for each prefix of a search query" → Trie with stored sorted suggestions; or sort + binary search
- "does any suffix of the current stream match a known word?" → reversed-word Trie; maintain list of active searches

**Time complexity:** LC 677: O(L) insert; O(L) sum | LC 1268: O(n log n + L²) sort approach; O(n×L) Trie | LC 1032: O(w×L) build; O(L) per query
**Space complexity:** O(total_chars) for all three

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Map Sum Pairs | 677 | Medium | Value-accumulating Trie (delta updates) | Store sum at each node; delta = new_val - old_val; propagate delta along key path |
| 2 | Search Suggestions System | 1268 | Medium | Sort + binary search (simpler) OR Trie with top-3 | Sort products; for each prefix, bisect to find start; return next 3 with matching prefix |
| 3 | Stream of Characters | 1032 | Hard | Reversed-word Trie + active search list | Insert reversed words; on each char, start new search + advance all existing; is_end = match |

---

### Code Skeleton
```python
import bisect

# Map Sum Pairs (LC 677)
class MapSum:
    def __init__(self):
        self.trie = {}   # nested dict; each node has '#sum' key
        self.key_vals = {}   # track existing key values for delta computation

    def insert(self, key, val):
        delta = val - self.key_vals.get(key, 0)
        self.key_vals[key] = val
        node = self.trie
        for ch in key:
            if ch not in node: node[ch] = {}
            node = node[ch]
            node['#sum'] = node.get('#sum', 0) + delta

    def sum(self, prefix):
        node = self.trie
        for ch in prefix:
            if ch not in node: return 0
            node = node[ch]
        return node.get('#sum', 0)

# Search Suggestions System (LC 1268) — Sort + Binary Search
def suggestedProducts(products, searchWord):
    products.sort()
    result = []
    prefix = ""
    for ch in searchWord:
        prefix += ch
        # Find the first product that starts with prefix
        lo = bisect.bisect_left(products, prefix)
        suggestions = []
        for i in range(lo, min(lo + 3, len(products))):
            if products[i].startswith(prefix):
                suggestions.append(products[i])
            else:
                break
        result.append(suggestions)
    return result

# Stream of Characters (LC 1032) — Reversed-word Trie
class StreamChecker:
    def __init__(self, words):
        self.trie = {}
        for word in words:
            node = self.trie
            for ch in reversed(word):   # insert reversed word
                if ch not in node: node[ch] = {}
                node = node[ch]
            node['#end'] = True   # mark end of (reversed) word
        self.active = []   # list of active Trie nodes (searches in progress)

    def query(self, letter):
        new_active = []
        # Advance all active searches
        for node in self.active:
            if letter in node:
                node = node[letter]
                if node.get('#end'): return True
                new_active.append(node)
        # Start a new search from root
        if letter in self.trie:
            node = self.trie[letter]
            if node.get('#end'): return True
            new_active.append(node)
        self.active = new_active
        return False
```

---

### Edge Cases to Trace Before Coding
- LC 677: re-insert same key with new value → delta = new - old; sum updates correctly; key not previously inserted → delta = val; prefix longer than any key → return 0
- LC 1268: no products share any prefix → empty lists for each step; single character searchWord → check all products starting with that char; duplicate products → return them (problem uses list as-is)
- LC 1032: single character word (e.g., "a") → match as soon as 'a' arrives; word longer than stream so far → active search accumulates; character matches no Trie child → drop that active search

---

### Interview Pattern Drill

| Trie variant | What each node stores | Insert time | Query time |
|-------------|----------------------|------------|-----------|
| Standard Trie | is_end flag | O(L) | O(L) |
| Map Sum Trie | Accumulated value sum | O(L) | O(L) |
| Autocomplete Trie | Sorted top-3 list | O(L log 3) | O(L) |
| Reversed stream Trie | End-of-word flag | O(L) per word | O(active_searches) |

**Binary search autocomplete (Search Suggestions) — why it works:**
After sorting, all products with the same prefix form a contiguous subarray. `bisect_left(products, prefix)` finds where the prefix would be inserted — the products at and after that index either start with the prefix (first 3) or don't (stopped immediately). Correct and simpler than a Trie for this problem.

---

## System Design (1 hour)
### Topic: Newsfeed — Scaling Feed Generation and Write Path Optimization

**Fanout service scaling:**
The fanout service is a separate pool of workers that consumes from a Kafka topic `new_posts`:

```
Post Service → Kafka "new_posts" topic
                    │
        [Fanout Worker Pool] (auto-scaled)
                    │
            For each post:
              1. Read follower list from Cassandra (paginated, 1000/batch)
              2. ZADD feed:{follower_id} {ts} {post_id} to Redis (batched pipeline)
              3. Publish to Kafka "notification_tasks" for notification workers
```

**Rate limiting fanout for large but non-celebrity accounts:**
- User with 100K followers: fanout completes in ~0.1 seconds at 1M Redis ops/sec
- User with 1M followers: ~1 second — still fast enough
- User with 10M followers: ~10 seconds — acceptable for eventual consistency
- Celebrity (100M+): skip fanout; use hybrid read-time merge

**Redis pipeline for fanout:**
Instead of one ZADD per follower, pipeline 1000 ZADD commands in one Redis round-trip:
```python
pipe = redis.pipeline()
for follower_id in batch:
    pipe.zadd(f"feed:{follower_id}", {post_id: timestamp})
pipe.execute()   # 1000 ZADDs in 1 round-trip
```

**Post DB write path:**
```
1. App server receives POST /post
2. Validate and generate Snowflake post_id
3. Write post to MySQL primary (synchronous)
4. Upload media to S3 (async, client handles retry)
5. Publish {post_id, user_id, timestamp} to Kafka "new_posts" (async)
6. Return post_id to client immediately (feed update is eventual)
```

**Cross-region replication:**
- Primary MySQL DB in us-east-1; replicas in eu-west-1 and ap-northeast-1
- Redis feed clusters per region: fans-out to regional Redis on post creation; 1-2 sec replication lag acceptable
- CDN caches post content globally; media from S3 behind CloudFront

**Write amplification math:**
- Average user: 200 followers × 1 ZADD = 200 ops per post
- 1 M posts/day × 200 ops = 200 M Redis ops/day = 2,315 ops/sec — trivial for Redis cluster
- Celebrity write path is O(1) regardless of follower count (just writes to `user_posts:{id}`)

**Interview talking point:** "Feed write latency is hidden behind Kafka. The user's post returns success immediately after the DB write — the fanout is asynchronous. A follower sees the post in their feed within 1–10 seconds (depending on follower count and worker load). This is the eventual consistency trade-off: we accept brief staleness in exchange for fast writes. The only exception is the poster themselves — we use optimistic UI updates to show their own post immediately in their feed."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock
- **Medium 1:** LC 677 Map Sum Pairs (target: 15 min)
- **Medium 2:** LC 1268 Search Suggestions System (target: 15 min — use sort + binary search)
- **Hard:** LC 1032 Stream of Characters (target: 28 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you designed a system that needed to process a continuous stream of data in real-time — how did you balance throughput and latency?
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does Map Sum Pairs update the Trie efficiently when a key is re-inserted? | Compute `delta = new_val - old_val`. Traverse the key's path in the Trie; add `delta` to the `#sum` field of every node visited. This avoids re-summing all values; only the changed amount propagates. |
| Why is sort + binary search simpler than a Trie for Search Suggestions System? | Sorted products with the same prefix form a contiguous subarray. `bisect_left` finds the insertion point for the prefix in O(log n); the next 3 entries are checked for the prefix. O(n log n) build, O(L log n) query — no Trie overhead. |
| How does Stream of Characters use a reversed-word Trie? | Words are inserted reversed into the Trie. On each new character: start a new search from root; advance all active searches by the character. A search reaching is_end means a word ending at the current stream position matches. |
| How does Redis pipeline reduce fanout latency? | Batching 1000 ZADD commands into one pipeline call reduces round-trip overhead from 1000 network RTTs to 1. At 0.1 ms RTT per Redis command, 1000 commands = 100 ms naively; pipelined = 0.1 ms. |
| Why does the post write path return success before fanout completes? | Fanout is async (via Kafka workers). Waiting for fanout would add seconds of latency for high-follower posts. Eventual consistency (followers see the post within seconds) is acceptable for social feeds. Only the poster sees their post immediately via optimistic UI update. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/15/28 min, no hints)
- [ ] Rewrote Map Sum delta-update Trie and Stream of Characters reversed-Trie from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain fanout pipeline and Redis pipelining cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
