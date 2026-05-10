# Day 65 — Graphs Advanced: BFS on Implicit State Graphs
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
BFS on implicit graphs: the graph is never explicitly stored — states are encoded as strings, tuples, or integers, and neighbours are generated on the fly. Minimum Knight Moves and Snakes and Ladders are standard BFS-on-grid-coordinates. Shortest Path to Get All Keys is BFS on a state = `(row, col, keys_bitmask)` — the classic multi-state BFS template.

---

## DSA (2 hours)
### Pattern: BFS on Implicit State Space + Bitmask State Encoding

**Minimum Knight Moves (LC 1197):**
From (0,0) to (x, y) on an infinite chessboard. Knight has 8 moves. BFS with visited set. Optimisation: by symmetry, `(x, y)` requires the same moves as `(|x|, |y|)`. Additionally, search space can be bounded — knight never needs to go far below or left of the origin, so bound to `[-1, max(|x|, |y|)+2]` range.

**Snakes and Ladders (LC 909):**
Boustrophedon (zigzag) numbering. BFS from cell 1 to cell n². At each state (cell number), try rolling 1–6; resolve snakes/ladders if the destination has one. Return minimum dice rolls. Key subtlety: when converting a cell number to (row, col), the row numbering is from the bottom, and alternating rows go right-to-left.

Boustrophedon conversion (cell `s` → `(row, col)`):
```python
s -= 1  # 0-indexed
row = s // n          # row from bottom
col = s % n
if row % 2 == 1:      # odd rows go right-to-left
    col = n - 1 - col
row = n - 1 - row     # flip to grid indexing (row 0 = bottom → top)
```

**Shortest Path to Get All Keys (LC 864):**
Grid with keys (a-f) and locks (A-F). BFS where state = `(row, col, keys_bitmask)`. Total states = m × n × 2^k where k = number of keys. A lock can only be passed if the corresponding key bit is set. When a key is collected, update bitmask. Start from `@`; goal = all k keys collected (bitmask = `(1 << k) - 1`).

**Trigger condition:**
- "minimum moves in an infinite/large grid with complex movement rules" → BFS from start; use visited set of (r, c) or just track visited; optionally exploit symmetry
- "minimum steps on a board with wormholes/teleporters" → BFS on cell number; resolve teleport on landing
- "minimum steps through a maze collecting items with unlock conditions" → BFS on `(r, c, bitmask)` state; k items → 2^k bitmask states

**Time complexity:** LC 1197: O(max(|x|, |y|)²) | LC 909: O(n²) | LC 864: O(m × n × 2^k)
**Space complexity:** O(max(|x|, |y|)²) / O(n²) / O(m × n × 2^k)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Knight Moves | 1197 | Medium | BFS from (0,0); exploit symmetry | Use `(abs(x), abs(y))`; bound search space; 8 knight directions |
| 2 | Snakes and Ladders | 909 | Medium | BFS on cell numbers; boustrophedon conversion | Convert cell# → (row, col) correctly; resolve snake/ladder before enqueueing |
| 3 | Shortest Path to Get All Keys | 864 | Hard | BFS on `(r, c, bitmask)` state | Total states = m×n×2^k; lock check = `(keys >> lock_idx) & 1`; goal = bitmask all 1s |

---

### Code Skeleton
```python
from collections import deque

# Minimum Knight Moves (LC 1197)
def minKnightMoves(x, y):
    x, y = abs(x), abs(y)   # symmetry
    queue = deque([(0, 0, 0)])
    visited = {(0, 0)}
    dirs = [(2,1),(2,-1),(-2,1),(-2,-1),(1,2),(1,-2),(-1,2),(-1,-2)]
    while queue:
        r, c, steps = queue.popleft()
        if r == x and c == y: return steps
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if (nr, nc) not in visited and nr >= -1 and nc >= -1:
                visited.add((nr, nc))
                queue.append((nr, nc, steps + 1))
    return -1

# Snakes and Ladders (LC 909)
def snakesAndLadders(board):
    n = len(board)

    def cell_to_rc(s):
        s -= 1
        row, col = s // n, s % n
        if row % 2 == 1:
            col = n - 1 - col
        return n - 1 - row, col

    visited = {1}
    queue = deque([(1, 0)])   # (cell, moves)
    while queue:
        cell, moves = queue.popleft()
        for roll in range(1, 7):
            next_cell = cell + roll
            if next_cell > n * n: break
            r, c = cell_to_rc(next_cell)
            if board[r][c] != -1:
                next_cell = board[r][c]   # follow snake or ladder
            if next_cell == n * n: return moves + 1
            if next_cell not in visited:
                visited.add(next_cell)
                queue.append((next_cell, moves + 1))
    return -1

# Shortest Path to Get All Keys (LC 864)
def shortestPathAllKeys(grid):
    rows, cols = len(grid), len(grid[0])
    start_r = start_c = 0
    total_keys = 0
    for r in range(rows):
        for c in range(cols):
            ch = grid[r][c]
            if ch == '@': start_r, start_c = r, c
            if ch.islower(): total_keys += 1

    all_keys = (1 << total_keys) - 1
    queue = deque([(start_r, start_c, 0, 0)])   # (r, c, keys_bitmask, steps)
    visited = {(start_r, start_c, 0)}
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    while queue:
        r, c, keys, steps = queue.popleft()
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] != '#':
                ch = grid[nr][nc]
                new_keys = keys
                if ch.isupper():                       # lock
                    if not (keys >> (ord(ch) - ord('A'))) & 1:
                        continue                        # don't have the key
                elif ch.islower():                     # key
                    new_keys |= (1 << (ord(ch) - ord('a')))
                if new_keys == all_keys: return steps + 1
                state = (nr, nc, new_keys)
                if state not in visited:
                    visited.add(state)
                    queue.append((nr, nc, new_keys, steps + 1))
    return -1
```

