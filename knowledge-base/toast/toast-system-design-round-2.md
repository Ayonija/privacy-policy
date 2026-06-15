# Toast — System Design Round 2: LLD, Code Review & Android Architecture

**Focus:** Object/concurrency design, code-review craft, Android-specific architecture
**Anchored on actual Toast questions:** Code-review round (R3 MenuDataResource), `getPrice` menu-tree problem, menu-item fetch, multi-threaded alarm clock, threads/processes/concurrency/parallelism deep-dive
**Study time:** ~75 min

> **Reality check from the dataset:** Toast's "second technical round" is usually a **peer-coding / code-review / LLD** round where the interviewer brings their own sample code. It is *not* a whiteboard distributed-systems round. The bar (per feedback) is **fully working code, running in the emulator, UI matching the sample**. So: write code that compiles and runs, not pseudocode.

---

## Part A — Code Review Round (R3, real code shipped to you)

The interviewer hands you `MenuDataResource` (a Dropwizard JAX-RS service that ingests restaurant menus from files/URLs) and asks you to review it. **They are testing whether you can spot security, correctness, and design defects in real code.** Walk it top-to-bottom in priority order. Lead with security and correctness, then design, then style.

### How to run the review (say this framing first)
> "I'll group findings by severity: **Security → Correctness/bugs → Resource & error handling → API/design → Testing & style.** I'll call out the bug and the fix for each."

### 🔴 Security (lead here — this service "allows everyone and everything")

1. **SSRF in `/loadUrl`** — takes an arbitrary user URL and `openConnection().getInputStream()`. An attacker passes `http://169.254.169.254/latest/meta-data/` (cloud metadata), `http://localhost:8080/admin`, or `file://`. **Fix:** allowlist schemes (http/https only), resolve the host and **block private/link-local/loopback ranges**, set connect/read timeouts, cap response size, disable redirects to internal hosts.
2. **Path traversal / arbitrary file read in `/loadFile`** — `Files.readAllBytes(Paths.get(location))` with a user-supplied path reads `/etc/passwd` or app secrets. **Fix:** resolve against a fixed base directory and verify the canonical path is still under it; reject `..`.
3. **Stored XSS in `list()`** — builds raw HTML: `menuData.getLocator()` is concatenated unescaped into an `<a href=...>`. A malicious locator injects script. **Fix:** HTML-escape all dynamic values, quote attributes; better, return JSON and let the client render.
4. **No authentication / authorization** — "we allow everyone" is stated as a feature but it means unbounded ingestion, abuse, and DoS. At minimum rate-limit and authenticate writes.

### 🟠 Correctness bugs

5. **`loadByes` always returns `true`** — the success/error branch in `loadFile`/`loadUrl` is dead code. On a DB failure it still says `"success!"`. Also `Reader.retrieve` returns `null` on `IOException`; `loadByes` then persists a row with `bytes = null`, but the column is `nullable = false` → it'll throw at flush, *after* you've told the caller... nothing consistent. **Fix:** return real status; check for null bytes before persisting; map failures to proper HTTP codes.
6. **`MenuData.equals`/`hashCode` use `Objects.equals`/`Objects.hash` on a `byte[]`** — that compares array **identity**, not contents, and hashes by identity. Two equal-content entities are "unequal." **Fix:** `Arrays.equals(this.bytes, that.bytes)` and `Arrays.hashCode(bytes)` (or `Objects.hash(id, locator, Arrays.hashCode(bytes))`). Also: entity equality on a mutable `byte[]` is a smell — prefer equality on `id` only for JPA entities.
7. **`parseItems` uses `obj.name == item.name`** (reference equality on Strings) in the `getPrice` cousin — should be `.equals`. Same class of bug; flag it if present.
8. **`get()` throws `new RuntimeException("Error 404 Not Found")`** — returns HTTP 500, not 404. **Fix:** `throw new WebApplicationException(Response.Status.NOT_FOUND)` (JAX-RS) so the status code is correct.

### 🟠 Resource & error handling

9. **`InputStream`/`URLConnection` not closed** in the URL branch — resource leak; `is` is never closed even on success. **Fix:** `try (InputStream is = conn.getInputStream())` (try-with-resources). Same for the buffer.
10. **Unbounded read into memory** — `ByteArrayOutputStream` grows with no size cap; a huge/malicious URL OOMs the JVM. **Fix:** enforce a max-bytes limit and abort past it.
11. **Swallowing `IOException` → `return null`** propagates a null downstream and loses the failure. **Fix:** translate to a domain exception / proper HTTP error.

