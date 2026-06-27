# UKG — Staff Software Engineer (P5) Interview Prep — MASTER INDEX

> **How to use this:** This is the map, not the territory. Each numbered topic below becomes its own exhaustive prep sheet (own file in this folder). Pick a topic by name, or say "go in order," and I produce one complete, teach-from-zero, present-out-loud sheet per response.
>
> **Target:** UKG Staff Software Engineer, P5 — full-stack, AI-forward. On-site Pune, **all rounds same day, in person, panel format.** At P5, **System Design + Technical Leadership carry the most weight; coding is a floor, not the bar.**

---

## 0. What the panel is actually buying at P5

A P5 is hired to **own ambiguous, cross-team technical problems and make other engineers better.** Every answer should ladder up to one of three signals:

1. **Depth** — you can go three "why?"s deep on anything you say. No buzzword survives follow-up.
2. **Judgement / trade-offs** — you don't recite "use Kafka"; you say *when not to* and what it costs.
3. **Leverage** — your impact is multiplied through tooling, standards, mentorship, and influence without authority — not just lines of code.

Your three behavioral anchors map perfectly onto signal #3 — **MCP server**, **CLI platform (zero-mandate adoption)**, **2× commit velocity**. Lean on them hard.

---

## TOPIC MAP — grouped by likely interview round

### 🟦 ROUND A — Coding / OA / DSA (the floor — clear it fast, don't shine here)
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 1 | **DSA & Coding patterns (Java)** — arrays/strings, hashing, two-pointer, sliding window, stack/queue, linked list, trees/BST, graphs (BFS/DFS), heaps, recursion/backtracking, binary search, intervals, greedy, DP + Big-O | C (STAR-per-pattern) | 4–5 h |
| 1b | **Targeted problem set** (from the lists you were sent — see §"Known asked problems" below) | drill | 3 h |

### 🟩 ROUND B — Core Java / JVM (a Senior Architect grilled this hard in the report you sent)
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 2 | **Core Java** — OOP, collections internals (HashMap, ArrayList, **ConcurrentHashMap vs synchronized HashMap**), generics, streams, Optional, equals/hashCode, immutability | A | 1.5 h |
| 3 | **Java Concurrency & JVM** — threads, executors, `CompletableFuture`, locks, `volatile` vs `synchronized`, deadlock; JVM memory model, GC, heap/stack | A | 1.5 h |

### 🟨 ROUND C — Spring / Backend / Data / Messaging
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 4 | **Spring & Backend** — **Spring vs Spring Boot**, IoC/DI & bean lifecycle, autoconfig, **Spring Boot annotations**, REST design (versioning, idempotency, status codes, pagination), Spring Security + OAuth2/OIDC/JWT, microservices patterns, service discovery, API gateway, resilience (circuit breaker/retry/timeout/bulkhead), `@Transactional` & isolation | A | 2 h |
| 5 | **Data Layer** — JPA/Hibernate + N+1, SQL vs NoSQL, **indexing (clustered vs non-clustered)** & query optimisation, **SQL joins/grouping/filters**, PostgreSQL/SQL Server, MongoDB modelling & aggregation, caching strategies & invalidation (Redis/Hazelcast) | A | 2 h |
| 6 | **Messaging — Kafka** — topics/partitions/offsets, consumer groups, delivery semantics (at-least/at-most/exactly-once), ordering, idempotent producers, back-pressure | A | 1.5 h |

### 🟧 ROUND D — Frontend / Micro-Frontends / Testing
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 7 | **Angular + TypeScript** — module/component architecture & lifecycle, change detection + zone.js + OnPush, RxJS (observables, operators, subjects, hot/cold), state mgmt, TS types/generics/narrowing, **CSR vs SSR**, performance (lazy loading, bundles) | A | 1.5 h |
| 8 | **Micro-Frontends** — Module Federation vs Single-SPA, shared dependency/version mgmt, independent deployment, coupling reduction, runtime integration pitfalls | A | 1 h |
| 9 | **Testing** — JUnit/Mockito/WireMock, Jest/Jasmine/Cypress, test pyramid, contract testing (Pact/Spring Cloud Contract), **SonarQube** | A | 1 h |

