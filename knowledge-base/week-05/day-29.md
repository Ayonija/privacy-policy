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
```java
class Solution {
    // Sum of Subarray Minimums (LC 907)
    public static int sumSubarrayMins(int[] arr) {
        final int MOD = 1_000_000_007;
        int n = arr.length;
        int[] left  = new int[n];   // number of subarrays where arr[i] is minimum, extending left
        int[] right = new int[n];   // extending right
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && arr[stack.peekLast()] >= arr[i]) {   // strict '>=' for left boundary
                stack.pollLast();
            }
            left[i] = stack.isEmpty() ? i + 1 : i - stack.peekLast();
            stack.addLast(i);
        }
        stack.clear();
        for (int i = n - 1; i >= 0; i--) {
            while (!stack.isEmpty() && arr[stack.peekLast()] > arr[i]) {    // strict '>' for right boundary
                stack.pollLast();
            }
            right[i] = stack.isEmpty() ? n - i : stack.peekLast() - i;
            stack.addLast(i);
        }
        long result = 0;
        for (int i = 0; i < n; i++) {
            result = (result + (long) arr[i] * left[i] * right[i]) % MOD;
        }
        return (int) result;
    }

    // Remove K Digits (LC 402) — skeleton
    public static String removeKdigits(String num, int k) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char digit : num.toCharArray()) {
            while (k > 0 && !stack.isEmpty() && stack.peekLast() > digit) {
                stack.pollLast();
                k--;
            }
            stack.addLast(digit);
        }
        // if k > 0, remove from end
        // strip leading zeros
        // return "0" if empty
        // TODO: implement post-loop steps
        return "0";
    }
}
```

### Interview Tips

- **LC 907 (Sum of Subarray Minimums) — state the contribution approach:** "Instead of iterating over all O(n²) subarrays, I count how many subarrays each element is the minimum of. For element at index i, that count = `left[i] * right[i]`. This reduces to O(n) with two monotonic stack passes."
- **Strict vs non-strict comparison for duplicate minimums:** "For the left boundary, I use `>=` (strict) so that equal elements on the left are counted as 'closer'. For the right boundary, I use `>` (non-strict). This ensures each subarray with a repeated minimum is counted exactly once." This is a senior-level detail that most candidates miss.
- **LC 402 (Remove K Digits) — three post-loop steps:** "After the main loop: (1) if `k > 0`, trim `k` characters from the right of the stack (loop used up all chances to pop). (2) Strip leading zeros using `lstrip('0')`. (3) If empty, return `'0'`." Missing any step gives a wrong answer on edge cases.
- **Brute force for LC 907:** O(n²) checking all subarrays and calling `min()` each time — O(n³) total. Contribution counting with monotonic stack is O(n).
- **Common mistake in LC 402:** forgetting `stack[-1] > digit` in the while condition (not `>=`) — equal digits should NOT be popped since popping equals doesn't make the number smaller.

### Edge Cases to Trace Before Coding
- LC 682: `"C"` as the first operation → stack is empty; problem guarantees this won't happen, but mention it shows constraint awareness
- LC 907: single-element array → `left[0] = 1`, `right[0] = 1`, contribution = `arr[0] * 1 * 1`; correct ✓
- LC 907: all same elements `[3, 3, 3]` → duplicate handling with strict/non-strict is critical
- LC 402: `k = 0` → no digits to remove; return `num` as-is
- LC 402: `num = "10"`, `k = 2` → pop both digits; empty stack; return `"0"` ✓

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
