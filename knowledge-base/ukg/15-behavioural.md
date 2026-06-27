# 15 — Behavioural & Technical Leadership (P5) — DECIDES THE LOOP

> At P5 the behavioural round isn't soft — it's where they decide if you operate at Staff altitude. The signal: **impact through leverage** (tooling, standards, influence), **judgement under ambiguity**, and **ownership**. Everything below is built from your 3 anchors so you tell *the same true stories* through different lenses.

---

## Your 3 anchors (memorize the metrics — they're your currency)
- **A1 — MCP server:** org's **first** MCP server exposing internal enterprise APIs to AI assistants. Owned API contract + security model + cloud integration. Became the **reference architecture / canonical demo**.
- **A2 — CLI developer platform:** cut onboarding **2 days → 15 min**; adopted organically across squads (**34+ MRs**) with **zero mandate**.
- **A3 — 2× commit velocity:** designed AI-assisted dev workflows + authored the templates/content teams adopted by default (Q1 2026 vs prior).
- *Supporting:* Angular 17→19 migration (**30% build-time cut, zero regression**), Spring Boot 2→3 / Java 11→17 (**20+ CVEs, zero critical debt**), open-source **AI Governance Copilot**.

**STAR =** Situation, Task, Action, Result. Always land on a **quantified Result** + a **reflection** ("what I'd do again / differently").

---

## Competency 1 — Influence without authority (THE Staff signal — likely asked)
**What they're testing:** Can you drive adoption/change across teams you don't manage? Staff impact is multiplied, not mandated.

**STAR (A2 — CLI platform):**
- **S:** New-app setup took ~2 days of manual, error-prone config; every squad reinvented CI/CD, auth, linting differently. I had no authority to mandate anything across squads.
- **T:** Reduce onboarding friction and raise the baseline — without a top-down mandate, which I knew would breed resistance.
- **A:** I treated developers as customers. Built a single-command CLI that scaffolds a production-ready app — CI/CD, tests, lint, auth baked in. I made *the right way the easy way*, so adoption was a choice, not an order. I dogfooded it, seeded it with two friendly squads, incorporated their feedback fast, wrote the docs/templates, and let word-of-mouth carry it. I showed up in architecture forums to demo, not to dictate.
- **R:** Onboarding **2 days → 15 minutes**; organic spread across multiple squads with **34+ merged requests**, **zero mandate**; it quietly raised engineering standards by default. I learned that adoption is a product problem — you win by reducing friction and earning trust, not by policy.

🎤 **90-second version:** *"The clearest example is a developer platform I built at the bank. New-app setup was a two-day manual slog and every squad did CI, auth, and linting differently — but I had no authority to standardize anything. So instead of pushing a mandate, which I knew would get resisted, I treated developers as customers and made the right way the easy way: a single command that scaffolds a production-ready app with CI/CD, tests, linting, and auth already wired. I dogfooded it, seeded it with two friendly squads, turned their feedback around fast, and let it spread by word of mouth. It went from two days to fifteen minutes and spread organically across squads — 34-plus merged requests with zero top-down mandate. The lesson I carry: at a senior level you don't get adoption by authority, you get it by removing friction and earning trust. That's how I'd drive any cross-team change."*

**Probes:** *Resistance?* "Two squads had homegrown setups; I didn't fight them — I made mine interoperate and let the time-savings argue for me." *Measure success?* "Adoption (MRs, squads), onboarding time, and standard-compliance by default." *Scale further?* "Feedback loop into a roadmap; treat it as a product with versioning and support."

---

## Competency 2 — Driving a technical decision / ambiguity
**What they're testing:** structured decision-making under uncertainty; owning a call and its trade-offs.

**STAR (A1 — MCP server):**
- **S:** AI coding assistants were arriving, but had no safe path to internal enterprise data — huge upside, but in a *regulated bank* the risk (data exposure, prompt injection) was real and the path was ambiguous; no precedent existed.
- **T:** Define *how* the org would let LLMs touch internal systems — safely and as a repeatable pattern.
- **A:** I scoped it to a thin, high-value slice first. The hard part wasn't the protocol — it was the **contract and security model**: I exposed intent-level tools, not raw APIs; enforced authZ per call with the *user's* permissions; treated tool inputs as untrusted; audit-logged everything. I socialized the design in architecture forums to pressure-test it, and built a working demo to make it concrete.
- **R:** Org's **first MCP server**; became the **reference architecture** for LLM-to-enterprise integration and the demo that convinced both leadership and engineers. I established the pattern, not just a feature.

🎤 **90-second version:** *"When AI assistants started landing, there was a real pull to connect them to internal systems and zero safe pattern for doing it — in a regulated bank, that ambiguity is risk. I took it on and built the org's first MCP server. I deliberately started with a thin, high-value slice rather than boiling the ocean. The engineering wasn't the protocol — it was the security model: I exposed intent-level operations instead of raw APIs, enforced authorization per call with the user's own permissions so the assistant never saw data the user couldn't, treated every tool input as untrusted because the model is now an actor in the system, and audit-logged everything. I pressure-tested the design in architecture forums and shipped a working demo. It became the reference architecture other teams now follow. What I'm proud of is that I delivered a *pattern*, not a one-off — that's the multiplier."*

**Probes:** *Biggest risk + mitigation?* prompt injection / excessive agency → least-privilege tools, authZ in code, human-in-loop on writes. *Trade-off you made?* scope (thin slice) over completeness, to ship and learn. *If it failed?* contained blast radius — narrow scope, audit trail, reversible.

