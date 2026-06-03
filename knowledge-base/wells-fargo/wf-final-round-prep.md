# Wells Fargo — Final Round Interview Prep
**Role:** R-SWIFT Senior Software Engineer — Full Stack Java + AI/GenAI/Agentic AI  
**Location:** Bengaluru | **Level:** Senior (5+ years)

---

## What the Final Round Tests

Based on the JD and Round 1 signals, the final round will assess across four dimensions:

| Dimension | What They Look For |
|---|---|
| **System Design** | Architect payments-grade distributed systems; microservices, Kafka, Redis, cloud |
| **Technical Depth** | Java/Spring Boot internals, SWIFT/ISO 20022, MongoDB, caching, concurrency |
| **GenAI/Agentic AI** | LLM integration, Agentic workflows, GitHub Copilot, test generation |
| **Leadership** | Driving initiatives, mentoring, cross-team collaboration, cloud migration |

Round 1 tested breadth (Transformer theory, Angular, React, Java generics, SQL). Final round tests **depth + design + domain**.

---

## Part 1 — System Design Questions

### SD1 — Design a Payment Wire Transfer Processing System

**Prompt:** "Design a system that processes international wire/SWIFT payments from initiation to settlement. Handle 50,000 TPS at peak, ensure exactly-once processing, compliance screening, and auditability."

---

#### Clarifying Questions to Ask First
- What is the SLA for payment confirmation? (real-time vs. batch?)
- Do we need idempotency guarantees? (yes — duplicate payment = disaster)
- Compliance requirements? (OFAC screening, AML, fraud checks)
- Settlement: direct to correspondent bank or via SWIFT network?
- What are the read/write ratios for the audit log?

---

#### High-Level Architecture

```
[Client Apps / Channel APIs]
        ↓
[API Gateway] → Auth (JWT), Rate Limiting, TLS termination
        ↓
[Payment Initiation Service] — validates, enriches, assigns UID
        ↓
[Kafka Topic: payment.initiated] ←→ [Schema Registry / ISO 20022 Avro]
        ↓                    ↓                   ↓
[Compliance Service]  [Fraud Engine]    [FX Rate Service]
(OFAC/AML screen)   (ML scoring)       (live rates, Redis cache)
        ↓
[Kafka Topic: payment.approved / payment.rejected]
        ↓
[Payment Execution Service]
  → SWIFT GPI / ISO 20022 MX message formatting
  → Correspondent bank / SWIFT network adapter
        ↓
[Kafka Topic: payment.settled / payment.failed]
        ↓
[Notification Service] → push, email, webhook
[Audit Log Service]    → append-only store (Cassandra/Oracle)
[Reconciliation Service] → T+0 / T+1 batch
```

---

#### Key Design Decisions — Be Ready to Defend Each

**Idempotency (Exactly-Once Processing):**
- Client sends `idempotency-key` (UUID) in header
- Payment Initiation Service checks Redis for key before processing
- If found: return cached response (no duplicate processing)
- If not: process, store result in Redis with TTL=24h, return response
- Kafka producer: use `transactional.id` for exactly-once semantics to topic

**SWIFT / ISO 20022 Integration:**
- Outbound: format pacs.008 (FI-to-FI Credit Transfer) or pacs.009 (Financial Institution Credit Transfer)
- Use SWIFT Alliance Gateway or SWIFT SDK for message transmission
- Message bus decouples internal processing from SWIFT network latency
- Track payment via SWIFT GPI (Global Payments Innovation): end-to-end tracking with `uetr` (unique end-to-end transaction reference)

**Compliance Screening (OFAC/AML):**
- Synchronous OFAC name screening (must block before execution)
- Async ML-based AML scoring for risk scoring (non-blocking, can flag for review)
- Cache OFAC list in Redis (updated daily) — avoids external API call on hot path

**Caching Strategy:**
- FX rates: Redis with 30-second TTL (rates change frequently)
- OFAC screening results for known entities: Redis with 1-hour TTL
- Account validation results: Redis with 5-minute TTL
- Do NOT cache payment state — state is source-of-truth in DB

**Database:**
- Payment state: Oracle (ACID guarantees, existing bank infra, regulatory compliance)
- Audit log: append-only partitioned table, never updated
- SWIFT message archive: separate cold storage (compliance mandates 7+ year retention)
- Config / routing tables: Oracle with Redis cache layer

**Failure Handling:**
- Dead Letter Queue (DLQ) in Kafka for failed messages after 3 retries
- Circuit breaker (Resilience4j) around SWIFT gateway calls
- Saga pattern: if SWIFT transmission fails, compensating transaction to reverse reservation
- Outbox pattern: write payment event to DB and outbox table atomically; Debezium CDC to Kafka

---

#### Scalability Numbers

| Component | Approach |
|---|---|
| API Gateway | Horizontal scaling, rate limit per client ID |
| Kafka | Partition by `payment_id` (ensures ordering per payment) |
| Payment Service | Stateless pods — scale to 50 replicas on PCF/OCP |
| Redis | Redis Cluster, read replicas for FX rate cache |
| Oracle | RAC (Real Application Clusters) for HA; read replicas for reporting |

