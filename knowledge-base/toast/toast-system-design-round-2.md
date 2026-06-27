# Toast — System Design / Coding Round 2: LLD, Code Review & Menu-Tree Extensions

**Focus:** Object/concurrency design, code-review craft, and *extensions* of the menu-tree problem you already saw in Round 1
**Anchored on actual Toast questions:** `getPrice` menu-tree (asked **R1** — expect a *variation* now), menu-item fetch, code-review round (MenuDataResource), multi-threaded alarm clock, threads/concurrency deep-dive
**Study time:** ~75 min

> ### 📅 YOUR INTERVIEW (confirmed)
> **Thu Jun 18, 2026 · 2:00–3:15 PM IST · Zoom + CodePair (HackerRank)**
> **Interviewer: Kalaiselvi Ravi — Senior Software Engineer, Tech Lead**
> - **Zoom:** https://toasttab.zoom.us/j/99474673848?pwd=dao1Yd1ntbsPaSIpTxBeImgjz2btLj.1
> - **CodePair:** https://hr.gs/6360a90
> - **Backup-email channel (if connection drops):** ayonija.mathur@gmail.com
> - **Scheduling:** Gouri (gouri.gindi@toasttab.com) · Other Qs: Priyalakshmi (priyalakshmi.r@toasttab.com)
>
> This is a **75-minute live round** with a **Tech Lead**. Tech leads lean toward **LLD + code quality + "make it extensible"** rather than pure DSA. Expect **Java**. The bar (per recent loops): **code that compiles and runs, clean structure, and continuous narration of your reasoning.**

> ### ⚠️ READ THIS FIRST — you already did `getPrice` in Round 1
> They will **not** re-ask the vanilla `getPrice`. Round-2 with a tech lead almost always means **(a)** an *extension* of the same menu model (find-all, path-to-item, iterative, modifiers/combos, caching), **(b)** a fresh **LLD/concurrency** problem (alarm clock), or **(c)** a **code review**. So the highest-leverage prep is: be able to *evolve* the menu-tree on demand, and have the LLD + review patterns ready. **Sections are ordered by likelihood for your round.** Start at Part A.

---

## Part A — Menu-Tree EXTENSIONS (most likely — they've seen your base `getPrice`) ⭐

