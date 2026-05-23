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
```java
import java.util.*;

class Solution {

    // Sequence Reconstruction (LC 444)
    public static boolean sequenceReconstruction(int[] org, List<List<Integer>> seqs) {
        Map<Integer, Set<Integer>> graph = new HashMap<>();
        Map<Integer, Integer> inDegree = new HashMap<>();
        Set<Integer> nodes = new HashSet<>();
        for (List<Integer> seq : seqs) {
            for (int node : seq) {
                nodes.add(node);
                graph.putIfAbsent(node, new HashSet<>());
                inDegree.putIfAbsent(node, 0);
            }
            for (int i = 0; i < seq.size() - 1; i++) {
                int u = seq.get(i), v = seq.get(i + 1);
                if (!graph.get(u).contains(v)) {
                    graph.get(u).add(v);
                    inDegree.put(v, inDegree.get(v) + 1);
                }
            }
        }
        Set<Integer> orgSet = new HashSet<>();
        for (int x : org) orgSet.add(x);
        if (!orgSet.equals(nodes)) return false;
        Deque<Integer> queue = new ArrayDeque<>();
        for (int n : nodes) {
            if (inDegree.get(n) == 0) queue.addLast(n);
        }
        int idx = 0;
        while (!queue.isEmpty()) {
            if (queue.size() > 1) return false; // not unique
            int node = queue.pollFirst();
            if (idx >= org.length || org[idx] != node) return false;
            idx++;
            for (int nb : graph.getOrDefault(node, Collections.emptySet())) {
                inDegree.put(nb, inDegree.get(nb) - 1);
                if (inDegree.get(nb) == 0) queue.addLast(nb);
            }
        }
        return idx == org.length;
    }

    // Parallel Courses III (LC 2050)
    public static int minimumTime(int n, int[][] relations, int[] time) {
        List<Integer>[] graph = new ArrayList[n + 1];
        for (int i = 0; i <= n; i++) graph[i] = new ArrayList<>();
        int[] inDegree = new int[n + 1];
        for (int[] rel : relations) {
            graph[rel[0]].add(rel[1]);
            inDegree[rel[1]]++;
        }
        int[] finishTime = new int[n + 1];
        for (int i = 1; i <= n; i++) finishTime[i] = time[i - 1];
        Deque<Integer> queue = new ArrayDeque<>();
        for (int i = 1; i <= n; i++) {
            if (inDegree[i] == 0) queue.addLast(i);
        }
        while (!queue.isEmpty()) {
            int u = queue.pollFirst();
            for (int v : graph[u]) {
                finishTime[v] = Math.max(finishTime[v], finishTime[u] + time[v - 1]);
                inDegree[v]--;
                if (inDegree[v] == 0) queue.addLast(v);
            }
        }
        int ans = 0;
        for (int i = 1; i <= n; i++) ans = Math.max(ans, finishTime[i]);
        return ans;
    }

    // Largest Color Value in Directed Graph (LC 1857)
    public static int largestPathValue(String colors, int[][] edges) {
        int n = colors.length();
        List<Integer>[] graph = new ArrayList[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        int[] inDegree = new int[n];
        for (int[] e : edges) {
            graph[e[0]].add(e[1]);
            inDegree[e[1]]++;
        }
        // dp[node][c] = max count of colour c on any path ending at node
        int[][] dp = new int[n][26];
        for (int i = 0; i < n; i++) {
            dp[i][colors.charAt(i) - 'a'] = 1; // each node contributes its own colour
        }
        Deque<Integer> queue = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            if (inDegree[i] == 0) queue.addLast(i);
        }
        int processed = 0;
        int ans = 0;
        while (!queue.isEmpty()) {
            int u = queue.pollFirst();
            processed++;
            for (int c = 0; c < 26; c++) ans = Math.max(ans, dp[u][c]);
            for (int v : graph[u]) {
                for (int c = 0; c < 26; c++) {
                    int add = (colors.charAt(v) - 'a' == c) ? 1 : 0;
                    dp[v][c] = Math.max(dp[v][c], dp[u][c] + add);
                }
                inDegree[v]--;
                if (inDegree[v] == 0) queue.addLast(v);
            }
        }
        return processed == n ? ans : -1; // cycle detected if not all processed
    }
}
```

---

### STAR Interview Framework

> **Unique Topological Sort Verification:** brute-force O(n! orderings) → this approach O(n + total_seq_length) time, O(n) space

**S:** "Given a permutation org of 1..n and a list of constraint sequences, determine if org is the only valid topological ordering. Brute force enumerates O(n!) orderings — infeasible for n=10,000."
**T:** "Need O(n + total_seq_length) by running Kahn's BFS with a uniqueness check at each step."
**A (60% of answer time):**
1. *Classify:* "'Is there a unique topological order?' — Kahn's uniqueness signal: at each step, the in-degree-zero queue must have exactly 1 element."
2. *Init:* "Build directed graph and in-degree map from all sequences; check node sets match org."
3. *Loop/Step:* "At each Kahn's step: if queue.size() > 1 → return false (ambiguous); dequeue the single node; verify it matches org[idx]; decrement neighbours' in-degrees."
4. *Termination:* "If idx == org.length after processing all nodes → unique and consistent."
5. *Gotcha:* "Two separate conditions must both pass: (a) queue never exceeds size 1, AND (b) the dequeued sequence exactly matches org. A sequence might be unique but reconstruct a different permutation — always verify against org."
**R:** "O(n + total_seq_length) time, O(n) space. Handles n=10,000 with 10,000 sequences in milliseconds."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| DFS-based topological sort | General DAG ordering | DFS doesn't naturally expose uniqueness — requires extra bookkeeping |
| Brute force all orderings | n ≤ 8 | O(n!) explosion; infeasible for n > 12 |

