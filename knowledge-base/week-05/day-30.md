# Day 30 — Stack Synthesis: Remove Duplicate Letters & 132 Pattern
**Week 05 | Phase 1: DSA Mastery | Month 1**

## Focus
Month 1 close-out: solve two advanced monotonic stack problems that combine greedy + counting logic, then complete the Month 1 review.

## DSA (2 hours)
### Pattern: Monotonic Stack — Greedy with Last-Occurrence Constraint & Behind-Pattern Detection
- Remove Duplicate Letters: keep the lexicographically smallest result; use a monotonic increasing stack, but only pop when the character to be removed still appears later in the string (check via last-occurrence index).
- 132 Pattern: find indices i < j < k where nums[i] < nums[k] < nums[j]; scan right-to-left, maintain a monotonic decreasing stack to track candidates for nums[k], and a running minimum for nums[i].
- Trigger condition: "lexicographically smallest result with all unique characters" → greedy + last-occurrence; "find a three-element subsequence with a specific ordering" → stack + running min.
- Time complexity: O(n) | Space complexity: O(n)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Remove Outermost Parentheses | 1021 | Easy | Depth counter | Track depth with a counter; output chars only when depth > 0 (exclude outermost open) or depth > 1 (exclude outermost close) |
| 2 | Remove Duplicate Letters | 316 | Medium | Monotonic stack + last-occurrence | Pop stack top only if it appears again later (`last[ch] > i`); mark chars in stack to avoid duplicates |
| 3 | 132 Pattern | 456 | Medium | Monotonic stack (right-to-left) + running min | Scan right-to-left; stack holds candidates for nums[k] (the "2"); pop when top < current (update third); return true if any nums[i] < third |

### Code Skeleton
```java
class Solution {
    // Remove Duplicate Letters (LC 316)
    public static String removeDuplicateLetters(String s) {
        Map<Character, Integer> last = new HashMap<>();
        for (int i = 0; i < s.length(); i++) {
            last.put(s.charAt(i), i);
        }
        Deque<Character> stack = new ArrayDeque<>();
        Set<Character> inStack = new HashSet<>();
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (inStack.contains(ch)) {
                continue;
            }
            while (!stack.isEmpty() && ch < stack.peekLast() && last.get(stack.peekLast()) > i) {
                inStack.remove(stack.pollLast());
            }
            stack.addLast(ch);
            inStack.add(ch);
        }
        StringBuilder sb = new StringBuilder();
        for (char c : stack) sb.append(c);
        return sb.toString();
    }

    // 132 Pattern (LC 456)
    public static boolean find132pattern(int[] nums) {
        Deque<Integer> stack = new ArrayDeque<>();   // candidates for nums[k] (the "2" in 132)
        int third = Integer.MIN_VALUE;               // current best nums[k]
        for (int i = nums.length - 1; i >= 0; i--) {
            int n = nums[i];
            if (n < third) {
                return true;
            }
            while (!stack.isEmpty() && stack.peekLast() < n) {
                third = stack.pollLast();
            }
            stack.addLast(n);
        }
        return false;
    }
}
```

### Interview Tips

- **LC 316 (Remove Duplicate Letters) — explain the three conditions for popping:** "I pop `stack[-1]` only when ALL THREE hold: (1) the new char `ch` is smaller than top, (2) the top character appears later in the string (`last[top] > i` — can still be added), (3) the top is not already committed (checked via `in_stack`). Missing any condition gives incorrect results."
- **LC 456 (132 Pattern) — explain the right-to-left scan:** "I scan right to left because I know nums[j] (the maximum) first. The stack holds candidates for nums[k] (the '2' — smaller than j but larger than i). Whenever I pop a value from the stack (because current is larger), that popped value becomes my best candidate for nums[k]."
- **LC 316 — `in_stack` set purpose:** "The set tracks which characters are already in the stack. If `ch in in_stack`, I skip — adding it again would create a duplicate in the result."
- **Month 1 synthesis tip:** you should now be able to look at a problem description and in 30 seconds decide: two-pointer, sliding window, or stack. Practice this pattern-recognition reflex — it's the core skill being tested in a real OA.
- **Common mistake in LC 316:** using `last[stack[-1]] >= i` instead of `> i` — at `i`, we're processing the character at that index, so `last[stack[-1]] > i` means there's still a future occurrence; `>=` would incorrectly block popping when the last occurrence is the current position.

### STAR Interview Framework

> **Monotonic Stack — Greedy with Last-Occurrence Constraint & Behind-Pattern Detection:** brute-force O(n! / duplicates) → this approach O(n) time, O(n) space

