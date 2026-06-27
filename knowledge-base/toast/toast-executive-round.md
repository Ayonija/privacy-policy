# Toast — Hiring Manager / Executive Round (Chance Kirsch)

**Focus:** High-level. Are you *senior*? Do you think beyond your ticket to **team, customer, and company impact**? Behavioral signals in **STAR**, anchored on YOUR resume.
**Anchored on actual rounds:** R5 HM round — *conflict management, projects you're proud of, what you expect in the role, how you work with co-workers.*

> ### 📅 YOUR INTERVIEW (confirmed)
> **Thu Jun 18, 2026 · 8:00–8:30 PM IST · Zoom**
> **Interviewer: Chance Kirsch — Senior Principal Software Engineer (Hiring Manager)**
> - **Zoom (all interviews):** https://toasttab.zoom.us/j/98176099273?pwd=0ok8VoaFfy1ElIPwAkJ9byUzTKKwDu.1
> - **Backup-email channel (connection drop):** ayonija.mathur@gmail.com
> - **Scheduling:** Gouri (gouri.gindi@toasttab.com) · Other Qs: Priyalakshmi (priyalakshmi.r@toasttab.com)
>
> **This is only 30 minutes** — it moves fast. Expect: a quick "tell me about yourself," **2–3 behavioral questions**, and time for *your* questions. Be tight: each story ≤ 2 min. This is the **hiring manager** — he decides whether you advance and what you'd own. Read every answer as: *can I trust this person with scope, ambiguity, and customers?*

> **What this round tests:** Not whether you can code — that's established. It's seniority, ownership, judgment, and culture fit. Keep answers **high-level, business-anchored, concise.** Don't dive into code unless asked.

---

## 🎤 Your 60-second opener ("Tell me about yourself")

> *"I'm a full-stack engineer with 5+ years at BNY Mellon, almost all of it on a **mission-critical, regulated financial platform** — the kind of environment where software that touches money has to be exactly right and always up. That instinct is what drew me to Toast: when a POS goes down, a restaurant can't take payment, so reliability isn't a feature, it's the product. For the last two years I've moved into **platform and developer-experience** work — I built my org's first MCP server connecting AI assistants to internal APIs, and a CLI platform that cut new-app onboarding from two days to fifteen minutes and spread to 30-plus teams with **zero top-down mandate**. What I love, and what I think maps directly to Toast, is **building things people genuinely adopt** — whether the user is a developer or a restaurant operator, you only win if the tool survives a chaotic, real-world workflow. I'm looking for exactly that: owning a system end-to-end where the customer feels the wins and losses immediately."*

**Why this works:** leads with the reliability/regulated-money bridge to Toast → pivots to adoption/empathy → ends with what you want. Memorize the three beats (regulated reliability → adoption with no mandate → end-to-end ownership), not the words.

---

## The 4 questions you WILL be asked — your scripted STAR answers

Use **STAR**: **S**ituation (1–2 sentences) → **T**ask (your goal) → **A**ction (the bulk — what *YOU* did) → **R**esult (quantified + lesson). ~2 min each. **"I," not "we,"** for your actions.

---

### 1. "Tell me about a conflict and how you managed it."
**What they listen for:** maturity, low ego, outcome over being right, customer/business as tie-breaker.

**Your story — the MCP server security model:**
- **S:** "When I built our org's first MCP server exposing internal platform APIs to AI assistants, a senior engineer wanted a broad, flexible natural-language query surface — basically let the LLM reach anything, for maximum usefulness in the demo. I disagreed: we're a **regulated bank**, and an over-permissioned LLM is an injection and data-exfiltration risk."
- **T:** "I needed us to land on a security model we could actually ship in a regulated environment, without it becoming a stalemate or me just overruling a senior."
- **A:** "I separated the person from the problem. I wrote both approaches down with their trade-offs, then re-anchored us on the real constraint — *what passes security review and protects customer data* — instead of who was right. I proposed a middle path: a **least-privilege, schema-constrained surface** — the LLM only gets explicitly allowlisted, contract-defined operations — which kept most of the usefulness he wanted while making the risk reviewable. We validated it with our security team rather than arguing in the abstract."
- **R:** "It passed review, shipped, and became the **reference architecture** other teams now copy for LLM integration. The relationship stayed strong — we still collaborate. The lesson: most 'technical' conflicts dissolve the moment you re-anchor on the shared constraint instead of defending a position."

> **Tie to Toast:** "I treat disagreements as a way to pressure-test a decision before it reaches the customer. At a company where a wrong call can mean a restaurant can't charge a guest, getting the decision *right* matters more than getting *my* answer adopted."

*(Backup conflict story if he probes for a second: the Angular 17→19 migration — product wanted features, I needed the migration done. I made the case in business terms — 30% build-time cut, security via ESLint 9 flat config, zero regression — and negotiated doing it incrementally along the feature path rather than as a blocking standalone sprint.)*

---

### 2. "Tell me about a project you're proud of."
**What they listen for:** scope, ownership, *impact* (not effort), tech win → business outcome.

