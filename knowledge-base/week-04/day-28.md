# Day 28 — Review Day: Stacks & Queues Week 4
**Week 04 | Phase 1: DSA Mastery | Month 1**

## Focus
Consolidate the full Stacks & Queues unit before the final two days; close the Week 04 flashcard deck.

## DSA (2 hours)
### Pattern: Review — Stack String Cleanup
- These three problems test stack fundamentals in a slightly new framing: removing stars (reverse undo), making a string great (adjacent pair cancellation), and the number of recent calls (queue window).
- Goal: solve each in under 15 minutes from scratch to confirm mastery before advancing to Medium/Hard in Month 2.
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Number of Recent Calls | 933 | Easy | Queue with deque | Pop requests older than `t − 3000` from the front; return queue size |
| 2 | Make The String Great | 1544 | Medium | Stack (case-pair cancellation) | Push char; if stack top is the same letter in opposite case, pop instead; join at end |
| 3 | Removing Stars From a String | 2390 | Medium | Stack (`*` pops previous char) | On `*`, pop stack; on letter, push; join at end |

### Code Skeleton
```java
// Number of Recent Calls (LC 933)
class RecentCounter {
    private Deque<Integer> q = new ArrayDeque<>();

    public int ping(int t) {
        q.addLast(t);
        while (q.peekFirst() < t - 3000) {
            q.pollFirst();
        }
        return q.size();
    }
}

// Make The String Great (LC 1544)
class Solution {
    public static String makeGood(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char ch : s.toCharArray()) {
            if (!stack.isEmpty() && stack.peekLast() != ch
                    && Character.toLowerCase(stack.peekLast()) == Character.toLowerCase(ch)) {
                stack.pollLast();
            } else {
                stack.addLast(ch);
            }
        }
        StringBuilder sb = new StringBuilder();
        for (char c : stack) sb.append(c);
        return sb.toString();
    }
}
```

## System Design (1 hour)
### Topic: CAP & Consistency — Week 4 Synthesis
- Full recap: CAP → always tolerate P; choose CP or AP during a partition. PACELC → even in normal operation, choose latency or consistency.
- Decision guide:
  - Financial transactions, inventory: **CP** — data correctness > availability
  - Social features (likes, feeds, cart): **AP + eventual** — availability > stale reads
  - Collaborative editing: **causal consistency** — see your own writes, ordered causally
  - Global coordination (locks, leader election): **CP** (Zookeeper/etcd)
- Common interview mistake: claiming "we need strong consistency everywhere" — always justify with the cost (higher latency, lower throughput) and check if the use case truly requires it.
- Interview talking point: "If asked to design a distributed counter for video view counts, answer: use AP + eventual — exact counts are not required; use a Redis INCR on a local shard, periodically flush totals to the main DB; slight staleness in view counts is completely acceptable."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- STAR prompt: Describe a time you performed a thorough end-of-sprint review to identify gaps before moving on — mirroring today's stack/queue consolidation before advancing to hashing.
- Leadership principle: Learn and Be Curious

## Flashcards

| Q | A |
|---|---|
| How does Number of Recent Calls maintain the sliding 3000ms window? | Append each ping to the deque; pop from the front while the front is older than `t − 3000`; return `len(deque)` |
| How does Make The String Great detect a "bad pair"? | `stack[-1].lower() == ch.lower()` and `stack[-1] != ch` — same letter, different case |
| How does Removing Stars From a String differ from Remove Adjacent Duplicates? | Stars explicitly trigger a pop of the preceding character; in adjacent duplicates the character itself triggers a pop when it matches the top |
| What is the three-step decision guide for choosing CP vs. AP? | 1. Can incorrect data cause harm (money, inventory)? → CP. 2. Is availability more important than perfect data (social, cart)? → AP. 3. Need causal ordering for collaboration? → causal consistency |
| How do you design a highly available distributed counter with acceptable staleness? | Redis INCR per shard (AP, O(1)); periodic async flush to main DB; no coordination between shards — tolerates ~seconds of staleness for view counts |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
