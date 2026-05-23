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

### Interview Tips

- **Number of Recent Calls window:** "The window is always [t − 3000, t] inclusive. Pop from the front while `front < t − 3000` (strict less-than) — you want to keep calls at exactly `t − 3000` in the window."
- **Make The String Great case comparison:** "Check `stack.peek().toLowerCase() == ch.toLowerCase() && stack.peek() != ch` — if you just check `toLowerCase` without the inequality check, you'd pop when the same letter appears twice in the same case, which is wrong."
- **Removing Stars vs. Adjacent Duplicates:** "Stars always pop the immediately preceding character regardless of what it is — there's no matching condition. Adjacent duplicates pop only when the top matches the incoming character exactly. Know how to articulate this distinction aloud."
- **This review day's goal:** Solve all three in under 15 minutes each. If any take longer, flag to the revision log — they signal gaps before advancing to hashing.

### STAR Interview Framework

> **Stack — String Cleanup Review (Adjacent Cancellation & Sliding Window Queue):** brute-force O(n²) → this approach O(n) time, O(n) space

**S:** "Given a string of n characters with either star-pop semantics or case-pair cancellation. Repeated passes through the string to find and remove pairs is O(n²) — too slow for n=10^5."
**T:** "Need O(n) by processing left to right with a stack that materialises the result after all cancellations. Goal: return the final cleaned string."
**A (60% of answer time):**
1. *Classify:* "The trigger is 'a character or token cancels the most recent surviving character' — stack with a conditional push/pop. For sliding window (Recent Calls), the trigger is 'maintain only elements within a fixed time window' — deque with front-eviction."
2. *Init:* "Empty stack (or deque). For Recent Calls, also store the incoming timestamp to compute the window boundary."
3. *Loop/Step:* "Stack cleanup: on each char, check cancel condition (star, case-pair) against stack top — pop if cancels, else push. Sliding window: push new timestamp, then pop from front while front < t − 3000."
4. *Termination:* "After one pass, join the stack contents. For Recent Calls, return the deque size."
5. *Gotcha:* "For case-pair cancellation (Make The String Great), the cancel condition requires both `toLowerCase` match AND the chars are not identical — missing the second check breaks it for repeated same-case letters."
**R:** "O(n) time, O(n) space. Solving all three review problems in under 15 minutes confirms mastery of the cancel-semantics pattern before advancing to hashing week."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Repeated regex replace in a loop | n ≤ 50, script context | O(n²) per repeated scan — unacceptable for n=10^5 |
| In-place two-pointer | Backspace Compare O(1) space (special case) | Cannot generalise across all three problem variants |

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
- Leadership principle: Learn and Be Curious

**Full STAR Story — "Running a Structured End-of-Quarter Retrospective That Uncovered a Hidden Reliability Gap":**

**S (20%):** "At a data engineering company, our team had just shipped a major pipeline refactor over an 8-week sprint. The sprint retrospective was traditionally a 30-minute team chat — no structured output, no action items tracked. After one such retrospective, a P1 incident revealed a known reliability gap in error handling that three engineers had privately noted during the sprint but never surfaced formally. The incident caused 6 hours of data pipeline downtime and delayed a customer report delivery."
**T:** "I volunteered to redesign the retrospective format for the next sprint. Goal: create a lightweight structured review process that surfaced technical gaps before they became incidents — without adding more than 1 hour of overhead per sprint."
**A (60% — use 'I' not 'we'):** "(1) I researched retrospective formats and designed a 3-section template: 'What we shipped' (fact-based), 'What we skipped or deferred' (explicit gap log), and 'What we need to understand before next sprint' (learning items). (2) I introduced a pre-retro async survey sent 24 hours in advance — engineers answered the three sections individually before the meeting, which surfaced things people hesitated to say live. (3) I ran the first structured retro myself, facilitated the gap-log section as a prioritised list, and assigned owners to each item with a two-week follow-up date. (4) I shared the format with two other teams and offered to run their first structured retros."
**R (20%):** "In the two sprints after introducing the format, our gap log surfaced 7 deferred items — 2 of which were subsequently upgraded to P1-risk items and addressed before causing incidents. The format was adopted by all three backend teams. My manager cited it in my performance review as an example of learning from an incident and institutionalising the lesson."
*Works for LP questions on: Learn and Be Curious; Ownership; Insist on the Highest Standards.*

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
