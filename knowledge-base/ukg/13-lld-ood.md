# 13 — Low-Level Design (LLD / OOD)

> Tests whether you can translate a problem into clean, extensible classes. At P5 the signal is: SOLID applied with judgement, the right pattern for the right problem (not pattern-soup), and clean contracts. Have **one worked LLD** ready to narrate end-to-end.

---

## PART A — SOLID (define each in one line + the smell it fixes)

- **S — Single Responsibility:** a class has one reason to change. *Smell it fixes:* a `User` class that also sends emails and writes SQL. Split concerns.
- **O — Open/Closed:** open for extension, closed for modification. Add behavior via new classes/strategies, not by editing a giant `switch`. *Fix:* polymorphism over conditionals.
- **L — Liskov Substitution:** a subtype must be usable anywhere its base is, without breaking expectations. *Smell:* `Square extends Rectangle` breaking `setWidth`. If an override throws "unsupported," LSP is violated.
- **I — Interface Segregation:** many small focused interfaces > one fat one. *Smell:* implementing `Machine` forces a printer to implement `fax()`. Split.
- **D — Dependency Inversion:** depend on abstractions, not concretions. High-level policy shouldn't import a concrete DB class. *Fix:* inject an interface (this is what Spring DI gives you — sheet 04).

🎤 **Say it like this:** *"SOLID for me is really one idea repeated: isolate what changes. Single Responsibility keeps reasons-to-change separate; Open/Closed means I extend with new types instead of editing existing code; Liskov keeps subtypes honest so polymorphism is safe; Interface Segregation stops fat interfaces forcing dead methods; and Dependency Inversion makes me depend on an interface, not a concrete class — which is exactly why dependency injection makes code testable. The practical payoff is that new requirements become new classes, not edits to battle-tested ones."*

---

## PART B — Core design patterns (know ~8 cold: definition → when → tiny example)

**Creational**
- **Factory Method** — a method decides which concrete class to instantiate. Use when creation logic shouldn't litter callers. `PaymentFactory.create("upi")`.
- **Builder** — construct a complex object step-by-step; great for many optional fields / immutability. `Pizza.builder().cheese().size(L).build()`.
- **Singleton** — one instance app-wide (config, connection pool). Beware: global state, hard to test, thread-safety (use enum or holder idiom). Spring beans are singletons by default — prefer that over hand-rolled.

**Structural**
- **Adapter** — wrap an incompatible interface to fit yours. Integrating a 3rd-party SDK behind your port.
- **Decorator** — add behavior by wrapping, not subclassing. `BufferedInputStream(new FileInputStream(...))`; or a caching/ logging wrapper around a service.
- **Facade** — one simple interface over a complex subsystem. Your service layer is a facade over repos + clients.
- **Proxy** — stand-in controlling access (lazy load, security, caching). Spring AOP proxies, Hibernate lazy proxies.

**Behavioral**
- **Strategy** — interchangeable algorithms behind one interface, chosen at runtime. *The Open/Closed workhorse* — replaces `if/switch` on type. `SortStrategy`, `PricingStrategy`, `RetryStrategy`.
- **Observer** — subscribers react to a subject's events (pub/sub in-process). Event listeners; the backbone of reactive/event-driven code.
- **Template Method** — base class fixes the skeleton, subclasses fill steps. Spring's `JdbcTemplate`.
- **Chain of Responsibility** — pass a request along handlers until one handles it. Servlet filters, validation pipelines, your governance rule pipeline.
- **State** — object changes behavior as its internal state changes (order: CREATED→PAID→SHIPPED), each state its own class — cleaner than enum + switch.

🎤 **Pattern judgement (the senior line):** *"I reach for patterns to remove conditionals and isolate change, not to show off. Most of my pattern use is Strategy — swap an algorithm without touching callers — and Factory to centralize creation. The anti-pattern is pattern-soup: forcing a GoF name onto a two-line problem. So my filter is: is something going to vary, and do I want to add a variation without editing existing code? If yes, a pattern earns its place; if not, a plain method is better."*

---

## PART C — Worked LLD: design a parking lot (the canonical one — adapt to any)

**Clarify:** multiple levels? vehicle types (bike/car/truck → spot sizes)? pricing (hourly/flat)? entry/exit + ticketing? payment? real-time availability?

