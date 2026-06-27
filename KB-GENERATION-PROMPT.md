# Knowledge Base Generation Prompt

Use the prompts below to build the **6-Month FAANG Knowledge Base** in this repository, **one slot at a time**. Each slot covers a 10-day block. Run it **18 times** (6 slots per month) to complete the full 180 days.

---

## Overview

| Phase | Months | Days | Slots | Focus |
|---|---|---|---|---|
| 1 — DSA Mastery | 1–3 | 1–90 | 1–9 | Pattern-by-pattern DSA + System Design foundations |
| 2 — Exposure & Assessment | 4 | 91–120 | 10–12 | DP deep dive + OA practice + mentor interactions |
| 3 — Mock Interviews | 5 | 121–150 | 13–15 | Hard problems + full mock interview loops |
| 4 — Final Sprint | 6 | 151–180 | 16–18 | Revision + timed OAs + intensive mock simulations |

---

## Daily Time Budget (by phase)

| Phase | DSA | System Design | Assessment / Mock | Behavioral |
|---|---|---|---|---|
| Months 1–3 | 2 h | 1 h | — | 30 min |
| Month 4 | 2 h | 30 min | 1 h contests/OA | 30 min |
| Month 5 | 1.5 h | 1 h | 1 h mock interview | 30 min |
| Month 6 | 1.5 h | 30 min | 1.5 h mock simulation | 30 min |

---

## Daily Problem Count (by phase)

| Days | Problems per day | Difficulty mix |
|---|---|---|
| 1–30 | 3 | Easy + Medium + Medium |
| 31–90 | 3 | Medium + Medium + Hard |
| 91–150 | 2 + 1 revision | Hard + Hard + revisit unsolved |
| 151–180 | 1 + 1 revision + 1 mock OA | Hard + revision + timed assessment |

---

## Slot Reference

| Run | Slot | Days | Weeks | DSA Focus | System Design Focus |
|-----|------|------|-------|-----------|---------------------|
| 1st | Slot 1 | 1–10 | 1–2 | Arrays & Strings: Two Pointers, Sliding Window | Big-O, RAM model |
| 2nd | Slot 2 | 11–20 | 2–3 | Linked Lists: Fast/Slow, Reversal, Merge | Scaling: H vs V, Load Balancers |
| 3rd | Slot 3 | 21–30 | 4–5 | Stacks & Queues: Monotonic Stack, Deque | CAP Theorem, Consistency models |
| 4th | Slot 4 | 31–40 | 5–6 | Hashing: HashMap, Frequency maps, advanced Sliding Window | DB Indexing, B-Trees |
| 5th | Slot 5 | 41–50 | 6–7 | Trees & BST: DFS Pre/In/Post, BFS, LCA, Serialize | Caching: Redis, Memcached, Eviction |
| 6th | Slot 6 | 51–60 | 8–9 | Graphs: DFS, BFS, Union-Find, Cycle detection | CDN, DNS, HTTP fundamentals |
| 7th | Slot 7 | 61–70 | 9–10 | Graphs advanced: Dijkstra, Topological Sort, Bellman-Ford | URL Shortener, Key-Value Store |
| 8th | Slot 8 | 71–80 | 11–12 | Heaps & Tries: Top-K, Merge K lists, Prefix trees | Newsfeed / Social Network |
| 9th | Slot 9 | 81–90 | 12–13 | Greedy & Backtracking: Intervals, Subsets, Permutations | Message Queues, Pub/Sub |
| 10th | Slot 10 | 91–100 | 13–14 | DP Basics: 1D DP, Fibonacci variants, House Robber | Chat Service / Notification System |
| 11th | Slot 11 | 101–110 | 15–16 | DP Intermediate: 0/1 Knapsack, LCS, LIS | API Design: REST vs GraphQL, Rate Limiting |
| 12th | Slot 12 | 111–120 | 16–17 | DP Advanced + Bit Manipulation | Consistent Hashing, LRU/LFU Cache Design |
| 13th | Slot 13 | 121–130 | 17–18 | Hard problems: Sliding Window hard, Two Pointer hard | Twitter / X full design |
| 14th | Slot 14 | 131–140 | 19–20 | Hard problems: Trees hard, Graphs hard | Uber / Lyft full design |
| 15th | Slot 15 | 141–150 | 21–22 | Hard problems: DP hard, Backtracking hard | YouTube / Netflix full design |
| 16th | Slot 16 | 151–160 | 22–23 | Full DSA revision: all patterns, templates from memory | Review all HLD components |
| 17th | Slot 17 | 161–170 | 24–25 | Timed OA simulation + contest problems | LLD deep dive: DB schema, class diagrams |
| 18th | Slot 18 | 171–180 | 25–26 | Intensive mock: weak area targeted drill | End-to-end system design mocks |

