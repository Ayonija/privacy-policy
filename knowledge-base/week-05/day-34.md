# Day 34 — Hashing: HashMap Design Problems
**Week 05 | Phase 1: DSA Mastery | Month 2**

## Focus
Build data structures with O(1) operations by combining HashMaps with auxiliary structures — a direct preview of system design cache implementations.

---

## DSA (2 hours)
### Pattern: HashMap + Auxiliary Structure for O(1) All Operations

**Core idea:**
HashMap alone gives O(1) get/put but not O(1) ordered access (min, max, LRU eviction). Combine it with a doubly-linked list (for order) or an array (for random access) to achieve O(1) across all required operations.

**LRU Cache:** HashMap(key → node) + doubly linked list (most-recently-used at head, least-recently-used at tail). On access: move node to head. On eviction: remove tail node.

**Insert/Delete/GetRandom O(1):** HashMap(val → array_index) + array(values). Delete: swap target with last element, update the swapped element's index in HashMap, pop last.

**All O'one:** Most complex — needs O(1) inc/dec and O(1) getMinKey/getMaxKey. Use a doubly linked list of *frequency nodes* (each holding a set of keys at that frequency) + a HashMap(key → frequency_node).

**Trigger condition:** "design a data structure with O(1) for operations that normally conflict (ordered + random access)" → the answer is almost always HashMap + secondary structure.

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | LRU Cache | 146 | Medium | HashMap + doubly linked list | `key→node` map gives O(1) lookup; DLL maintains recency order; dummy head/tail simplify edge cases |
| 2 | Insert Delete GetRandom O(1) | 380 | Medium | HashMap + array | Array enables O(1) getRandom; HashMap enables O(1) lookup; swap-with-last enables O(1) delete |
| 3 | All O`one Data Structure | 432 | Hard | DLL of frequency nodes + two HashMaps | `key→freq` map + `freq→DLL_node` map; DLL ordered by freq; O(1) inc/dec moves key between adjacent freq nodes |

---

### Code Skeleton
```java
// LRU Cache (LC 146)
class LRUCache {
    private class Node {
        int key, val;
        Node prev, next;
        Node(int key, int val) { this.key = key; this.val = val; }
    }

    private int cap;
    private Map<Integer, Node> cache = new HashMap<>();   // key → node
    // doubly linked list with dummy head (MRU side) and tail (LRU side)
    private Node head = new Node(0, 0), tail = new Node(0, 0);

    public LRUCache(int capacity) {
        this.cap = capacity;
        head.next = tail;
        tail.prev = head;
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    public int get(int key) {
        if (!cache.containsKey(key)) return -1;
        remove(cache.get(key));
        insertFront(cache.get(key));
        return cache.get(key).val;
    }

    public void put(int key, int val) {
        if (cache.containsKey(key)) remove(cache.get(key));
        Node node = new Node(key, val);
        cache.put(key, node);
        insertFront(node);
        if (cache.size() > cap) {
            Node lru = tail.prev;
            remove(lru);
            cache.remove(lru.key);
        }
    }
}

// Insert Delete GetRandom O(1) (LC 380)
class RandomizedSet {
    private Map<Integer, Integer> valToIdx = new HashMap<>();
    private List<Integer> vals = new ArrayList<>();

    public boolean insert(int val) {
        if (valToIdx.containsKey(val)) return false;
        valToIdx.put(val, vals.size());
        vals.add(val);
        return true;
    }

    public boolean remove(int val) {
        if (!valToIdx.containsKey(val)) return false;
        int idx = valToIdx.get(val);
        int last = vals.get(vals.size() - 1);
        vals.set(idx, last);
        valToIdx.put(last, idx);
        vals.remove(vals.size() - 1);
        valToIdx.remove(val);
        return true;
    }

