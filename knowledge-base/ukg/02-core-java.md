# 02 — Core Java

> A Senior Architect grilled core Java in the report you have. Collections internals + equals/hashCode + immutability are the highest-yield. Be able to go under the hood, not just name things.

---

## 2.1 OOP — the 4 pillars (one line + the *why*)
- **Encapsulation** — bundle data + behavior, hide internals behind methods. *Why:* control invariants, change internals freely.
- **Abstraction** — expose *what*, hide *how* (interfaces/abstract classes). *Why:* depend on contracts, not implementations.
- **Inheritance** — reuse/extend a base type. *Use sparingly* — **favor composition over inheritance** (inheritance couples tightly; composition is flexible).
- **Polymorphism** — one interface, many implementations; the runtime picks the method (dynamic dispatch). *Why:* extend behavior without editing callers (Open/Closed).

**Abstract class vs interface:** abstract class = shared state + partial implementation, single inheritance; interface = a contract, multiple inheritance, `default` methods since Java 8. Default to interfaces; use abstract class when sharing state/implementation.

---

## 2.2 Collections internals (the most-probed area)

**HashMap (know this cold):**
- Stores key→value in an **array of buckets**; bucket index = `(n-1) & hash(key)` where `hash` spreads bits (`h ^ (h>>>16)`).
- **Collision** (two keys → same bucket): chained in a **linked list**; since Java 8, a bucket **treeifies** to a **red-black tree** when it exceeds 8 entries (and table ≥ 64) → O(log n) instead of O(n) worst case.
- **Load factor** 0.75: when `size > capacity*0.75`, it **resizes** (doubles) and **rehashes** — amortized O(1) get/put.
- **Needs correct `equals`/`hashCode`** — `hashCode` finds the bucket, `equals` finds the entry within it.
- **Not thread-safe.** Concurrent writes can corrupt it (pre-8 could infinite-loop on resize).

**ConcurrentHashMap vs synchronized HashMap (asked explicitly — report Round 5):**
- `Collections.synchronizedMap` / `Hashtable`: lock the **whole map** on every op → one lock, low concurrency.
- **ConcurrentHashMap:** lock-free reads; writes lock only a **single bucket/bin** (CAS + synchronized on the bin head, Java 8+; "lock striping" pre-8). → far higher concurrent throughput. Doesn't allow null keys/values. Iterators are **weakly consistent** (don't throw `ConcurrentModificationException`, reflect some-but-not-all concurrent updates).

🎤 **Say it like this:** *"A HashMap is an array of buckets; the key's hash, after a bit-spreading step, picks the bucket. Collisions chain in a list that converts to a red-black tree past eight entries, so worst-case lookup is O(log n) instead of O(n). At 75% full it doubles and rehashes, which keeps get and put amortized O(1). It relies entirely on correct equals and hashCode — hashCode finds the bucket, equals finds the entry. For concurrency I don't wrap a HashMap with a global lock; I use ConcurrentHashMap, which keeps reads lock-free and locks only the individual bin on write, so throughput scales with cores instead of serializing on one lock."*

**ArrayList vs LinkedList:** ArrayList = dynamic array, O(1) random access, O(1) amortized append (doubles on grow), O(n) middle insert/delete. LinkedList = doubly-linked, O(1) insert/delete at ends, O(n) access. **Default to ArrayList** (cache-friendly); LinkedList rarely wins in practice.

**HashSet** = HashMap with dummy values. **TreeMap/TreeSet** = red-black tree, sorted, O(log n). **LinkedHashMap** = HashMap + insertion/access order (access-order mode → easy **LRU cache**).

---

## 2.3 equals & hashCode (the contract — classic question)
**Contract:**
1. If `a.equals(b)` then `a.hashCode() == b.hashCode()` (**equal objects → equal hashes**).
2. Unequal objects *may* share a hashCode (collision) — but good distribution matters.
3. Both must be consistent and based on the **same fields**.

**Why it matters:** break it and `HashMap`/`HashSet` misbehave — you store a key and can't find it (different hashCode → wrong bucket), or duplicates appear. Override **both, together**, using the same fields. Use `Objects.hash(...)` / `Objects.equals(...)`.

```java
@Override public boolean equals(Object o){
  if (this==o) return true;
  if (!(o instanceof User u)) return false;       // pattern match (Java 16+)
  return id == u.id && Objects.equals(email, u.email);
}
@Override public int hashCode(){ return Objects.hash(id, email); }
```

🎤 **Say it like this:** *"The rule is: equal objects must have equal hash codes, and I always override the two together on the same fields. If I break that, a HashMap puts the key in one bucket and looks for it in another, so I can store something and never find it again. The reverse — unequal objects sharing a hash code — is fine; that's just a collision the map handles. For mutable keys there's a subtle trap: if a field used in hashCode changes after insertion, the entry is effectively lost, so hash keys should be immutable."*

---

## 2.4 Generics, Streams, Optional, immutability
- **Generics** — compile-time type safety + reuse without casts. **Type erasure:** generics exist at compile time only; at runtime `List<String>` is just `List`. Wildcards: `? extends T` (read/producer), `? super T` (write/consumer) — **PECS** (Producer Extends, Consumer Super).
- **Streams** — declarative pipelines over collections: **intermediate** ops (lazy: `map`, `filter`) + a **terminal** op (triggers: `collect`, `reduce`, `forEach`). Enable readable transforms and easy parallelism (`parallelStream` — use carefully). Lazy = nothing runs until the terminal op.
- **Optional** — a container that may hold a value; signals "might be absent" in the type system to avoid NPEs. Use as a **return type**; don't use for fields/params. `map`/`orElse`/`orElseThrow` over `get()`.
- **Immutability** — object can't change after construction (`final` fields, no setters, defensive copies). *Why:* inherently thread-safe, safe as map keys, easier to reason about. Records (Java 16+) are concise immutable carriers. **Prefer immutable** by default.

🎤 **Streams line:** *"Streams let me express a transformation as a pipeline — filter, map, collect — instead of imperative loops, and they're lazy: intermediate operations build a plan and nothing executes until the terminal operation. That readability is the main win; parallelism is available but I reach for it only for large, CPU-bound, stateless work because the overhead and ordering surprises often aren't worth it."*

---

## ⚠️ Pitfalls & seniority signals
- Overriding `equals` but not `hashCode` (or different fields) — silent map bugs.
- Mutable objects as hash keys.
- Claiming "HashMap is thread-safe" or "synchronizedMap == ConcurrentHashMap" — know the locking difference.
- `Optional.get()` without checking; `Optional` fields.
- **Seniority signal:** explain HashMap *treeification + resize*, ConcurrentHashMap *bin-level locking*, and the equals/hashCode failure mode in terms of *buckets* — that depth is what the architect is probing.

---

*Next topic, or drill deeper on this one?*
