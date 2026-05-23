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
```java
s -= 1; // 0-indexed
int row = s / n;       // row from bottom
int col = s % n;
if (row % 2 == 1)      // odd rows go right-to-left
    col = n - 1 - col;
row = n - 1 - row;     // flip to grid indexing (row 0 = bottom → top)
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
```java
import java.util.*;

class Solution {

    // Minimum Knight Moves (LC 1197)
    public static int minKnightMoves(int x, int y) {
        x = Math.abs(x);
        y = Math.abs(y); // symmetry
        Deque<int[]> queue = new ArrayDeque<>();
        queue.addLast(new int[]{0, 0, 0});
        Set<String> visited = new HashSet<>();
        visited.add("0,0");
        int[][] dirs = {{2,1},{2,-1},{-2,1},{-2,-1},{1,2},{1,-2},{-1,2},{-1,-2}};
        while (!queue.isEmpty()) {
            int[] cur = queue.pollFirst();
            int r = cur[0], c = cur[1], steps = cur[2];
            if (r == x && c == y) return steps;
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                String key = nr + "," + nc;
                if (!visited.contains(key) && nr >= -1 && nc >= -1) {
                    visited.add(key);
                    queue.addLast(new int[]{nr, nc, steps + 1});
                }
            }
        }
        return -1;
    }

    // Snakes and Ladders (LC 909)
    public static int snakesAndLadders(int[][] board) {
        int n = board.length;

        // Convert cell number to (row, col)
        int[] cellToRC = new int[n * n + 1];
        // helper: returns {row, col} for cell s (1-indexed)
        // (inlined below for clarity)

        Set<Integer> visited = new HashSet<>();
        visited.add(1);
        Deque<int[]> queue = new ArrayDeque<>();
        queue.addLast(new int[]{1, 0}); // (cell, moves)
        while (!queue.isEmpty()) {
            int[] cur = queue.pollFirst();
            int cell = cur[0], moves = cur[1];
            for (int roll = 1; roll <= 6; roll++) {
                int next = cell + roll;
                if (next > n * n) break;
                int s = next - 1;
                int row = s / n;
                int col = s % n;
                if (row % 2 == 1) col = n - 1 - col;
                row = n - 1 - row;
                if (board[row][col] != -1) next = board[row][col]; // follow snake or ladder
                if (next == n * n) return moves + 1;
                if (!visited.contains(next)) {
                    visited.add(next);
                    queue.addLast(new int[]{next, moves + 1});
                }
            }
        }
        return -1;
    }

    // Shortest Path to Get All Keys (LC 864)
    public static int shortestPathAllKeys(String[] grid) {
        int rows = grid.length, cols = grid[0].length();
        int startR = 0, startC = 0;
        int totalKeys = 0;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                char ch = grid[r].charAt(c);
                if (ch == '@') { startR = r; startC = c; }
                if (Character.isLowerCase(ch)) totalKeys++;
            }
        }
        int allKeys = (1 << totalKeys) - 1;
        Deque<int[]> queue = new ArrayDeque<>();
        queue.addLast(new int[]{startR, startC, 0, 0}); // (r, c, keys_bitmask, steps)
        Set<String> visited = new HashSet<>();
        visited.add(startR + "," + startC + "," + 0);
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

        while (!queue.isEmpty()) {
            int[] cur = queue.pollFirst();
            int r = cur[0], c = cur[1], keys = cur[2], steps = cur[3];
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                char ch = grid[nr].charAt(nc);
                if (ch == '#') continue;
                int newKeys = keys;
                if (Character.isUpperCase(ch)) {         // lock
                    if (((keys >> (ch - 'A')) & 1) == 0) continue; // don't have key
                } else if (Character.isLowerCase(ch)) { // key
                    newKeys |= (1 << (ch - 'a'));
                }
                if (newKeys == allKeys) return steps + 1;
                String state = nr + "," + nc + "," + newKeys;
                if (!visited.contains(state)) {
                    visited.add(state);
                    queue.addLast(new int[]{nr, nc, newKeys, steps + 1});
                }
            }
        }
        return -1;
    }
}
```

---

### STAR Interview Framework

> **BFS on Implicit State Space + Bitmask State Encoding:** brute-force O(m × n × k!) → this approach O(m × n × 2^k) time, O(m × n × 2^k) space

**S:** "Given a maze with k keys and k locks (k ≤ 6), navigate from start '@' to collect all keys. Naive path enumeration O(m × n × k!) exceeds time limits at grid size 30×30 with 6 keys."
**T:** "Need O(m × n × 2^k) by encoding collected keys as a bitmask in the BFS state."
**A (60% of answer time):**
1. *Classify:* "Collecting items with unlock conditions on a grid — classic bitmask BFS signal: finite item set (k ≤ 6), each item changes future valid moves."
2. *Init:* "State = (row, col, keys_bitmask); visited = Set of these triples; queue starts at ('@', 0); allKeys = (1 << k) - 1."
3. *Loop/Step:* "For each (r, c, keys): expand 4 neighbours; skip walls '#'; skip locked doors if corresponding key bit not set; OR in new key bits on collection; return steps if keys == allKeys."
4. *Termination:* "BFS guarantees shortest path; finite state space m × n × 2^k ensures termination."
5. *Gotcha:* "Visited must track (r, c, keys_bitmask) not just (r, c) — the same cell is validly re-visited with a larger key set. Forgetting bitmask in the visited key causes TLE or incorrect pruning."
**R:** "O(m × n × 2^k) time, O(m × n × 2^k) space. At k=6, m=n=30: ~57,600 states vs 30! factorial explosion — tractable in < 1ms vs effectively infinite brute force."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Pure BFS (no bitmask) | No locks or dependencies between items | Ignores key/lock ordering; produces wrong paths through locked doors |
| DFS + backtracking | Small grids, enumerate all paths | Exponential in path length; doesn't guarantee shortest path |
| Dijkstra with state | Edge weights vary | All moves cost 1; BFS is O(V+E) vs Dijkstra's O((V+E) log V) overhead |

---

> **BFS on Board with Teleporters (Boustrophedon):** brute-force O(n² × 6^(n²)) → this approach O(n²) time, O(n²) space

**S:** "Given an n×n board with snakes and ladders encoded in zigzag numbering. Naive try-all-dice-sequences is O(6^(n²)) — infeasible for n=20."
**T:** "Need O(n²) by treating board as a graph and BFS on cell numbers — guaranteed shortest path in fewest dice rolls."
**A (60% of answer time):**
1. *Classify:* "Minimum moves on a board with teleporters — BFS on state (cell number), not grid coordinates."
2. *Init:* "visited = Set{1}; queue = [(cell=1, moves=0)]; goal = n×n."
3. *Loop/Step:* "For each cell: try rolls 1–6; convert next cell number to (row, col) via boustrophedon formula; if board[row][col] != -1, follow the snake/ladder; enqueue if unvisited."
4. *Termination:* "At most n² cells; each visited once; BFS terminates and returns minimum rolls."
5. *Gotcha:* "Boustrophedon row direction: even rows (from bottom) go left-to-right, odd rows right-to-left. Grid row 0 is the top, but board row 0 is the bottom — double flip needed. Off-by-one here produces wrong cells and fails silently."
**R:** "O(n²) time, O(n²) space. n=20 board: 400 states vs millions of dice-roll paths."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| DFS | Finding any path, not shortest | BFS guarantees minimum dice rolls; DFS may find longer paths first |
| Dijkstra | Varying edge weights | All dice rolls cost 1 move; BFS is simpler and O(n²) |

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
**Full STAR Story — "Bitmask State BFS: Eliminating Combinatorial Lock-Dependency Bugs":**
**S (20%):** "At a fintech platform, our payment-routing service processed transactions through a pipeline of 5 approval stages (KYC, fraud, AML, limit-check, FX). State management was ad-hoc — engineers manually tracked which checks had passed per transaction, causing 3 production incidents in 6 months where transactions were approved with missing stages."
**T:** "I was tasked with redesigning the state-tracking layer to guarantee every transaction passed every required stage before approval — measurable goal: zero approval-with-missing-stage incidents."
**A (60% — 'I' not 'we'):** "(1) I modelled each approval stage as a bitmask bit, reducing the 5-stage combination space from 5! orderings to 2^5 = 32 deterministic states. (2) I refactored the state store to persist a single bitmask per transaction, where each stage atomically ORed in its bit on completion. (3) I added a pre-approval guard: `if (bitmask != ALL_STAGES_MASK) reject()` — one line eliminating the class of bugs. (4) I validated correctness by replaying 90 days of historical transactions through the new state machine and confirmed zero false approvals."
**R (20%):** "Approval-with-missing-stage incidents dropped from 3 in 6 months to zero over the following year. Transaction processing throughput increased 15% because the guard short-circuited unnecessary DB queries. The bitmask pattern was later adopted across 4 other pipeline services."
*Works for LP questions on: Dive Deep, Insist on the Highest Standards, Earn Trust.*

---

## Flashcards

| Q | A |
|---|---|
| How do you handle symmetry in Minimum Knight Moves BFS? | Use `(abs(x), abs(y))` as the target — by symmetry, all four quadrants require the same number of moves. Also bound the search space to `r >= -1, c >= -1` to avoid infinite expansion. |
| What is the boustrophedon conversion in Snakes and Ladders? | Cell `s` (1-indexed): `s -= 1; row = s / n; col = s % n; if row % 2 == 1: col = n-1-col; row = n-1-row`. Even rows go left-to-right; odd rows right-to-left; row 0 is at the top of the grid (row n-1 is bottom). |
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