---

## Competency 3 — Mentorship / making others better
**What they're testing:** do you scale yourself through others (core P5 expectation)?

**STAR (A3 — velocity + content):**
- **S:** Teams were adopting AI-assisted development ad hoc — inconsistent quality, people unsure how to use LLMs well or safely.
- **T:** Level up the team's ability to build *with* LLMs, by default.
- **A:** I authored the tutorials, templates, and reusable workflows that became the team's default setup; encoded standards into tooling (hooks, skills) so the good path was automatic; ran as the go-to person engineers consulted on LLM integration and API design; synthesized their feedback into the roadmap and shipped against it.
- **R:** **2× commit velocity** (Q1 2026 vs prior); became the internal developer advocate / primary technical voice in cross-squad forums. I scaled my impact through content and tooling rather than doing the work for people.

🎤 **90-second version:** *"My favorite kind of impact is the kind that outlives me. Teams were adopting AI-assisted development unevenly — some great, some risky. Rather than fix it case by case, I built the defaults: tutorials, templates, and reusable workflows that became the team's standard setup, and I encoded the standards into tooling so the good path was the automatic path. I made myself available as the person people consulted on LLM integration and API design, and I fed what I learned back into the roadmap. The measurable result was a doubling of commit velocity quarter over quarter, but the part I value is that it came from making other engineers faster and safer, not from me writing more code. That's how I think about seniority — leverage, not heroics."*

**Probes:** *Someone resisted your approach?* I treat that as signal — usually they have a constraint I missed; I adapt the tool. *Mentoring a struggling engineer?* pair, give context not answers, small wins, gradually hand over ownership.

---

## Competency 4 — Disagreement / conflict
**What they're testing:** can you disagree productively and commit?

**STAR (Angular 17→19 migration):**
- **S:** Some engineers wanted to defer the Angular 17→19 + Karma→Vitest migration as risky on a large live codebase; I believed the compounding cost of staying behind was worse.
- **T:** Align on doing it, safely, without production regression.
- **A:** I didn't argue from opinion — I de-risked it with data: a spike branch, an incremental migration plan, automated tests as the safety net, and a measured build-time benchmark. I steel-manned their risk concern and addressed it directly (incremental, reversible, tested) rather than dismissing it.
- **R:** **30% build-time reduction, zero production regression.** The disagreement made the plan safer. Lesson: win disagreements with evidence and by addressing the other side's real concern, then commit fully either way ("disagree and commit").

🎤 **Probes:** *Lost a technical argument?* "I commit fully once the team decides — and I make the new path succeed; a decision relitigated forever is worse than a slightly-suboptimal one owned by everyone." *Disagree with your manager?* private, evidence-based, their call after.

---

## Competency 5 — Failure / ownership
**What they're testing:** honesty, accountability, learning (no "my weakness is I work too hard").

**STAR (pick a real, contained one — e.g. early CLI adoption stall or a migration hiccup):**
- **S/T:** Early CLI version made assumptions about squad setups; one squad couldn't adopt it cleanly.
- **A:** I owned it, talked to them directly, learned my tool was too opinionated, refactored it to be configurable, and added a feedback channel so I'd catch this earlier next time.
- **R:** That squad adopted it; the feedback loop became part of how the tool evolved. Lesson: build *with* users, ship smaller, validate assumptions earlier.

🎤 **Line that lands:** *"I'd shipped my assumptions instead of validating them. Owning that — and turning it into a feedback loop — is what made the platform actually spread."*

---

## Competency 6 — Logistics questions (prep crisp, confident answers)
- **Why switch?** *"I've had deep impact at the bank — built its first MCP server and a platform adopted with zero mandate. I'm looking for a Staff-level scope where AI-forward, full-stack product work is the core mission, not a side effort, and where I can multiply a broader engineering org. UKG's combination of large-scale full-stack and a serious AI push is exactly that."* (Frame as **toward**, never **away/bashing**.)
- **Why relocate (Pune)?** Be direct and positive: on-site collaboration, the role/scope, you're committed to the location. Keep it short and certain.
- **Expected CTC?** *"I'm targeting a band appropriate for a Staff/P5 role with my full-stack plus AI specialization; I'm confident we can align if the fit is right. Do you have a band in mind for this level?"* (Give a researched range if pushed; deflect-then-anchor, stay collaborative.)
- **"How do you give an accurate answer when unsure?"** *"I say what I know, flag the boundary of what I don't, state my best hypothesis and how I'd verify it quickly. I'd rather be transparently uncertain than confidently wrong — especially in a regulated environment."*
- **"Troubleshoot an intermittent bug / no documentation?"** *"Make it reproducible first — logs, metrics, traces to find a pattern (load? a specific node? timing?). Form a hypothesis, isolate by bisecting, add observability where it's blind. Intermittent usually means concurrency, resource limits, or an environment difference. No docs just means I instrument and read the source."*

---

## ⚠️ What makes a story land vs fall flat at Staff level
- **Land:** quantified result, *your* specific decisions ("I chose X over Y because…"), scope beyond yourself (cross-team, pattern, tooling), a real reflection.
- **Flat:** "we" with no clear *you*; no metric; describing activity not impact; no trade-off; blaming others; rehearsed-robotic delivery.
- **The P5 multiplier test:** every story should answer "how did this make *other people or the org* better?" — not just "I built a thing."
- Keep a 90-second version *and* a 20-second version of each; let them pull detail with follow-ups rather than front-loading everything.

---

*Next topic, or drill deeper on this one?*