---

### Edge Cases to Trace Before Coding
- LC 1197: x=0, y=0 → return 0; x=1, y=1 → cannot be reached in 1 move; requires at least 2 moves (classic tricky case)
- LC 909: no snakes or ladders → pure BFS dice roll; starting cell has a ladder → resolve immediately; cell n² is the goal, don't enqueue it, return moves+1
- LC 864: no keys → return 0 immediately if start is at the end (or check if any cell is unreachable); k=6 → 64 bitmask states per cell → still tractable (m×n ≤ 900 → 57,600 total states)

---

### Interview Pattern Drill

| Implicit graph type | State representation | Neighbour generation | Visited key |
|--------------------|---------------------|---------------------|------------|
| Knight moves | `(row, col)` | 8 knight offsets | `(row, col)` |
| Board with teleporters | `cell_number` | `cell + 1..6`, resolve teleport | `cell_number` |
| Maze with keys+locks | `(row, col, keys_bitmask)` | 4-directional; lock check; key collection | `(row, col, keys_bitmask)` |
| Word ladder | `word_string` | All valid 1-char substitutions | `word_string` |

---

## System Design (1 hour)
### Topic: Key-Value Store — Distributed Design and Consistent Hashing

**Why distributed?**
A single-node KV store is bounded by one machine's RAM and disk. A distributed KV store shards data across N nodes and replicates each shard to tolerate failures.

**Sharding with consistent hashing:**
In naive hash sharding, `key % N` means adding/removing a node re-shards almost every key. Consistent hashing arranges nodes on a ring and assigns keys to the nearest clockwise node. Adding a node only re-shards keys between the new node and its predecessor.

```
Ring: [Node A] --- [Node B] --- [Node C] --- [Node A]
Key K → hash(K) → falls on ring → first node clockwise = responsible node
```

**Virtual nodes (VNodes):**
If one physical node owns a small arc of the ring, it gets less data. To balance load: each physical node is represented by 150–200 virtual nodes scattered around the ring. This ensures each physical node owns roughly 1/N of the keyspace even with unequal placement.

**Replication:**
Each key is stored on N nodes (the responsible node + N-1 successors on the ring). `N = 3` is the standard (e.g., Dynamo, Cassandra). This means the system tolerates up to N-1 node failures without data loss.

**Quorum reads and writes (R + W > N):**
- Write: send to all N replicas; wait for W acknowledgements → success
- Read: send to R replicas; return the value with the highest vector clock

Common configurations (with N=3):
| Config | W | R | Guarantee |
|--------|---|---|-----------|
| Strong consistency | 2 | 2 | R + W = 4 > 3; any read sees the last write |
| Write-optimised | 1 | 3 | Fast writes; reads guaranteed fresh |
| Read-optimised | 3 | 1 | Slower writes; reads always fresh |

**Hinted handoff:**
If the target replica for a write is temporarily down, another node stores the write with a "hint" that it belongs to the unavailable node. When the node recovers, the hinted data is forwarded. Maintains write availability during transient failures without waiting for full recovery.

**Interview talking point:** "In Dynamo-style systems, we sacrifice strong consistency for availability (AP in CAP theorem). With N=3, W=2, R=2, the system can tolerate 1 node failure for both reads and writes while ensuring R+W > N for eventual consistency. A client may see stale data if it reads from a replica that hasn't received the latest write — resolved by comparing vector clocks and using read repair."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 1197 Minimum Knight Moves (target: 15 min)
- **Medium 2:** LC 909 Snakes and Ladders (target: 18 min)
- **Hard:** LC 864 Shortest Path to Get All Keys (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you designed a system that needed to balance availability and consistency — what was the failure mode you were protecting against?
- Leadership principle: Dive Deep

---

## Flashcards

| Q | A |
|---|---|
| How do you handle symmetry in Minimum Knight Moves BFS? | Use `(abs(x), abs(y))` as the target — by symmetry, all four quadrants require the same number of moves. Also bound the search space to `r >= -1, c >= -1` to avoid infinite expansion. |
| What is the boustrophedon conversion in Snakes and Ladders? | Cell `s` (1-indexed): `s -= 1; row = s // n; col = s % n; if row % 2 == 1: col = n-1-col; row = n-1-row`. Even rows go left-to-right; odd rows right-to-left; row 0 is at the top of the grid (row n-1 is bottom). |
| What is the state space size for Shortest Path to Get All Keys? | `m × n × 2^k` where k = number of keys (at most 6 → 2^6 = 64 bitmask states per cell). Visited set uses `(row, col, keys_bitmask)` triples. |
| How does consistent hashing reduce resharding when adding a node? | Keys are assigned to the nearest clockwise node on a hash ring. Adding a node only moves keys between the new node and its predecessor — O(keys/N) moved vs O(keys) in naive `key % N` hashing. |
| What is the Quorum condition R + W > N and why does it guarantee consistency? | With N replicas, if W nodes must ack a write and R nodes must respond to a read, then R + W > N means the read set and write set must overlap by at least one node — that node has the latest write, ensuring the read sees it. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 15/18/25 min, no hints)
- [ ] Rewrote bitmask BFS template and boustrophedon conversion from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can draw consistent hashing ring and explain quorum cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