**Your story — the Developer Platform CLI (most Toast-shaped: adoption without mandate):**
- **S:** "Spinning up a new application in my org took about **two days** of manual setup — CI/CD, testing, linting, auth — and everyone did it slightly differently, which created security and quality drift."
- **T:** "I owned the idea and the build: a single-command CLI that provisions a production-ready app with all of that baked in."
- **A:** "I built the scaffolding engine end-to-end, made the secure, tested, compliant setup the **default path** rather than something teams had to opt into, and iterated tightly on developer feedback. Critically, I didn't try to mandate it — I made it good enough that people *chose* it."
- **R:** "Onboarding dropped from **two days to fifteen minutes**, it spread to multiple squads with **34+ merged contributions purely by word-of-mouth, zero mandate**, and it raised our security/quality baseline by default. It's the work I'm proudest of because adoption was *organic* — that's the real proof it solved a felt problem."

> **Make it Toast-shaped (say this):** "This is why Toast appeals to me — you can't force a busy, under-staffed restaurant to use a feature any more than you can force a developer. It has to be so simple and resilient that they adopt it on their own. Building for that reality is exactly what I did here."

*(Alt story if he wants reliability over adoption: the MCP server — org's first, owned API contract + security model + cloud integration, now the reference architecture and the demo that convinced leadership. Or the BNY reliability story: remediated 20+ Veracode CVEs and ran Spring Boot 2→3 / Java 11→17 upgrades with **zero production regression** on a live financial platform — eliminated 100% of critical security debt.)*

---

### 3. "What do you expect from this role?"
**What they listen for:** your motivations match what the role/company offers → you'll stay and thrive.

**Your answer:**
- **Ownership:** "A system I own end-to-end, with scope that grows as I show impact — I do my best work when I own the outcome, not just a ticket."
- **Mission fit:** "I want my work to visibly help a real user. At a bank the feedback loop is slow and abstract; Toast's restaurants feel a software win or loss **immediately**. That tight loop is exactly what motivates me."
- **Craft + team:** "A strong engineering culture — I saw code review and reliability are part of how Toast works, and I want teammates who raise my bar."
- **Two-way (ask back):** "What does success in this role look like at 6 and 12 months?"

> Avoid: "I expect promotion in a year" or pure comp framing. Frame around **impact and growth** — that's what a hiring manager funds.

---

### 4. "How do you work with co-workers?"
**What they listen for:** collaboration style, giving/receiving feedback, reliability as a teammate.

**Your answer (use real proof points):**
- **Transparency:** "I over-communicate status and flag blockers early — no surprises. On distributed, cross-functional releases — I shipped white-label identity verification across 5 environments and multiple brands over 12+ versions — async clarity is the whole game."
- **Review as teaching, not gatekeeping:** "As an internal developer advocate I was consulted across squads on LLM integration and API design. I review for security and correctness first, always explain the *why*, and I take feedback on my own code without ego."
- **Lift the team:** "I drove 2× commit velocity not just by my own output but by writing the templates, tutorials, and workflows that made the *whole team* faster. I leave the codebase — and the team — better than I found them."
- **Reliable:** "I own my commitments and the parts that break."

---

## Executive framing: tie YOUR work to Toast's objectives

Whatever he asks, answer it, then zoom out to one of these company truths and connect your work to it:

| Toast company objective | Your soundbite (resume-backed) |
|---|---|
| **Reliability = revenue** (POS down → no payments) | "I come from a regulated financial platform under SLA — I ran Spring Boot/Java upgrades and CVE remediation with *zero* production regression. Uptime and correctness are my default, not an afterthought." |
| **Win + retain restaurants** | "I build for chaotic real-world adoption — my CLI platform spread to 30+ teams with no mandate because it survived real workflows. Same instinct a restaurant floor demands." |
| **Grow transaction/payment volume** | "I think about whether work deepens the core loop, not just isolated features — I built platform tooling that made *other* teams ship faster, which is leverage, not local optimization." |
| **Trust** (money + hours must be exact) | "I've worked where money data must be exactly right — entitlement microservices, financial flows. I treat those paths as auditable and idempotent, never destructive edits." |
| **Velocity at scale** | "I favor clean seams — events, flags, contracts, code-gen — so the team adds capability without destabilizing the critical path. Speed *and* safety." |

> **The senior move:** answer the specific thing, then ladder it up to *customer → restaurant → Toast's business.* That's what gets you onto a hiring manager's "I'd bet on this person" list.

---

## Questions to ask Chance (signals seniority + genuine interest)
Pick 2–3 — it's only 30 min:
- "As a Senior Principal, what are the biggest technical bets the org is making this year, and where does this team sit in them?"
- "Where does reliability/uptime rank against feature velocity right now, and how is that tension managed day to day?"
- "What does a great hire in this role do in their first 90 days?"
- "How does the company measure whether engineering is succeeding — what's the north-star metric?"
- "What's the hardest scaling challenge you see as Toast grows — transaction volume, the offline-sync model, multi-region?"

---

## Final delivery checklist
- [ ] **60-sec opener** rehearsed — three beats (regulated reliability → adoption with no mandate → end-to-end ownership).
- [ ] Conflict (MCP security model) + proud-of (CLI platform) stories ≤ 2 min each, ending in **quantified result + business impact**.
- [ ] "I" for your actions; "we" only for genuine team context.
- [ ] Stay **high-level** — no code/architecture dives unless invited.
- [ ] Re-anchor every answer to **customer → restaurant → company objective**.
- [ ] 3 sharp questions ready; a 30-min HM round with no questions reads as low interest.
- [ ] Test Zoom + mic/cam ~10 min early; have ayonija.mathur@gmail.com open for drop-emails.
- [ ] Calm, concise, confident. Seniority shows in what you *leave out*.
