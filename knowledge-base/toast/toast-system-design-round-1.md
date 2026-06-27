# Toast — System Design Round 1: Real-Time Restaurant Operations

**Focus:** Scalability + real-time data handling
**Anchored on actual Toast questions:** Labor / time-tracking service (R2), Restaurant reservation/table-booking system (R4)
**Study time:** ~60 min

> ### 📅 YOUR INTERVIEW (confirmed)
> **Mon Jun 15, 2026 · 9:00–10:00 PM IST · Zoom + CodePair (HackerRank)**
> **Interviewer: Patrick Flynn — Senior Software Engineer**
> This is a **60-minute live-coding round** on HackerRank CodePair. Toast's R2/R3 pattern: the interviewer brings *their own* problem — usually **implement a method on a menu/data model, design a small REST controller, or review/extend sample code.** Expect Java. The bar (per recent feedback): **fully working code that compiles and runs**, clean structure, and you **talking through your reasoning the whole time.** See the "Round 2 — Recent Questions" and "CodePair Playbook" sections at the bottom — start there if short on time.

> **Why these two?** Both showed up verbatim in recent loops. Toast is a restaurant POS company, so every design prompt is a thin wrapper over the same hard parts: **concurrent writes from many devices, real-time aggregation, and correctness of money/hours.** Master these two and you can derive the rest (KDS, online ordering, payments).

---

## Part A — Labor / Time-Tracking Service (R2, asked as a REST API task)

> *"We are building a labor service to track when restaurant employees are on the job. Data is used in payroll, scheduling, and other user-facing products. Write the REST endpoints to: clock-in an employee, clock-out an employee, list currently clocked-in employees, return hours worked this week (last 7 days)."*

This is asked as a **coding/API-design** round but graded like a mini system design. Lead with the API, then volunteer the scaling story.

### S — Situation / Clarify first (say these out loud)
- One employee can work at **multiple restaurants**? Assume scoped to one restaurant per shift; key on `(restaurantId, employeeId)`.
- Can an employee clock in twice without clocking out? **No** — must reject double clock-in (idempotency + conflict).
- "This week" = rolling **last 7 days (168h)**, not calendar week. (The prompt says last 7 days — confirm.)
- Time source: **server time**, UTC stored, restaurant timezone for display. Never trust client clocks for payroll.
- Auto clock-out for forgotten punches? Mention it as a real-world concern (a shift open for 18h is a data-quality bug).

### T — Task / Core challenge
The hard part is **not** the CRUD. It's: (1) preventing concurrent double clock-ins, (2) computing hours efficiently without scanning all history, (3) correctness for payroll (money depends on it).

### A — Data model
A shift is an **interval**, modeled as an append-only event or a row with nullable `clockOut`:

```
TimeEntry
  id           UUID (PK)
  restaurantId UUID         -- partition / shard key
  employeeId   UUID
  clockInAt    TIMESTAMP    -- UTC, NOT NULL
  clockOutAt   TIMESTAMP    -- NULL == currently on the clock
  source       ENUM(POS, MOBILE, MANAGER_EDIT)
  createdAt / updatedAt

-- The invariant that makes everything work:
UNIQUE INDEX uniq_open_shift (restaurantId, employeeId) WHERE clockOutAt IS NULL
```

That **partial unique index** is the whole trick: the database itself guarantees an employee can have at most one open shift. No app-level lock needed; a concurrent second clock-in fails with a constraint violation → you return `409 Conflict`.

### A — Endpoints

```
POST   /v1/restaurants/{rid}/employees/{eid}/clock-in
       body: { idempotencyKey }            -> 201 { entryId, clockInAt }
                                           -> 409 if already clocked in

POST   /v1/restaurants/{rid}/employees/{eid}/clock-out
                                           -> 200 { entryId, clockInAt, clockOutAt, durationMinutes }
                                           -> 409 if not currently clocked in

GET    /v1/restaurants/{rid}/shifts/active
       ?cursor=&limit=50                   -> 200 { activeShifts: [...], nextCursor }

GET    /v1/restaurants/{rid}/employees/{eid}/hours?window=7d
                                           -> 200 { hours: 30.5 }
```

