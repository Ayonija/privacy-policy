# Day 29 — Monotonic Stack: Advanced Applications
**Week 05 | Phase 1: DSA Mastery | Month 1**

## Focus
Tackle harder monotonic stack problems — sum of subarray minimums and greedy digit removal — that combine the pattern with counting and ordering constraints.

## DSA (2 hours)
### Pattern: Monotonic Stack — Counting & Greedy Removal
- Sum of Subarray Minimums: for each element, use a monotonic stack to find how many subarrays it is the minimum of — multiply by value, sum all contributions; handle duplicates with strict vs. non-strict comparison on left/right.
- Remove K Digits: greedy — maintain an increasing monotonic stack; pop larger digits when a smaller digit arrives (as long as k > 0); pad or truncate the result to the correct length.
- Baseball Game: simulate with a stack — numbers push, `+` sums top two, `D` doubles top, `C` pops top.
- Trigger condition: "sum of minimums over all subarrays" (contribution counting) OR "smallest number by removing k digits" (greedy monotonic stack).
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Baseball Game | 682 | Easy | Stack simulation | `+`: sum top two; `D`: double top; `C`: pop; number: push; return sum of stack |
| 2 | Sum of Subarray Minimums | 907 | Medium | Monotonic Stack + contribution | For each element find left/right boundaries; use MOD 10^9+7; handle duplicate minimums with strict left, non-strict right |
| 3 | Remove K Digits | 402 | Medium | Monotonic increasing stack + greedy | Pop larger digits while k > 0; strip leading zeros; if k > 0 after loop, truncate right end |

### Code Skeleton
```python
# Sum of Subarray Minimums (LC 907)
def sum_subarray_mins(arr):
    MOD = 10**9 + 7
    n = len(arr)
    left  = [0] * n   # number of subarrays where arr[i] is minimum, extending left
    right = [0] * n   # extending right
    stack = []
    for i in range(n):
        while stack and arr[stack[-1]] >= arr[i]:   # strict '>=' for left boundary
            stack.pop()
        left[i] = i - stack[-1] if stack else i + 1
        stack.append(i)
    stack = []
    for i in range(n - 1, -1, -1):
        while stack and arr[stack[-1]] > arr[i]:    # strict '>' for right boundary
            stack.pop()
        right[i] = stack[-1] - i if stack else n - i
        stack.append(i)
    return sum(arr[i] * left[i] * right[i] for i in range(n)) % MOD

# Remove K Digits (LC 402) — skeleton
def remove_k_digits(num, k):
    stack = []
    for digit in num:
        while k and stack and stack[-1] > digit:
            stack.pop(); k -= 1
        stack.append(digit)
    # if k > 0, remove from end
    # strip leading zeros
    # return "0" if empty
    pass
```

## System Design (1 hour)
### Topic: CAP in Real Systems — Zookeeper, Cassandra, DynamoDB
- **Zookeeper (CP):** used for distributed coordination — leader election, distributed locks, configuration management; refuses reads during a partition to prevent stale config data; clients must retry.
- **Cassandra (AP + tunable):** AP by default; tunable consistency per request (ONE → QUORUM → ALL); uses gossip protocol for node discovery, anti-entropy repair for convergence; no single point of failure.
- **DynamoDB (AP + eventual, with optional strong reads):** eventually consistent reads by default (lower cost); strongly consistent reads available at 2× cost; global tables use last-write-wins with timestamps.
- **etcd (CP):** Raft-based; used by Kubernetes for all cluster state; prioritises consistency over availability — a minority partition of etcd will halt the Kubernetes control plane rather than serve stale cluster state.
- Interview talking point: "If asked which database to use for storing Kubernetes cluster state vs. user profile data, answer: etcd (CP) for cluster state — stale leader elections would be catastrophic; DynamoDB or Cassandra (AP) for user profiles — a millisecond of stale profile data is harmless, and availability is critical for user-facing traffic."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you made a greedy decision — took the locally best action at each step — that led to a globally optimal result, analogous to Remove K Digits removing the largest digits greedily from the left.
- Leadership principle: Bias for Action

## Flashcards

| Q | A |
|---|---|
| How do you handle duplicate minimum values in Sum of Subarray Minimums? | Left boundary: use `>=` (strict includes equal on left to avoid double-counting). Right boundary: use `>` (non-strict excludes equal on right) — or vice versa consistently |
| What are the three post-loop steps in Remove K Digits? | 1. If k > 0, truncate `k` characters from the right. 2. Strip leading zeros. 3. Return "0" if the result is empty |
| Why does Zookeeper choose CP over AP? | It stores coordination data (locks, leader election, config); stale data here causes split-brain or incorrect cluster behaviour — unavailability is safer than incorrect coordination |
| How does Cassandra achieve high availability without a single master? | Gossip protocol for node discovery and failure detection; any node can handle any request (token ring); replicas self-heal via anti-entropy repair after partition resolution |
| What is the cost difference between eventually consistent and strongly consistent reads in DynamoDB? | Strongly consistent reads cost 2× the read capacity units of eventually consistent reads — because they must hit the leader shard, not a replica |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
