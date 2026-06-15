# Week 20 Review — LLD Foundations
**Week 20 | Phase: LLD Mastery | Month 4**

## Focus
Consolidate OOP pillars, SOLID principles, and creational design patterns. Identify gaps before moving to structural patterns next week. Every item in this review should be answerable in under 30 seconds.

---

## Summary Table: Everything Covered This Week

### OOP — 4 Pillars

| Pillar | One-liner | Key analogy | Confusion pair |
|---|---|---|---|
| Encapsulation | Bundle data + behavior; restrict direct field access | ATM — you use the keypad, not the cash mechanism | vs. Abstraction |
| Abstraction | Expose *what*, hide *how* | Steering wheel — you turn it, not the hydraulics | vs. Encapsulation |
| Inheritance | Child acquires parent's properties; IS-A relationship | Parent–child traits | Inheritance vs. Composition (HAS-A) |
| Polymorphism | One interface, type-specific behavior | `run` command to human / dog / robot | Overriding vs. Overloading |

**Abstraction vs. Encapsulation in one breath:** Abstraction hides implementation; Encapsulation hides data.

**Overriding vs. Overloading in one breath:** Overriding = runtime dispatch in subclass; Overloading = compile-time signature difference in same class.

---

### SOLID — 5 Principles

| Principle | One-liner | Smell it fixes | Analogy |
|---|---|---|---|
| S — Single Responsibility | One class, one reason to change | God class | Chai dukaan owner makes chai only |
| O — Open/Closed | Open for extension, closed for modification | if-else chain grows every sprint | Plug socket — never rewired |
| L — Liskov Substitution | Subclass must honor parent's contract | `Penguin.fly()` throws exception | Penguin IS-A Bird (biologically) but breaks Bird.fly() |
| I — Interface Segregation | No class forced to implement unused methods | Fat interface with stub/exception impls | Café doesn't need the hotel spa menu |
| D — Dependency Inversion | High-level depends on abstractions, not concretions | `new ConcreteClass()` inside business logic | USB-C socket — laptop never rewired for a new charger |

---

### Creational Patterns — 4 + Prototype

| Pattern | One-liner | Machine coding trigger |
|---|---|---|
| Singleton | Exactly one instance; global access point | "There should be only one X" |
| Factory Method | Delegate type selection; caller never calls `new` | "Create the right type of X" |
| Abstract Factory | Family of compatible objects from one factory | "Create a compatible set of X, Y, Z" |
| Builder | Step-by-step assembly; fluent chaining; immutable output | "Build configurable X with many optional parts" |
| Prototype | Clone existing object instead of rebuilding | "Create a new X similar to this existing X" |

---

## Most-Tested Interview Questions This Week

1. "What is the difference between abstraction and encapsulation?"
2. "Explain polymorphism. What is the difference between overriding and overloading?"
3. "What is SRP? How do you identify a violation?"
4. "How does the Liskov Substitution Principle relate to inheritance?"
5. "What is Dependency Inversion? How is it different from Dependency Injection?"
6. "What is the Singleton pattern? How do you make it thread-safe?"
7. "When would you use Builder over Factory?"
8. "Design a parking lot." (machine coding)
9. "Walk me through the SOLID principles with a real example for each."
10. "Which creational pattern would you use for an HTTP client with optional headers?"

---

## Common Mistakes to Avoid

| Mistake | Correction |
|---|---|
| Confusing abstraction and encapsulation | Abstraction = hiding *implementation* (what it does); Encapsulation = hiding *data* (what it holds) |
| Saying LSP is just "inheritance" | LSP is about *behavioral contracts* — subclass must not break what callers expect from the parent type |
| Using lazy Singleton without `volatile` | Instruction reordering allows a partially-constructed object to be read; `volatile` prevents this |
| Writing Factory as a growing switch-case | Use registration map (`Map<String, Supplier<T>>`) to honor OCP |
| Building a Builder without final fields in product | Product must have `private final` fields; no setters; all set in `build()` only |
| ISP = OCP confusion | ISP is about *interfaces being too fat*; OCP is about *classes being modified to add behavior* |
| Calling DIP the same as DI | DIP is the principle (depend on abstractions); Dependency Injection is the implementation pattern |
| Abstract Factory vs. Factory method mix-up | Factory Method = one product type; Abstract Factory = family of compatible products |