### 🟡 API & design

12. **`GET` used for state-changing `loadFile`/`loadUrl`** — these create rows. GET must be safe/idempotent; caches, prefetchers, and crawlers will trigger ingestion. **Fix:** `POST`.
13. **`list()` returns **all** rows, no pagination** — table scan + huge response as the dataset grows ("all the menus in the world"). **Fix:** cursor/limit pagination.
14. **`Reader` is a `static` nested class doing I/O** — hard to mock/test, couples HTTP and file logic. **Fix:** inject a `MenuFetcher` interface (`FileFetcher`, `UrlFetcher` impls) — also makes the `ReaderType` switch unnecessary (polymorphism over enum-switch).
15. **`findAll` does `SELECT d FROM MenuData d`** pulling full `byte[]` blobs for a list view — expensive. **Fix:** projection of `(id, locator)` only.

### 🟡 Testing

16. **`ReaderTest.retrieveTest` hits `http://www.example.com`** — a network call in a unit test = flaky, slow, fails offline. **Fix:** mock the fetcher or use a local stub server (WireMock). Also it never tests failure paths (bad path, IOException, oversized input).

### How to close the review
> "Top priorities before this ships: fix the SSRF and path-traversal (security), the `byte[]` equals/hashCode and the always-true `loadByes` (correctness), close the streams (leak), and move mutations off GET. Then pagination and dependency-inject the fetcher for testability."

**Practice tip:** rehearse saying these in ~8 minutes. The signal is *prioritization* — security/correctness first, not nitpicking braces.

---

## Part B — `getPrice`: Menu Tree with Inherited Pricing (real coding problem)

> Given nested `Menu → MenuGroup → (MenuGroup | MenuItem)`, each with a **nullable** price. An item's effective price is its own price, or if null, the **nearest non-null ancestor's** price (group, then parent group, … then menu). Implement `getPrice(menus, item)`.

### S/T — The crux
It's a **DFS over a tree** carrying an **inherited value** down the recursion. The candidate solution in the wild had three bugs: (1) returned `0.0` instead of propagating "not found" (null), (2) returned on the **first** group instead of searching siblings, (3) used `==` on names. Don't repeat them.

### A — Clean recursive solution

```java
public static Double getPrice(Collection<Menu> menus, MenuItem target) {
    for (Menu menu : menus) {
        Double price = search(menu.groups, menu.price, target); // menu price is the seed
        if (price != null) return price;                        // found in this menu
    }
    return null; // not found anywhere
}

// `inherited` = nearest non-null ancestor price seen so far.
private static Double search(List<MenuGroup> groups, Double inherited, MenuItem target) {
    if (groups == null) return null;
    for (MenuGroup g : groups) {
        Double current = (g.price != null) ? g.price : inherited; // group overrides if set

        // 1) direct items in this group
        if (g.items != null) {
            for (MenuItem item : g.items) {
                if (item.name.equals(target.name)) {              // .equals, not ==
                    return (item.price != null) ? item.price : current;
                }
            }
        }
        // 2) recurse into subgroups, carrying the updated inherited price
        Double sub = search(g.groups, current, target);
        if (sub != null) return sub;                              // propagate the find
    }
    return null; // not found in this branch — keep searching siblings
}
```

