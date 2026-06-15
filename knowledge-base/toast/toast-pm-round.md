# Toast — Product Manager / Cross-Functional Round

**Focus:** Scenario-based questions connecting **tech decisions → business goals**
**Anchored on:** Toast's product (restaurant POS, online ordering, payroll/labor, payments) + the behavioral signals seen in HM/peer rounds
**Study time:** ~45 min

> **What this round tests:** Not coding. Can you (1) reason about *product* trade-offs as an engineer, (2) translate a technical choice into restaurant/business impact, and (3) collaborate with a PM without either rubber-stamping or stonewalling them. Toast's customers are restaurants — every answer should ground in *"what does this do for the restaurant operator and their guests?"*

---

## Know Toast's business before you walk in (90 seconds of grounding)
- **Who pays Toast:** restaurants (SaaS subscription) + **payment processing** (a % of every transaction). So *transaction volume and uptime* directly drive Toast revenue.
- **Why uptime is existential:** if the POS is down, the restaurant **cannot take payment** — they lose money in real time and churn. Reliability isn't a nice-to-have; it's the product.
- **Product surface:** in-store POS (Android terminals), online ordering, KDS (kitchen display), payroll & team management (the Labor service you saw in R2), reporting/analytics, capital/lending.
- **Customer constraints:** restaurants are low-margin, low-tech-staff, high-chaos environments. Hardware gets dropped, Wi-Fi drops, staff turn over weekly. **Software must be dead simple and work offline.**

Use these facts as the backbone of every scenario answer.

---

## Scenario questions & how to answer them

### Format: **STAR-for-scenarios** → Situation, Trade-off, Action, Result/Reasoning
For hypotheticals you don't have a past story for, externalize your *reasoning framework*: clarify → list options → name the trade-off → recommend → tie to business metric.

---

### Q1. "We can ship the new online-ordering feature now with a known limitation, or delay two weeks to do it fully. PM wants it now. What do you do?"
**Framework answer:**
- **Clarify:** What's the limitation's blast radius? Does it touch **payments or order accuracy** (never ship) or a cosmetic/edge UX (shippable behind a flag)?
- **Options:** (a) ship behind a **feature flag** to 5% of restaurants, measure, ramp; (b) ship with the limitation documented + a fast-follow committed; (c) hold.
- **Recommend:** "If it can't cause a wrong charge or a lost order, I'd ship behind a flag to a small cohort now — that gets the PM their learning/speed *and* caps risk. If it touches money or the kitchen ticket, I hold; a wrong order costs the restaurant a guest and costs Toast trust."
- **Business tie:** speed-to-learning vs. reliability-is-the-product. Show you can *both* move fast and protect the transaction.

### Q2. "Engineering wants to spend a sprint on tech debt / refactoring the menu service. Product wants features. Make the case to the PM."
- Frame debt in **business terms, not engineering terms:** "This refactor isn't cosmetic — the current menu-pricing code has a class of bugs (inherited pricing resolved wrong) that surface as **incorrect prices charged to guests**. Each such bug is a support ticket, a refund, and an eroded-trust event for the restaurant."
- **Quantify:** "Right now a menu change takes 3 days and risks a pricing bug; after, it's same-day and safe. That unblocks the next 3 features on the roadmap."
- **Compromise:** propose **20% allocation** or fixing debt *along the path* of the next feature rather than a standalone sprint. PMs say yes to debt that's tied to roadmap velocity.

### Q3. "A restaurant reports the labor/time-tracking dashboard is slow at shift change. How do you prioritize the fix vs. the planned feature work?"
- **Connect to user pain:** shift change is the *worst* time for slowness — managers are reconciling who's on the clock for payroll. Slowness here = payroll errors = staff trust + compliance risk.
- **Diagnose cheaply first:** is it the unbounded "list all active employees" query (the R2 design pitfall)? If a cache + pagination fixes it in a day, do it now — high impact, low cost.
- **Prioritize via impact × reach × effort:** payroll correctness is high-impact, the fix is low-effort → it jumps the queue. Communicate the trade to the PM with that reasoning, not "engineers want to fix it."

### Q4. "How would you decide whether to build a reservations feature in-house vs. integrate a third party (OpenTable)?"
- **Strategic lens:** Does it deepen the **core loop** (more transactions flow through Toast)? Reservations → seated guests → orders → payments processed by Toast → revenue. That argues *build* (own the data + the transaction).
- **Counter:** integration is faster to market and avoids rebuilding a mature product; build only if you can differentiate (e.g. reservations that feed directly into Toast's table/KDS/payroll data).
- **Recommend:** "Integrate first to validate demand, instrument it, then build in-house once it proves it drives transaction volume — buy-then-build to de-risk." Shows product judgment + capital efficiency.

### Q5. "A feature you built isn't getting adopted by restaurants. What do you do?"
- **Don't assume it's a code problem.** Adoption gaps are usually **discoverability, onboarding, or it solves a problem the operator doesn't feel.**
- **Action:** look at the funnel (do they reach it? start it? complete it? return?); talk to 3 customers / support tickets; check if training/rollout was the gap.
- **Tie to business:** "Restaurants are busy and under-staffed — if a feature needs a manual to use, it won't get used. I'd push for in-product guidance and measure activation, not just shipped."

---

## Connecting tech ↔ business (have these mappings memorized)
| Technical decision | Business consequence (say it this way) |
|---|---|
| Offline-first / local SoT on the POS | Restaurant keeps taking payments when Wi-Fi drops → no lost revenue, no churn. |
| Idempotency keys on payments/orders | No double-charges → guest trust, fewer refunds/chargebacks. |
| Optimistic locking on bookings/inventory | No double-booked tables / oversold items → no in-person guest conflict. |
| Pre-aggregated payroll rollups | Fast, correct hours at shift change → managers trust the data, payroll is compliant. |
| Feature flags + gradual ramp | Ship fast *and* contain blast radius → speed without betting the fleet. |
| Pagination on list endpoints | Dashboards stay fast as the restaurant grows → retention at scale. |
| Event-driven fan-out (Kafka) | New consumers (analytics, KDS, notifications) without touching the write path → roadmap velocity. |

---

## Collaboration signals (the round also reads how you work with PMs)
- **Disagree with reasoning, not authority:** "I'd push back, but with data — here's the risk and here's the cheaper path to the same goal."
- **Default to the customer:** when eng and product disagree, re-anchor on "what does the restaurant operator actually need?" It dissolves most debates.
- **Translate, don't condescend:** explain tech trade-offs in operator/business terms, never "it's too complex for you to get."
- **Bring options, not blockers:** never just "no" — "no, *and here are two ways to get most of what you want.*"

---

## Questions to ask the PM/panel (shows product thinking)
- "What's the metric this team is measured on — transaction volume, retention, new-restaurant activation?"
- "Where does this feature sit in the restaurant's daily workflow — is it in the critical path of taking an order?"
- "How do we measure success post-launch, and what's the rollback plan if it regresses?"
