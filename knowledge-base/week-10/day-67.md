# Day 67 — Graphs Advanced: Topological DP & Critical Path
**Week 10 | Phase 1: DSA Mastery | Month 3**

## Focus
Sequence Reconstruction verifies that a given topological order is the unique order consistent with a set of sequences. Parallel Courses III computes the critical path length — the minimum time to complete all courses using topological DP. Largest Color Value in a Directed Graph is a topological DP where each node tracks the maximum frequency of each colour on any path reaching it.

---

## DSA (2 hours)
### Pattern: Unique Topological Sort Verification + Critical Path DP + Color Frequency DP on DAG

**Sequence Reconstruction (LC 444):**
Given `org` (a permutation of 1..n) and `seqs` (subsequences), determine if `org` is the UNIQUE topological order consistent with all sequences. Condition for uniqueness: at every step in Kahn's BFS, the in-degree-0 queue must have exactly 1 node. If at any point the queue has ≥ 2 nodes, the topological order is not unique. Also verify the reconstructed sequence equals `org`.

**Parallel Courses III (LC 2050):**
Each course has a duration and prerequisites. Find the minimum time to complete all courses if you can take any independent courses simultaneously. This is the critical path problem: for each course v, `time[v] = duration[v] + max(time[u] for u in prerequisites)`. Compute with Kahn's topological sort, updating `time[v]` when processing each edge. Answer = `max(time)`.

**Largest Color Value in a Directed Graph (LC 1857):**
Each node has a colour ('a'–'z'). Find the maximum frequency of any single colour on any path in the directed graph. If a cycle exists, return -1. DP: `dp[node][c]` = max count of colour c on any path ending at node. Propagate in topological order. When processing edge `(u, v)`: for each colour c, `dp[v][c] = max(dp[v][c], dp[u][c] + (1 if color[v] == c else 0))`. Answer = max over all `dp[node][color[node]]`.

**Trigger condition:**
- "is there a unique topological order?" → Kahn's; unique if and only if queue always has exactly 1 element
- "minimum time to complete all tasks with parallelism and dependencies?" → topological DP; `time[v] = duration[v] + max(time of prereqs)`
- "maximum frequency of a property on any path in a DAG?" → topological DP with per-colour count array per node

**Time complexity:** LC 444: O(n + total_seq_length) | LC 2050: O(V+E) | LC 1857: O(26 × (V+E))
**Space complexity:** O(n) / O(V+E) / O(26 × V)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Sequence Reconstruction | 444 | Medium | Kahn's uniqueness check | Queue must have exactly 1 node at each Kahn's step; also verify result == org |
| 2 | Parallel Courses III | 2050 | Medium | Topological DP critical path | `time[v] = duration[v] + max(time[prereq])`; answer = max(time) |
| 3 | Largest Color Value in Directed Graph | 1857 | Hard | Topological DP with 26-colour count array | `dp[v][c]` = max count of colour c ending at v; propagate via Kahn's; cycle → -1 |

---

### Code Skeleton
```python
from collections import defaultdict, deque

# Sequence Reconstruction (LC 444)
def sequenceReconstruction(org, seqs):
    graph = defaultdict(set)
    in_degree = defaultdict(int)
    nodes = set()
    for seq in seqs:
        for node in seq: nodes.add(node)
        for i in range(len(seq) - 1):
            u, v = seq[i], seq[i+1]
            if v not in graph[u]:
                graph[u].add(v)
                in_degree[v] += 1
    if set(org) != nodes: return False
    queue = deque(n for n in nodes if in_degree[n] == 0)
    idx = 0
    while queue:
        if len(queue) > 1: return False   # not unique
        node = queue.popleft()
        if idx >= len(org) or org[idx] != node: return False
        idx += 1
        for nb in graph[node]:
            in_degree[nb] -= 1
            if in_degree[nb] == 0:
                queue.append(nb)
    return idx == len(org)

# Parallel Courses III (LC 2050)
def minimumTime(n, relations, time):
    graph = defaultdict(list)
    in_degree = [0] * (n + 1)
    for prev, next_ in relations:
        graph[prev].append(next_)
        in_degree[next_] += 1
    finish_time = [0] * (n + 1)
    for i in range(1, n + 1):
        finish_time[i] = time[i - 1]
    queue = deque(i for i in range(1, n + 1) if in_degree[i] == 0)
    while queue:
        u = queue.popleft()
        for v in graph[u]:
            finish_time[v] = max(finish_time[v], finish_time[u] + time[v - 1])
            in_degree[v] -= 1
            if in_degree[v] == 0:
                queue.append(v)
    return max(finish_time[1:])

# Largest Color Value in Directed Graph (LC 1857)
def largestPathValue(colors, edges):
    n = len(colors)
    graph = defaultdict(list)
    in_degree = [0] * n
    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1
    # dp[node][c] = max count of colour c on any path ending at node
    dp = [[0] * 26 for _ in range(n)]
    for i in range(n):
        dp[i][ord(colors[i]) - ord('a')] = 1   # each node contributes its own colour
    queue = deque(i for i in range(n) if in_degree[i] == 0)
    processed = 0
    ans = 0
    while queue:
        u = queue.popleft()
        processed += 1
        ans = max(ans, max(dp[u]))
        for v in graph[u]:
            for c in range(26):
                dp[v][c] = max(dp[v][c], dp[u][c] + (1 if ord(colors[v]) - ord('a') == c else 0))
            in_degree[v] -= 1
            if in_degree[v] == 0:
                queue.append(v)
    return ans if processed == n else -1   # cycle detected if not all processed
```

---