### 🟥 ROUND E — Cloud / DevOps / Observability
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 10 | **Cloud & DevOps** — **Docker**, **Kubernetes (pods/deployments/services/ingress/HPA)**, **CI/CD pipeline design**, Azure/AWS/GCP essentials, **Git & version control** | A | 1.5 h |
| 11 | **Observability** — logs vs metrics vs traces, distributed tracing, SLO/SLI/SLA, Datadog & Prometheus/Grafana, alerting | A | 1 h |

### 🟪 ROUND F — System Design (HLD + LLD) — **HIGHEST WEIGHT**
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 12 | **System Design (HLD)** — scalability, **scale 1 → 10,000 req/sec globally**, load balancing, caching layers, **CDN (what & why)**, CAP & consistency, sharding, replication, rate limiting, idempotency, queues, multi-tenancy. Worked example: **Design a review/rating system** | B | 3 h |
| 13 | **Low-Level Design (LLD/OOD)** — SOLID, core design patterns, class modelling, API/contract design | B | 1.5 h |

### 🟫 ROUND G — AI / LLM Engineering — **YOUR EDGE — go deepest**
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 14 | **AI/LLM Engineering** — LLM integration & structured/schema-constrained outputs, prompt engineering & context mgmt & token efficiency, RAG + vector DBs + embeddings + chunking, agentic workflows, **MCP**, AI security (prompt injection, context leakage, excessive agency, OWASP LLM Top 10), AI observability (latency/token cost/drift), RAG-vs-fine-tune-vs-prompt decision | A (multi-part) | 3 h |
| 14b | **Claude Code as a force-multiplier** — skills, subagents, worktrees, hooks, MCP in CC, slash commands (ties to your "2× velocity" anchor) | A | 0.5 h |

### ⬛ ROUND H — Behavioural & Technical Leadership (P5) — **decides the loop**
| # | Topic | Sheet type | Budget |
|---|-------|-----------|--------|
| 15 | **Behavioural / Leadership** — driving technical decisions, **influence without authority**, mentorship, disagreement/conflict, ambiguity & failure, cross-functional collab, ownership & measurable impact. Plus logistics: **Why switch? Expected CTC? Why relocate?** | D (STAR) | 1.5 h |

---

## 📅 THE 2-DAY PLAN (≈12 h/day) — read this, then study the sheets in this order

The order is deliberate: **front-load your differentiators and the highest-weight rounds (System Design, AI/LLM, Behavioural), then the backend core, then the fundamentals/floor.**

### DAY 1 — "The deciders" (System Design, AI, Backend, Leadership)
| Block | Sheets | Time |
|-------|--------|------|
| Morning | **14** AI/LLM → **14b** Claude Code | 3.5 h |
| Midday | **12** System Design HLD → **13** LLD/OOD | 4.5 h |
| Afternoon | **04** Spring/Backend → **06** Kafka | 3.5 h |
| Evening | **15** Behavioural — write & say the 3 STARs out loud | 1.5 h |

### DAY 2 — "Fundamentals + floor + breadth"
| Block | Sheets | Time |
|-------|--------|------|
| Morning | **02** Core Java → **03** Concurrency/JVM → **05** Data Layer | 4.5 h |
| Midday | **01** DSA patterns + drill the §"Known asked problems" list | 4 h |
| Afternoon | **07** Angular/TS → **08** Micro-Frontends | 2.5 h |
| Evening | **09** Testing → **10** Cloud/DevOps → **11** Observability | 3 h |

### MORNING OF THE PANEL (45 min — review only, no new material)
Re-read **only** the "🎤 Say it like this" blocks, the 3 behavioural STARs, and the System Design narrative (§12). Glance at the cheat sheet (`99-cheat-sheet.md`).

> **If a day collapses to 8 h:** do 14, 12, 04, 15, and the top-frequency DSA problems (Two Sum, Valid Palindrome, LRU, primes, root-to-leaf sum). Those cover ~80% of a P5 loop's deciding moments.

> **Study technique for a 2-day cram:** For each sheet — (1) read it once for understanding, (2) re-read **only** the 🎤 block and say it aloud, (3) cover the follow-up ladder and answer from memory. Speaking beats re-reading 3:1 for retention under panel pressure.

