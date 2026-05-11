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
- STAR prompt: Tell me about a time you designed a system where you needed instant access by multiple different criteria simultaneously — analogous to needing both LRU eviction order and O(1) key lookup in LRU Cache.
- Leadership principle: Think Big

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