The data model from Round 1 (reuse it; don't redefine from scratch):

```java
class Menu      { Double price; List<MenuGroup> groups; }
class MenuGroup { String name; Double price; List<MenuGroup> groups; List<MenuItem> items; }
class MenuItem  { String name; Double price; Set<String> tags; }   // tags e.g. "vegan","gluten-free"
```

> **Opening line to the interviewer:** *"I solved the base inherited-price lookup last round — happy to re-derive it quickly, but I'm guessing you want a variation. Want me to extend it to find-all / return the path / make it iterative, or take it toward a real menu service?"* — proactively naming the variations signals you've thought past the base problem.

### A1 — Find ALL items in a category (the "vegan salads" question)

> *"Fetch every item under a given category (e.g. all items in 'Salads', or all items tagged vegan)."*

**S — Situation:** Same nested menu, but now **collect-all** instead of **find-first**. Classic place candidates break: returning early on the first match instead of accumulating.

**T — Task:** DFS the whole tree, accumulate every matching item into an output list. Don't stop early.

**A — Approach (Java, runnable):**
```java
// Collect every item whose enclosing group name matches `category`.
public static List<MenuItem> findByCategory(List<MenuGroup> groups, String category, List<MenuItem> out) {
    if (groups == null) return out;
    for (MenuGroup g : groups) {
        boolean match = category.equalsIgnoreCase(g.name);
        if (match && g.items != null) out.addAll(g.items);          // whole category
        findByCategory(g.groups, category, out);                    // ALWAYS recurse — no early return
    }
    return out;
}

// Variant: collect by dietary tag, anywhere in the tree.
public static List<MenuItem> findByTag(List<MenuGroup> groups, String tag, List<MenuItem> out) {
    if (groups == null) return out;
    for (MenuGroup g : groups) {
        if (g.items != null)
            for (MenuItem it : g.items)
                if (it.tags != null && it.tags.contains(tag)) out.add(it);
        findByTag(g.groups, tag, out);
    }
    return out;
}
```

**R — Result/close:** "Same DFS as `getPrice`, but I accumulate into `out` and never short-circuit. **O(N)** time, **O(H)** recursion space (plus the result list)."

**Add-ons to expect:** combine category + tag filter; case-insensitive; dedupe by name; sort by price.

### A2 — Return the PATH to an item (breadcrumb)

> *"Return the full path to an item, e.g. `["Dinner", "Entrees", "Toast", "Avocado"]`."*

**S/T:** Need the trail of group names from root to the matched item — push on the way down, **pop on the way back up** (backtracking).

**A — Approach:**
```java
public static List<String> findPath(List<MenuGroup> groups, String target, List<String> path) {
    if (groups == null) return null;
    for (MenuGroup g : groups) {
        path.add(g.name);                                      // enter group
        if (g.items != null)
            for (MenuItem it : g.items)
                if (it.name.equals(target)) {
                    path.add(it.name);
                    return new ArrayList<>(path);              // copy — caller keeps an immutable result
                }
        List<String> sub = findPath(g.groups, target, path);
        if (sub != null) return sub;                           // propagate the found path up
        path.remove(path.size() - 1);                          // BACKTRACK — leave group
    }
    return null;
}
```

**R:** "The one thing people miss is the **backtrack** (`path.remove`) when a branch fails — without it the path leaks sibling names. I also return a **defensive copy** so later mutation of `path` doesn't corrupt the answer."

### A3 — Make `getPrice` iterative (explicit stack)

> *"What if the menu is very deep — avoid recursion / stack overflow."*

**S/T:** Replace the call stack with an explicit stack of `(group, inheritedPrice)` frames.

**A — Approach:**
```java
public static Double getPriceIterative(Collection<Menu> menus, MenuItem target) {
    Deque<Map.Entry<MenuGroup, Double>> stack = new ArrayDeque<>();
    for (Menu menu : menus)
        if (menu.groups != null)
            for (MenuGroup g : menu.groups)
                stack.push(Map.entry(g, menu.price));          // seed with menu price

    while (!stack.isEmpty()) {
        Map.Entry<MenuGroup, Double> frame = stack.pop();
        MenuGroup g = frame.getKey();
        Double inherited = (g.price != null) ? g.price : frame.getValue();

        if (g.items != null)
            for (MenuItem it : g.items)
                if (it.name.equals(target.name))
                    return (it.price != null) ? it.price : inherited;

        if (g.groups != null)
            for (MenuGroup child : g.groups)
                stack.push(Map.entry(child, inherited));        // carry inherited down
    }
    return null;
}
```

**R:** "Same O(N)/O(H), but heap-allocated frames instead of the call stack — safe for pathological depth. Note `Map.entry` is immutable and null-hostile, so if `inherited` can be null I'd use a tiny `Frame` class instead."

### A4 — "Make it a service / it's called constantly" (tech-lead favorite)

> *"This price lookup is hit thousands of times per second on the POS. How do you make it production-grade?"*

**S — Situation:** The naive recursion is fine for one call, but a POS resolves prices on every screen tap. Tech leads want to hear you think past the algorithm.

**T — Task:** Turn a tree-walk into a fast, correct, maintainable component.

**A — Talking points (say these, light code):**
- **Precompute / flatten:** on menu publish, do **one** DFS that resolves every item's effective price into a `Map<itemId, Double>`. Lookups become O(1). Trade memory for speed; rebuild on menu change (menus change rarely, are read constantly).
- **Caching + invalidation:** cache resolved prices keyed by `(menuVersion, itemId)`; bump `menuVersion` on edit so stale prices can never be served. **Correctness tie-in:** a stale price means a guest is **charged wrong** — invalidation must be airtight.
- **Immutability / thread-safety:** publish an immutable resolved snapshot; readers never see a half-updated menu. Swap the reference atomically (`volatile Map` / `AtomicReference`).
- **Extensibility:** new node types (combos, modifiers) → favor **polymorphism** (a `PricedNode.resolvePrice(inherited)` method per type) over `instanceof`/switch. This is the "design for change" signal a tech lead grades.
- **Edge correctness:** all-null prices → surface as an error, not `0.0` (never silently sell free food); duplicate item names → resolve by id/path, not name.

**R — Close:** "I'd keep the recursive resolver as the source of truth, but **materialize an immutable price map at publish time**, cache by menu version, and resolve new node types via polymorphism so the model extends without touching the traversal."

---

## Part B — Multi-Threaded Alarm Clock (LLD concurrency round — high likelihood for a Tech Lead)

> *"Implement a multi-threaded alarm clock with `setAlarm` and `ringAlarm`."* Asked as an LLD round; they probe **threads, processes, concurrency vs parallelism** around it.

**S — Situation:** Multiple alarms, each fires at a future time, possibly overlapping. Must be thread-safe and cancellable.

**T — Task:** `setAlarm(when, callback)`, fire the callback at `when`, support cancel, don't leak threads.

**A — Idiomatic solution (`ScheduledExecutorService`):**
```java
public class AlarmClock {
    private final ScheduledExecutorService scheduler;
    private final Map<String, ScheduledFuture<?>> alarms = new ConcurrentHashMap<>();

    public AlarmClock(int threads) {
        this.scheduler = Executors.newScheduledThreadPool(threads);
    }

    public String setAlarm(Instant when, Runnable onRing) {
        long delayMs = Duration.between(Instant.now(), when).toMillis();
        if (delayMs < 0) throw new IllegalArgumentException("alarm in the past");
        String id = UUID.randomUUID().toString();
        ScheduledFuture<?> f = scheduler.schedule(() -> {
            try { onRing.run(); }          // ringAlarm callback
            finally { alarms.remove(id); } // self-cleanup
        }, delayMs, TimeUnit.MILLISECONDS);
        alarms.put(id, f);
        return id;
    }

    public boolean cancelAlarm(String id) {
        ScheduledFuture<?> f = alarms.remove(id);
        return f != null && f.cancel(false);
    }

    public void shutdown() { scheduler.shutdown(); }
}
```

**Talking points the panel wants:**
- **Why `ScheduledExecutorService` over `wait/notify` + a sleeping thread?** Thread reuse (no thread-per-alarm), built-in timing, interruptible/cancellable. If they want the **from-scratch** version: a single timer thread holding a **`PriorityQueue` ordered by fire-time** + `ReentrantLock`/`Condition`, doing `condition.awaitNanos(timeToNextAlarm)` and waking early when an earlier alarm is inserted (classic Timer design).
- **Thread-safety:** `ConcurrentHashMap` registry; the callback runs on a pool thread, so `onRing` must itself be thread-safe.
- **Concurrency vs parallelism:** *concurrency* = many alarms interleaved by few threads; *parallelism* = multiple alarms firing **at once** on a multi-core pool. Single-thread pool = concurrent, not parallel.

**R — Close:** "`ScheduledExecutorService` for the timing wheel, a `ConcurrentHashMap` registry for cancel/cleanup, self-removing futures so it doesn't leak — and I can drop to a `PriorityQueue` + `Condition.awaitNanos` if you want the primitives."

### Core-CS deep-dive (the "intense" threads/processes round)
| Concept | One-liner that signals depth |
|---|---|
| Process vs Thread | Process = own address space; thread = shares heap, has own stack/PC. Context-switch cheaper between threads. |
| Concurrency vs Parallelism | Concurrency = *dealing with* many things (interleaving, structure); parallelism = *doing* many at once (hardware). Concurrent on one core is possible. |
| Race condition | Two threads touch shared mutable state without sync → outcome depends on timing. Fix: locks, atomics, immutability, confinement. |
| Deadlock | Circular wait on locks. Avoid via lock ordering, `tryLock` with timeout, lock-free structures. |
| `synchronized` vs `Lock` | `Lock` adds `tryLock`, timed/interruptible acquisition, fairness, multiple conditions. |
| Atomics / CAS | `AtomicInteger` uses compare-and-swap → lock-free, non-blocking. |
| `volatile` | Visibility (no caching), **not** atomicity. `count++` is still unsafe on a volatile. |
| Thread pool sizing | CPU-bound ≈ #cores; I/O-bound ≈ higher (cores × (1 + wait/compute)). |

---

## Part C — Code Review Round (interviewer hands you real code)

The interviewer hands you `MenuDataResource` (a Dropwizard JAX-RS service that ingests menus from files/URLs) and asks you to review it. **They test whether you can spot security, correctness, and design defects.** The signal is **prioritization** — security/correctness first, not nitpicking braces.

**Framing line (say this first):**
> "I'll group findings by severity: **Security → Correctness/bugs → Resource & error handling → API/design → Testing & style.** I'll name the bug and the fix for each."

### 🔴 Security (lead here)
1. **SSRF in `/loadUrl`** — takes an arbitrary URL and `openConnection().getInputStream()`. Attacker hits `http://169.254.169.254/latest/meta-data/` (cloud metadata), `http://localhost:8080/admin`, or `file://`. **Fix:** allowlist http/https only, resolve host and **block private/link-local/loopback ranges**, connect/read timeouts, cap response size, no redirects to internal hosts.
2. **Path traversal in `/loadFile`** — `Files.readAllBytes(Paths.get(location))` on a user path reads `/etc/passwd`/secrets. **Fix:** resolve against a fixed base dir, verify canonical path stays under it, reject `..`.
3. **Stored XSS in `list()`** — raw HTML built from `menuData.getLocator()` unescaped into `<a href=...>`. **Fix:** HTML-escape dynamic values, quote attributes; better, return JSON and let the client render.
4. **No authn/authz** — "we allow everyone" means unbounded ingestion/abuse/DoS. At minimum rate-limit and authenticate writes.

### 🟠 Correctness bugs
5. **`loadByes` always returns `true`** — success/error branch is dead code; says `"success!"` on DB failure. `Reader.retrieve` returns `null` on `IOException`, then persists `bytes = null` into a `nullable = false` column → throws at flush after lying to the caller. **Fix:** return real status; null-check bytes before persist; map failures to HTTP codes.
6. **`equals`/`hashCode` use `Objects.equals`/`Objects.hash` on a `byte[]`** — compares array **identity**, not contents. **Fix:** `Arrays.equals` / `Arrays.hashCode`; better, equality on `id` only for JPA entities.
7. **`parseItems` uses `obj.name == item.name`** (reference equality on Strings) — should be `.equals`. (Same bug class as Round-1 `getPrice`.)
8. **`get()` throws `new RuntimeException("Error 404 Not Found")`** → returns HTTP 500. **Fix:** `throw new WebApplicationException(Response.Status.NOT_FOUND)`.

### 🟠 Resource & error handling
9. **`InputStream`/`URLConnection` not closed** → leak. **Fix:** try-with-resources.
10. **Unbounded read into memory** — `ByteArrayOutputStream` with no cap; huge URL OOMs the JVM. **Fix:** enforce max-bytes, abort past it.
11. **Swallowing `IOException` → `return null`** loses the failure. **Fix:** translate to a domain exception / proper HTTP error.

### 🟡 API & design
12. **`GET` used for state-changing `loadFile`/`loadUrl`** — GET must be safe/idempotent; caches/crawlers will trigger ingestion. **Fix:** `POST`.
13. **`list()` returns all rows, no pagination** — table scan + huge response. **Fix:** cursor/limit pagination.
14. **`Reader` is a `static` nested class doing I/O** — hard to mock. **Fix:** inject a `MenuFetcher` interface (`FileFetcher`/`UrlFetcher`) — polymorphism over the enum switch.
15. **`findAll` does `SELECT d FROM MenuData d`** pulling full blobs for a list view. **Fix:** project `(id, locator)` only.

### 🟡 Testing
16. **`ReaderTest` hits `http://www.example.com`** — flaky/slow/offline-fails. **Fix:** mock the fetcher / WireMock; add failure-path tests.

**Close:** "Top priorities before ship: SSRF + path-traversal (security), `byte[]` equals/hashCode + always-true `loadByes` (correctness), close the streams (leak), move mutations off GET. Then pagination and DI the fetcher for testability."

---

## Part D — `getPrice` base solution (reference only — already asked in R1)

Keep this fresh in case they ask you to re-derive it before extending. Full trace below.

```java
public static Double getPrice(Collection<Menu> menus, MenuItem target) {
    for (Menu menu : menus) {
        Double price = search(menu.groups, menu.price, target); // menu price is the seed
        if (price != null) return price;
    }
    return null; // not found anywhere
}
// `inherited` = nearest non-null ancestor price seen so far.
private static Double search(List<MenuGroup> groups, Double inherited, MenuItem target) {
    if (groups == null) return null;
    for (MenuGroup g : groups) {
        Double current = (g.price != null) ? g.price : inherited;   // group overrides if set
        if (g.items != null)
            for (MenuItem item : g.items)
                if (item.name.equals(target.name))                  // .equals, not ==
                    return (item.price != null) ? item.price : current;
        Double sub = search(g.groups, current, target);             // recurse, carry inherited
        if (sub != null) return sub;                                // propagate the find
    }
    return null; // keep searching siblings
}
```
**3 bugs to avoid:** returning `0.0` instead of `null` for not-found; returning on the first group instead of searching siblings; `==` on names.
**Trace:** `"Wings"` → Appetizers `5.00` overrides menu `11.50`, Wings null → `5.00`. `"Iced Latte"` → own `6.50` regardless of ancestors. **O(N)/O(H).**

---

## Part E — Android Architecture (only if your role is Android-leaning)

- **MVVM + unidirectional data flow:** View → ViewModel (survives config changes) → Repository → data sources; UI observes `StateFlow`/`LiveData`. MVVM over MVP: lifecycle-aware, testable ViewModel with no Android deps.
- **Repository + single source of truth:** Room is the SoT; network syncs *into* Room; UI reads from Room. Critical for a **POS that must work offline** — orders/payments can't fail on a Wi-Fi drop.
- **Offline-first sync:** queue mutations locally (WorkManager), replay on reconnect, idempotency keys to dedupe, last-write-wins / version-vector conflict resolution.
- **Coroutines + structured concurrency:** `viewModelScope` cancels on VM clear; `Dispatchers.IO` for DB/network; `Flow` to stream live ticket state to the kitchen display.
- **Jetpack:** ViewModel, Room, WorkManager, Navigation, Hilt, Compose — know *why* each (testability, lifecycle-safety, separation of concerns).
- **Real-time perf:** keep main thread free (POS responsive while printing/charging); paginate long order lists; `DiffUtil`/Compose keys to avoid full redraws.

---

## 🎯 CodePair Playbook — 75 min with a Tech Lead

This round is **~70% communication, 30% the answer.** A correct silent solution loses to a slightly-imperfect well-narrated one.

**Structure (drive it):**
1. **0–5 min — Clarify.** Restate the problem; ask about input shape, nulls, duplicates, empties, scale, output type. Confirm the data model. *Proactively name the menu-tree variations.*
2. **5–10 min — Plan out loud.** "I'll DFS carrying inherited X; base case…; recursive case…; O(N)/O(H)." Get a nod before typing.
3. **10–45 min — Code while narrating.** **Runnable Java** — real signatures, no `// pseudocode`. Build small, run early with the Run button.
4. **45–60 min — Test & edge cases out loud.** Walk a concrete trace; name empties/not-found/all-null/duplicates; fix what breaks.
5. **60–75 min — Add-ons + your questions.** They'll extend (find-all → path → iterative → service). Handle one. Leave 4–5 min for *your* questions.

**Communication that reads as senior:**
- Think out loud constantly; silence reads as stuck.
- Brute force → optimal; state complexity (time *and* space) **unprompted**.
- Edge cases before they ask; take hints gracefully ("Good point — let me adjust").
- Clean code = the signal: meaningful names, small helpers, early returns, no dead code (no always-`true`-return).

**Toast-specific framing (earns points):** Tie correctness to the business — *"Getting inherited pricing wrong means a guest is charged the wrong amount, so I'll test the null-inheritance carefully."* Reliability-first: validate inputs, handle the failure path, don't swallow exceptions.

**Smart questions for Kalaiselvi (asking shows seniority):**
- "What does your team own day-to-day, and where does this kind of menu/pricing logic live in the stack?"
- "How does Toast balance feature velocity against POS reliability?"
- "What's the hardest scaling or correctness problem the team is working on right now?"
- "As a tech lead, what separates a strong senior engineer from a mid-level one on your team?"

---

## ✅ Pre-interview checklist (do before 2:00 PM IST Thu Jun 18)
- [ ] Test **Zoom** (mic/cam) + **CodePair** (`hr.gs/6360a90`) ~15 min early; have `ayonija.mathur@gmail.com` open for connection-drop emails.
- [ ] Quiet room, charged laptop + charger, wired/hotspot internet backup, water.
- [ ] Blank local Java scratch file ready in case CodePair lags; know `List`/`Map`/`Deque`/`Optional`/recursion cold without autocomplete.
- [ ] Warm up: re-type **base `getPrice`** once, then **find-all** and **find-path** from memory.
- [ ] Can write the **alarm clock** (`ScheduledExecutorService` + from-scratch `PriorityQueue` version) and articulate concurrency vs parallelism / deadlock / `volatile` vs atomic.
- [ ] Can run the **code review** security-first in ~8 min.
- [ ] **Write code that compiles and runs.** The feedback explicitly rewards working code over clever pseudocode.

---

## 🧪 Part F — Runnable practice harness (paste & hit Run)

Self-contained single file. Sample menu mirrors the Round-1 trace data (menu `11.50`; Appetizers `5.00` → Wings null; Entrees null → Toast `6.50` → Avocado null; Iced Latte own `6.50`). Covers **base getPrice, find-by-category, find-by-tag, find-path, and iterative getPrice** with asserts. Paste into `MenuPractice.java` and run `java MenuPractice.java` (JDK 11+), or drop into CodePair's main class.

```java
import java.util.*;

public class MenuPractice {

    // ---------- model ----------
    static class Menu      { Double price; List<MenuGroup> groups = new ArrayList<>(); }
    static class MenuGroup { String name; Double price; List<MenuGroup> groups = new ArrayList<>(); List<MenuItem> items = new ArrayList<>();
        MenuGroup(String n, Double p) { name = n; price = p; } }
    static class MenuItem  { String name; Double price; Set<String> tags;
        MenuItem(String n, Double p, String... t) { name = n; price = p; tags = new HashSet<>(Arrays.asList(t)); } }

    // ---------- A/D: base getPrice ----------
    static Double getPrice(Collection<Menu> menus, MenuItem target) {
        for (Menu menu : menus) {
            Double p = search(menu.groups, menu.price, target);
            if (p != null) return p;
        }
        return null;
    }
    static Double search(List<MenuGroup> groups, Double inherited, MenuItem target) {
        if (groups == null) return null;
        for (MenuGroup g : groups) {
            Double current = (g.price != null) ? g.price : inherited;
            if (g.items != null)
                for (MenuItem it : g.items)
                    if (it.name.equals(target.name))
                        return (it.price != null) ? it.price : current;
            Double sub = search(g.groups, current, target);
            if (sub != null) return sub;
        }
        return null;
    }

    // ---------- A1: find by category ----------
    static List<MenuItem> findByCategory(List<MenuGroup> groups, String category, List<MenuItem> out) {
        if (groups == null) return out;
        for (MenuGroup g : groups) {
            if (category.equalsIgnoreCase(g.name) && g.items != null) out.addAll(g.items);
            findByCategory(g.groups, category, out);
        }
        return out;
    }

    // ---------- A1 variant: find by tag ----------
    static List<MenuItem> findByTag(List<MenuGroup> groups, String tag, List<MenuItem> out) {
        if (groups == null) return out;
        for (MenuGroup g : groups) {
            if (g.items != null)
                for (MenuItem it : g.items)
                    if (it.tags != null && it.tags.contains(tag)) out.add(it);
            findByTag(g.groups, tag, out);
        }
        return out;
    }

    // ---------- A2: find path (with backtracking) ----------
    static List<String> findPath(List<MenuGroup> groups, String target, List<String> path) {
        if (groups == null) return null;
        for (MenuGroup g : groups) {
            path.add(g.name);
            if (g.items != null)
                for (MenuItem it : g.items)
                    if (it.name.equals(target)) { path.add(it.name); return new ArrayList<>(path); }
            List<String> sub = findPath(g.groups, target, path);
            if (sub != null) return sub;
            path.remove(path.size() - 1);   // backtrack
        }
        return null;
    }

    // ---------- A3: iterative getPrice ----------
    static Double getPriceIterative(Collection<Menu> menus, MenuItem target) {
        Deque<Object[]> stack = new ArrayDeque<>();   // [MenuGroup, Double inherited] — Object[] tolerates null
        for (Menu menu : menus)
            for (MenuGroup g : menu.groups) stack.push(new Object[]{g, menu.price});
        while (!stack.isEmpty()) {
            Object[] frame = stack.pop();
            MenuGroup g = (MenuGroup) frame[0];
            Double inherited = (Double) frame[1];
            Double current = (g.price != null) ? g.price : inherited;
            if (g.items != null)
                for (MenuItem it : g.items)
                    if (it.name.equals(target.name))
                        return (it.price != null) ? it.price : current;
            if (g.groups != null)
                for (MenuGroup child : g.groups) stack.push(new Object[]{child, current});
        }
        return null;
    }

    // ---------- sample data ----------
    static List<Menu> sampleMenu() {
        Menu m = new Menu(); m.price = 11.50;

        MenuGroup apps = new MenuGroup("Appetizers", 5.00);
        apps.items.add(new MenuItem("Wings", null, "gluten-free"));
        apps.items.add(new MenuItem("Garden Salad", null, "vegan", "gluten-free"));

        MenuGroup entrees = new MenuGroup("Entrees", null);          // inherits menu 11.50
        MenuGroup toast = new MenuGroup("Toast", 6.50);             // overrides
        toast.items.add(new MenuItem("Avocado", null, "vegan"));    // -> 6.50
        entrees.groups.add(toast);
        entrees.items.add(new MenuItem("Steak", 24.00));           // own price

        MenuGroup drinks = new MenuGroup("Drinks", null);
        drinks.items.add(new MenuItem("Iced Latte", 6.50));        // own price

        m.groups.addAll(Arrays.asList(apps, entrees, drinks));
        return Collections.singletonList(m);
    }

    // ---------- tiny assert helper ----------
    static int passed = 0, failed = 0;
    static void check(String label, Object actual, Object expected) {
        boolean ok = Objects.equals(actual, expected);
        System.out.printf("%s %-28s got=%-18s expected=%s%n", ok ? "PASS" : "FAIL", label, actual, expected);
        if (ok) passed++; else failed++;
    }

    public static void main(String[] args) {
        List<Menu> menus = sampleMenu();
        List<MenuGroup> roots = menus.get(0).groups;

        // base getPrice
        check("getPrice Wings",     getPrice(menus, new MenuItem("Wings", null)),      5.00);
        check("getPrice Avocado",   getPrice(menus, new MenuItem("Avocado", null)),    6.50);
        check("getPrice Steak",     getPrice(menus, new MenuItem("Steak", null)),      24.00);
        check("getPrice Iced Latte",getPrice(menus, new MenuItem("Iced Latte", null)), 6.50);
        check("getPrice not-found", getPrice(menus, new MenuItem("Unicorn", null)),    null);

        // iterative must match recursive
        check("iter Wings",   getPriceIterative(menus, new MenuItem("Wings", null)),   5.00);
        check("iter Avocado", getPriceIterative(menus, new MenuItem("Avocado", null)), 6.50);

        // find by category
        List<MenuItem> apps = findByCategory(roots, "appetizers", new ArrayList<>());
        check("category Appetizers size", apps.size(), 2);

        // find by tag
        List<MenuItem> vegan = findByTag(roots, "vegan", new ArrayList<>());
        check("tag vegan size", vegan.size(), 2);   // Garden Salad, Avocado

        // find path
        check("path Avocado", findPath(roots, "Avocado", new ArrayList<>()),
              Arrays.asList("Entrees", "Toast", "Avocado"));
        check("path Wings",   findPath(roots, "Wings", new ArrayList<>()),
              Arrays.asList("Appetizers", "Wings"));
        check("path missing", findPath(roots, "Unicorn", new ArrayList<>()), null);

        System.out.printf("%n=== %d passed, %d failed ===%n", passed, failed);
    }
}
```

**Expected output:** all 12 `PASS`, final line `=== 12 passed, 0 failed ===`.

**Drill tonight:** delete each method body and re-type from the asserts until they all pass without peeking — that's exactly the muscle memory you need for CodePair.