### Edge Cases to Trace Before Coding
- LC 444: `seqs` contains a node not in `org` → return False; `seqs` are consistent but don't enforce a unique order → queue has > 1 element → return False; single node `org = [1]` with `seqs = [[1]]` → trivially True
- LC 2050: no prerequisites (independent courses) → each course finishes at its own duration; all courses sequential (chain) → critical path = sum of all durations
- LC 1857: cycle in graph → `processed < n` after Kahn's → return -1; all nodes have the same colour → answer = length of the longest path

---

### Interview Pattern Drill

| Pattern | Kahn's modification | DP state | Answer |
|---------|--------------------|---------:|--------|
| Standard topological sort | None | — | Ordering |
| Unique topological sort | Check `len(queue) == 1` each step | — | Boolean |
| Critical path | `time[v] = max(time[v], time[u] + dur[v])` | `time[v]` | `max(time)` |
| Color value DP | `dp[v][c] = max(dp[v][c], dp[u][c] + indicator)` | `dp[v][26]` | `max(dp[node][color[node]])` |

---

## System Design (1 hour)
### Topic: Key-Value Store — Fault Tolerance, Gossip Protocol, and Anti-Entropy

**The fault tolerance problem:**
In a distributed KV store with N=3 replicas, a node may go down for hours. During that time:
- Writes to the unavailable node are stored as hints on other nodes (hinted handoff)
- On recovery, the node needs to reconcile its state with the rest of the cluster

**Gossip protocol:**
Nodes periodically exchange cluster state with a few random peers. Each node knows:
- Which other nodes are alive/dead (failure detection)
- Each node's ring membership and VNode assignments

```
Every T seconds:
  Node A randomly picks 2 peers (B, C)
  A sends its "membership list" to B and C
  B and C merge A's info with their own (take the max version/timestamp per field)
  A receives B's and C's lists and merges similarly
```
This propagates information exponentially fast: after O(log N) rounds, all nodes have converged on the same cluster state. Gossip is fault-tolerant because it doesn't depend on a single coordinator.

**Anti-entropy with Merkle trees:**
When a node recovers from failure, it needs to sync its data with current replicas without transferring the full dataset. Merkle trees make this efficient:
1. Each node builds a Merkle tree over its data (leaf = hash of key-value pair; parent = hash of children)
2. Two nodes compare Merkle tree roots — if roots match, all data is consistent
3. If roots differ, they traverse the tree to find the specific subtree with differences — only those key ranges need syncing

```
Full dataset: 100 GB
Difference: 10 MB
Merkle tree sync: transfers 10 MB + O(log n) comparison messages
Naive sync: transfers 100 GB
```

**Failure detection:**
- Heartbeat: each node sends periodic pings; if no response within timeout → suspected dead
- Phi Accrual Failure Detector (used in Cassandra): computes a probability score of node failure based on inter-arrival times of heartbeats, rather than a hard timeout. More adaptive to network conditions.

**Sloppy quorum and hinted handoff together:**
- If the W required replicas include a down node, write to the next available node with a "hint" (destination node + key)
- The hint-holder monitors the down node; on recovery, forwards the hinted writes
- This maintains write availability during failures without sacrificing eventual consistency

**Interview talking point:** "When a node in a Dynamo-style cluster comes back online after 6 hours, it uses Merkle tree comparison to find which key ranges differ from its peers — typically a small fraction of total data. It syncs only the changed ranges (anti-entropy). Meanwhile, hinted handoff delivers the writes that occurred during its downtime. After both steps, the node is fully consistent."

---

## Assessment / Mock (1 hour)
### Activity: Timed mock — pick one from each difficulty
- **Medium 1:** LC 444 Sequence Reconstruction (target: 18 min)
- **Medium 2:** LC 2050 Parallel Courses III (target: 15 min)
- **Hard:** LC 1857 Largest Color Value in Directed Graph (target: 25 min)

After each: state time complexity, space complexity, and one edge case aloud.

---

## Behavioral (30 min)
- STAR prompt: Describe a time you had to coordinate multiple parallel workstreams with dependencies — how did you identify and protect the critical path?
- Leadership principle: Deliver Results

---

## Flashcards

| Q | A |
|---|---|
| How do you verify a topological order is unique using Kahn's? | At each BFS step, the in-degree-0 queue must have exactly 1 node. If ever `len(queue) > 1`, multiple valid orderings exist — the sequence is not uniquely reconstructable. |
| What is the topological DP formula for the critical path (Parallel Courses III)? | `finish_time[v] = time[v] + max(finish_time[u] for u in prerequisites of v)`. Process in topological order; answer = `max(finish_time)`. |
| How does Largest Color Value DP propagate? | `dp[v][c] = max(dp[v][c], dp[u][c] + (1 if color[v] == c else 0))` for each predecessor u. Initialize `dp[v][color[v]] = 1` for each node. Cycle → return -1 (not all nodes processed). |
| What is gossip protocol and why is it fault-tolerant? | Nodes periodically exchange cluster state with random peers. Information propagates in O(log N) rounds. Fault-tolerant because there is no single coordinator — any subset of alive nodes can continue gossiping. |
| What does a Merkle tree enable in a distributed KV store? | Efficient anti-entropy: two nodes compare Merkle tree roots; if equal, all data matches. If not, they traverse to find differing subtrees — only those key ranges need sync, not the full dataset. |

---

## Checklist
- [ ] Solved all 3 problems (timed — 18/15/25 min, no hints)
- [ ] Rewrote topological DP and color-value DP templates from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design — can explain gossip + Merkle tree anti-entropy cold
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