**Core entities & responsibilities (SRP):**
```java
enum VehicleType { BIKE, CAR, TRUCK }
enum SpotType   { SMALL, MEDIUM, LARGE }

abstract class Vehicle { VehicleType type; String plate; }
class Car extends Vehicle { ... }

class ParkingSpot {            // knows only its own state
    String id; SpotType type; boolean free = true; Vehicle current;
    boolean canFit(Vehicle v) { /* size rule */ }
}

class ParkingLevel {           // owns its spots + finds one
    List<ParkingSpot> spots;
    Optional<ParkingSpot> findSpot(Vehicle v) { ... }   // Strategy candidate
}

class Ticket { String id; Vehicle v; ParkingSpot spot; Instant in; }

interface PricingStrategy { Money price(Ticket t, Instant out); }   // Strategy: flat vs hourly
class FeeCalculator { PricingStrategy strategy; ... }

interface PaymentStrategy { boolean pay(Money m); }                  // Strategy: card/upi/cash

class ParkingLot {             // Facade/orchestrator over levels
    List<ParkingLevel> levels;
    Ticket parkVehicle(Vehicle v) { /* find spot across levels, occupy, issue ticket */ }
    Money unpark(Ticket t)        { /* free spot, price via strategy, take payment */ }
}
```

**Why this is "senior":**
- **SRP:** spot tracks state, level finds spots, lot orchestrates, calculator prices, gateway pays — no god-class.
- **Strategy** for pricing, payment, *and* spot-selection (nearest/random/level-fill) → Open/Closed: add UPI or surge pricing = new class, zero edits.
- **Concurrency** (the part juniors miss): two cars racing for the last spot → guard spot assignment (synchronize per level, or atomic compare-and-set on `free`, or a DB row lock / optimistic version). *Call this out unprompted.*
- **Extensibility:** electric spots, reservations, multiple gates — all additive.

🎤 **Walk it:** *"I model responsibilities, not nouns blindly. A spot owns only its own free/occupied state; a level owns its spots and the search for one; the lot is a facade that orchestrates park and unpark. The two decisions I'd defend: pricing, payment, and spot-selection are all Strategy interfaces, so adding surge pricing or UPI is a new class with zero edits to existing code — Open/Closed in practice. And the bit that's easy to forget — concurrency: two cars can race for the last spot, so spot assignment has to be atomic, either a synchronized search-and-occupy per level or a compare-and-set on the free flag, or a row lock if it's DB-backed. Designing the data structure without the race is the difference between a toy and a real system."*

**Follow-up ladder:**
- *Q: Find the nearest free spot efficiently?* Keep a per-level priority structure / free-list per spot type instead of scanning all spots — O(1)/O(log n) instead of O(n).
- *Q: Make spot assignment thread-safe without locking the whole level?* Per-spot atomic flag (CAS) so only the contended spot serializes; or a concurrent free-queue you `poll()` from.
- *Q: Persist this?* Repository interfaces per aggregate (DIP); the lot depends on `SpotRepository`, not a concrete DB.
- *Q: New requirement — reservations?* Add a `Reservation` + a `ReservationSpotStrategy`; existing flow untouched (Open/Closed payoff).

---

## PART C2 — UKG-SPECIFIC LLD (🔥 most likely tomorrow — Pune 2026)

> UKG LLD rounds favor **workforce-domain** problems. The parking lot proves you can model anything; this proves you understand *their* domain. The standout patterns here are **State** (approval/shift lifecycle) and **Strategy/Chain** (pluggable business rules). Bring up **concurrency** (two approvers, double-booking) and **audit trail** unprompted — that's the P5 signal.

### UKG LLD 1 — Leave / Time-off Management System 🔥 #1 likely

**Clarify:** Leave types (casual/sick/earned)? Approval chain (manager → HR)? Balance per type? Carry-forward/accrual? Cancellation? Half-days? Multi-tenant?

**The lifecycle is a state machine → use the State pattern** (cleaner than enum + switch):
```
   submit          approve            (auto on start date)
DRAFT ───▶ PENDING ───────▶ APPROVED ──────────▶ TAKEN
             │   ▲                │
       reject│   │withdraw  cancel│
             ▼   │                ▼
          REJECTED            CANCELLED   (balance refunded)
```

**Core classes (responsibilities, not nouns):**
```java
enum LeaveType { CASUAL, SICK, EARNED }

class LeaveRequest {                 // the aggregate
    String id, tenantId, employeeId;
    LeaveType type;
    LocalDate from, to; boolean halfDay;
    LeaveState state;                // State pattern, not an enum+switch
    List<AuditEntry> history;        // append-only: who/what/when (compliance)
    void submit(); void approve(Approver a); void reject(...); void cancel();
}

interface LeaveState {               // STATE PATTERN — each transition validated per state
    LeaveState approve(LeaveRequest r, Approver a);   // illegal transitions throw
    LeaveState reject(LeaveRequest r, String reason);
    LeaveState cancel(LeaveRequest r);
}
class PendingState implements LeaveState { ... }   // only PENDING allows approve/reject
class ApprovedState implements LeaveState { ... }  // only APPROVED allows cancel→refund

interface LeavePolicy {              // STRATEGY/CHAIN — pluggable business rules
    void validate(LeaveRequest r, LeaveBalance bal);  // throws if violated
}
class SufficientBalancePolicy implements LeavePolicy { ... }
class NoOverlapPolicy        implements LeavePolicy { ... }   // no overlapping requests
class BlackoutDatePolicy     implements LeavePolicy { ... }   // e.g. no leave at month-end
class NoticePeriodPolicy     implements LeavePolicy { ... }   // min N days advance

class LeaveBalance {                 // per (employee, type, year)
    int entitledMinutes, usedMinutes, pendingMinutes;
    int available() { return entitledMinutes - usedMinutes - pendingMinutes; }
    // reserve on submit, commit on approve, release on reject/cancel
}

interface ApprovalWorkflow { List<Approver> approversFor(LeaveRequest r); } // manager→HR chain

class LeaveService {                 // facade/orchestrator
    List<LeavePolicy> policies;      // validation chain
    LeaveRequest submit(LeaveRequest r) {
        policies.forEach(p -> p.validate(r, balanceOf(r)));   // Open/Closed: add a rule = new class
        balance.reserve(r);          // reserve so concurrent requests can't oversubscribe
        return r.submit();
    }
}
```

