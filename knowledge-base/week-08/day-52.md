# Day 52 — Graphs: BFS Shortest Path on Grids & Word Graphs
**Week 08 | Phase 1: DSA Mastery | Month 2**

## Focus
BFS guarantees the shortest path in unweighted graphs. Multi-source BFS initialises the queue with all starting nodes simultaneously. Word Ladder generalises BFS to an implicit word graph where edges are generated on-the-fly.

---

## DSA (2 hours)
### Pattern: BFS Shortest Path — Multi-Source, Grid, and Implicit Graphs

**Rotting Oranges (LC 994):**
Multi-source BFS: initialise the queue with ALL rotten oranges at time 0. BFS spreads rot simultaneously from all sources — each level represents one time unit. After BFS, if any fresh orange remains → return -1; else return the number of levels processed.

**Shortest Path in Binary Matrix (LC 1091):**
Standard BFS from `(0,0)`. Move in 8 directions (including diagonals). Level = path length. Goal: reach `(n-1, n-1)`. Path length = number of cells visited (not edges) — answer is the level when the goal is dequeued + 1 (include the starting cell).

**Word Ladder (LC 127):**
Implicit graph BFS: each word is a node; two words are adjacent if they differ by exactly one character. Generate neighbours by replacing each character position with all 26 letters and checking against a word set. BFS from `beginWord` to `endWord`; return the shortest transformation sequence length.

**Trigger condition:**
- "minimum time/steps for N things spreading simultaneously" → multi-source BFS (all sources in queue at t=0)
- "shortest path in unweighted grid" → BFS with 4 or 8 directions
- "minimum word transformations" → BFS on implicit word graph; generate neighbours by character substitution

**Time complexity:** LC 994: O(m×n) | LC 1091: O(n²) | LC 127: O(M²×N) where M=word length, N=wordList size
**Space complexity:** O(m×n) / O(n²) / O(M²×N)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Rotting Oranges | 994 | Medium | Multi-source BFS from all rotten cells | Init queue with all rotten cells; BFS levels = time; if fresh remain → -1 |
| 2 | Shortest Path in Binary Matrix | 1091 | Medium | BFS on grid, 8-directional | Dequeue and enqueue 8 neighbours; mark visited on enqueue not dequeue to prevent re-queuing |
| 3 | Word Ladder | 127 | Hard | BFS on implicit word graph | Generate all possible 1-char substitutions; check against word set O(1); BFS levels = sequence length |

---

### Code Skeleton
```python
from collections import deque

# Rotting Oranges (LC 994)
def orangesRotting(grid):
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2: queue.append((r, c))
            elif grid[r][c] == 1: fresh += 1
    if fresh == 0: return 0
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]
    minutes = 0
    while queue:
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for dr, dc in dirs:
                nr, nc = r+dr, c+dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    queue.append((nr, nc))
        minutes += 1
    return minutes - 1 if fresh == 0 else -1
    # -1 because we increment minutes after the last level even when queue empties

# Shortest Path in Binary Matrix (LC 1091)
def shortestPathBinaryMatrix(grid):
    n = len(grid)
    if grid[0][0] == 1 or grid[n-1][n-1] == 1: return -1
    dirs = [(-1,-1),(-1,0),(-1,1),(0,-1),(0,1),(1,-1),(1,0),(1,1)]
    queue = deque([(0, 0, 1)])   # (row, col, path_length)
    grid[0][0] = 1               # mark visited
    while queue:
        r, c, dist = queue.popleft()
        if r == n-1 and c == n-1: return dist
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < n and 0 <= nc < n and grid[nr][nc] == 0:
                grid[nr][nc] = 1   # mark visited on enqueue
                queue.append((nr, nc, dist+1))
    return -1

# Word Ladder (LC 127)
def ladderLength(beginWord, endWord, wordList):
    word_set = set(wordList)
    if endWord not in word_set: return 0
    queue = deque([(beginWord, 1)])   # (word, sequence_length)
    visited = {beginWord}
    while queue:
        word, length = queue.popleft()
        for i in range(len(word)):
            for ch in 'abcdefghijklmnopqrstuvwxyz':
                candidate = word[:i] + ch + word[i+1:]
                if candidate == endWord: return length + 1
                if candidate in word_set and candidate not in visited:
                    visited.add(candidate)
                    queue.append((candidate, length + 1))
    return 0
```

---

### Edge Cases to Trace Before Coding
- LC 994: no fresh oranges at start → return 0 immediately; fresh oranges unreachable from any rotten → return -1; grid with a single fresh, single rotten, not adjacent → -1
- LC 1091: start or end cell is `1` (blocked) → return -1 immediately; n=1 and grid[0][0]=0 → return 1 (start = end)
- LC 127: `endWord` not in `wordList` → return 0; `beginWord == endWord` → problem guarantees they differ, but handle gracefully