**Design notes the interviewer is listening for:**
- **POST not GET** for clock-in/out — they mutate state. (The R3 code review round penalizes GET-with-side-effects; don't repeat that mistake here.)
- **Idempotency key** on clock-in: a flaky tablet retries; you must not create two shifts. Store the key, return the original result on replay.
- **Pagination/cursor** on the active list — a stadium concession has hundreds of staff. Never return an unbounded list.
- Clock-out computes `durationMinutes = clockOutAt - clockInAt`. Reject if `clockOut < clockIn` (clock skew / manual edit).

### A — Computing "hours this week" at scale

Naive: `SELECT entries WHERE employeeId AND clockInAt >= now-7d`, sum durations. Fine at small scale. Make it scale:

```sql
SELECT COALESCE(SUM(
   EXTRACT(EPOCH FROM (COALESCE(clockOutAt, now()) - clockInAt))
), 0) / 3600.0 AS hours
FROM time_entry
WHERE restaurantId = ? AND employeeId = ?
  AND clockInAt >= now() - interval '7 days';
```
- Index on `(restaurantId, employeeId, clockInAt)` → bounded range scan, not a table scan.
- Include the **currently-open shift** by treating a null `clockOutAt` as "up to now" — payroll dashboards want live hours.
- **Edge case to volunteer:** a shift that *spans* the 7-day boundary (clocked in 8 days ago, out 6 days ago). Decide: count only the portion inside the window, or count the whole shift if it ends in-window. Clarify with interviewer — this is the kind of correctness nuance Toast probes.

### A — Scaling to "real-time across all restaurants"

| Concern | Approach |
|---|---|
| **Write volume** (every punch is a write, shift-change spikes) | Shard by `restaurantId`. A single restaurant's writes are low; the fleet is large but cleanly partitionable. |
| **Read amplification** (dashboards polling active staff) | Cache `active shifts` per restaurant in Redis (set of employeeIds); update on clock-in/out. Dashboard reads hit cache, not DB. |
| **Live updates** (manager screen shows who's on the clock) | Push via WebSocket/SSE per restaurant channel on each event, instead of polling. This is the "real-time data handling" Toast wants to hear. |
| **Payroll correctness** | TimeEntry is the **source of truth, append-only/auditable**. Manager edits create a new versioned row, never destructive update — payroll disputes need history. |
| **Aggregation for payroll runs** | Don't recompute from raw on every payroll. Materialize daily `hours_by_employee_day` via a streaming job (events → Kafka → aggregator) or nightly batch; "weekly hours" = sum of 7 daily rollups + today's live partial. |

### R — One-liner summary to close
> "Append-only TimeEntry as the source of truth, a partial unique index to enforce one-open-shift-per-employee, range queries on an indexed `(restaurant, employee, clockInAt)` for hours, Redis + WebSocket for the real-time active list, and pre-aggregated daily rollups so payroll never scans raw events."

---

## Part B — Restaurant Table-Booking / Reservation System (R4 + "Reservations for Restaurants")

> *Asked twice in the dataset: "Design a restaurant table booking system" and "System design is Reservations for Restaurants."* This is **the** Toast system design question. Know it cold.

### Step 1 — Requirements (drive this; don't wait to be asked)

**Functional**
- Diner searches restaurants by location/time/party size; sees available slots.
- Diner books a slot → reservation confirmed; can modify/cancel.
- Restaurant configures tables, capacities, opening hours, slot duration.
- Restaurant sees the day's book; host seats walk-ins (consumes inventory too).

**Non-functional (state numbers — Toast likes concreteness)**
- **No double-booking** — strong consistency on the booking write. This is the heart of the problem.
- High read:write ratio (browsing >> booking), ~100:1. Reads can be eventually consistent.
- Low latency search (<200ms). Availability is the hot read path.
- Availability — diners book peak dinner hours; thundering herd on Friday 7pm slots.

### Step 2 — Scale estimate
- Say 50K restaurants, avg 30 tables, 4 time-slots/table/evening → ~6M bookable slots/day. Modest data; the challenge is **contention**, not volume.
- Booking writes peak-spiky; search reads constant and high. → Separate the read and write paths.

### Step 3 — Core data model

```
Restaurant(id, name, geoHash, timezone, openingHours, slotMinutes)
Table(id, restaurantId, capacity, isCombinable)
ReservationSlot(id, restaurantId, tableId, startTime, endTime,
                status ENUM(AVAILABLE, HELD, BOOKED), version)
Reservation(id, slotId, dinerId, partySize, status, createdAt)
```

Model availability as **discrete slots** (table × time-window). Booking = flip one slot AVAILABLE → BOOKED. Discrete slots make the concurrency problem tractable (you're contending on one row, not a range).

### Step 4 — The concurrency core (the part they grade)

Two diners race for the last 7pm table. Solutions, weakest → strongest:

1. **Optimistic locking (version column)** — read slot with `version=v`; `UPDATE ... SET status=BOOKED WHERE id=? AND version=v`. If 0 rows updated, someone won the race → retry/return "just taken." Best default: cheap, no held locks, correct.
2. **Pessimistic lock** — `SELECT ... FOR UPDATE` on the slot row inside the booking txn. Simpler to reason about, but holds DB locks; fine at this scale.
3. **Two-phase hold** — UX reality: diner needs ~2 min to confirm party details. Move slot to **HELD** with a TTL (Redis key `hold:slotId` expires in 120s); confirm flips HELD→BOOKED, expiry flips back to AVAILABLE. This is how OpenTable/Resy actually work — mention it; it shows product sense.

> **Say this:** "The booking write needs strong consistency, so it lives in a transactional store (Postgres) with optimistic locking on the slot row. Everything else — search, availability browse — can be served from a read replica or cache and be eventually consistent."

### Step 5 — Architecture / scaling

```
              ┌─────────────┐
   Diner ───▶ │  API Gateway│
              └──────┬──────┘
         ┌───────────┴───────────┐
   (reads)│                       │(writes)
   ┌──────▼──────┐         ┌──────▼───────┐
   │ Search svc  │         │ Booking svc  │
   │ (Elastic/   │         │ (txn, opt.   │
   │  geo index) │         │  lock)       │
   └──────┬──────┘         └──────┬───────┘
   ┌──────▼──────┐         ┌──────▼───────┐
   │Availability │◀────────│  Postgres    │ (source of truth)
   │ cache(Redis)│  CDC/   │  sharded by  │
   └─────────────┘  events │  restaurantId│
                           └──────┬───────┘
                                  │ emits ReservationCreated
                           ┌──────▼───────┐
                           │ Kafka → notif│ (email/SMS, KDS, analytics)
                           └──────────────┘
```

- **Shard by `restaurantId`** — a reservation never spans restaurants, so this partitions perfectly with no cross-shard transactions.
- **Search** uses a geo index (geohash / Elasticsearch) over restaurants + a denormalized availability summary. Search is eventually consistent — a slot shown "available" may be gone by booking time; the booking write is the authority that rejects with 409.
- **Availability cache** in Redis keyed by `restaurantId:date`, invalidated on each booking via change-data-capture / event. Absorbs the browse traffic.
- **Real-time host view** — restaurant's floor screen subscribes (WebSocket) to its restaurant's reservation events; updates live as bookings/cancellations/walk-ins land.
- **Async fan-out** — confirmation email/SMS, kitchen display, analytics all consume the `ReservationCreated` event off Kafka. Keep the booking write path thin and fast.

### Step 6 — Edge cases to raise unprompted
- **No-shows / cancellations** → free the slot; optionally overbook by a small % like airlines (product call).
- **Table combining** — party of 6 needs two 4-tops joined; booking must atomically reserve *both* slots (multi-row txn). Good "going deeper" topic.
- **Walk-ins** consume the same inventory — host seating must write a reservation too, or you double-seat.
- **Clock/timezone** — store UTC, render in restaurant timezone; DST boundaries.
- **Idempotency** — diner double-taps "Book"; idempotency key prevents two reservations.

### R — Closing summary
> "Discrete slot inventory in a transactional store sharded by restaurant, optimistic locking (or a Redis TTL hold) to kill double-booking, a separate eventually-consistent search/availability read path backed by a geo index and Redis cache, and Kafka-driven async fan-out for notifications and the live host view."

---

## Rapid-fire follow-ups (both designs)
- *"How do you prevent double-booking / double clock-in?"* → unique constraint or optimistic version check at the DB; the DB is the arbiter, not app code.
- *"How do reads scale?"* → separate read path, replicas + Redis, eventual consistency is acceptable for browse; only the write is strongly consistent.
- *"Real-time updates?"* → event on every state change → WebSocket/SSE per restaurant channel; don't poll.
- *"How do you shard?"* → by `restaurantId`; operations are naturally restaurant-scoped, so no cross-shard transactions.
- *"Source of truth for money/hours?"* → append-only, audited, versioned; never destructive updates on payroll/payment data.

---

# 🔥 Round 2 — Recent CodePair Questions (most likely for your 60-min round)

> Patrick will bring **one** problem and go deep with add-ons. Across recent Toast loops the R2 coding round almost always lands on one of the patterns below. All are **Java**, all are tree/collection-over-a-menu-or-restaurant-model. Practice writing each so it **compiles and runs** — that's the explicit bar.

### Pattern 1 — `getPrice`: menu tree with inherited pricing ⭐ (most frequent)
Nested `Menu → MenuGroup → (MenuGroup | MenuItem)`, each with a **nullable** price. An item's price = its own, else the **nearest non-null ancestor's** price.

```java
public static Double getPrice(Collection<Menu> menus, MenuItem target) {
    for (Menu menu : menus) {
        Double p = search(menu.groups, menu.price, target); // menu price seeds inheritance
        if (p != null) return p;
    }
    return null; // not found
}
private static Double search(List<MenuGroup> groups, Double inherited, MenuItem target) {
    if (groups == null) return null;
    for (MenuGroup g : groups) {
        Double current = (g.price != null) ? g.price : inherited;     // group overrides if set
        if (g.items != null)
            for (MenuItem it : g.items)
                if (it.name.equals(target.name))                       // .equals, NOT ==
                    return (it.price != null) ? it.price : current;
        Double sub = search(g.groups, current, target);               // recurse, carry inherited
        if (sub != null) return sub;                                   // propagate the find
    }
    return null;                                                       // keep searching siblings
}
```
**3 bugs candidates make (don't):** returning `0.0` instead of `null` for not-found; `return`ing on the first group instead of searching siblings; `==` on names. **O(N) time, O(H) space.** Full trace + this exact problem live in [toast-system-design-round-2.md](toast-system-design-round-2.md) Part D — and since you already did this in R1, see Part A there for the **extensions** (find-all, path, iterative, service) they'll likely ask next.

### Pattern 2 — "Fetch items from the menu" (the *vegan salads* question)
They give a menu data structure, then *"fetch a specific item / all items in a category (e.g. vegan salads)."* Same DFS as `getPrice` but **collect-all instead of find-first**:
```java
public static List<MenuItem> findByCategory(List<MenuGroup> groups, String category, List<MenuItem> out) {
    if (groups == null) return out;
    for (MenuGroup g : groups) {
        boolean match = category.equalsIgnoreCase(g.name);
        if (match && g.items != null) out.addAll(g.items);   // whole category
        findByCategory(g.groups, match ? g.name : category, out); // recurse
    }
    return out;
}
```
**Likely add-ons:** filter by a tag/dietary flag (`item.tags.contains("vegan")`); return the **path** to each item (pass a `List<String>` down); handle duplicate names; make it **iterative** with an explicit stack.

### Pattern 3 — Implement / extend a REST controller
*"Here's a Rest Controller — add an endpoint / implement this method."* This is the **Labor service** (Part A above) territory, or a menu CRUD controller. What they grade:
- Correct HTTP verbs (**POST** for mutations, **GET** safe/idempotent), correct status codes (`201`/`200`/`404`/`409`), **input validation**, pagination on list endpoints.
- Clean separation: controller → service → DAO. Don't put logic in the controller.
- Idempotency on writes; meaningful error responses (not `RuntimeException`).

### Pattern 4 — Code review of sample code
*"Review this code."* Lead **security → correctness → resources → API/design → tests**, prioritized. (SSRF, path traversal, `byte[]` equals/hashCode, unclosed streams, GET-with-side-effects, no pagination.) Full worked review in [toast-system-design-round-2.md](toast-system-design-round-2.md) Part C.

### Pattern 5 — Concurrency LLD (multi-threaded alarm clock)
*"Multi-threaded alarm clock: `setAlarm`, `ringAlarm`."* `ScheduledExecutorService` + `ConcurrentHashMap` registry; be ready for the from-scratch `PriorityQueue` + `Condition.awaitNanos` version, and to explain concurrency vs parallelism / deadlock / `volatile`. Full code in [toast-system-design-round-2.md](toast-system-design-round-2.md) Part B.

### Pattern 6 — Stock profit with 5-second gap (DSA warm-up)
Some loops open with a pure DSA question. The known Toast one: max single buy/sell profit where `sell_index − buy_index ≥ 5`. One-pass with a lagged running-min. Covered in [toast-sde-prep.md](toast-sde-prep.md).

> **What to drill tonight (in order):** `getPrice` until automatic → category-fetch variant → REST controller (clock-in/out from Part A) → skim the code-review checklist. These four cover ~80% of the R2 surface.

---

# 🎯 CodePair Playbook — how to present & get shortlisted

**Your setup:** HackerRank CodePair, shared editor, Patrick watching you type. This round is **70% communication, 30% the answer.** A correct silent solution loses to a slightly-imperfect well-narrated one.

### The 60-minute structure (drive it)
1. **0–5 min — Clarify before coding.** Restate the problem, ask about input shape, nulls, duplicates, empty collections, size/scale, expected output type. Confirm the data model. *Toast loves the inherited-null-price nuance — surface it yourself.*
2. **5–10 min — Plan out loud.** "I'll DFS the tree carrying the inherited price; base case…; recursive case…; complexity O(N)/O(H)." Get a nod before typing.
3. **10–40 min — Code while narrating.** Write **runnable Java**: real signatures, real types, no `// pseudocode`. Build small, run early. Use the CodePair "Run" button — make the sample case pass.
4. **40–50 min — Test & edge cases out loud.** Walk a concrete example (e.g. trace "Wings" → 5.00). Name edge cases: empty menu, item not found, all-null prices, duplicate names. Fix anything that breaks.
5. **50–60 min — Add-ons + your questions.** They'll extend the problem; handle one. Leave 3–4 min for *your* questions to Patrick.

### Communication rules that read as "senior"
- **Think out loud constantly.** Silence reads as stuck. Narrate trade-offs: "I could use `==` but that's reference equality on Strings — `.equals` is correct here."
- **Brute force → optimal.** State the naive approach and its complexity, then improve. Shows range.
- **State complexity unprompted** before you finish: time *and* space.
- **Edge cases before they ask.** Nulls, empties, not-found, duplicates — call them out, then handle them.
- **Take hints gracefully.** A hint is help, not a strike. "Good point — let me adjust."
- **Clean code = the signal.** Meaningful names, small helper methods, early returns, no dead code (the always-`true`-return bug is a known Toast anti-pattern — don't write it).

### Toast-specific framing (earns bonus points)
- Tie correctness to the business: "Getting inherited pricing wrong means a guest is **charged the wrong amount** — so I'll be careful with the null-inheritance and test it." That restaurant/money mindset is exactly Toast's culture.
- Reliability-first instinct: validate inputs, handle the failure path, don't swallow exceptions.

### Logistics (do these before 9 PM IST)
- [ ] Test the **Zoom link** + mic/camera and the **CodePair link** (`hr.gs/2499583`) ~15 min early; have `ayonija.mathur@gmail.com` open in case they email on a connection drop.
- [ ] Quiet room, charged laptop + charger, stable internet (wired/hotspot backup), water.
- [ ] Have a **blank Java scratch file** ready locally in case CodePair lags — know Java collections cold (`List`, `Map`, `Optional`, recursion) without autocomplete.
- [ ] Warm up 20 min before: re-type `getPrice` from memory once.
- [ ] **Smart questions for Patrick** (asking shows seniority): "What does the team own day-to-day?" · "How does Toast balance feature velocity against POS reliability?" · "What's the hardest scaling problem you're working on right now?"

### Red flags to avoid (recent ghosting feedback)
- Pseudocode that never runs → **always make it compile/run.**
- Going silent for long stretches.
- Jumping to code before clarifying.
- Ignoring the null/empty/not-found cases.
- Not testing your own solution before saying "done."