---

#### Trade-offs to Volunteer

- **Kafka vs. direct DB calls:** Kafka adds latency but gives durability, replay, and decoupling. For payments, durability > latency.
- **Saga vs. 2PC:** Distributed transactions (2PC) are slow and fragile. Saga with compensating transactions is the microservices standard.
- **Sync compliance vs. async:** OFAC must be synchronous (legal requirement). ML fraud scoring can be async (flag for manual review post-execution for low-risk transactions).

---

### SD2 — Design a Caching Layer for a High-Volume Financial API

**Prompt:** "Our payment status API is called 500,000 times/day. 80% of calls are for the same 1,000 recently-completed payments. Design a caching solution."

#### Answer Framework

**Cache-Aside Pattern (most common in Spring Boot):**
```java
@Cacheable(value = "payment-status", key = "#paymentId", 
           unless = "#result.status == 'PENDING'")  // don't cache pending
public PaymentStatus getPaymentStatus(String paymentId) {
    return paymentRepository.findById(paymentId);
}

@CacheEvict(value = "payment-status", key = "#payment.id")
public void updatePaymentStatus(Payment payment) { ... }
```

**Redis Configuration:**
- Key structure: `payment:status:{paymentId}` — namespaced, avoids collisions
- TTL: settled payments = 1 hour; failed = 30 min; pending = NO CACHE (state changes)
- Eviction policy: `allkeys-lru` — evict least-recently-used when memory full
- Redis Cluster for HA: 3 primary + 3 replica nodes

**Cache Coherence:**
- Event-driven invalidation: when payment status changes, Kafka consumer fires `@CacheEvict`
- Prevents stale reads without requiring synchronous DB calls

**Multi-Level Caching:**
- L1: Application-level cache (Caffeine) — in-memory, ~1ms latency, 10K entries, 5-min TTL
- L2: Redis — distributed, ~5ms latency, 1M entries, 1-hour TTL
- L3: Oracle — source of truth

**Cache Stampede Prevention:**
- Use Redis `SET NX PX` (set-if-not-exists with expiry) as a distributed lock
- When cache misses, only ONE thread fetches from DB; others wait for lock release
- Spring `@Cacheable` does NOT prevent stampede — implement with Redisson `RLock`

---

### SD3 — Design an Agentic AI Workflow for Payment Exception Handling

**Prompt:** "Payments sometimes fail or get flagged for compliance review. Design an AI agent workflow that triages these exceptions, determines action, and either auto-resolves or escalates to a human."

#### Architecture

```
[Exception Event on Kafka] → [Agent Orchestrator Service]
                                        ↓
                            [LLM Tool-Use Router]
                           /        |         \
               [SWIFT Lookup]  [Account Check]  [Risk Scorer]
               Tool               Tool            Tool (ML model)
                           \        |         /
                            [LLM Decision Engine]
                                    ↓
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
             [Auto-Resolve]   [Request Docs]   [Escalate Human]
             (retry SWIFT)   (send customer    (ops queue with
                              notification)      AI summary)
```

**Key Components:**

**Agent Orchestrator (LangChain4j / Spring AI):**
- Receives exception event with full payment context
- Calls LLM with tools: `lookup_swift_status`, `check_beneficiary_account`, `get_customer_risk_score`, `check_ofac_status`
- LLM decides which tools to call, in what order, based on exception type
- Returns structured decision: `{action: "RETRY|ESCALATE|REQUEST_DOCS", reason: "...", confidence: 0.92}`

**Tools (function calling):**
- Each tool is a Spring `@Service` method annotated as an LLM tool
- Tools have deterministic outcomes — LLM handles reasoning, tools handle execution
- All tool calls logged for auditability and compliance

**Human-in-the-Loop:**
- Low confidence (<0.7): always escalate to human reviewer
- High-value payments (>$1M): always require human approval regardless of confidence
- Agent generates a structured summary for the reviewer: what it found, what it recommends, why

**LLM Selection:**
- Use Claude or Azure OpenAI (enterprise-grade, data privacy)
- Prompt caching for static context (compliance rules, SWIFT codes) → reduces cost 80%
- System prompt contains SWIFT exception taxonomy, resolution playbooks

---

### SD4 — Design a Notification System for Payment Status Updates

**Prompt:** "After a payment settles (or fails), customers and internal teams need real-time notification via multiple channels: push, email, webhook, internal Slack. Design it."

#### Key Design Points

- **Kafka consumer group per channel** — push consumers scale independently of email consumers
- **Delivery guarantee:** at-least-once with idempotency key (avoid double notifications)
- **Channel abstraction:** `NotificationChannel` interface; add new channels without changing core
- **Template engine:** Thymeleaf/Freemarker for email; parameterized templates stored in DB
- **Webhook retry:** exponential backoff (1s, 2s, 4s, 8s), max 5 retries, then DLQ + alert
- **Solace vs. Kafka:** Solace is better for pub/sub fan-out to many consumers; Kafka better for log-based replay. Wells Fargo uses both — Kafka for internal event streaming, Solace for application messaging.

---

## Part 2 — Java & Spring Boot Deep Questions

### Java Concurrency