**S:** "Given a string, find the lexicographically smallest subsequence using each unique character exactly once. Brute force over all character subsets is factorial — fails even at n=20."
**T:** "Need O(n) by greedily maintaining an increasing character stack, constrained by last-occurrence positions."
**A (60% of answer time):**
1. *Classify:* "'Lexicographically smallest result with all unique characters' → greedy monotonic stack + last-occurrence guard. 'Three-element subsequence i<j<k with nums[i]<nums[k]<nums[j]' → right-to-left stack + running minimum."
2. *Init:* "For LC 316: precompute `last[ch]` = last index of each char. Stack + `in_stack` set. For LC 456: stack of candidates for nums[k], `third = -∞`."
3. *Loop/Step:* "LC 316: skip if already in stack; pop top if `top > ch AND last[top] > i`; push. LC 456: if `nums[i] < third` return true; pop into `third` while stack top < current; push."
4. *Termination:* "Each element is pushed and popped at most once → O(n) total."
5. *Gotcha:* "In LC 316, use `last[top] > i` (strict greater-than) — `>=` would block popping when the current index IS the last occurrence, leaving a suboptimal character on the stack. State this before coding."
**R:** "O(n) time, O(n) space. The last-occurrence guard is the detail that separates correct from plausible-looking solutions — mentioning it proactively signals senior-level depth."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Recursive backtracking | n ≤ 10, need all solutions | Exponential — times out at n=10⁴ |
| Sort + deduplicate | Anagram grouping, not ordering | Destroys original order needed for subsequence constraint |
| Prefix min array for LC 456 | Offline, need all valid triplets | O(n²) — nested scan for valid third element |

### Edge Cases to Trace Before Coding
- LC 316: single unique character (e.g., `"aaaa"`) → only one 'a' in result; `in_stack` prevents duplicates ✓
- LC 316: already sorted string (e.g., `"abc"`) → no pops ever; result is the unique characters in their original order ✓
- LC 456: monotonically increasing array → no valid 132 pattern; `third` never exceeds `-inf`; return False ✓
- LC 456: `nums.length < 3` → impossible to have i < j < k; return False immediately (add guard)
- LC 1021: `"(()())(())"` → depth tracker correctly outputs only inner characters

## System Design (1 hour)
### Topic: Month 1 System Design Synthesis — Big-O → Scaling → CAP
- **Month 1 system design arc:** Big-O + RAM model → Vertical/Horizontal scaling → Load balancers (L4 vs L7, algorithms, health checks) → CAP theorem (CP vs AP) → Consistency models (strong, causal, eventual) → PACELC → Real systems (Zookeeper, Cassandra, DynamoDB, Spanner).
- **The interview flow:** when asked to design any distributed system, the conversation naturally follows: capacity estimates → single server → scale the app tier → scale the data tier → choose consistency model → handle failures.
- The CAP decision is always tied to the data type: financial data → CP; user-facing features where availability matters → AP; real-time collaboration → causal.
- Always end a system design answer with failure modes: "what happens when a node crashes?" and "what happens during a network partition?"
- Interview talking point: "If asked to start a system design for any large-scale service, answer: 1. Clarify requirements and scale (DAU, QPS, storage). 2. High-level components (LB → App servers → Cache → DB). 3. Deep dive on the hardest component. 4. Discuss consistency model choice. 5. Address failures and monitoring."

## Assessment / Mock (—)
### Activity: —

## Behavioral (30 min)
- Leadership principle: Learn and Be Curious

**Full STAR Story — "Diagnosing a Month-Long Learning Gap and Closing It":**
**S (20%):** "After 30 days of algorithm study at a self-directed pace, I noticed I was consistently solving easy problems quickly but freezing on medium problems involving monotonic stacks — I was re-reading the pattern every time rather than internalising it."
**T:** "I needed to close the monotonic stack gap before Month 2 began, without adding extra study hours — I had a firm 2-hour daily limit."
**A (60% — 'I' not 'we'):** "(1) I ran a diagnostic: timed myself on five representative problems, identified that my failure mode was choosing a wrong data structure in the first 5 minutes. (2) I created a trigger-word flashcard — e.g., 'next greater element → decreasing stack; sum of minimums → contribution counting' — and drilled it for 10 minutes each morning. (3) I replaced one practice problem per day with a deliberate re-solve of the hardest problem from the day before, without notes. (4) By day 35 I was solving monotonic stack mediums within 20 minutes on first attempt."
**R (20%):** "Pattern recognition speed improved from ~15 minutes to under 3 minutes on monotonic stack triggers. This compounding habit added zero extra hours and became my standard gap-closing technique across all patterns in Month 2."
*Works for: Learn and Be Curious, Dive Deep, Invent and Simplify.*

## Flashcards

| Q | A |
|---|---|
| What is the key guard condition that prevents Remove Duplicate Letters from removing a character that won't appear again? | `last[stack[-1]] > i` — the character at the top must still appear at a future index for it to be safe to pop |
| How do you avoid re-processing already-added characters in Remove Duplicate Letters? | Maintain an `in_stack` set; skip any character that is already in the stack |
| In the 132 Pattern right-to-left scan, what does the `third` variable represent? | The best candidate for nums[k] (the "2" in 1-3-2) — the largest value seen so far that was popped because a bigger value arrived |
| What is the correct order of steps in a system design interview? | 1. Clarify requirements + scale. 2. High-level components. 3. Deep-dive hardest piece. 4. Consistency model choice. 5. Failure handling + monitoring |
| What is the system design question to always end with? | "What happens when a node crashes?" and "What happens during a network partition?" — this shows you understand distributed systems |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
