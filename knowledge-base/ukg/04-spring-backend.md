# 04 — Spring & Backend

> The backend core. Includes the explicitly-asked **Spring vs Spring Boot**, **annotations**, **REST basics**, plus the microservices/security/resilience material a P5 must own.

---

## 4.1 Spring vs Spring Boot (asked directly)

**Plain English:** **Spring** is the framework (dependency injection + a huge ecosystem). **Spring Boot** is an opinionated layer *on top of* Spring that removes the configuration pain — auto-config, embedded server, starters — so you can run an app in minutes.

| | Spring (Framework) | Spring Boot |
|---|---|---|
| Config | Manual (XML/Java config, wire everything) | **Auto-configuration** (sensible defaults) |
| Server | Deploy a WAR to external Tomcat | **Embedded** server, runs as a `java -jar` |
| Dependencies | Pick + align versions yourself | **Starters** (`spring-boot-starter-web`) bundle + version-align |
| Boilerplate | High | Low |
| Production | DIY | **Actuator** (health/metrics) built in |

🎤 **Say it like this:** *"Spring is the core framework — its heart is dependency injection and the bean container. Spring Boot isn't a different framework; it's an opinionated convenience layer on top. It gives you three things that matter: auto-configuration, so it wires sensible defaults based on what's on the classpath; starter dependencies, so you pull one artifact and get a version-aligned stack; and an embedded server with Actuator, so the app is a self-contained runnable jar with health and metrics out of the box. The mental model: Spring gives you the power, Boot removes the ceremony."*

**Follow-up:** *Q: How does auto-config work?* `@EnableAutoConfiguration` scans `spring.factories`/`AutoConfiguration.imports`; each auto-config class is gated by `@Conditional` (e.g. `@ConditionalOnClass`, `@ConditionalOnMissingBean`) — so a bean is created only if the class is present and you haven't defined your own. That last part is key: **your bean always wins over the default.**

---

## 4.2 IoC / DI & bean lifecycle (asked — "beans, their lifecycle")

**IoC (Inversion of Control)** — you don't `new` your dependencies; the **container** creates and injects them. **DI (Dependency Injection)** is how IoC is achieved — the container "injects" collaborators.

**Why:** decoupling + testability — a class depends on an interface; the container (or a test) supplies the implementation. (This is Dependency Inversion from SOLID, made concrete.)

**A bean** = an object the Spring container manages. Default **scope = singleton** (one per container), also `prototype`, `request`, `session`.

**Bean lifecycle (be able to recite):**
1. Container starts, **scans** for `@Component`/`@Service`/`@Repository`/`@Controller` (and `@Bean` methods in `@Configuration`).
2. **Instantiate** the bean.
3. **Populate dependencies** (inject — constructor/setter/field).
4. **Aware** callbacks (`BeanNameAware`, `ApplicationContextAware`).
5. **`BeanPostProcessor.postProcessBeforeInitialization`**.
6. **`@PostConstruct`** / `InitializingBean.afterPropertiesSet` / custom init.
7. **`postProcessAfterInitialization`** (this is where **AOP proxies** are created — e.g. `@Transactional` wrapping).
8. Bean is **ready / in use**.
9. On shutdown: **`@PreDestroy`** / `DisposableBean.destroy` / custom destroy.