**Q: How would you make a payment processing service thread-safe?**
```java
// ConcurrentHashMap for idempotency cache — atomic putIfAbsent
private final ConcurrentHashMap<String, PaymentResult> idempotencyCache = new ConcurrentHashMap<>();

public PaymentResult process(String idempotencyKey, PaymentRequest request) {
    return idempotencyCache.computeIfAbsent(idempotencyKey, key -> executePayment(request));
    // computeIfAbsent is atomic — only ONE thread executes the mapping function
}
```

**Q: Explain `volatile` vs `synchronized` vs `AtomicInteger`.**
- `volatile`: guarantees visibility (reads/writes go to main memory, not CPU cache); does NOT guarantee atomicity of compound operations
- `synchronized`: guarantees both visibility AND atomicity for a block; has overhead
- `AtomicInteger`: lock-free CAS (Compare-And-Swap) operations; best for simple counters

**Q: What is a CompletableFuture and when do you use it in a payments system?**
```java
// Run compliance check and fraud check in parallel, combine results
CompletableFuture<Boolean> ofac = CompletableFuture.supplyAsync(() -> ofacService.check(payment));
CompletableFuture<Double> fraud = CompletableFuture.supplyAsync(() -> fraudService.score(payment));

CompletableFuture.allOf(ofac, fraud)
    .thenApply(v -> ofac.join() && fraud.join() < 0.8)  // both must pass
    .exceptionally(ex -> { log.error(...); return false; });
```

---

### Spring Boot Internals

**Q: How does Spring Boot auto-configuration work?**
- `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
- Spring Boot scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
- Each autoconfiguration class is annotated with `@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.
- If condition met AND no user bean overrides it → Spring Boot creates the default bean
- Example: `DataSourceAutoConfiguration` runs only if JDBC driver is on classpath AND no `DataSource` bean defined

**Q: What is Spring Boot Actuator and what endpoints matter for a payments service?**
- `/actuator/health` — liveness/readiness for k8s / PCF — critical for zero-downtime deploys
- `/actuator/metrics` — Micrometer metrics (payment.processed.count, payment.latency.p99)
- `/actuator/circuitbreakers` — Resilience4j circuit breaker state for SWIFT gateway
- `/actuator/prometheus` — Prometheus scraping endpoint for Grafana dashboards

**Q: How do you implement circuit breaker for SWIFT gateway calls?**
```java
@CircuitBreaker(name = "swift-gateway", fallbackMethod = "swiftFallback")
@Retry(name = "swift-gateway")
@TimeLimiter(name = "swift-gateway")
public CompletableFuture<SwiftResponse> sendSwiftMessage(SwiftMessage msg) {
    return CompletableFuture.supplyAsync(() -> swiftGateway.send(msg));
}

public CompletableFuture<SwiftResponse> swiftFallback(SwiftMessage msg, Exception ex) {
    // Queue for retry, alert ops team
    retryQueue.enqueue(msg);
    return CompletableFuture.completedFuture(SwiftResponse.pending(msg.getUetr()));
}
```

**Resilience4j config:**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      swift-gateway:
        slidingWindowSize: 10
        failureRateThreshold: 50      # open if >50% fail
        waitDurationInOpenState: 30s
        permittedCallsInHalfOpenState: 3
```

---

### Microservices Patterns

**Q: How do you handle distributed transactions across Payment Service, Account Service, and Audit Service?**

**Saga Pattern — Choreography-based:**
```
PaymentService → publishes: PaymentInitiated
AccountService → consumes PaymentInitiated → reserves funds → publishes: FundsReserved
SwiftService   → consumes FundsReserved → sends message → publishes: SwiftSent
AuditService   → consumes all events → writes audit record

On failure:
SwiftService fails → publishes: SwiftFailed
AccountService → consumes SwiftFailed → releases reservation → publishes: FundsReleased
PaymentService → consumes FundsReleased → marks payment FAILED → notifies customer
```

**Why not 2PC (Two-Phase Commit)?**
- 2PC blocks all participants until coordinator confirms — catastrophic if coordinator fails
- In microservices with independent DBs, 2PC requires XA transactions which most cloud DBs don't support
- Saga provides eventual consistency with bounded failure scope

**Q: How do you prevent duplicate event processing in Saga consumers?**
- Each event has a unique `eventId`
- Consumers store processed `eventId` in their DB (idempotency table)
- Before processing: `SELECT * FROM processed_events WHERE event_id = ?`
- If found: skip (already processed)
- If not: process + insert `eventId` atomically in same transaction

---

### Spring Security & JWT

**Q: How do you implement JWT authentication in a payments microservice?**
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, ...) {
        String token = extractToken(req);  // from Authorization: Bearer <token>
        if (token != null && jwtService.isValid(token)) {
            UsernamePasswordAuthenticationToken auth = 
                new UsernamePasswordAuthenticationToken(
                    jwtService.getUsername(token), null, 
                    jwtService.getAuthorities(token));
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(req, res);
    }
}
```

- Validate: signature (HMAC-SHA256 or RS256), expiry (`exp` claim), issuer (`iss` claim)
- For service-to-service: use mutual TLS (mTLS) or short-lived service tokens
- Store secrets in Vault or Azure Key Vault — NEVER in application.properties

