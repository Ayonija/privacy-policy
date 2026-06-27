# 11 — Observability

> Short, high-signal. The core distinction (logs/metrics/traces), distributed tracing, SLO/SLI/SLA, and tools (Datadog, Prometheus/Grafana). Ties to your OpenTelemetry experience.

---

## Monitoring vs Observability
- **Monitoring** — watching **known** failure modes (dashboards/alerts for things you predicted).
- **Observability** — being able to ask **new** questions about your system from its outputs, to debug problems you *didn't* predict. *"Monitoring tells you something is wrong; observability tells you why."*

---

## The three pillars (define each + when to reach for it)
- **Logs** — timestamped, discrete event records. Best for **detailed context** on a specific event ("what exactly happened here?"). Use **structured logging** (JSON with fields) so they're queryable. Cost: high volume.
- **Metrics** — numeric measurements aggregated over time (counters, gauges, histograms). Best for **trends, dashboards, alerts** — cheap, efficient ("is error rate rising?"). Low cardinality.
- **Traces** — the **end-to-end journey of one request** across services. Best for **"where is the latency / where did it fail?"** in a distributed call graph.

🎤 *"I think in three pillars. Metrics are cheap aggregates — counters and histograms — great for dashboards and alerts that tell me something's wrong, like error rate climbing. Logs are detailed event records for context on a specific occurrence, and I keep them structured so they're queryable. Traces follow a single request across every service it touches, which is the only practical way to answer 'where did the latency come from' in a microservices call graph. The mantra: metrics tell me what's wrong, traces tell me where, logs tell me why."*

---

## Distributed tracing (the microservices essential)
**Plain English:** Follow one request as it hops across services. Each service adds a **span** (a timed unit of work) under a shared **trace ID**; together they form a tree showing where time went and where it broke.
- **Trace** = whole request; **span** = one operation; **context propagation** = passing the trace ID across service boundaries (HTTP headers).
- **OpenTelemetry (OTel)** — the vendor-neutral standard for generating traces/metrics/logs (you've used it). Instrument once, export anywhere (Datadog, Jaeger, Grafana Tempo).

---

## SLI / SLO / SLA (know the chain cold)
- **SLI (Indicator)** — the actual measured number: e.g. "99.95% of requests succeeded," "p99 latency 180ms." *What you measure.*
- **SLO (Objective)** — your internal target for an SLI: "99.9% success over 30 days." *What you aim for.*
- **SLA (Agreement)** — the contractual promise to customers with consequences (credits/penalties) if missed: "99.5% uptime or you get a refund." *What you promise (looser than the SLO, deliberately).*
- **Error budget** — `1 − SLO`. If SLO is 99.9%, you have 0.1% "budget" to spend on risk/releases. Budget exhausted → freeze risky changes. *This is how SLOs drive engineering decisions.*

🎤 *"They form a chain. The SLI is the measured reality — say the percentage of successful requests. The SLO is the internal target for that indicator, like 99.9% over 30 days. The SLA is the external contractual version with penalties, and I deliberately set it looser than my SLO so I have margin before I'm breaching a customer promise. The practical lever is the error budget: one minus the SLO. As long as I have budget I can ship aggressively; if I burn through it, that's a signal to slow down and stabilize. It turns reliability into a quantified, shared decision instead of an argument."*

---

## Tools + alerting
- **Prometheus** — pull-based metrics DB + query language (PromQL); scrapes `/metrics` endpoints. **Grafana** — dashboards/visualization on top (and alerting). Common OSS pairing.
- **Datadog** — managed, all-in-one (metrics + logs + APM/traces + LLM observability). Less ops, paid.
- **The RED method** (services): **R**ate, **E**rrors, **D**uration — the three metrics to watch per endpoint. (**USE method** for resources: Utilization, Saturation, Errors.)
- **Alerting:** alert on **symptoms users feel** (error rate, latency SLO burn), not every cause; avoid alert fatigue (noisy alerts get ignored). Page on actionable, user-impacting issues; everything else is a dashboard.

🎤 *"For services I instrument the RED metrics — rate, errors, and duration — per endpoint, because those map directly to user pain. I alert on symptoms, not causes: latency breaching the SLO or error rate spiking, rather than every CPU blip, because noisy alerts train people to ignore the pager. Stack-wise, Prometheus and Grafana are my default open-source pairing — Prometheus scrapes metrics, Grafana visualizes and alerts — and Datadog when I want managed APM and tracing in one place. I standardize instrumentation on OpenTelemetry so I'm not locked to one backend."*

---

## ⚠️ Pitfalls & seniority signals
- Logging everything unstructured → unsearchable + expensive. Structure + sample.
- Alerting on causes not symptoms → alert fatigue.
- Confusing SLO (internal target) with SLA (external contract).
- No tracing in a microservices system → can't localize latency.
- **Seniority signal:** RED/USE methods, error-budget-driven release decisions, OTel for vendor-neutral instrumentation, "alert on symptoms," and tying AI observability (tokens/latency/drift — sheet 14.7) into the same framework.

---

*Next topic, or drill deeper on this one?*