---

## Master Prompt (copy, fill the 3 placeholders, run)

```
You are building a structured 6-month FAANG interview knowledge base inside this repository.
The knowledge base lives in the `knowledge-base/` directory.

## Task
Generate [SLOT N]: Days [START]–[END] of the 6-Month FAANG Full-Stack Preparation Plan.

## Output structure
For each day, create a file at:
  knowledge-base/week-[WW]/day-[DD].md

Use the week number that corresponds to the day number (Day 1–7 = week-01, Day 8–14 = week-02, etc.).

Each day file must follow this exact template:

---
# Day [DD] — [TOPIC NAME]
**Week [WW] | [PHASE NAME] | Month [M]**

## Focus
One-sentence description of what this day builds toward in the 6-month plan.

## DSA ([X] hours — see phase budget)
### Pattern: [PATTERN NAME]
- Core idea (2–3 sentences, no jargon padding)
- Trigger condition: when to reach for this pattern
- Time complexity: O(?) | Space complexity: O(?)

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | [title] | [number] | Easy/Medium/Hard | [pattern] | [insight] |
| 2 | [title] | [number] | Easy/Medium/Hard | [pattern] | [insight] |
| 3 | [title] | [number] | Easy/Medium/Hard | [pattern] | [insight] |

(For Days 151–180: 1 hard problem + 1 revision problem. For Days 91–150: 2 hard + 1 revision.)

### Code Skeleton
```python
# Minimal skeleton — delete this and rewrite from memory before solving
```

## System Design ([X] hours — see phase budget)
### Topic: [SYSTEM DESIGN TOPIC or "—" if no SD component today]
- Core components (bullet list, max 5)
- Key trade-offs (2–3 bullets)
- Interview talking point: "If asked [question], answer: [concrete answer]"

## Assessment / Mock (Month 4+: [X] hours or "—" for Months 1–3)
### Activity: [LeetCode contest / HackerRank OA / Mock interview / "—"]
- Goal for this session
- Debrief prompt: [one question to reflect on after the session]

## Behavioral (30 min)
- STAR prompt: [one specific scenario tied to the DSA topic or company LP]
- Leadership principle: [LP name]

## Flashcards
5 recall-based cards (not recognition — phrase as "How do you X?" not "What is X?"):

| Q | A |
|---|---|
| [Question] | [Concise answer] |
| [Question] | [Concise answer] |
| [Question] | [Concise answer] |
| [Question] | [Concise answer] |
| [Question] | [Concise answer] |

## Checklist
- [ ] Solved all problems (timed, no hints for first 20 min)
- [ ] Wrote pattern skeleton from memory after reading it
- [ ] Stated time + space complexity aloud for each solution
- [ ] Completed system design component (if scheduled)
- [ ] Reviewed flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`

---

## Weekly Review File
On the last day of each calendar week (Day 7, 14, 21 … 182), also create:
  knowledge-base/week-[WW]/weekly-review.md

Weekly review template:

---
# Week [WW] Review — [DATE RANGE PLACEHOLDER]

## Phase
[Phase name and month]

## Patterns Covered This Week
- [list all patterns]

## System Design Topics Covered
[1–2 sentence summary of each topic]

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| [pattern] | | [drill more / move on] |

## Weekly Flashcard Deck
[Aggregate all 5-per-day flashcards from the 7 days into one table — 35 cards]

| Q | A |
|---|---|
[all 35 rows]

## Mock / Contest Log (Month 4+)
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

---

## Monthly Milestone Review
On Day 30, 60, 90, 120, 150, and 180, also create:
  knowledge-base/month-[M]-review.md

Monthly review template:

---
# Month [M] Review

## Phase
[Phase name and goal]

## Patterns Mastered (comfort ≥ 4/5)
- [list]

## Patterns Needing Drill
- [list]

## System Design Topics Mastered
- [list]

## OA / Mock Interview Stats (Month 4+)
- Contests attempted:
- OAs completed:
- Mock interviews done:
- Average completion rate under time pressure:

