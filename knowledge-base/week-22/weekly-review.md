# Week 22 Review — Days 145–151
**Phase: LLD Mastery | Month 4**

---

## Phase
LLD Mastery, Month 4. This week completed the full behavioral pattern catalog and synthesized it into two canonical machine-coding problems. The shift from "can I name the patterns" to "can I code them from memory and pick the right one under interview pressure" is the target of this week's work.

---

## Patterns Covered This Week

- **Day 145 — Strategy:** Family of interchangeable algorithms; caller swaps from outside; composition over inheritance
- **Day 146 — Observer:** Subject auto-notifies all observers; push vs pull model; thread-safety snapshot rule; Kafka/SNS as Observer at scale
- **Day 147 — State:** Object transitions its own behavior; eliminates status if-else chains; state machine diagrams for Order and Vending Machine
- **Day 148 — Command:** Request as object; undo/redo via history stack; Invoker/Receiver/ConcreteCommand roles + Template Method: skeleton fixed, subclass fills steps; hook methods
- **Day 149 — Confusion Pairs:** Strategy vs State, Observer vs Mediator, Command vs Strategy, Decorator vs Template Method, Decorator vs Proxy — side-by-side pseudocode + diagnostic tests
- **Day 150 — Remaining GoF:** Chain of Responsibility, Iterator, Mediator, Memento, Visitor, Interpreter, Abstract Factory (deeper), Prototype (deeper)
- **Day 151 — Machine Coding:** Full 12-pattern cheat sheet; Vending Machine (State + Strategy); Splitwise (Strategy + Observer); 6 common LLD interview mistakes

---

## Complete GoF Pattern Reference

### Creational (5)

| Pattern | One-Line Recall |
|---|---|
| Singleton | One instance, global access point |
| Factory Method | Subclass decides which object to create |
| Abstract Factory | Families of related objects, cross-product consistency guaranteed |
| Builder | Step-by-step construction, optional fields, readable DSL |
| Prototype | Clone expensive object instead of rebuilding |

### Structural (7)

| Pattern | One-Line Recall |
|---|---|
| Adapter | Bridge two incompatible interfaces, neither changes |
| Bridge | Decouple abstraction from implementation; vary both independently |
| Composite | Tree of objects; clients treat leaf and composite identically |
| Decorator | Add behavior at runtime via stackable composition wrappers |
| Facade | Single clean entry point hiding complex subsystem |
| Flyweight | Share intrinsic state to support large numbers of fine-grained objects |
| Proxy | Control access to object (lazy load, protection, caching) |

### Behavioral (11)

| Pattern | One-Line Recall |
|---|---|
| Chain of Responsibility | Pass request along handler chain; each handles or passes on |
| Command | Wrap request as object; enables undo/queue/log |
| Interpreter | Grammar as class hierarchy; evaluate by composing expressions |
| Iterator | Walk collection without exposing its internal structure |
| Mediator | All participants communicate through one hub; O(n) vs O(n²) connections |
| Memento | Opaque state snapshot; restore without exposing internals |
| Observer | Subject auto-notifies all registered watchers |
| State | Object transitions its own behavior as lifecycle status changes |
| Strategy | Caller swaps interchangeable algorithm from outside |
| Template Method | Base fixes skeleton; subclasses override specific steps |
| Visitor | Add operation to stable hierarchy without modifying it |

---

## The 5 Mix-Ups — Screenshot-Worthy Table

| Pair | Pattern A | Pattern B | One-Line Tell |
|---|---|---|---|
| Strategy vs State | Caller injects algo externally | Object transitions itself | "Who switches?" — caller = Strategy; self = State |
| Observer vs Mediator | Subject broadcasts to passive watchers | All route through central hub | "Direct participant-to-participant talk?" — No = Mediator |
| Command vs Strategy | Wraps REQUEST (has undo/queue/receiver) | Wraps ALGORITHM choice (no undo) | "Need undo or queue?" → Command. "Need algo swap?" → Strategy |
| Decorator vs Template Method | Runtime composition, stackable | Compile-time inheritance, fixed sequence | "Runtime wrapping?" = Decorator. "Fixed skeleton, vary step?" = Template Method |
| Decorator vs Proxy | Adds behavior (enhancement) | Controls access (protection/lazy-load) | "Adding?" = Decorator. "Controlling?" = Proxy |

---

## Machine Coding Pattern Map

| Problem | Patterns | Key Entities |
|---|---|---|
| Vending Machine | State, Strategy | VendingMachine, IdleState/HasMoneyState/DispensingState, PaymentStrategy |
| Splitwise | Strategy, Observer | User, Expense, EqualSplit/ExactSplit/PercentSplit, BalanceSheet |
| Parking Lot | Strategy (pricing), Factory (vehicle), Singleton (ParkingLot) | ParkingLot, ParkingSpot, Vehicle, PricingStrategy, Ticket |
| Elevator | State (IDLE/MOVING/MAINTENANCE), Strategy (scheduling: SCAN/FCFS) | Elevator, Request, ElevatorState, DispatchStrategy |
| Library Management | Observer (due-date notify), Strategy (fine calculation) | Book, Member, Loan, FineStrategy |
| Hotel Booking | Strategy (pricing/loyalty), Builder (booking builder) | Room, Booking, PricingStrategy, BookingBuilder |
| ATM | State (IDLE/CARD_INSERTED/AUTHENTICATING/DISPENSING), Strategy (auth method) | ATM, ATMState, AuthStrategy |
| Chess | Command (move + undo), Strategy (piece movement) | Board, Piece, MoveCommand, MovementStrategy |

---

## Interview Trigger → Pattern Table (12-Row Cheat Sheet)