**Injection types:** prefer **constructor injection** (immutable, makes deps explicit, fails fast, easy to test, no `@Autowired` needed on a single ctor) over field injection (hidden deps, can't make final, harder to test).

🎤 **Say it like this:** *"A bean is just an object whose lifecycle the Spring container owns. It's discovered by component scanning, instantiated, then its dependencies are injected, then initialization callbacks like @PostConstruct run, and crucially post-processors wrap it — that's where AOP proxies like @Transactional get applied. It lives as a singleton by default and gets @PreDestroy on shutdown. I default to constructor injection because it makes dependencies explicit and final, fails fast if something's missing, and needs no Spring at all to unit-test — I just call the constructor."*

**Follow-ups:**
- *Q: Why is @Transactional ignored on a self-invoked or private method?* It works via a proxy; a call from within the same bean bypasses the proxy, so the advice never runs. Private methods can't be proxied.
- *Q: Singleton bean with mutable state — problem?* Yes — shared across threads → race conditions. Keep beans stateless, or use thread-safe state / request scope.
- *Q: Two beans of the same type?* Disambiguate with `@Qualifier` or `@Primary`.

---

## 4.3 Spring Boot annotations (asked) — the essential map
- **`@SpringBootApplication`** = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` (the bootstrap trio).
- **Stereotypes:** `@Component` (generic bean), `@Service` (business logic), `@Repository` (data access + translates persistence exceptions), `@Controller`/`@RestController` (web; `@RestController` = `@Controller` + `@ResponseBody`).
- **DI:** `@Autowired`, `@Qualifier`, `@Primary`, `@Value` (inject properties), `@Bean` (in `@Configuration`).
- **Web:** `@RequestMapping`, `@GetMapping/@PostMapping/...`, `@PathVariable`, `@RequestParam`, `@RequestBody`, `@ResponseStatus`, `@RestControllerAdvice` + `@ExceptionHandler` (global error handling).
- **Data:** `@Entity`, `@Transactional`, `@Query`.
- **Config/conditional:** `@Configuration`, `@ConfigurationProperties`, `@Profile`, `@ConditionalOnProperty`.

---

## 4.4 REST API design (asked — "REST API basics")

**REST** = an architectural style: **resources** (nouns) identified by URLs, manipulated via **HTTP verbs**, **stateless** (each request self-contained — no server session), with standard status codes.

**Verbs & safety:** `GET` (read, **safe + idempotent**), `POST` (create, *not* idempotent), `PUT` (replace, idempotent), `PATCH` (partial), `DELETE` (idempotent). *Idempotent* = repeating it gives the same result/state.

**Status codes:** 200 OK, 201 Created (+`Location`), 204 No Content, 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (authenticated, not allowed), 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500 Server Error, 503 Unavailable.

**Design checklist:** plural nouns (`/orders/{id}`), **versioning** (`/v1`), **pagination** (cursor for scale), **filtering/sorting** via query params, consistent **error shape**, **idempotency keys** on POST for safe retries, HATEOAS optional. **REST vs GraphQL:** REST = many fixed endpoints, simple/cacheable; GraphQL = one endpoint, client picks fields (kills over/under-fetching) but caching + rate-limiting are harder. Use GraphQL for rich, varied client data needs; REST for simple resource CRUD.

🎤 **Say it like this:** *"REST treats everything as a resource addressed by a URL and acted on with HTTP verbs, and it's stateless — every request carries its own auth and context, which is what lets me scale horizontally behind a load balancer. The discipline is in the details: right status codes, GET safe and idempotent, versioning so I can evolve without breaking clients, cursor pagination at scale, a consistent error shape, and idempotency keys on POST so a retried create doesn't duplicate. When clients need flexible, nested data I'll consider GraphQL to kill over-fetching, accepting that I lose easy HTTP caching."*

---

## 4.5 Spring Security + OAuth2 / OIDC / JWT
- **AuthN vs AuthZ** — authentication = *who you are*; authorization = *what you're allowed to do*.
- **OAuth2** — a **delegated authorization** framework: a client gets an **access token** to call an API on a user's behalf, without the user's password. Roles: resource owner, client, authorization server, resource server.
- **OIDC (OpenID Connect)** — an **identity** layer on top of OAuth2; adds an **ID token** (a JWT proving *who* the user is). OAuth2 = access; OIDC = login.
- **JWT (JSON Web Token)** — a signed, self-contained token: `header.payload.signature`, base64url. The signature (HMAC or RSA/EC) lets any service verify it **without a DB lookup** — stateless auth. Carries claims (sub, exp, roles).
- **Trade-off:** JWTs are hard to revoke before expiry → keep them short-lived + use refresh tokens; or a denylist for emergencies. **Never put secrets in a JWT** (payload is readable, only signed not encrypted).
- **Spring Security flow:** a filter chain authenticates the request (validates the JWT/session), populates the `SecurityContext`, then method/URL authorization rules (`@PreAuthorize`, `requestMatchers`) gate access.

🎤 **Say it like this:** *"I separate authentication — who you are — from authorization — what you can do. OAuth2 is delegated authorization: a client gets a scoped access token instead of the user's password. OIDC adds identity on top with an ID token, so it's what I use for actual login. JWTs make this stateless — they're signed, so any service verifies them locally without a session lookup, which is huge for microservices. The catch is revocation: a signed token is valid until it expires, so I keep access tokens short-lived with refresh tokens, and I never store anything secret in the payload because it's only signed, not encrypted."*

---

## 4.6 Microservices patterns, service discovery, gateway, resilience

**Service discovery** — services have dynamic IPs (containers come and go). A **registry** (Eureka/Consul) tracks live instances; clients look up by name. **Client-side** (client picks an instance, e.g. Eureka + Ribbon) vs **server-side** (LB/gateway does it). Spring Cloud Gateway routes + cross-cuts.

**API gateway** — single entry: routing, auth, rate limiting, TLS, aggregation. Keeps cross-cutting concerns out of each service.

**Resilience patterns (define each — common follow-up):**
- **Timeout** — never wait forever on a dependency; bound every remote call.
- **Retry** (with **exponential backoff + jitter**) — retry transient failures, spacing attempts out to avoid a retry storm. Only for idempotent ops.
- **Circuit breaker** — after N failures, "open" the circuit and **fail fast** for a cooldown instead of hammering a sick service; "half-open" to test recovery. (Resilience4j.) *Prevents cascading failure.*
- **Bulkhead** — isolate resources (separate thread pools/connection pools per dependency) so one slow dependency can't exhaust everything — like watertight compartments in a ship.
- **Fallback / graceful degradation** — return cached/default/partial data when a dependency is down.

🎤 **Say it like this:** *"In microservices the network is the enemy, so every remote call gets a timeout — waiting forever is how one slow service takes down the whole call graph. On top of that: retries with exponential backoff and jitter for transient blips, but only on idempotent calls; a circuit breaker so after repeated failures I fail fast and give the dependency room to recover instead of hammering it; and bulkheads — separate thread pools per dependency — so one sick downstream can't exhaust my threads and cascade. The goal is graceful degradation: serve cached or partial data rather than fall over."*

**Follow-ups:**
- *Q: Distributed transaction across services?* Avoid 2PC; use the **Saga** pattern — a sequence of local transactions with compensating actions on failure (orchestration or choreography). Embrace eventual consistency.
- *Q: How do services stay consistent on data they both need?* Each service owns its data; share via events (event-driven) + the **outbox pattern** (write event + state in one local tx, relay async) for reliable publishing.
- *Q: `@Transactional` isolation levels?* READ_UNCOMMITTED (dirty reads), READ_COMMITTED (default in PG/SQL Server — no dirty reads), REPEATABLE_READ (no non-repeatable reads), SERIALIZABLE (full isolation, least concurrency). Higher isolation = fewer anomalies, more locking/contention. Anomalies: dirty read, non-repeatable read, phantom read.

---

## ⚠️ Pitfalls & seniority signals
- Saying "Spring Boot is a framework" — it's a *layer on Spring*. Precision matters.
- Not knowing **why** `@Transactional` fails on self-invocation/private methods (proxy mechanics) — a classic senior probe.
- Field injection everywhere — name the constructor-injection trade-offs.
- Treating JWT as encrypted/revocable — it's signed and valid-till-expiry.
- **Seniority signal:** connect DI → testability → SOLID's DIP; explain resilience as *preventing cascading failure*, not just "we add retries"; mention Saga/outbox for distributed data without being asked.

---

*Next topic, or drill deeper on this one?*
