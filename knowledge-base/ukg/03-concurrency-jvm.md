# 03 — Java Concurrency & JVM

> Report flagged multithreading + ConcurrentHashMap + (C/C++) malloc/calloc/segfaults. Focus: thread-safety mechanisms, the memory model, executors/CompletableFuture, GC. Plus a short C-memory appendix since a panel mixed it in.

---

## 3.1 Threads, executors, CompletableFuture
- **Thread** — independent path of execution; threads in a process share heap memory (hence the need for synchronization). Creating raw threads is expensive → pool them.
- **ExecutorService / thread pool** — reuse a fixed set of worker threads to run submitted tasks. `Executors.newFixedThreadPool(n)`, `newCachedThreadPool`, `newVirtualThreadPerTaskExecutor()` (Java 21 virtual threads — cheap, for blocking I/O). *Why pool:* bound resource use, avoid thread-creation cost, apply back-pressure via a bounded queue.
- **Future / CompletableFuture** — handle to an async result. `CompletableFuture` composes async work without blocking: `thenApply`, `thenCompose` (flat-map), `thenCombine` (join two), `exceptionally` (handle errors), `allOf`. *Why:* non-blocking pipelines, parallel fan-out/fan-in.

```java
CompletableFuture<User> u = supplyAsync(() -> userClient.get(id), pool);
CompletableFuture<Cart> c = supplyAsync(() -> cartClient.get(id), pool);
u.thenCombine(c, (user, cart) -> render(user, cart))   // runs both in parallel, joins
 .exceptionally(ex -> fallback());                      // resilience
```

🎤 **Say it like this:** *"I almost never create raw threads — I submit tasks to an executor so the pool bounds resource use and a bounded queue gives me back-pressure. For composing async calls I use CompletableFuture: I can fire two service calls in parallel with supplyAsync and join them with thenCombine, which turns sequential latency into the max of the two instead of the sum, and exceptionally gives me a clean fallback. On Java 21 I'd reach for virtual threads for blocking I/O — they make the simple thread-per-request model cheap again."*

---

## 3.2 Thread-safety: volatile vs synchronized, locks, atomics
- **Race condition** — two threads access shared mutable state and at least one writes, with no ordering → corrupt/lost updates.
- **`synchronized`** — mutual exclusion: only one thread holds the monitor at a time; also establishes a **happens-before** memory barrier (changes visible to the next thread). Use for compound actions (check-then-act, read-modify-write).
- **`volatile`** — **visibility only, not atomicity.** Guarantees reads/writes of that variable go to/from main memory (no stale cached copy) and aren't reordered. Good for a flag (`volatile boolean running`); **does NOT** make `count++` safe (that's read-modify-write = 3 ops).
- **`java.util.concurrent.locks` (ReentrantLock)** — like synchronized but with `tryLock`, timeout, fairness, multiple condition variables. **ReadWriteLock** — many readers OR one writer (read-heavy data).
- **Atomics** (`AtomicInteger`, `AtomicLong`) — lock-free thread-safe single-variable updates via **CAS** (compare-and-swap: atomically "set to new if still equals expected, else retry"). Use for counters instead of synchronized.

🎤 **volatile vs synchronized (classic):** *"They solve different problems. volatile gives visibility — every thread sees the latest value, no stale cache — but it does not give atomicity, so a counter increment, which is read-modify-write, is still unsafe with volatile. synchronized gives both mutual exclusion and visibility, so it's what I use for compound actions like check-then-act. For a simple counter I skip both and use an AtomicInteger, which is lock-free compare-and-swap and faster under contention."*

**Deadlock** — two threads each hold a lock the other needs → both wait forever. **4 Coffman conditions:** mutual exclusion, hold-and-wait, no preemption, circular wait. **Prevent:** acquire locks in a **global order**, use `tryLock` with timeout, minimize lock scope, prefer higher-level concurrency utilities. (Also: livelock = keep reacting but no progress; starvation = a thread never gets the resource.)

---

## 3.3 JVM memory model & garbage collection
- **JMM (Java Memory Model)** — defines when one thread's writes become visible to another; the **happens-before** relationship is the guarantee (`synchronized` release→acquire, `volatile` write→read, thread start/join all establish it).
- **Memory areas:** **Heap** (objects, shared, GC'd) · **Stack** (per-thread: frames, locals, return addresses — `StackOverflowError` on deep recursion) · **Metaspace** (class metadata, off-heap since Java 8, replaced PermGen) · PC register · native stacks.
- **Heap generations:** **Young** (Eden + 2 Survivor spaces — most objects die young) and **Old/Tenured**. **Minor GC** cleans Young (frequent, fast, copying); **Major/Full GC** cleans Old (slower, can pause).
- **GC = automatic memory reclamation** of unreachable objects (reachability from GC roots). Collectors: **G1** (default since Java 9 — region-based, low-pause, balances throughput/latency), **ZGC / Shenandoah** (ultra-low pause, large heaps), Parallel (throughput), Serial (small). **`StackOverflowError`** (stack, recursion) vs **`OutOfMemoryError`** (heap exhausted / leak).

🎤 **GC line:** *"The heap is split into young and old generations on the weak-generational hypothesis — most objects die young. Minor GC frequently and cheaply collects the young generation by copying survivors; objects that live long enough get promoted to the old generation, which is collected less often but more expensively. The default G1 collector divides the heap into regions and targets a pause-time goal, so I get predictable latency without hand-tuning. A memory leak in Java isn't unfreed memory — it's unintended reachability, like a static collection that keeps growing, so objects stay 'alive' and never get collected."*

**Follow-ups:** *Memory leak in Java?* objects still referenced (static maps, listeners not removed, ThreadLocals) → never collected; find with a heap dump. *Stack vs heap?* stack = per-thread locals/frames (fast, auto-freed on return); heap = shared objects (GC'd). *Why ConcurrentHashMap over synchronized?* bin-level locking + lock-free reads (sheet 02).

---

## 3.4 C/C++ memory appendix (a panel mixed these in — quick, confident answers)
- **`malloc` vs `calloc`** — both allocate heap memory. `malloc(size)` leaves it **uninitialized** (garbage); `calloc(n, size)` allocates `n*size` and **zero-initializes** it. `calloc` also guards against multiplication overflow.
- **Segmentation fault** — a program accesses memory it isn't allowed to (dereferencing NULL/dangling/freed pointer, out-of-bounds, stack overflow). The OS's memory protection traps it. Common causes: use-after-free, buffer overrun, null deref.
- **(If asked) `free`/dangling pointer** — freeing memory then using the pointer = undefined behavior; set to NULL after free. Java avoids all this via GC + no raw pointers.

🎤 *"malloc gives you uninitialized memory; calloc zero-initializes and is overflow-safe on the size calc. A segfault is the OS trapping an illegal memory access — null or dangling pointer, or out-of-bounds. The reason I work in Java for this kind of system is that the GC and the absence of raw pointer arithmetic remove this entire class of bug, at the cost of GC pauses I manage with collector choice."*

---

## ⚠️ Pitfalls & seniority signals
- Saying `volatile` makes `count++` thread-safe — it doesn't (no atomicity).
- Confusing visibility (volatile) with mutual exclusion (synchronized).
- Creating raw threads instead of pooling; unbounded pools/queues (OOM, no back-pressure).
- "GC frees memory I don't reference" — leak = *unintended reachability*, not unfreed memory.
- **Seniority signal:** happens-before reasoning, CAS/atomics for counters, generational GC + G1 region model, deadlock prevention via lock ordering — these read senior.

---

*Next topic, or drill deeper on this one?*