## Month Flashcard Master Deck
[Link to each weekly-review.md in this month — learner aggregates]

## Next Month Goals
1. [Goal 1]
2. [Goal 2]
3. [Goal 3]

---

## Slot context for this run

You are generating [SLOT N] which covers Days [START]–[END].

Phase schedule reminder:
- Days 1–90 (Slots 1–9): Phase 1, DSA Mastery, Months 1–3
  - 2h DSA + 1h System Design + 30min Behavioral
  - Problems: Days 1–30 = Easy+Medium+Medium; Days 31–90 = Medium+Medium+Hard
- Days 91–120 (Slots 10–12): Phase 2, Exposure & Assessment, Month 4
  - 2h DSA + 30min System Design + 1h Contest/OA + 30min Behavioral
  - Problems: 2 Hard + 1 revision
- Days 121–150 (Slots 13–15): Phase 3, Mock Interviews, Month 5
  - 1.5h DSA + 1h System Design + 1h Mock interview + 30min Behavioral
  - Problems: 2 Hard + 1 revision
- Days 151–180 (Slots 16–18): Phase 4, Final Sprint, Month 6
  - 1.5h DSA revision + 30min System Design + 1.5h Mock simulation + 30min Behavioral
  - Problems: 1 Hard + 1 revision + 1 timed OA problem

DSA topics and System Design topics for this slot (use the Slot Reference table):
[PASTE THE ROW FROM THE SLOT REFERENCE TABLE FOR THIS SLOT]

## Hard rules
1. Use real LeetCode problem numbers and titles. Do not invent problems.
2. Do not repeat any problem that would appear in an earlier slot.
3. Flashcard questions must test recall, not recognition.
4. Code skeletons must be minimal — learner fills in the logic.
5. Every system design day must include one concrete interview talking point.
6. Weekly review files are created at the end of Day 7, 14, 21, 28 … within this slot range.
7. Monthly review files are created if Day 30, 60, 90, 120, 150, or 180 falls in this slot range.

Now generate [SLOT N]: Days [START]–[END].
Create all day files, any weekly-review files, and any monthly-review files that fall within this slot.
```

---

## Continuation Prompt (if output is cut off mid-slot)

```
Continue the knowledge base from Day [N], same format, same rules, same slot context.
Do not regenerate days already created. Pick up exactly at Day [N].
```

---

## Index Generation Prompt (run once after all 18 slots are complete)

```
The `knowledge-base/` directory now contains all 180 day files, 26 weekly-review files,
and 6 monthly-review files.

Generate `knowledge-base/README.md` as the master index with these sections:

1. **Navigation table** (180 rows):
   Day | File link | Topic | Patterns | System Design | Month

2. **Pattern frequency table**:
   Pattern | Days it appears | Difficulty ramp (Easy→Hard over which days)

3. **Flashcard master deck index**:
   Link to each weekly-review.md. Total cards available: 26 weeks × 35 = 910 cards.

4. **Revision log** (empty — learner fills):
   | Date | Problem | LC # | What went wrong | Retried on | Solved clean |

5. **Mock interview log** (empty — learner fills):
   | Date | Type (DSA/SD/Behavioral) | Platform | Result | Follow-up action |

6. **OA contest log** (empty — learner fills):
   | Date | Platform | Contest name | Problems solved | Rank | Notes |

7. **Monthly milestone tracker**:
   Month | Review file | Patterns mastered | Patterns to drill | Mock stats

8. **6-Month progress checklist** (one line per day, linked to file, checkbox):
   - [ ] [Day 01 — Topic](week-01/day-01.md)
   - [ ] [Day 02 — Topic](week-01/day-02.md)
   ... (all 180 days)

File: knowledge-base/README.md
```

---

## Tips for Best Results

- Run each slot in a **fresh Claude Code session** to avoid context overflow.
- Paste the exact slot row from the Slot Reference table into the `[PASTE THE ROW...]` placeholder.
- If output is cut off, use the Continuation Prompt — do not re-run the full slot prompt.
- After each slot, commit: `git add knowledge-base/ && git commit -m "feat: knowledge-base slot [N] days [START]-[END]"`
- Weekly flashcard decks are Anki-compatible — each `| Q | A |` row maps to one card.
- Monthly reviews are natural checkpoints to adjust difficulty or slow down if gaps are large.
