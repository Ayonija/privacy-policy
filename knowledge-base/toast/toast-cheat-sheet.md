# Toast — One-Page Cheat Sheet (glance during Zoom)

**Thu Jun 18 · Zoom:** https://toasttab.zoom.us/j/98176099273?pwd=0ok8VoaFfy1ElIPwAkJ9byUzTKKwDu.1 · drop-email: ayonija.mathur@gmail.com
- **8:00–8:30 PM IST** — Chance Kirsch (Sr Principal SWE) — **HM round**, behavioral
- **9:00–10:00 PM IST** — Alan Quigley (Principal SWE) — **deep-dive + scenarios**, not coding

> **The thread through every answer:** *POS down = restaurant can't take payment.* Reliability = revenue. Build for a chaotic, offline-prone floor. Adoption can't be mandated.

---

## 🎤 Opener (3 beats — don't memorize words)
**Regulated-financial-platform reliability** → **adoption with zero mandate** → **end-to-end ownership.**
> "5+ yrs at BNY Mellon on a regulated, money-handling platform — software has to be exactly right and always up. Moved into platform/DevEx: built our org's first MCP server, and a CLI that cut onboarding 2 days → 15 min, spread to 30+ teams with zero mandate. I love building things people genuinely adopt — same bar as a restaurant floor. Want end-to-end ownership where the user feels wins immediately."

---

## 4 STORY ONE-LINERS (each ≤ 2 min, "I" not "we", end on impact)
- **Conflict →** MCP security model: senior wanted broad LLM access, I wanted least-privilege (regulated bank). Wrote down both + trade-offs, re-anchored on *what passes security review*, shipped schema-constrained allowlist → **became the reference architecture.** *Lesson: re-anchor on the shared constraint, not who's right.*
- **Proud of →** CLI platform: 2-day setup → **15 min**, secure/tested setup as the default, **34+ MRs, organic adoption, zero mandate.** Adoption proves it solved a felt problem.
- **Reliability (backup) →** **20+ Veracode CVEs** fixed, **Spring Boot 2→3 / Java 11→17, zero prod regression** on a live financial platform under SLA.
- **Conflict (backup) →** Angular 17→19 migration sold in business terms (30% build-time cut, security, zero regression), run incrementally so it never blocked features.

## Other behavioral beats
- **Expect from role:** end-to-end ownership · immediate user feedback loop · strong review culture · ask back "success at 6 & 12 months?"
- **Work with co-workers:** over-communicate / no surprises · review = teaching, no ego · drove **2× commit velocity** by enabling the whole team · own what breaks.

---

## 🔧 Alan (9 PM): deep-dive ready (10+ min, defend decisions)
Pick by his steer: **architecture/security → MCP server** · **adoption/impact → CLI platform** · **uptime → CVE/Spring upgrades.**
Structure: **problem (business terms) → constraint → options considered → decision → what I'd change.** Say *"I considered X but rejected it because Y."*

## Scenario framework: **clarify → options → trade-off → recommend → business metric**
- **Ship now vs delay:** touches money/order accuracy? → hold. Else → **flag to ~5%, ramp.**
- **Tech debt vs features:** frame as *wrong prices charged to guests* = tickets/refunds/trust; propose **~20% / along the feature path.**
- **Slow shift-change dashboard:** worst time (payroll) → cheap diagnose (unbounded query?) → **cache + pagination**, jumps queue on impact×reach×effort.
- **Build vs buy reservations:** deepens core loop (→ build), but **integrate first to validate, then build.**
- **Low adoption:** not a code problem — funnel + talk to 3 customers + onboarding; instrument **activation, not "shipped."**

---

## ❓ Ask them (2–3 each)
**Chance (HM):** biggest tech bets this year? · reliability vs velocity tension? · great hire's first 90 days? · north-star metric?
**Alan (Principal):** what metric is the team measured on? · is this in the critical path of taking an order? · hardest architecture problem now (offline sync / scale / multi-region)? · rollback plan when something regresses?

---

## ⚠️ Watch-outs
- Resume says **OpenAI API** for Governance but certs are **Anthropic/Claude** — have a clean reason ready.
- Stay high-level in HM round; go deep only when Alan invites it.
- "I" for your actions. End every story on **business impact**, not cleverness.
- Test Zoom/mic 10 min early; water; quiet room for the full 2-hr block.
