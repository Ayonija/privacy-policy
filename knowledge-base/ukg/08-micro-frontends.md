# 08 — Micro-Frontends (MFE)

> You built a Module Federation code-gen engine — this is home turf. Keep it crisp: what, why, the two approaches, and the real pitfalls (shared deps, versioning).

---

## What & why
**Plain English:** Micro-frontends apply the microservices idea to the UI — **split a big frontend into independently developed and deployed pieces**, each owned by a team, composed into one app at runtime.

**Why it exists:** a single large SPA becomes a bottleneck — one repo, one release train, one tech-version lockstep, teams stepping on each other. MFEs give **independent deployment + team autonomy + isolated blast radius**, at the cost of integration complexity.

**When NOT to:** small app / single team — the overhead (shared deps, versioning, runtime integration) isn't worth it. MFEs solve an *organizational* scaling problem, not a technical one.

---

## The two main approaches
**Module Federation (Webpack 5 / now also rspack):**
- **Runtime** integration: a **host** (shell) loads **remote** bundles at runtime; remotes expose modules the host imports dynamically. Apps can **share dependencies** (load one copy of Angular/React across remotes via the `shared` config with version negotiation).
- Pro: true runtime composition, shared singletons, no rebuild of the host to update a remote. Con: tight build-tool coupling, version-mismatch risk.

**Single-SPA:**
- An **orchestrator** that registers applications and mounts/unmounts them by route (lifecycle: bootstrap → mount → unmount). Framework-agnostic (mix Angular + React).
- Pro: mix frameworks, clear lifecycle. Con: you handle shared deps yourself; more orchestration glue.

(Other patterns: **iframes** — strongest isolation, worst UX/communication; **Web Components** — standards-based, framework-neutral, your "Angular Elements" engine; **build-time integration** — simplest but loses independent deploy.)

🎤 **Say it like this:** *"Micro-frontends extend the microservices philosophy to the UI: independently built and deployed pieces, each team-owned, composed into one app. The point is organizational — independent deploys and team autonomy — so I wouldn't use them for a small single-team app where the integration overhead isn't justified. The two mainstream approaches are Module Federation, which composes remote bundles at runtime and lets apps share a single copy of a framework via version negotiation, and Single-SPA, which is a route-level orchestrator that's framework-agnostic so you can even mix Angular and React. I've built on Module Federation with an Angular Elements code-gen engine, so I've lived the hard part — shared dependency and version management."*

---

## The real pitfalls (where seniority shows)
1. **Shared dependency / version management** — if each remote bundles its own Angular, you ship it many times (bloat) and risk **multiple framework instances** (broken DI/zone state). Fix: declare frameworks as **shared singletons** with compatible version ranges; align major versions across teams (governance problem).
2. **Version skew** — host and remote built against different versions → runtime breakage. Need a contract + compatibility policy.
3. **CSS/style leakage** — global styles from one MFE bleed into another. Fix: scoping (Shadow DOM / CSS modules / prefixing).
4. **Cross-MFE communication** — avoid tight coupling; communicate via a thin shared contract: custom events / a shared event bus / URL/state — **not** by reaching into each other's internals.
5. **Shared state & auth** — single sign-on/token shared via the shell; don't duplicate auth per MFE.
6. **Performance** — multiple bundles, duplicated vendor code, waterfall loads. Mitigate with shared deps + lazy/prefetch.
7. **Coupling creep** — the whole point is decoupling; a shared "common" library that everything depends on quietly re-couples teams (versioning hell). Keep shared surface minimal and stable.

🎤 **The pitfalls line:** *"The failure mode everyone hits is shared dependencies. If each micro-frontend bundles its own copy of the framework, you both bloat the download and risk two framework instances fighting over things like dependency injection and change-detection state. So the framework gets declared a shared singleton with a compatible version range, and major-version alignment across teams becomes a governance conversation, not just a config. The other quiet killer is coupling creep — a giant shared library that everything depends on re-couples the teams you split apart — so I keep the shared surface minimal and communicate across MFEs through thin contracts like custom events, never by reaching into another app's internals."*

**Follow-ups:**
- *Q: Two MFEs need different Angular major versions?* Either don't share (accept the bloat/isolation) or align — you can't run two incompatible singletons safely; this is why version policy matters.
- *Q: How do MFEs talk without coupling?* Shared events/bus or URL state with a documented contract; treat each other as black boxes (same discipline as microservice APIs).
- *Q: Independent deploy in practice?* Each remote deployed to its own URL; the shell loads the latest remote entry at runtime — so a team ships without a host rebuild.

---

## ⚠️ Seniority signals
- Framing MFEs as an **organizational** scaling tool with a real cost, and naming **when not to** use them.
- Knowing the **multiple-framework-instance** danger and shared-singleton fix.
- Treating cross-MFE comms like service contracts (decoupled), and calling out coupling-creep via shared libs.

---

*Next topic, or drill deeper on this one?*