---

## Quick-Fire Quiz — 10 Questions

Attempt each from memory before checking answers below.

1. Which OOP pillar does `private` field + `getBalance()` represent?
2. A `Bird.fly()` interface is implemented by `Penguin` which throws `UnsupportedOperationException`. Which SOLID principle is violated?
3. `new MySQLRepo()` inside `OrderService`. Which SOLID principle? What's the fix?
4. You have a `WorkerInterface` with 8 methods and `RobotWorker` can only do 3 of them. Which principle? Fix?
5. You add a new payment type but must edit `processPayment()`. Which principle? Fix?
6. You need exactly one `ConfigManager`. Which pattern?
7. You need a `Pizza` with 10 optional toppings. Which pattern?
8. You need `WindowsButton` + `WindowsCheckbox` to always match. Which pattern?
9. An existing `Document` object took 2 seconds to construct from a DB. You need 50 copies. Which pattern?
10. You need to create `Car`, `Truck`, or `Motorcycle` from a type string. Which pattern?

### Answers

| # | Answer |
|---|---|
| 1 | Encapsulation |
| 2 | LSP — subclass violates parent's behavioral contract |
| 3 | DIP — fix: introduce `OrderRepository` interface; inject via constructor |
| 4 | ISP — fix: split into role interfaces; `RobotWorker` implements only what it can |
| 5 | OCP — fix: Strategy pattern or registration map; new type = new class, not modified existing |
| 6 | Singleton |
| 7 | Builder |
| 8 | Abstract Factory |
| 9 | Prototype |
| 10 | Factory Method |

Score yourself: 9–10 = ready to move on; 7–8 = review weak areas; <7 = re-read days 131–137.

---

## STAR Stories Ready to Deploy

| Topic | Story summary |
|---|---|
| OOP | Refactored monolithic payment processor into 4-layer design using all 4 pillars |
| SRP | Decomposed 1,200-line `UserService` into 6 focused services; zero cross-concern failures |
| DIP | Introduced `NotificationChannel` interface; went from 12% to 78% test coverage |
| Singleton | Thread-safe async logger with `BlockingQueue`; 99% log correlation |
| Factory | Notification factory with registration map; new channel in 2 hours |
| Builder | HTTP client config; replaced 12-arg constructor; zero misconfiguration incidents |
| Parking Lot | Machine coding: Singleton + Factory + Strategy; interviewer noted correct pattern instinct |

---

## Next Week Preview: Structural Patterns

| Day | Pattern |
|---|---|
| Day 138 | Adapter Pattern — interface compatibility shim |
| Day 139 | Decorator Pattern — runtime behavior layering |
| Day 140 | Facade Pattern — simplified interface over complex subsystem |
| Day 141 | Proxy Pattern — access control, lazy init, logging proxy |
| Day 142 | Composite Pattern — tree structures of objects |
| Day 143 | Bridge Pattern — decoupling abstraction from implementation |
| Day 144 | Structural Patterns Review + Machine Coding: Library Management System |

**Heads-up:** Decorator and Proxy are the most-confused pair next week — know the distinction before Day 139.

---

## Week 20 Completion Checklist

**OOP:**
- [ ] Can define all 4 pillars + write a code example for each from memory
- [ ] Can clearly distinguish abstraction vs. encapsulation
- [ ] Can distinguish method overriding from method overloading

**SOLID:**
- [ ] Can state all 5 principles with smell-it-fixes and analogy
- [ ] Can identify which principle a given code smell violates (full table)
- [ ] Can write bad + fixed code for each principle

**Creational Patterns:**
- [ ] Can write thread-safe Singleton (double-checked locking) from memory
- [ ] Can write Factory with registration map
- [ ] Can write PizzaBuilder with fluent chaining and immutability
- [ ] Can design Abstract Factory for two product families
- [ ] Can describe Prototype with clone() use case

**Machine Coding:**
- [ ] Can apply the 4-step method (nouns/verbs/varies/simple) to any new problem
- [ ] Can design Parking Lot entities, methods, patterns, and edge cases without notes

**STAR:**
- [ ] Have at least 3 STAR stories ready: OOP/SOLID, Singleton/Factory, Builder/Parking Lot
- [ ] Each story has specific metrics (lines of code, test coverage %, time saved)