---

## 🎯 KNOWN ASKED PROBLEMS — pulled from what you sent (prioritised)

**Must-solve DSA (high frequency / explicitly asked):**
- Two Sum (1) · Valid Palindrome (125) · LRU Cache (146) — *design + hashmap + DLL, classic*
- Print primes 1–200 with minimal complexity *(asked in report — Sieve of Eratosthenes)*
- Root-to-leaf path sum equals target *(asked in report — tree DFS/backtracking)*
- Valid Parentheses · Implement Queue using Stacks · Move Zeroes · Sort Colors · Reverse Linked List · Climbing Stairs · Permutations · Decode String · Daily Temperatures · Kth Largest Element · Trapping Rain Water
- Harder/less common: Special Binary String, Minimum Length of Anagram Concatenation, Fraction to Recurring Decimal, Continuous Subarray Sum, Kth Smallest in Sorted Matrix

**Core Java / systems (from the round-by-round report):**
- Beans + lifecycle + multithreading · ConcurrentHashMap vs synchronized HashMap · malloc vs calloc · segmentation faults · HTTP vs SOAP · Clustered vs non-clustered index · what is an index

**System design (explicitly asked across companies):**
- Scale 1 → 10,000 req/sec globally · Design review/rating system · CDN what & why · Caching usage & need

**Ops / breadth (asked):**
- `kill` default signal (SIGTERM, 15) · troubleshoot intermittent bug · CSR vs SSR · BST search · Git · SonarQube · Docker · K8s · CI/CD · SQL joins/grouping

**Behavioural / logistics:**
- Why switch? Expected CTC? Why relocate? · Situational questions · "accurate answer when unsure" · "no documentation, how troubleshoot?"

> Note: some questions in your dump are from a **Support Engineer** loop (Fibre Channel bandwidth, BOOT.INI, Safe Mode vs Recovery Env). Those are **not** P5 SWE material — I've flagged them out of scope so you don't waste a weekend on them. Tell me if you want a quick "just in case" appendix.

---

## ✅ Files in this folder (ALL COMPLETE)

| File | Topic |
|------|-------|
| `00-master-index.md` | This index + 2-day plan |
| `01-dsa-coding-patterns.md` | DSA patterns + the explicitly-asked problems |
| `02-core-java.md` | Collections internals, equals/hashCode, ConcurrentHashMap |
| `03-concurrency-jvm.md` | Threads, volatile/synchronized, GC, + C-memory appendix |
| `04-spring-backend.md` | Spring vs Boot, beans, REST, security, resilience |
| `05-data-layer.md` | Indexes (clustered/non-clustered), SQL, NoSQL, JPA, caching |
| `06-kafka.md` | Partitions, consumer groups, delivery semantics, lag |
| `07-angular-typescript.md` | Change detection, RxJS, CSR vs SSR |
| `08-micro-frontends.md` | Module Federation vs Single-SPA, shared deps |
| `09-testing.md` | Test pyramid, Mockito/WireMock, contract testing, SonarQube |
| `10-cloud-devops.md` | Docker, K8s, CI/CD, deployments, Git |
| `11-observability.md` | Logs/metrics/traces, SLO/SLI/SLA, RED |
| `12-system-design-hld.md` | Framework + 1→10k req/s + review/rating system |
| `13-lld-ood.md` | SOLID, patterns, worked LLD (parking lot) |
| `14-ai-llm.md` | LLM integration, RAG, agents, MCP, AI security, observability |
| `14b-claude-code.md` | Skills, subagents, worktrees, hooks (velocity story) |
| `15-behavioural.md` | STARs from your 3 anchors + logistics answers |
| `99-cheat-sheet.md` | **Morning-of one-glance recall** |

---

**How to use over 2 days:** follow the Day-1/Day-2 table above. For each sheet — read once, then re-read only the 🎤 blocks and say them aloud, then cover the follow-up ladder and answer from memory. Morning of the panel: `99-cheat-sheet.md` only.

**Want me to go deeper on any sheet, add more worked DSA solutions, or run a mock Q&A?** Paste a specific interviewer question and I'll give the ideal spoken answer + follow-up ladder + the trap to avoid.