---

> **Topological DP — Critical Path:** brute-force O(2^V path enumeration) → this approach O(V+E) time, O(V+E) space

**S:** "Given n courses with durations and prerequisite dependencies (DAG), find the minimum time to complete all courses when independent courses can run in parallel. Naive simulation of all schedule permutations is exponential."
**T:** "Need O(V+E) by computing finish times via topological DP — each course's start time is the maximum finish time of its prerequisites."
**A (60% of answer time):**
1. *Classify:* "'Minimum time with parallelism and dependencies' — critical path signal; topological DP where finish_time[v] = duration[v] + max(finish_time[prereqs])."
2. *Init:* "finishTime[v] = time[v] for all v (no prerequisites case); in-degree array; enqueue all in-degree-0 nodes."
3. *Loop/Step:* "When processing edge (u → v): finishTime[v] = max(finishTime[v], finishTime[u] + time[v-1])."
4. *Termination:* "After Kahn's completes, answer = max(finishTime)."
5. *Gotcha:* "finishTime[v] must be initialised to time[v] (not 0) before processing edges. If initialised to 0, courses with no prerequisites finish at 0 — wrong. Also the update adds time[v-1] not time[u] — you're adding v's duration, not u's."
**R:** "O(V+E) time, O(V+E) space. n=50,000 courses processed in < 100ms; exponential simulation is completely infeasible."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| DFS post-order DP | Same complexity; recursive | Recursive DFS risks stack overflow for chains of 50,000 nodes; Kahn's is iterative |
| Bellman-Ford (longest path) | General graphs with cycles | This is a DAG; Bellman-Ford is O(V×E) — unnecessary overhead |

---

> **Color Frequency DP on DAG:** brute-force O(26 × V! path enumeration) → this approach O(26 × (V+E)) time, O(26 × V) space

**S:** "Given a directed graph where each node has a colour ('a'–'z'), find the maximum frequency of any single colour on any path. Brute force enumerates all paths — exponential."
**T:** "Need O(26 × (V+E)) by tracking a per-colour count array at each node during topological sort; return -1 if a cycle exists."
**A (60% of answer time):**
1. *Classify:* "'Maximum frequency of a property on any path in a DAG' — topological DP with per-node count array per colour."
2. *Init:* "dp[v][c] = 0 for all; dp[v][color[v]-'a'] = 1 for each node; Kahn's queue with in-degree-0 nodes."
3. *Loop/Step:* "For edge (u → v), for each colour c: dp[v][c] = max(dp[v][c], dp[u][c] + (1 if color[v]=='c' else 0))."
4. *Termination:* "After processing, if processed < n → cycle detected → return -1. Answer = max over all dp[node][color[node]]."
5. *Gotcha:* "The answer comes from dp[node][color[node]] — the node's own colour at the node itself — not max over all 26 colours at all nodes. This is because you want the maximum frequency on a path ending here, counting only one specific colour per path."
**R:** "O(26 × (V+E)) time, O(26 × V) space. V=100,000 nodes: ~2.6M operations vs exponential path enumeration."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| 26 separate DFS runs (one per colour) | Conceptually simpler | Same asymptotic cost but 26× constant overhead; combined Kahn's does it in one pass |
| Floyd-Warshall | All-pairs path properties, n ≤ 200 | n up to 100,000; O(n³) = 10^15 operations — impossible |

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
**Full STAR Story — "Critical Path DP: Shipping a Multi-Team Feature by Protecting the Longest Dependency Chain":**
**S (20%):** "At a SaaS company, a cross-functional feature (real-time collaboration) had 8 parallel engineering workstreams with complex prerequisites — infrastructure, auth, websocket layer, UI components, testing. The project was 3 weeks behind schedule because teams were not coordinating around the true bottleneck."
**T:** "As tech lead, I owned end-to-end delivery. Goal: ship the feature within 4 weeks — a fixed external commitment to a major enterprise customer worth $1.2M ARR."
**A (60% — 'I' not 'we'):** "(1) I mapped all workstreams and prerequisites into a directed dependency graph and computed the critical path (the longest dependency chain) using topological DP — identified the websocket layer as 11 days on the critical path, auth integration as 7 days, UI as 5 days. (2) I reassigned two engineers from non-critical-path work (analytics dashboard) to unblock the websocket team, reducing that leg from 11 to 6 days. (3) I introduced a daily 15-minute standup scoped only to critical-path owners, cutting decision latency from 48 hours to same-day. (4) I tracked finish-time estimates daily and re-ran the critical path calculation when scope or estimates changed — adjusting priorities proactively rather than reactively."
**R (20%):** "Delivered the feature in 3.5 weeks, 2.5 days ahead of commitment. The enterprise customer signed the expanded contract. Post-mortem established critical-path tracking as a standard practice for all cross-team projects at the company."
*Works for LP questions on: Deliver Results, Bias for Action, Ownership.*

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