---

## Part 3 — SWIFT & ISO 20022 (Domain)

This is what makes this role unique — most candidates won't know this.

### SWIFT Basics

**SWIFT:** Society for Worldwide Interbank Financial Telecommunication — the messaging network banks use to communicate payment instructions globally.

**Key Message Types:**

| Message | Purpose |
|---------|---------|
| **MT103** | Single Customer Credit Transfer — most common wire instruction |
| **MT202** | Financial Institution Transfer — bank-to-bank (no customer info) |
| **MT199/299** | Free-format messages for queries/confirmations |
| **pacs.008** | ISO 20022 equivalent of MT103 (FI-to-FI Credit Transfer) |
| **pacs.009** | ISO 20022 equivalent of MT202 |
| **camt.053** | End-of-day account statement |
| **camt.056** | Payment cancellation request |

### MT103 Structure (Old SWIFT / MT format)
```
{1:F01WELLSFGBGAXXX0000000000}   ← Block 1: Basic Header
{2:I103DEUTDEDBXXXXN}            ← Block 2: Application Header  
{4:
:20:REF123456789                 ← Transaction Reference
:23B:CRED                        ← Bank Operation Code
:32A:230601USD10000,00           ← Value Date, Currency, Amount
:50K:/123456789                  ← Ordering Customer (sender)
WELLS FARGO BANK
:59:/DE89370400440532013000      ← Beneficiary (IBAN)
JOHN DOE
:70:INVOICE 12345               ← Remittance Info
:71A:OUR                        ← Charges (OUR=sender pays all)
-}
```

### ISO 20022 Migration (2025 deadline)

The global banking industry is migrating from MT format to **ISO 20022 MX** (XML-based) format.

**Why ISO 20022 matters:**
- Richer data: full structured remittance info, purpose codes, LEI identifiers
- Better AML/compliance: more data fields = better screening
- Interoperability: consistent schema across all global banks
- Wells Fargo is actively migrating → this is why the role mentions ISO 20022

**pacs.008 (ISO 20022 equivalent of MT103):**
```xml
<FIToFICstmrCdtTrf>
  <GrpHdr>
    <MsgId>WFUS-20230601-001</MsgId>
    <CreDtTm>2023-06-01T10:00:00</CreDtTm>
    <NbOfTxs>1</NbOfTxs>
    <SttlmInf><SttlmMtd>CLRG</SttlmMtd></SttlmInf>
  </GrpHdr>
  <CdtTrfTxInf>
    <PmtId>
      <EndToEndId>E2E-REF-001</EndToEndId>
      <UETR>a1b2c3d4-...</UETR>      ← Unique End-to-End Transaction Reference (SWIFT GPI)
    </PmtId>
    <Amt><InstdAmt Ccy="USD">10000.00</InstdAmt></Amt>
    <Cdtr><Nm>John Doe</Nm></Cdtr>
    <CdtrAcct><Id><IBAN>DE89370400440532013000</IBAN></Id></CdtrAcct>
  </CdtTrfTxInf>
</FIToFICstmrCdtTrf>
```

### SWIFT GPI (Global Payments Innovation)

- Real-time payment tracking end-to-end
- Every payment gets a **UETR** (Unique End-to-End Transaction Reference) — a UUID
- Banks update the SWIFT GPI tracker at each processing step
- Customers can see: "Payment is in transit at Correspondent Bank X"
- This is now MANDATORY for all MT103/pacs.008 messages

**Interview talking point:** "In my experience building payment systems, SWIFT GPI significantly reduces payment investigations. Before GPI, tracing a failed wire required emailing 3-4 correspondent banks manually — days of effort. With GPI and the UETR, status is available in minutes via the gCCT (GPI Credit Confirmation) tracker API."

---

## Part 4 — GenAI & Agentic AI

### GitHub Copilot in a Banking Engineering Team

**Q: How have you used GitHub Copilot in your workflow?**

STAR Answer:
- **Situation:** Team was building boilerplate Spring Boot microservices — lots of repetitive controller/service/repository layers
- **Task:** Reduce time on boilerplate without sacrificing code quality standards
- **Action:** Used Copilot for: generating JUnit test cases from method signatures, scaffolding DTO classes from API contracts, suggesting regex patterns for SWIFT reference validation, generating SQL migration scripts. For all AI suggestions: reviewed, tested, and adjusted before committing.
- **Result:** Estimated 30% reduction in time spent on boilerplate. Code review time actually increased slightly — we invested in reviewing AI output carefully. No AI-generated code went to production without human review.

**Key points to make:**
- Copilot for test generation is high-value — it writes happy-path tests, you write edge cases
- Never trust AI-generated security code without audit (auth, encryption, input validation)
- Use for documentation and javadoc generation — huge time saver
- Copilot Business/Enterprise includes IP indemnification — important for banking

---

### LLM-Based Test Generation

**Q: How would you use LLMs to improve test coverage for a payment processing service?**

