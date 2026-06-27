# 07 — Angular + TypeScript

> You led an Angular 17→19 migration — lean on that. Includes the asked **CSR vs SSR**. Focus: change detection, RxJS, and TS narrowing (the parts that separate seniors).

---

## 7.1 Component architecture & lifecycle
- **Component** = template + class + styles; the UI building block. **Module** (`NgModule`) groups components/services — though Angular 17+ favors **standalone components** (no NgModule), with `@Input()`/`@Output()` for parent↔child, or **signals** (17+) for reactive state.
- **Lifecycle hooks (order):** `ngOnChanges` (input changes) → `ngOnInit` (init, fetch data) → `ngDoCheck` → `ngAfterViewInit` (view ready) → `ngOnDestroy` (cleanup — **unsubscribe here** to avoid leaks).

---

## 7.2 Change detection + zone.js + OnPush (the key Angular topic)
**Plain English:** Change detection = how Angular figures out what changed and updates the DOM.

**How it works:** **zone.js** monkey-patches async APIs (events, `setTimeout`, XHR/fetch, promises) so Angular *knows* when something async finished and runs **change detection** — it walks the component tree top-down, compares bound values, and updates the DOM where they changed.

- **Default strategy:** checks the **whole tree** on every event. Simple but can be wasteful in big apps.
- **OnPush:** a component is re-checked **only when** (a) an `@Input` **reference** changes, (b) an event fires inside it, or (c) an observable bound with `async` emits. → big performance win. *Implication:* with OnPush you must treat data as **immutable** (new reference) and/or use signals — mutating an object in place won't trigger it.
- **Signals (Angular 17+):** fine-grained reactivity — Angular tracks exactly which signals a template reads and updates only those, moving toward **zoneless** change detection (drop zone.js entirely).

🎤 **Say it like this:** *"Change detection is how Angular keeps the DOM in sync with component state. zone.js patches async APIs so Angular knows when to check; by default it walks the entire component tree on every event and compares bindings. In a large app that's wasteful, so I use OnPush, which only re-checks a component when an input *reference* changes, an event fires inside it, or an async pipe emits. The catch OnPush forces is immutability — I have to pass new references, not mutate in place — which is good discipline anyway. The newer model is signals: Angular tracks exactly which signals a template reads, so it updates just those, which is the path to dropping zone.js entirely and going zoneless."*

**Follow-up:** *Q: OnPush component not updating?* You mutated an object instead of replacing the reference, or updated state outside Angular's zone — pass a new reference, use signals, or `markForCheck()`.

---

## 7.3 RxJS (observables, operators, subjects, hot/cold)
- **Observable** — a stream of values over time you **subscribe** to (vs a Promise = single future value). Lazy: nothing runs until subscribed.
- **Cold** — produces values per-subscriber (each subscription re-runs, e.g. an HTTP call). **Hot** — shares one production among subscribers (e.g. a Subject, DOM events). `share()`/`shareReplay()` makes cold hot.
- **Subject** — both observable *and* observer (you can `next()` into it) → an event bus / multicast. **BehaviorSubject** holds a current value (great for state). **ReplaySubject** replays last N.
- **Key operators:** `map`, `filter`, `switchMap` (cancel previous inner — **typeahead/search**), `mergeMap` (concurrent), `concatMap` (ordered, queued), `exhaustMap` (ignore new while busy — **prevent double-submit**), `debounceTime` (wait for quiet — search input), `distinctUntilChanged`, `combineLatest`, `catchError`, `takeUntil` (unsubscribe pattern), `retry`.

🎤 **switchMap vs mergeMap (classic):** *"They differ in how they handle a new outer value while an inner stream is still running. switchMap cancels the previous inner observable and switches to the new one — perfect for a search box, because I don't care about results for a query the user already changed. mergeMap runs them all concurrently and keeps every result — right when each emission matters, like firing independent saves. concatMap queues them in order, and exhaustMap ignores new ones until the current finishes, which is how I stop a double-click double-submit."*

**Memory-leak fix:** subscriptions live until completed/unsubscribed. Use `takeUntil(destroy$)` in `ngOnDestroy`, or the **`async` pipe** (auto-subscribes/unsubscribes — preferred).

---

## 7.4 State management
- Small: component state + services with `BehaviorSubject`, or **signals**.
- Large/shared: **NgRx** (Redux pattern — single immutable store, actions, reducers, effects, selectors). *Why:* predictable, debuggable, time-travel; *cost:* boilerplate. Don't reach for it unless shared cross-feature state justifies it.

---

## 7.5 TypeScript essentials
- **Types vs interfaces** — interface for object shapes (extendable/merged), type for unions/intersections/aliases.
- **Generics** — reusable typed code: `function id<T>(x: T): T`.
- **Narrowing** — TS shrinks a union to a specific type via `typeof`, `instanceof`, `in`, discriminated unions (`kind` field), truthiness. Enables type-safe branching.
- **Utility types** — `Partial<T>`, `Pick`, `Omit`, `Record<K,V>`, `Readonly`. **`unknown` vs `any`:** `unknown` forces you to narrow before use (safe); `any` disables checking (avoid). `strictNullChecks` to catch null bugs.

🎤 *"TypeScript's value is catching errors at compile time. The feature I lean on most is narrowing — TS shrinks a union type based on runtime checks like typeof or a discriminated-union tag, so each branch is fully typed and I can't, say, call a string method on a number. And I prefer unknown over any for external data because unknown forces me to validate before use, where any silently turns off the type checker."*

---

## 7.6 CSR vs SSR (asked directly)
- **CSR (Client-Side Rendering)** — server sends a near-empty HTML + JS bundle; the **browser** builds the UI. Pro: rich app-like interactivity, cheap server. Con: slow first paint, weaker SEO (crawlers see empty HTML), big JS download.
- **SSR (Server-Side Rendering)** — server renders **full HTML** per request; browser shows it immediately, then JS **hydrates** it (attaches interactivity). Pro: fast first paint, SEO-friendly. Con: heavier server load, more complexity. (**Angular Universal**; **SSG** = pre-render at build time for static content.)

🎤 **Say it like this:** *"With client-side rendering the server ships a JS bundle and the browser builds the page — great for highly interactive apps but the first paint is slow and SEO suffers because crawlers initially see an empty shell. Server-side rendering returns fully-formed HTML so the user sees content immediately and search engines can read it, then the JS hydrates to add interactivity — at the cost of more server work. I choose by need: SSR or static generation for public, SEO-sensitive, content-heavy pages; CSR for authenticated, app-like dashboards where SEO doesn't matter and interactivity does."*

---

## 7.7 Performance
- **Lazy loading** routes (load a feature's bundle only when visited) → smaller initial bundle.
- **OnPush / signals** to cut change-detection work; `trackBy` in `*ngFor` to avoid re-rendering unchanged list items.
- **Bundle optimization:** tree-shaking, code-splitting, `@defer` (Angular 17 deferred loading), analyze with source-map-explorer. *(Your migration cut build time 30% — mention the Vitest swap + esbuild/flat-config.)*

---

## ⚠️ Pitfalls & seniority signals
- Not unsubscribing (memory leaks) — use `async` pipe / `takeUntil`.
- Mutating state with OnPush and wondering why the view won't update (reference equality).
- `switchMap` vs `mergeMap` confusion in a search box (cancellation semantics).
- `any` everywhere — defeats TS.
- **Seniority signal:** explain change detection via zone.js + reference-equality, pick the right RxJS operator *by cancellation behavior*, and frame SSR/CSR as a per-page trade-off — plus your real migration metrics.

---

*Next topic, or drill deeper on this one?*