    public int getRandom() {
        return vals.get((int)(Math.random() * vals.size()));
    }
}
```

---

### Edge Cases to Trace Before Coding
- LRU: capacity = 1; put same key twice (update, not duplicate)
- RandomizedSet: remove the only element; getRandom on single-element set
- All O'one: inc a key that doesn't exist yet (treat as freq 0 → 1); dec a key with freq 1 (remove from data structure entirely)

---

### STAR Interview Framework

> **HashMap + Auxiliary Structure for O(1) All Operations:** brute-force O(n) eviction / random access → this approach O(1) time for all operations, O(n) space

**S:** "Design a cache supporting O(1) get, put, and LRU eviction for 500K sessions. HashMap alone gives O(1) get/put but O(n) eviction — scanning 500K entries every eviction causes latency spikes at scale."
**T:** "Need O(1) eviction by augmenting the HashMap with a doubly-linked list that tracks recency order — the LRU node is always at the tail."
**A (60% of answer time):**
1. *Classify:* "'O(1) get/put + eviction order' → HashMap + DLL (LRU). 'O(1) insert/delete + uniform random' → HashMap + array with swap-to-last delete. 'O(1) inc/dec + getMin/getMax' → DLL of frequency nodes + two HashMaps (All O'one)."
2. *Init:* "LRU: dummy `head` (MRU side) and dummy `tail` (LRU side); `cache` HashMap of `key → Node`. Two helper methods: `remove(node)` and `insertFront(node)`."
3. *Loop/Step:* "On get: look up in map; call `remove`, then `insertFront`; return val. On put: if exists, remove old; create new node; insert at front; if `cache.size() > cap`, remove `tail.prev` and delete its key from map."
4. *Termination:* "Every operation is O(1) — pointer rewiring, no traversal."
5. *Gotcha:* "Always store the `key` inside the Node — when you evict `tail.prev`, you need its key to delete it from the HashMap. Forgetting this means you can't clean up the map entry. State this before coding."
**R:** "O(1) time for all operations, O(n) space. For RandomizedSet: swap-with-last delete is the key — O(1) vs O(n) array shift. At 10K QPS: consistent sub-millisecond vs 200ms eviction spikes."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| OrderedDict (Python) | Python only; interview flexibility | Language-specific — explain the underlying DLL mechanism anyway |
| Sorted set / TreeMap by timestamp | Need arbitrary timestamp queries | O(log n) per operation — too slow for O(1) requirement |
| Single HashMap with timestamp field | Simple LRU approximation | O(n) eviction scan — the exact problem we're solving |

---

## System Design (1 hour)
### Topic: Clustered vs. Non-Clustered Indexes

**Clustered Index:**
- The table rows are physically stored *in the order of the index key*
- There can be **only one** clustered index per table (the data can only be sorted one way)
- In MySQL InnoDB: the PRIMARY KEY is always the clustered index; row data lives in the B+Tree leaf nodes
- Range queries on the clustered key are extremely fast — rows are adjacent on disk

**Non-Clustered (Secondary) Index:**
- A separate B+Tree structure; leaf nodes store the index key + a pointer to the row (or the primary key in InnoDB)
- Multiple secondary indexes allowed per table
- A lookup requires two B+Tree traversals: first the secondary index, then the primary index (called a **double lookup** or **bookmark lookup**)

**Covering Index:**
- A secondary index that contains *all columns needed by the query* — no need for the second lookup
- Example: `SELECT email FROM users WHERE username = 'alice'` — if the index is on `(username, email)` the query never touches the main table

**Practical guidance:**
| Scenario | Recommendation |
|----------|---------------|
| Equality lookup on primary key | Free — clustered index |
| Equality lookup on non-primary column | Add secondary index |
| Range query on non-primary column | Add secondary index; consider covering index |
| Multi-column WHERE clause | Composite index in selectivity order |

**Interview talking point:** "If asked why InnoDB tables should always have a small integer primary key (not a UUID), answer: the primary key is the clustered index — all secondary indexes store the PK value as the row pointer. A UUID (16 bytes) vs. an int (4 bytes) means every secondary index is 4× larger. Also, random UUID inserts cause random B+Tree leaf splits (no sequential order), fragmenting the index and causing many more disk writes than auto-increment integers."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Think Big

**Full STAR Story — "Designing a Multi-Criteria O(1) Access Layer for a High-Traffic Cache":**
**S (20%):** "At a streaming company, a recommendation service cached 500K user sessions in memory. When the cache filled, the eviction policy did a full O(n) scan to find the least-recently-used session — under 10K QPS this caused 200ms spikes every 30 seconds."
**T:** "I was asked to redesign the eviction layer to bring worst-case eviction latency under 1ms, without changing the cache API or increasing memory footprint."
**A (60% — 'I' not 'we'):** "(1) I identified the root cause: O(n) eviction scan because the only data structure was a HashMap — no ordering information. (2) I added a doubly-linked list tracking recency: on every cache hit I moved the accessed node to the head (O(1)); on eviction I removed the tail node (O(1)). (3) I stored the list node pointer inside the HashMap entry so lookup and move were both O(1) with no secondary search. (4) I wrote a 72-hour load test simulating 15K QPS with randomised access patterns to verify that the eviction spike disappeared before proposing the change to the infra review board."
**R (20%):** "Eviction latency dropped from a 200ms spike to a consistent sub-millisecond operation. P99 API latency improved from 340ms to 28ms. The pattern was adopted as the standard session-cache design across three other services within two months."
*Works for: Think Big, Invent and Simplify, Deliver Results.*

---

## Flashcards

| Q | A |
|---|---|
| How does LRU Cache achieve O(1) eviction? | Doubly-linked list — least-recently-used node is always at the tail (before the dummy tail node); removal is O(1) with prev/next pointers |
| How does RandomizedSet delete in O(1) without shifting the array? | Swap the target element with the last element; update the last element's index in the HashMap; pop the last element — O(1) with no shifting |
| What is a covering index and when does it eliminate the double lookup? | A covering index includes all columns referenced in the query (SELECT + WHERE); the DB can answer entirely from the index B+Tree without touching the main table |
| Why should InnoDB primary keys be small sequential integers rather than UUIDs? | UUIDs are 16 bytes (vs 4 bytes for int) — all secondary indexes are larger. Random UUIDs cause random B+Tree splits fragmenting the clustered index; sequential ints fill leaves predictably |
| What is the maximum number of clustered indexes a table can have? | Exactly one — table rows can only be physically sorted in one order (the clustered index order) |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