```java
// Prompt engineering for test generation:
// "Given this Java method that validates a SWIFT BIC code, 
//  generate JUnit 5 test cases covering: valid BIC formats,
//  invalid length, invalid characters, null input, empty string"

// What LLM generates (starting point, not final):
@ParameterizedTest
@ValueSource(strings = {"WELLSGB2LXXX", "DEUTDEDB", "CHASUSMT"})
void validBicCodes_shouldPass(String bic) {
    assertTrue(SwiftValidator.isValidBic(bic));
}

@ParameterizedTest  
@ValueSource(strings = {"WEL", "WELLS1234567890", "WELL$GB2"})
void invalidBicCodes_shouldFail(String bic) {
    assertFalse(SwiftValidator.isValidBic(bic));
}
```

**Workflow:**
1. Paste method signature + javadoc into LLM prompt
2. LLM generates test skeleton with obvious cases
3. Engineer adds domain-specific edge cases (SWIFT-specific invalid patterns)
4. Run mutation testing (PIT) to verify tests actually catch bugs
5. LLM can also suggest additional tests based on mutation testing gaps

**What LLMs are bad at for test generation:**
- Integration tests requiring complex setup
- Performance/load tests
- Tests that need domain knowledge (SWIFT message format nuances)
- Security tests (injection, auth bypass)

---

### Agentic AI Workflows

**Q: What is an Agentic AI workflow and how does it differ from a simple LLM call?**

| Simple LLM Call | Agentic Workflow |
|---|---|
| Single prompt → single response | Multi-step, tool-calling, iterative |
| Stateless | Maintains state across steps |
| No external actions | Can call APIs, query DBs, send emails |
| Fixed output | Dynamic: decides which tools to use |

**Components of an Agentic System:**
- **LLM (the brain):** Reasons, plans, decides which tool to call next
- **Tools (the hands):** Functions the LLM can invoke — API calls, DB queries, computations
- **Memory:** Short-term (conversation context) + long-term (vector DB retrieval)
- **Orchestrator:** Manages the loop: LLM → tool call → result → LLM → ...

**Practical example: Agentic Payment Exception Handler**
```
User: "Payment REF123 is stuck in compliance review for 2 hours"

Agent loop:
1. LLM: "I'll check the SWIFT status first" → calls tool: get_swift_status(uetr)
   Tool returns: "Message delivered to Deutsche Bank, awaiting processing"
2. LLM: "Message delivered — check beneficiary account" → calls tool: validate_account(iban)
   Tool returns: "Account active, no holds"
3. LLM: "Check for OFAC match" → calls tool: ofac_screen(name="JOHN DOE", country="DE")
   Tool returns: "No match found"
4. LLM: "No issues found. Likely Deutsche Bank processing delay. 
          Recommend: wait 4 more hours then send MT199 inquiry"
   → Generates structured resolution: {action: WAIT, deadline: "2023-06-01T16:00:00", 
      next_step: "Send MT199 if not settled by deadline"}
```

**Tools used in production for agents:**
- LangChain4j (Java) or LangChain (Python) for orchestration
- Spring AI for Spring Boot integration with OpenAI/Azure OpenAI/Anthropic
- Vector databases (Pinecone, Weaviate, pgvector) for long-term memory / RAG
- MCP (Model Context Protocol) for standardized tool definitions

---

### RAG (Retrieval-Augmented Generation) in Banking

**Q: How would you build a RAG system for Wells Fargo's internal SWIFT documentation assistant?**

```
Documents: SWIFT user guides, ISO 20022 specs, internal SOPs (PDFs, Word docs)
          ↓
[Document Loader] → [Text Splitter: 512-token chunks with 50-token overlap]
          ↓
[Embedding Model] (text-embedding-ada-002 or BGE-large)
          ↓
[Vector Store] (pgvector on PostgreSQL, or Pinecone)
          ↓
At query time:
User: "What fields are mandatory in a pacs.008 message?"
          ↓
[Query Embedding] → [Similarity Search: top-k=5 relevant chunks]
          ↓
[Prompt: "Based on these SWIFT docs: {context}, answer: {question}"]
          ↓
[LLM] → Accurate, grounded answer with source citation
```

**Why RAG beats fine-tuning for this:**
- Documents change frequently (SWIFT releases new specs annually)
- Fine-tuning is expensive and requires retraining for each update
- RAG: just re-index new documents → LLM automatically uses updated info
- RAG provides citations → compliance teams can verify answers

---

## Part 5 — Data, Caching & Messaging

### MongoDB for Payments

**Q: How would you model a payment document in MongoDB?**

```javascript
// Document structure for a payment
{
  "_id": "PAY-2023-001",
  "uetr": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "SETTLED",
  "statusHistory": [                           // embedded for fast access
    {"status": "INITIATED", "at": ISODate("2023-06-01T10:00:00Z"), "by": "user@wf.com"},
    {"status": "COMPLIANCE_PASS", "at": ISODate("2023-06-01T10:00:05Z")},
    {"status": "SWIFT_SENT", "at": ISODate("2023-06-01T10:01:00Z")},
    {"status": "SETTLED", "at": ISODate("2023-06-01T14:30:00Z")}
  ],
  "amount": {"value": NumberDecimal("10000.00"), "currency": "USD"},
  "sender": {"accountId": "ACC-001", "name": "ACME Corp", "bankBic": "WELLSGB2"},
  "beneficiary": {"iban": "DE89370400440532013000", "name": "John Doe", "bankBic": "DEUTDEDB"},
  "swiftMessages": [                           // embedded — avoids joins
    {"type": "pacs.008", "sentAt": ISODate("..."), "ref": "REF123"}
  ],
  "complianceFlags": [],
  "createdAt": ISODate("2023-06-01T10:00:00Z"),
  "updatedAt": ISODate("2023-06-01T14:30:00Z")
}
```

