# Week 21 Review — Structural Patterns
**Week 21 | Phase: LLD Mastery | Month 4**

## Focus
Consolidate all seven structural patterns, lock in the confusion pairs, map patterns to machine coding problems, and self-assess with a 10-question quick-fire quiz. Exit this review ready to identify any structural pattern in under 30 seconds and articulate the design rationale in an interview.

---

## Master Summary Table: All 7 Structural Patterns

| Pattern | Problem Solved | Analogy | One-Line Trigger | Day |
|---|---|---|---|---|
| Adapter | Two incompatible interfaces must work together | Foreign plug + Indian socket | "Integrate library with incompatible interface" | 138 |
| Decorator | Add features dynamically without changing original class | Pizza toppings layered on base | "Optional add-ons in any combination, avoid subclass explosion" | 139 |
| Facade | Complex subsystem needs one simple entry point | Restaurant waiter | "Too many classes to coordinate — give callers one clean API" | 140 |
| Proxy | Control access, lazy-load, cache, or log transparently | Club bouncer | "Control access / lazy-load / cache without changing real object" | 141 |
| Composite | Treat leaf and container identically (tree structure) | Folder-within-folder | "Uniform interface over part-whole hierarchy" | 142 |
| Flyweight | Reduce memory for millions of similar objects | Chess pawns sharing one type | "Millions of similar objects — share intrinsic state" | 142 |
| Bridge | Let abstraction and implementation vary independently | Remote control + TV separate | "Two orthogonal dimensions of variation" | 142 |

---

## 5 Confusion Pairs Reference Card

### 1. Adapter vs Facade

| | Adapter | Facade |
|---|---|---|
| Problem | Incompatibility between two interfaces | Complexity of one subsystem |
| Structure | Wraps one interface to match another | Wraps multiple classes under one API |
| Test | "Do two things not fit each other?" | "Is one thing too hard to use directly?" |

### 2. Decorator vs Proxy

| | Decorator | Proxy |
|---|---|---|
| Intent | Add new capability / behavior | Control access (gate, defer, cache) |
| Core behavior | Changed — augmented output | Unchanged — same result, different path |
| Stacking | Designed to stack | Typically singular |
| Test | "Am I adding capability?" | "Am I adding a gatekeeper?" |

### 3. Adapter vs Decorator

Both wrap an object — the distinction is whether you're changing the interface (Adapter) or extending behavior while keeping the same interface (Decorator).

### 4. Facade vs Proxy

Both provide an intermediary. Facade simplifies many-to-one; Proxy provides one-to-one interception.

### 5. Aggregation vs Composition

| | Aggregation | Composition |
|---|---|---|
| Ownership | Weak — parts exist independently | Strong — parts die with owner |
| UML | Hollow diamond | Filled diamond |
| Test | "Can the part exist without the container?" Yes → Aggregation | No → Composition |

---

## Machine Coding Problems → Pattern Fit

| Machine Coding Problem | Primary Patterns | Day to Review |
|---|---|---|
| Elevator System | State + Strategy + Observer + Facade | 143 |
| Parking Lot | State (spot status) + Factory (vehicle types) + Strategy (pricing) | 144 |
| Coffee/Beverage Ordering | Decorator | 139 |
| Home Theater | Facade | 140 |
| File System | Composite | 142 |
| Chess / Game Engine | Flyweight + State | 142 |
| Image Gallery / Lazy Load | Proxy (Virtual) | 141 |
| Legacy API Integration | Adapter | 138 |
| Library Management | Aggregation/Composition + Factory | 144 |
| ATM System | State + Facade + Strategy | 143 (4-step method) |

---

## LLD Interview Quick Reference

### 5-Step Method
1. Clarify (functional + non-functional requirements)
2. Identify entities → nouns → classes
3. Identify behaviors → verbs → methods
4. Find what varies → apply pattern at variation points only
5. Start simple, extend on interviewer push

### UML Arrows in 30 Seconds
- Solid line = Association (uses-a)
- Hollow diamond = Aggregation (has-a, weak)
- Filled diamond = Composition (has-a, strong)
- Solid + hollow triangle = Inheritance (extends)
- Dashed + hollow triangle = Implementation (implements)

---

## Quick-Fire Quiz: 10 Questions

Answer these cold before checking answers below.

1. You need to integrate a legacy SOAP XML API into a new REST JSON service. Which pattern?
2. A coffee shop app lets customers add milk, sugar, whip in any combination. Design the pricing model. Which pattern?
3. Loading a 500-page PDF — delay loading page previews until the user scrolls to them. Which pattern?
4. Your checkout system calls PaymentService, InventoryService, and ShippingService in a specific order. You want the client to call one method. Which pattern?
5. You have 1 million map tiles, each with the same texture but different X/Y position. Which pattern reduces memory?
6. A file system must let callers call `getSize()` on both files and folders without knowing which they're dealing with. Which pattern?
7. An elevator system needs different dispatch algorithms (SCAN, FCFS, nearest-first) swappable at runtime. Which pattern?
8. Department has Professors. Professors existed before the department and can leave. Which class relationship?
9. House has Rooms. Rooms have no meaning outside a house. Which class relationship?
10. What is the key distinction between Proxy and Decorator?

### Answers

1. **Adapter** — two incompatible interfaces, neither can change
2. **Decorator** — adding behavior (cost + description) in any combination; same `Coffee` interface throughout
3. **Virtual Proxy** — lazy-load expensive object (page preview) on first access
4. **Facade** — one clean entry point (`OrderFacade.placeOrder()`) hiding multi-subsystem orchestration
5. **Flyweight** — share texture (intrinsic state); pass position (extrinsic) at draw time
6. **Composite** — Component interface, File (Leaf), Folder (Composite with children list)
7. **Strategy** — algorithm is the variation point; Strategy makes it swappable
8. **Aggregation** — weak has-a; professors exist independently of the department
9. **Composition** — strong has-a; rooms cannot exist without the house
10. Proxy: same behavior, adds control (gate/defer/cache). Decorator: new behavior, augments capability.

**Score guide:** 10/10 = ready. 8-9/10 = review missed patterns. ≤7 = re-read the failed days before interviews.

---

## Patterns by Category — Exit Mental Model

```
Two incompatible things ──────────────────────► Adapter
One complex thing, simplify surface ──────────► Facade
Add behavior dynamically ─────────────────────► Decorator
Control access / defer / cache ───────────────► Proxy
Tree: uniform leaf + container ───────────────► Composite
Memory: share common state across N objects ──► Flyweight
Two orthogonal variation axes ────────────────► Bridge
```

---

## Next Week Preview — Week 22: Behavioral Patterns

| Day | Topic |
|---|---|
| 145 | Observer Pattern (Event systems, pub-sub) |
| 146 | Strategy + Template Method (Algorithm variation) |
| 147 | State Pattern (State machines in depth) |
| 148 | Command Pattern (Undo/redo, job queues) |
| 149 | Chain of Responsibility + Iterator |
| 150 | Behavioral Confusion Pairs |
| 151 | Machine Coding: Notification System / Snake and Ladder |

---

## Week 21 Checklist
- [ ] Quiz scored — reviewed any pattern with wrong answers
- [ ] 5 confusion pairs stated cold (no notes)
- [ ] Elevator system: 4 patterns identified and state machine drawn from memory
- [ ] UML arrows for all 5 relationship types drawn from memory
- [ ] 5-step LLD interview method stated out loud for a fresh problem
- [ ] Marked any weak areas in `knowledge-base/revision-log.md`
- [ ] Previewed Week 22 topics — confirmed behavioral patterns are next