**Why this is "senior":**
- **State pattern** makes illegal transitions *impossible* — you can't approve a CANCELLED request because that state object has no valid `approve()`. Cleaner and safer than a giant switch.
- **Policy chain (Strategy/Open-Closed):** labor rules vary by tenant/country → each is a class; adding "no leave during blackout week" is additive, zero edits.
- **Concurrency (call out unprompted):** two requests racing against the same balance → **reserve `pendingMinutes` atomically** (DB row lock / optimistic version on the balance row) so you can't approve two leaves that together exceed the balance. *Balance must be strongly consistent — it's an entitlement ledger.*
- **Audit trail:** every transition appends to `history` (immutable) — compliance + dispute resolution.
- **Balance lifecycle:** reserve-on-submit → commit-on-approve → release-on-reject — prevents the "approved over balance" race.

🎤 **Walk it:** *"A leave request is fundamentally a state machine, so I model it with the State pattern — each state is a class that only permits its legal transitions, which makes 'approve a cancelled request' impossible by construction rather than by an if-check I might forget. The business rules — sufficient balance, no overlap, blackout dates, notice period — vary by tenant and country, so each is a Strategy in a validation chain; a new rule is a new class, never an edit. The part juniors miss is the balance race: two requests could each pass the balance check and both get approved over the limit, so I reserve pending minutes atomically with an optimistic version on the balance row, and treat the balance as a strongly-consistent ledger. And because this is payroll-adjacent, every transition appends to an immutable audit history."*

**Follow-up ladder:**
- *Multi-level approval (manager → HR)?* `ApprovalWorkflow` returns the ordered approver chain; request stays PENDING until the last approves (Chain of Responsibility).
- *Accrual (earn 1.5 days/month)?* An `AccrualStrategy` run by a scheduled job credits `entitledMinutes` — pluggable per leave type.
- *Make balance update safe under load?* Optimistic lock (version) on the balance row; on conflict, re-read and retry.

---

### UKG LLD 2 — Shift Scheduler (quick model, if they pivot)
```java
class Shift { String id, tenantId; LocalDateTime start, end; Set<String> requiredSkills;
              String assignedEmployeeId; int version; }   // version → optimistic concurrency
interface SchedulingRule { void check(Shift s, Employee e, Schedule ctx); } // Chain
class MaxHoursRule, MinRestRule, SkillMatchRule, AvailabilityRule implements SchedulingRule;
class Schedule { assign(Shift s, Employee e){ rules.forEach(r->r.check(s,e,this)); /* CAS on version */ } }
```
- **Same backbone:** rules as a **Chain/Strategy**, **optimistic version** on a shift so two managers (or an open-shift grab) can't double-assign — last writer fails and retries. Mention both unprompted.

---

## PART D — API / contract design (LLD for services)
- **Resource-oriented, noun URLs, verbs via HTTP** (`POST /reviews`, not `/createReview`).
- **Versioning** (`/v1`), **pagination** (cursor > offset at scale), **consistent error shape** (`{code, message, traceId}`), **idempotency keys** on unsafe writes.
- **Contract-first:** define the OpenAPI/interface before implementation so consumers and tests can proceed in parallel (ties to contract testing — sheet 09).
- **DTO vs entity:** never leak your persistence entity as the API contract — map to a DTO so DB changes don't break clients.

---

## ⚠️ Pitfalls & seniority signals
- **God class / anemic model** — one class does everything, or classes are just data bags with logic elsewhere. Balance.
- **Pattern-soup** — forcing patterns where a method suffices.
- **Forgetting concurrency & error states** in the model — the single biggest seniority differentiator in LLD.
- **Leaking entities through the API** — couples clients to your schema.
- **Seniority signal:** start from responsibilities, use interfaces at the seams, name where things will vary and isolate them, and proactively raise thread-safety and failure handling before being asked.

---

*Next topic, or drill deeper on this one?*