**Indexing strategy:**
```javascript
db.payments.createIndex({ "uetr": 1 }, { unique: true })
db.payments.createIndex({ "status": 1, "createdAt": -1 })  // ops dashboard queries
db.payments.createIndex({ "sender.accountId": 1, "createdAt": -1 })  // customer history
db.payments.createIndex({ "beneficiary.iban": 1 })         // beneficiary lookup
```

**Q: When would you choose MongoDB over Oracle for a payments table?**
- MongoDB: payment documents with variable structure (different fields per payment type), high write throughput, horizontal scaling, flexible schema for evolving requirements
- Oracle: audit tables requiring strict ACID (money movement), regulatory reporting requiring complex SQL joins, existing bank infrastructure, compliance mandates
- **Wells Fargo reality:** Oracle for core payment state and audit trail; MongoDB for supplementary data (enrichment, metadata, analytics). Use the right tool per use case.

---

### Redis Caching Patterns

**Q: Explain different Redis data types and their use in a payments system.**

| Data Type | Command | Payments Use Case |
|---|---|---|
| `String` | `GET/SET/SETEX` | Idempotency keys, session tokens, cached API responses |
| `Hash` | `HGET/HSET` | FX rate table `{USD:EUR: 0.92, USD:GBP: 0.79}` |
| `Sorted Set` | `ZADD/ZRANGEBYSCORE` | Priority queue for payment processing; payment SLA timers |
| `List` | `RPUSH/LPOP` | Simple job queue, retry queue |
| `Pub/Sub` | `PUBLISH/SUBSCRIBE` | Real-time payment status updates to UI |
| `Stream` | `XADD/XREAD` | Persistent ordered event log (Kafka-lite) |

**Distributed Lock for Idempotency (Redisson):**
```java
RLock lock = redisson.getLock("payment:" + idempotencyKey);
try {
    if (lock.tryLock(100, 10000, TimeUnit.MILLISECONDS)) {
        // Only one instance processes this payment
        return processPayment(request);
    }
} finally {
    lock.unlock();
}
```

**Cache Invalidation Strategy:**
- Proactive: invalidate on write (`@CacheEvict` on status update)
- Reactive: TTL-based expiry as fallback
- Versioned keys: `payment:v2:{id}` — on schema change, old keys naturally expire

---

### Kafka for Payment Events

**Q: How do you ensure message ordering for a payment's state machine in Kafka?**

- **Partition by payment ID:** `ProducerRecord<String, PaymentEvent>(topic, paymentId, event)`
- All events for the same payment go to the same partition → guaranteed ordering within partition
- Consumer group per service ensures parallel processing while maintaining per-payment order

**Q: What is the difference between Kafka and Solace? When does Wells Fargo use each?**

| Feature | Kafka | Solace |
|---|---|---|
| Model | Log-based (pull) | Message broker (push) |
| Retention | Long-term (days/forever) | Short-term (until consumed) |
| Replay | Yes — consumers can replay from offset | No (once consumed, gone) |
| Throughput | Very high | High |
| Protocol | Kafka binary | AMQP, JMS, MQTT, REST |
| Use at WF | Internal event streaming, audit log | Application messaging, legacy system integration |

**When to choose Solace:**
- Fan-out pub/sub to many subscribers
- Integration with legacy systems (JMS clients)
- Low-latency financial messaging requirements (sub-millisecond)
- WildCard subscriptions (`payments.>` matches all payment subtopics)

**Consumer group lag monitoring:**
```bash
# Check if consumers are falling behind — critical alert in payments
kafka-consumer-groups.sh --describe --group payment-processor \
  --bootstrap-server kafka:9092
# Alert if LAG > 1000 on any partition
```

---

## Part 6 — Cloud & DevOps

### PCF/OCP/Azure/GCP

**Q: What is the difference between PCF and OCP (OpenShift)?**
- **PCF (Pivotal Cloud Foundry):** Platform-as-a-Service; developer pushes app, platform manages containers, routing, scaling. Less control, more abstraction.
- **OCP (OpenShift Container Platform):** Enterprise Kubernetes; developer writes Kubernetes manifests. More control, more responsibility.
- Wells Fargo uses both — PCF for legacy apps (easier migration path), OCP/Kubernetes for new microservices

**Q: How do you implement zero-downtime deployments for a payments service?**

**Rolling Update (Kubernetes):**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%        # at most 25% extra pods during update
    maxUnavailable: 0    # no downtime — always at full capacity
