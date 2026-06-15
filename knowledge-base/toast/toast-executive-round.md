# Toast — Executive / Hiring-Manager Round

**Focus:** High-level. How your work ties to Toast's **broader company objectives.** Behavioral signals in **STAR**.
**Anchored on actual rounds:** R5 HM round — *conflict management, projects you're proud of, what you expect in the role, how you work with co-workers.* Plus the standard exec/leadership round.
**Study time:** ~45 min

> **What this round tests:** Not whether you can code — that's already established. It's: *Are you senior?* Do you think beyond your ticket to **team, customer, and company impact**? Are you someone a leader can trust with ambiguity and put in front of customers? Keep answers **high-level, business-anchored, concise.** Don't dive into code unless asked.

> **A note on the dataset feedback:** one candidate was told *"due to business changes they paused recruitment."* That's outside your control — it is **not** a reflection of performance. Prepare to be excellent; some outcomes are macro, not you.

---

## The 4 questions you WILL be asked (R5 confirmed) — prep one STAR story each

Use **STAR**: **S**ituation (context, 1–2 sentences) → **T**ask (your responsibility/goal) → **A**ction (what *you* did — the bulk) → **R**esult (quantified outcome + what you learned). Keep each to ~2 minutes. **"I," not "we,"** when describing *your* actions.

### 1. "Tell me about a conflict and how you managed it."
**What they're listening for:** maturity, low ego, focus on outcome over being right, customer/business as the tie-breaker.

**STAR skeleton (fill with your real story):**
- **S:** Two engineers (or you + a senior) disagreed on an approach — e.g. ship a quick fix vs. a proper refactor, or REST vs. event-driven for a service.
- **T:** I needed us to land on a decision without it festering or blocking the release.
- **A:** I separated the *people* from the *problem* — got both positions written down with their trade-offs, re-anchored on the **actual goal** (what does the customer/business need, not who's right), proposed a path that took the strongest part of each, and agreed on a way to validate it (data/spike) rather than argue in the abstract.
- **R:** We shipped on time; the chosen approach held up; the relationship stayed strong (we still pair regularly). **Lesson:** most "technical" conflicts dissolve when you re-anchor on the shared objective.

> **Tie to company objective:** "I treat disagreements as a way to pressure-test a decision before it reaches the customer — at a company where downtime means a restaurant can't take payment, getting the decision right matters more than getting *my* answer adopted."

### 2. "A project you're proud of."
**What they're listening for:** scope, ownership, *impact* (not just effort), and whether you can connect a technical win to a business outcome.

**STAR skeleton:**
- **S:** A meaningful problem with a business stake (a system that was slow/unreliable/blocking the team).
- **T:** My ownership — I led/drove X.
- **A:** The key technical *and* collaborative decisions — what I built, who I aligned, what I cut.
- **R:** **Quantify:** "reduced X by N%," "unblocked the team to ship Y," "cut on-call pages by Z." Then *why it mattered to the business*.

> **Make it Toast-shaped:** pick a story about **reliability, real-time data, or correctness of money/hours** if you have one — it maps directly to Toast's POS/payments world. End every "proud of" story with the **business impact**, not the cleverness.

### 3. "What do you expect from this role?"
**What they're listening for:** that your motivations match what the role/company actually offers → you'll stay and thrive.

**Strong answer structure:**
- **Growth:** "Ownership of a system end-to-end, and scope that grows with impact."
- **Mission fit:** "I want my work to visibly help a real user — Toast's restaurants feel software wins/losses *immediately*; that tight feedback loop is motivating."
- **Craft + team:** "Strong engineering culture, code review that raises the bar (I saw that's part of your process), and teammates I learn from."
- **Two-way:** ask back — "What does success in this role look like at 6 and 12 months?"

> Avoid: "I expect to be promoted in a year" / pure comp framing. Frame around **impact and growth**, which is what executives fund.

### 4. "How do you work with co-workers?"
**What they're listening for:** collaboration style, how you give/receive feedback, reliability as a teammate.

**Hit these points:**
- **Default to transparency:** I over-communicate status, flag blockers early, no surprises.
- **Code review as teaching, not gatekeeping:** I review for security/correctness first, explain the *why*, and I take feedback on my own code without ego (the R3-style review round is normal craft, not criticism).
- **Lift the team:** I document, I pair with newer folks, I leave the codebase easier than I found it.
- **Reliable:** I own my commitments and the parts that break.

---

## Executive-level framing: tie YOUR work to Toast's objectives

When an exec asks anything, zoom out to one of these company-level truths and connect your work to it:

| Toast company objective | How an engineer's work ladders up to it (your soundbites) |
|---|---|
| **Reliability = revenue** (POS down → restaurant can't take payment) | "I optimize for correctness and uptime first; in this business an outage isn't a bug ticket, it's a restaurant losing money in real time." |
| **Win + retain restaurants** | "Every feature has to survive a chaotic, low-tech, offline-prone restaurant floor. I build for that reality — simple, resilient, offline-first." |
| **Grow transaction/payment volume** (Toast's core revenue) | "I think about whether my work deepens the core loop — more orders and payments flowing through Toast — not just isolated features." |
| **Trust** (money + hours must be exactly right) | "Payroll hours and guest charges have to be exactly right. I treat those data paths as append-only, auditable, and idempotent — no destructive edits on money." |
| **Velocity at scale** | "I favor designs (events, flags, clean seams) that let the team add capability without destabilizing the write path — speed *and* safety." |

> **The senior move:** whatever they ask, answer the specific thing, then connect it upward to *customer → restaurant → Toast's business*. That's what separates "a good engineer" from "someone I'd put on my leadership's radar."

---

## Questions to ask the executive (signals seniority + genuine interest)
- "What are the biggest technical bets the org is making over the next year, and where does this team sit in them?"
- "Where does reliability/uptime rank against feature velocity right now, and how is that tension managed?"
- "As Toast scales, what's the hardest scaling challenge you see — transaction volume, the offline-sync model, multi-region?"
- "What does a great hire in this role do in their first 90 days?"
- "How does the company measure whether engineering is succeeding — what's the north-star metric?"

---

## Final delivery checklist for the exec round
- [ ] Each STAR story ≤ 2 min, ends with **quantified result + business impact**.
- [ ] "I" for your actions; "we" only for genuine team context.
- [ ] Stay **high-level** — no code/architecture dives unless invited.
- [ ] Re-anchor every answer to **customer → restaurant → company objective**.
- [ ] Have 3–4 sharp questions ready; an exec round with no questions reads as low interest.
- [ ] Calm, concise, confident. Seniority is conveyed as much by *what you leave out* as what you say.