### Trace (from the sample data)
- `getPrice(..., "Wings")`: Appetizers group price = `5.00` (non-null → overrides menu's `11.50`). Wings item price = `null` → inherits `5.00`. ✅
- `"Avocado"` (under Entrees→Toast): Entrees price `null` → inherits menu `11.50`; Toast price `6.50` overrides → Avocado null → `6.50`. ✅
- `"Iced Latte"`: own price `6.50` → returns `6.50` regardless of ancestors. ✅

### Complexity
- **Time O(N)** — visits each node once (N = items + groups). **Space O(H)** — recursion depth = tree height.

### Likely add-ons (the dataset mentions "some add-on questions")
- *"Return the full path to the item"* → pass/accumulate a `List<String>` of names down the recursion.
- *"Find all items under a category (e.g. all vegan salads)"* → DFS collecting matches into a list instead of returning the first (this is the separate "fetch vegan salads from the menu" question — same traversal, collect-all instead of find-first).
- *"Handle duplicate item names"* → return all matches, or qualify by path.
- *"Make it iterative"* → explicit stack of `(group, inheritedPrice)` pairs.

---

## Part C — Multi-Threaded Alarm Clock (LLD concurrency round)

> *"Implement a multi-threaded alarm clock with `setAlarm` and `ringAlarm`."* Asked as a panel LLD round. They probe **threads, processes, concurrency vs parallelism** around it.

### A — Design with `ScheduledExecutorService` (the idiomatic answer)

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
- **Why `ScheduledExecutorService` over `wait/notify` + a sleeping thread?** A thread pool reuses threads (no thread-per-alarm), the scheduler handles the timing wheel, and it's interruptible/cancellable. If they want the "from scratch" version, show a single timer thread holding a **`PriorityQueue` ordered by fire-time** + `ReentrantLock`/`Condition`, doing `condition.awaitNanos(timeToNextAlarm)` and waking early when an earlier alarm is inserted (the classic Timer design).
- **Thread-safety:** `ConcurrentHashMap` for the registry; the callback runs on a pool thread, so `onRing` must itself be thread-safe.
- **Concurrency vs parallelism:** *concurrency* = many alarms managed/interleaved by few threads; *parallelism* = multiple alarms firing **at once** on a multi-core pool. Single-thread pool = concurrent but not parallel.

### Core-CS deep-dive (the "intense" threads/processes round)
| Concept | One-liner that signals depth |
|---|---|
| Process vs Thread | Process = own address space; thread = shares heap, has own stack/PC. Context-switch is cheaper between threads. |
| Concurrency vs Parallelism | Concurrency = *dealing with* many things (interleaving, structure); parallelism = *doing* many at once (hardware). You can be concurrent on one core. |
| Race condition | Two threads touch shared mutable state without synchronization → outcome depends on timing. Fix: locks, atomics, immutability, confinement. |
| Deadlock | Circular wait on locks. Avoid via lock ordering, tryLock with timeout, or lock-free structures. |
| `synchronized` vs `Lock` | `Lock` adds tryLock, timed/interruptible acquisition, fairness, multiple conditions. |
| Atomics / CAS | `AtomicInteger` etc. use compare-and-swap → lock-free, no blocking. |
| `volatile` | Visibility (no caching), not atomicity. `count++` is still unsafe on a volatile. |
| Thread pool sizing | CPU-bound ≈ #cores; I/O-bound ≈ higher (cores × (1 + wait/compute)). |

---

## Part D — Android-Specific Architecture (be ready to discuss)

Toast ships Android POS hardware, so for an Android-leaning role expect architecture questions. Have crisp answers:

- **MVVM + unidirectional data flow:** View → ViewModel (survives config changes, holds UI state) → Repository → data sources. UI observes `StateFlow`/`LiveData`. Why MVVM over MVP: lifecycle-aware, testable ViewModel with no Android deps.
- **Repository pattern + single source of truth:** Room (local DB) is the SoT; network syncs *into* Room; UI always reads from Room. Critical for a **POS that must work offline** — orders/payments can't fail because Wi-Fi dropped. (Tie back to "real-time data handling": local-first + background sync + conflict resolution on reconnect.)
- **Offline-first sync:** queue mutations locally (WorkManager), replay to server on reconnect, idempotency keys to dedupe, last-write-wins or version-vector conflict resolution. This is the mobile mirror of the Labor/Booking backend designs.
- **Coroutines + structured concurrency:** `viewModelScope` cancels work on VM clear; `Dispatchers.IO` for DB/network; `Flow` for streaming the live order/ticket state to the kitchen display.
- **Jetpack stack:** ViewModel, Room, WorkManager, Navigation, Hilt (DI), Compose (UI). Know *why* each: testability, lifecycle-safety, separation of concerns.
- **Performance/real-time:** keep the main thread free (POS must stay responsive while printing/charging); paginate long order lists; use `DiffUtil`/Compose keys to avoid full redraws on live updates.

---

## 60-second pre-interview checklist
- [ ] Can review the `MenuDataResource` code in 8 min, security-first.
- [ ] Can write `getPrice` correctly (carry inherited price, `.equals`, propagate null, search siblings).
- [ ] Can write the alarm clock with `ScheduledExecutorService` *and* explain the from-scratch PriorityQueue version.
- [ ] Can articulate concurrency vs parallelism, deadlock, `volatile` vs atomic.
- [ ] Can explain Android MVVM + offline-first SoT and tie it to real-time POS needs.
- [ ] **Write code that compiles and runs.** The feedback explicitly rewards working, emulator-running code over clever pseudocode.