```

**Blue-Green for high-risk releases:**
- Run Blue (current) and Green (new) simultaneously
- Route 5% traffic to Green via API Gateway (canary phase)
- Monitor error rates, latency, business metrics for 30 min
- If healthy: shift 100% to Green; if not: instant rollback to Blue

**Key for payments:**
- Database migrations must be backward-compatible (add columns, don't rename/remove until old version is fully retired)
- Expand-contract pattern: add new column (nullable) → deploy new code that writes both → backfill old rows → make column required → remove old column in later release

---

### CI/CD Pipeline for Payments

**Q: Describe a production-grade CI/CD pipeline for a Java payments microservice.**

```
Developer pushes to feature branch
↓
[GitHub] → triggers GitHub Actions / Jenkins pipeline
↓
[Build Stage]
├── mvn compile
├── mvn test (unit tests, mocked)
├── SonarQube analysis (code quality gates: coverage >80%, no critical vulnerabilities)
├── OWASP Dependency-Check (vulnerable dependency scan)
└── mvn package → Docker image build
↓
[Test Stage]
├── Integration tests (Testcontainers: real Kafka, Redis, Oracle in Docker)
├── Contract tests (Pact: verify consumer-provider API contracts)
└── DAST scan (OWASP ZAP against deployed test environment)
↓
[Artifactory] ← Push Docker image, Maven artifacts
↓
[Deploy to DEV] → smoke tests → auto-promote if pass
↓
[Deploy to SIT] → regression suite → manual approval gate
↓
[Deploy to UAT] → performance tests (Gatling) → stakeholder sign-off
↓
[Deploy to PROD] → Blue-Green deployment → monitoring 30 min → full rollout
```

**SonarQube quality gates for a bank:**
- No new critical/blocker issues
- Code coverage ≥ 80%
- Technical debt ratio ≤ 5%
- No hardcoded credentials (Gitleaks scan)
- OWASP Top 10 security hotspots resolved

---

## Part 7 — Technical Leadership Questions

### Behavioral Questions with STAR Answers

---

**Q: Tell me about a time you led a technically complex initiative.**

**Situation:** Our team was tasked with migrating a legacy MT SWIFT message processing system to ISO 20022 MX format — a multi-year industry mandate affecting all global banks.  
**Task:** As the tech lead, I needed to design the migration architecture, coordinate with 4 downstream teams, and ensure zero payment failures during the transition.  
**Action:**
- Designed a dual-format processing layer that could parse both MT and MX messages — the "co-existence" phase required by SWIFT (banks must handle both formats through the transition period)
- Implemented message transformation using SWIFT's free Translator tool + custom enrichment for fields ISO 20022 adds that MT doesn't have (e.g., LEI, purpose codes)
- Created a contract testing suite (Pact) between all producer/consumer microservices
- Ran daily translation accuracy reports comparing MT → MX conversion output against expected schema
- Presented weekly progress to the CTO and compliance team with traffic light status  
**Result:** Completed Phase 1 (outbound MX messages) ahead of the SWIFT deadline. Zero production incidents during cutover. The dual-format layer I designed became the template for two other teams' migrations.

---

**Q: How do you handle disagreements on technical approach with senior stakeholders?**

**Situation:** An architect proposed using a single Oracle stored procedure to handle the entire payment validation + compliance check + SWIFT routing logic — arguing it was simpler and fewer moving parts.  
**Task:** I disagreed because this approach would make unit testing impossible, couldn't scale independently, and violated the separation of concerns we'd need for future compliance rule changes.  
**Action:**
- Prepared a structured comparison: stored procedure approach vs. microservices approach — performance, testability, deployability, compliance change velocity
- Invited the architect to a whiteboard session; asked questions rather than stating positions: "How do we unit test OFAC logic if it's embedded in a stored proc?" "If compliance rules change next month, what's the deployment impact?"
- Proposed a compromise: the orchestration logic in Java (testable, scalable) with the DB storing routing rules and configuration (the architect's valid concern about configuration centralization)
**Result:** Architect adopted the hybrid approach. More importantly, the process strengthened the relationship — disagreement handled professionally led to a better solution than either of us had individually.

---

**Q: How do you mentor junior engineers?**

**Situation:** Onboarded a junior engineer who was strong in Python but new to Java enterprise patterns and payments domain.  
**Task:** Get them productive on the payments codebase within 6 weeks while maintaining team velocity.  
**Action:**
- Week 1-2: Paired programming on small bug fixes — they drove, I navigated, explaining patterns as we went
- Week 3-4: Assigned a self-contained feature (add a new notification channel) with clear acceptance criteria; daily 15-min check-in to unblock without doing the work for them
- Gave specific, actionable code review feedback: not "this is wrong" but "this HashMap won't work under concurrent load because of X — consider ConcurrentHashMap or why do you think it won't be concurrent here?"
- Connected them with the domain: walked through a real wire transfer from initiation to settlement using our own monitoring tools
**Result:** They delivered the notification feature independently in week 5. Six months later, they were leading their own feature stream. Best investment of 10 minutes/day I've made.

---

**Q: Tell me about a production incident you led the resolution of.**

**Situation:** Payments started failing at 2 AM with "SWIFT gateway timeout" errors — about 200 payments queued, unable to send.  
**Task:** As on-call lead, diagnose and resolve within our 15-minute P1 SLA.  
**Action:**
- Checked Grafana dashboard: SWIFT gateway circuit breaker had opened — 100% failure rate for 5 min
- Checked SWIFT Alliance Gateway logs: TLS certificate had expired at midnight (had been renewed in UAT but deployment to PROD was missed in the change freeze)
- Immediate fix: emergency cert rotation using runbook (pre-approved for P1 incidents, no change approval needed)
- Circuit breaker reset automatically after SWIFT gateway recovered
- Queued payments replayed from Kafka DLQ — all processed successfully within 20 min of initial alert
- Post-incident: added certificate expiry monitoring alert (14 days / 7 days / 1 day before expiry), automated cert rotation using Vault PKI  
**Result:** 200 payments delayed ~45 min, no financial loss, root cause eliminated. The cert monitoring was added to the standard runbook for all teams.

---

**Q: How do you approach cloud migration of a legacy application?**

Framework I follow:

1. **Assess:** Inventory dependencies (DB, messaging, external APIs), classify by migration complexity (R's: Rehost, Replatform, Refactor, Retire)
2. **Strangler Fig pattern:** Don't big-bang migrate. Route new traffic to cloud-native service while legacy handles existing. Gradually shift.
3. **Data migration:** Schema conversion (Oracle → Aurora if applicable), data sync with CDC (Debezium), parallel run with validation
4. **Decouple first, migrate second:** Extract tight coupling (direct DB calls between services) BEFORE moving to cloud. Otherwise you're lifting-and-shifting the coupling.
5. **Observability first:** Deploy monitoring (Prometheus/Grafana, distributed tracing with Jaeger) before go-live — you need visibility on day 1 in the new environment.

**STAR Answer:**
- **Situation:** Legacy payment reconciliation job: monolithic Java application running on on-premises server, no CI/CD, manual deployment, no monitoring
- **Task:** Migrate to Azure (AKS) with zero disruption to daily reconciliation runs
- **Action:** Containerized (Dockerfile), extracted config to ConfigMaps/Secrets, added health endpoints, deployed to AKS with Helm chart. Ran parallel for 4 weeks — compared on-prem output vs. AKS output daily. Gradually shifted to AKS as primary.
- **Result:** Successfully migrated. Added auto-scaling: job now completes 40% faster during month-end peaks by scaling to 10 pods. Infra cost reduced 30% (pay-per-use vs. always-on server).

---

## Part 8 — Questions to Ask Your Interviewer

These signal strategic thinking and genuine interest:

**On Technology:**
- "Where is the team currently in the MT-to-ISO 20022 migration journey? What's the biggest technical challenge remaining?"
- "What does the current Kafka / Solace topology look like — is the team working toward consolidation?"
- "How is GenAI being evaluated for use in the payments processing pipeline? Is there a formal framework for responsible AI use in production?"

**On Team & Process:**
- "What does the sprint ceremony look like? How many weeks per sprint, and how are priorities balanced between feature work and technical debt?"
- "What does the escalation path look like when a junior engineer is blocked on something domain-specific, like a SWIFT message format question?"

**On Growth:**
- "What does a successful first 90 days look like for someone in this role?"
- "Is there a path toward architecture ownership — contributing to the tech roadmap beyond delivery?"

---

## Quick Reference: Key Technology Checklist

### Before the Interview, Be Ready to Discuss:

- [ ] **Java:** Thread safety, CompletableFuture, Stream API, GC tuning basics
- [ ] **Spring Boot:** Auto-configuration, Actuator, Security (JWT), Circuit Breaker (Resilience4j)
- [ ] **Microservices:** Saga pattern, Outbox pattern, Service Mesh (Istio basics)
- [ ] **SWIFT/ISO 20022:** MT103 vs pacs.008, UETR, SWIFT GPI, message structure
- [ ] **Kafka:** Partitioning, consumer groups, exactly-once semantics, DLQ
- [ ] **Redis:** Data types, caching patterns, distributed locking, eviction policies
- [ ] **MongoDB:** Document modeling, indexing strategies, aggregation pipeline
- [ ] **Angular/React:** Round 1 topics — ViewEncapsulation, HttpClient, Fragments
- [ ] **Python:** GIL, threading vs multiprocessing, async (from Round 1)
- [ ] **SQL:** Window functions, CTEs, query optimization, EXPLAIN
- [ ] **GenAI:** RAG, Agentic workflows, tool calling, LangChain4j / Spring AI
- [ ] **Cloud:** AKS/PCF basics, CI/CD pipeline, zero-downtime deployment
- [ ] **Leadership:** 3 STAR stories ready — complex initiative, conflict resolution, mentoring

### Three Things to Emphasize Throughout

1. **Payments domain seriousness:** Money movement requires exactly-once guarantees, auditability, and compliance. Every design decision should acknowledge this.
2. **GenAI pragmatism:** You use AI tools to amplify productivity (test gen, code review, documentation), but apply human judgment to security, compliance, and correctness.
3. **Engineering maturity:** You think about observability, failure modes, and operational concerns before writing the happy path.
