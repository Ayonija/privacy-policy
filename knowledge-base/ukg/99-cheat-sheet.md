# 99 — MORNING-OF CHEAT SHEET (read in the last 45 min, nothing else)

> One-glance recall of the highest-yield facts + your spoken hooks. If a line here feels shaky, open that topic's sheet; otherwise just say the lines out loud.

---

## 🎯 The 3 anchors (your currency — say the metrics)
1. **MCP server** — org's *first*; owned contract + security model; became the *reference architecture*.
2. **CLI platform** — onboarding *2 days → 15 min*; *34+ MRs*, organic, *zero mandate*.
3. **2× commit velocity** — AI workflows + templates teams adopted by default.
*(Backups: Angular 17→19 = 30% build cut, zero regression. Spring Boot 2→3 = 20+ CVEs cleared. Governance Copilot = OWASP LLM Top 10.)*

## P5 mantra
**Impact through leverage** (tooling/standards/influence) · **trade-offs with the cost named** · **depth that survives 3 follow-ups** · coding is a floor, design + leadership is the bar.

---

## One-liners (the lines that read senior)
- **Spring vs Boot:** "Spring gives the power (DI/container); Boot removes the ceremony — auto-config, starters, embedded server + Actuator."
- **Bean lifecycle:** scan → instantiate → inject → @PostConstruct → post-processors wrap (AOP/@Transactional) → in use → @PreDestroy. Constructor injection by default.
- **@Transactional self-call fails:** it's a proxy; internal calls bypass the proxy.
- **volatile vs synchronized:** volatile = visibility only (no atomicity); synchronized = mutual exclusion + visibility; counter → AtomicInteger (CAS).
- **HashMap:** array of buckets, hash→bucket, chain→tree past 8, resize at 0.75. **ConcurrentHashMap:** lock-free reads + per-bin lock (not whole-map).
- **equals/hashCode:** equal objects → equal hashCodes; override both on same fields; immutable keys.
- **GC:** young (minor, copying) / old (major); G1 region-based, pause-target. Leak = unintended reachability.
- **Index:** sorted B-tree, O(log n). **Clustered** = physical order, 1/table, leaf=row. **Non-clustered** = separate, many, extra hop unless covering.
- **WHERE vs HAVING:** WHERE filters rows before grouping; HAVING filters groups after aggregation.
- **N+1:** 1 query for parents + N for children → JOIN FETCH / entity graph; keep lazy + fetch explicitly.
- **ACID vs BASE; CAP:** under a partition pick CP (correct) or AP (available); PACELC: else latency vs consistency. Strong for money, eventual for counts.
- **Kafka:** partition = ordering + parallelism unit; consumer group splits partitions (1 per group); at-least-once + **idempotent consumers**; watch **consumer lag**.
- **Idempotency:** client idempotency key → server stores key→result → replay returns stored result (pairs with retries/at-least-once).
- **JWT:** signed (not encrypted), stateless verify, valid-till-expiry → short TTL + refresh; OAuth2 = authz, OIDC = identity/login.
- **Resilience:** timeout every call → retry+backoff+jitter (idempotent only) → circuit breaker (fail fast) → bulkhead (isolate pools) → fallback. Saga for distributed txns.
- **Change detection:** zone.js patches async → checks tree; OnPush = re-check only on input *reference* change/event/async pipe → needs immutability.
- **switchMap vs mergeMap:** switchMap cancels previous (search box); mergeMap concurrent; exhaustMap ignores new (anti double-submit).
- **CSR vs SSR:** CSR = browser builds (interactive, weak SEO); SSR = server HTML + hydrate (fast paint, SEO). SSR/SSG for public pages, CSR for dashboards.
- **MFE:** org-scaling tool; Module Federation (runtime, shared singletons) vs Single-SPA (route orchestrator, framework-agnostic); danger = duplicate framework instances → shared singleton + version policy.
- **Docker:** image = layered snapshot; container = isolated process sharing host kernel (lighter than VM); multi-stage + non-root.
- **K8s:** pod (ephemeral) → Deployment (N replicas, rolling) → Service (stable L4 endpoint) → Ingress (L7 edge routing) → HPA (autoscale); readiness/liveness probes.
- **Deploy:** canary (ramp on metrics) / blue-green (instant rollback) / rolling; feature flags → deploy ≠ release.
- **Observability:** metrics=what's wrong, traces=where, logs=why; RED (rate/errors/duration); SLI<SLO<SLA; error budget = 1−SLO drives release risk.
- **SonarQube:** static analysis + quality gate in CI blocks merge; SAST(code) vs SCA(deps) vs DAST(running).
- **Test pyramid:** many unit / some integration / few E2E; contract testing (Pact) for microservices; test behavior not internals.

---

## AI/LLM (your edge — go deepest)
- **"Prompts are pleading, schemas are physics"** — structured outputs mask the token sampler so invalid JSON *can't* generate.
- **RAG:** chunk(semantic+overlap) → embed → vector DB → retrieve top-k(cosine) → re-rank → augment+cite. Hybrid (vector+keyword) for exact terms. Not for whole-doc tasks.
- **Agents:** simplest thing that works — workflow (predictable) over agent (dynamic); bound steps/cost; schema-validated quality gates; human-in-loop for irreversible.
- **MCP:** USB-C for AI; server exposes tools (model-called actions) / resources (read-only data) / prompts; hard part = contract + security model.
- **AI security:** model = untrusted actor; prompt injection (esp. indirect) can't be prompted away → authZ in code with *user's* perms, least-privilege tools, human-in-loop. OWASP LLM Top 10.
- **AI observability:** tokens (cost), TTFT/p95 (latency), eval suite in CI (drift/quality), traces per call.
- **RAG vs fine-tune vs prompt:** prompt first → RAG for *facts* → fine-tune for *behavior* (not facts). Often combine.

---

## DSA quick-fire (the asked ones)
- **Primes 1→200:** Sieve of Eratosthenes, O(n log log n), cross multiples from p².
- **Root-to-leaf sum:** DFS, subtract per node, check remainder at *leaf*. O(n)/O(h).
- **Two Sum:** one-pass hashmap of complement. O(n).
- **LRU:** HashMap + doubly-linked list (or access-order LinkedHashMap). O(1).
- **BST search:** go left if smaller, right if larger. O(h).
- Patterns: hashing · two-pointer · sliding window · stack · binary search · BFS/DFS · heap(top-K) · backtracking · intervals · DP. **State Big-O before coding; test edges after.**

---

## Logistics answers (have these ready)
- **Why switch:** toward Staff scope where AI-forward full-stack is the core mission + bigger org to multiply. (Never bash current.)
- **Why relocate:** positive, certain, on-site collaboration + the role.
- **CTC:** "targeting a band appropriate for Staff/P5 with my full-stack + AI specialization — do you have a band in mind?"
- **Unsure:** state what I know, flag the boundary, give a hypothesis + how I'd verify. Better transparently uncertain than confidently wrong.
- **Intermittent bug:** make it reproducible → logs/metrics/traces for a pattern → hypothesis → isolate/bisect → add observability. Usually concurrency / resource / env difference.

---

## Behaviour under pressure
- **Clarify before you design.** State scale, then architecture. Narrate evolution under load.
- **Every tech choice gets a "but" (the cost).** No buzzword without a trade-off.
- **Don't gold-plate coding** — clear the floor, show judgement, move on.
- **"We" → say what *you* did.** Land on a metric + a reflection.
- Breathe. You've built the things they're asking about. Tell the truth, with depth.

---

*Good luck. You've got the depth — now just say it out loud.*