| Interviewer Says | Pattern |
|---|---|
| "Only one instance across the system" | Singleton |
| "Create objects without naming the exact class" | Factory Method |
| "Families of related objects, consistent together" | Abstract Factory |
| "Too many constructor parameters, optional fields" | Builder |
| "Clone an expensive object instead of rebuilding" | Prototype |
| "Incompatible interface, can't change either side" | Adapter |
| "Add behavior at runtime, stackable" | Decorator |
| "Hide a complex subsystem behind one clean entry" | Facade |
| "Multiple ways to do the same thing, swap at runtime" | Strategy |
| "Notify many when one changes, pub-sub" | Observer |
| "Object behavior changes as lifecycle status changes" | State |
| "Support undo/redo, queue operations" | Command |

---

## 10-Question Quick-Fire Quiz

Answer without notes. Check answers at the bottom.

1. You have a checkout service that must support credit card, UPI, and wallet payments, with new methods added every quarter. The checkout service must never change when a new method is added. Which pattern?

2. An Order moves through PLACED → SHIPPED → DELIVERED → RETURNED. Adding a RETURN_REQUESTED status currently requires modifying 4 methods. Which pattern eliminates this?

3. Your system needs to notify email, push, and SMS services whenever a product's price drops. Adding a WhatsApp notifier must require zero changes to the product service. Which pattern?

4. You need undo/redo for a text editor. Ctrl+Z must reverse the last 10 operations. Which pattern?

5. A report pipeline has the fixed sequence: fetch data → format → export. Fetch is always the same; format and export vary by report type. Which pattern?

6. In an LLD interview, the interviewer asks about a payment system. You say "Strategy." They ask: "How is that different from State?" What is your one-line answer?

7. You are building a chat system where users send messages to each other. As the system grows, every user needs a direct reference to every other user, creating an O(n²) connection problem. Which pattern fixes this?

8. Your vending machine needs to handle cash, card, and UPI payment. You also need to model the machine's behavior in IDLE, HAS_MONEY, and DISPENSING states. Name both patterns and which part of the design each handles.

9. You want to add a PDF export operation to a document model (Paragraph, Table, Image) without modifying those classes. Which pattern?

10. A new subclass is added: `ExcelReport extends Report`. It uses the same fetch pipeline but only overrides `format()`. What pattern is in use, and what is the method the base class declares final?

**Answers:**
1. Strategy
2. State
3. Observer
4. Command (history stack of Command objects with undo())
5. Template Method
6. "Strategy: external caller injects the algorithm. State: object transitions its own behavior based on internal status."
7. Mediator
8. State handles IDLE/HAS_MONEY/DISPENSING; Strategy handles cash/card/UPI payment
9. Visitor
10. Template Method; the base class declares `generate()` final

---

## Strength / Gap Assessment

| Pattern | Comfort (1–5) | Action |
|---|---|---|
| Strategy | | |
| Observer | | |
| State | | |
| Command | | |
| Template Method | | |
| Chain of Responsibility | | |
| Mediator | | |
| Memento | | |
| Strategy vs State distinction | | |
| Observer vs Mediator distinction | | |
| Vending Machine end-to-end | | |
| Splitwise end-to-end | | |

---

## Problems to Revisit

| Problem | What blocked me | Retry date |
|---|---|---|
| (fill in after each session) | | |

---

## 20 Minutes Before the Round — LLD Exam Prep

**Run through these in order, out loud, without notes:**

1. State the one-line recall for Strategy, Observer, State, Command — 30 seconds
2. Say the Strategy vs State diagnostic test: "Who switches?"
3. Say the Decorator vs Proxy diagnostic test: "Adding vs Controlling"
4. Name the 4 roles in Command: Command interface, ConcreteCommand, Receiver, Invoker
5. Trace the Vending Machine from `insertMoney()` to `dispense()`: Idle → HasMoney → Dispensing → Idle
6. Trace Splitwise `addExpense()`: validate → calculateSplit → updateBalances → notifyMembers
7. Remind yourself: code to interfaces, not concrete classes. No god classes. No forced Singletons.
8. Remind yourself: when stuck, narrate your thinking — "I'm choosing Strategy here because the algorithm needs to be swappable from outside."

**If you have 5 more minutes:** draw the State machine diagram for Vending Machine on paper. No code needed — just the bubbles and arrows.

---

## Week 22 → Week 23 Transition

**Consolidate before moving on:**
- Any pattern with comfort < 3/5 in the table above: schedule a 30-minute targeted re-drill before Week 23
- Code Vending Machine from scratch in a blank editor, timing yourself — target: under 20 minutes
- Code Splitwise `addExpense()` + `BalanceSheet.addDebt()` from memory — target: under 15 minutes
- Review all 50 flashcards from Days 145–151 as one deck
- Week 23 focus: system design deep dives and concurrency patterns in LLD

---

## Week 22 STAR Quick Reference

| Pattern | One-line STAR hook |
|---|---|
| Strategy | "Checkout team added BNPL by writing one class — Strategy pattern meant zero changes to CheckoutService." |
| Observer | "Price-drop notification fan-out decoupled from product service via Observer — added WhatsApp channel in 30 minutes." |
| State | "Order lifecycle refactor: State pattern eliminated 6 scattered if-else blocks, added RETURN_REQUESTED in one class." |
| Command | "Undo/redo feature shipped in 2 days using Command + history stack — zero regression to existing editor flows." |
| Template Method | "Report pipeline deduplication: Template Method eliminated 4 copy-pasted 200-line pipelines, Excel export in 30 minutes." |
| Vending Machine | "Machine coding: State for lifecycle + Strategy for payment — added out-of-stock state as one new class." |
| Splitwise | "Machine coding: Strategy for split types + Observer for balance updates — simplify debts in O(n log n) greedy." |