---

### Interview Pattern Drill
| BFS variant | Queue initialisation | Level tracking | Goal |
|-------------|---------------------|---------------|------|
| Single-source BFS | One start node | `level_size = len(queue)` | Reach target |
| Multi-source BFS | All source nodes at t=0 | Same level tracking | Explore from all at once |
| Implicit graph BFS | Start state | Generate neighbours on-the-fly | Reach goal state |

---

## System Design (1 hour)
### Topic: DNS — How Domain Resolution Works

**The DNS hierarchy:**
```
Root Nameservers (13 clusters, Anycast)
  └─ TLD Nameservers (.com, .org, .io, …)
       └─ Authoritative Nameservers (owned by the domain's registrar/DNS provider)
```

**Full resolution of `api.example.com`:**
1. Browser checks its cache → miss
2. OS checks `/etc/hosts` and resolver cache → miss
3. OS forwards to **Recursive Resolver** (ISP's resolver or 8.8.8.8 or 1.1.1.1)
4. Resolver asks a **Root Nameserver**: "Who handles `.com`?"
5. Root returns the address of the `.com` TLD nameserver
6. Resolver asks the `.com` TLD: "Who handles `example.com`?"
7. TLD returns the authoritative nameserver for `example.com`
8. Resolver asks the **authoritative nameserver**: "What is the IP of `api.example.com`?"
9. Authoritative returns the A record (IPv4) or AAAA record (IPv6)
10. Resolver caches the result for the record's TTL; browser connects to the IP

**Common DNS record types:**
| Record | Purpose | Example |
|--------|---------|---------|
| A | Hostname → IPv4 | `api.example.com → 93.184.216.34` |
| AAAA | Hostname → IPv6 | `api.example.com → 2001:db8::1` |
| CNAME | Alias → another hostname | `www.example.com → example.com` |
| MX | Mail server for domain | `example.com → mail.example.com` |
| NS | Authoritative nameservers | `example.com → ns1.cloudflare.com` |
| TXT | Arbitrary text (SPF, DKIM, verification) | `"v=spf1 include:..."` |

**DNS TTL and propagation:**
- TTL (Time To Live) is set by the domain owner on each record
- Short TTL (60 s): changes propagate quickly; more resolver load; use during migrations or failover
- Long TTL (86 400 s = 1 day): less resolver load; changes take up to 24 h to propagate globally
- **Before a migration:** lower TTL to 60 s at least 2× the previous TTL in advance, so resolvers refresh before you change the record

**DNS caching at each layer:**
- Root and TLD nameservers: high TTL (days) — changes rarely
- Authoritative: set by you — your application's TTL
- Recursive resolver: respects TTL, often floors at 30–60 s
- OS: respects TTL, platform-dependent minimum
- Browser: often ignores OS TTL and caps at ~60 s internally

**Interview talking point:** "If asked how to design a blue-green deployment with zero downtime using DNS, answer: run two production environments (blue = current, green = new). Lower DNS TTL to 60 s 48 hours before deploy. Deploy new version to green. Test green thoroughly. Update DNS A record to point to green's IP. Clients using the old IP (blue) will continue working until their TTL expires — at most 60 s. Blue stays running as a hot standby for 5 minutes. After TTL elapses, raise TTL back to 3600 s."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to trace through multiple layers of a system to find the root cause of a problem — analogous to following the DNS resolution chain to understand where a failure occurred.
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How does multi-source BFS for Rotting Oranges differ from single-source BFS? | All rotten cells are enqueued simultaneously at time 0; BFS levels represent time steps — rot spreads in parallel from all sources, not sequentially |
| Why mark cells visited on enqueue (not dequeue) in BFS? | Marking on dequeue allows the same cell to be enqueued multiple times before it's processed, causing O(n²) work; marking on enqueue ensures each cell enters the queue exactly once |
| How does Word Ladder generate neighbours in the implicit graph? | For each of the M character positions, substitute all 26 letters; check each candidate against the word set (O(1)); valid unseen words become BFS neighbours |
| What are the 4 steps in a full DNS resolution? | (1) Recursive resolver → Root (returns TLD server). (2) Resolver → TLD server (returns authoritative server). (3) Resolver → Authoritative server (returns IP). (4) Resolver caches and returns IP to client |
| Why lower DNS TTL before a production migration? | Resolvers cache DNS records for the full TTL duration; lowering TTL ensures the old record expires quickly after the switch so traffic drains from the old server within seconds instead of hours |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
