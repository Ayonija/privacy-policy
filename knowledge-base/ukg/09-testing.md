# 09 — Testing & Quality

> Includes the asked **SonarQube**. Focus: the test pyramid as a philosophy, Java + Angular tooling, and contract testing (the microservices-era senior topic).

---

## The test pyramid (the framing that reads senior)
**Plain English:** Have **many fast unit tests, fewer integration tests, very few end-to-end tests** — because higher up = slower, flakier, more expensive.
- **Unit** (base, most): one class/function in isolation, dependencies mocked. Milliseconds. JUnit + Mockito / Jest + Jasmine.
- **Integration** (middle): components together — service + real DB (Testcontainers), API + WireMock. Seconds.
- **E2E** (tip, fewest): the whole app through the UI/API as a user. Cypress / Selenium. Slow, flaky → keep minimal, cover critical journeys only.

🎤 *"I think in the test pyramid: a broad base of fast unit tests, a thinner layer of integration tests, and a few end-to-end tests for critical journeys. The reasoning is feedback speed and reliability — unit tests run in milliseconds and pin down logic, while end-to-end tests are slow and flaky, so a top-heavy 'ice-cream cone' suite gives slow, unreliable feedback. I optimize for confidence per second of test runtime."*

---

## Java testing tools
- **JUnit 5** — the test framework (`@Test`, `@BeforeEach`, `@ParameterizedTest`, assertions).
- **Mockito** — create **mocks** (fake collaborators) to isolate the unit. `when(repo.find(1)).thenReturn(user)`; `verify(repo).save(any())`. **Mock** (programmed behavior + verify interactions) vs **stub** (just returns canned data) vs **spy** (real object, selectively stubbed).
- **WireMock** — stubs an **external HTTP service** so you can test your HTTP client without the real dependency (returns canned responses, simulates latency/errors).
- **`@SpringBootTest`** — loads the Spring context for integration tests; **`@WebMvcTest`** / **`@DataJpaTest`** — slice tests (just the web or JPA layer, faster).
- **Testcontainers** — spin up a real DB/Kafka in Docker for honest integration tests.

```java
@Test void returnsUserWhenFound() {
  when(repo.findById(1L)).thenReturn(Optional.of(new User(1L,"a@b.com"))); // stub
  var dto = service.get(1L);
  assertEquals("a@b.com", dto.email());
  verify(repo).findById(1L);                                               // interaction check
}
```

---

## Angular / frontend testing
- **Jasmine** (assertion/spec framework) + **Karma** (browser test runner — *legacy; you migrated off it*) → **Jest / Vitest** (faster, jsdom, no browser). *(Your 17→19 migration swapped Karma→Vitest — a great concrete example.)*
- **TestBed** — Angular's utility to configure a testing module + render components.
- **Cypress** — modern E2E: drives a real browser, time-travel debugging, auto-wait. Use for critical user flows.

---

## Contract testing (the microservices senior topic)
**Plain English:** Verify that a **provider** (API) and **consumer** (client) agree on the API shape — **without** spinning up both together in a slow E2E.

**How (Pact / Spring Cloud Contract):** the consumer defines expectations (a "pact"); the provider runs tests against that contract in CI. If the provider changes the API in a breaking way, *its* build fails — you catch integration breakage early, fast, per-service.

🎤 *"In a microservices world, end-to-end tests across services are slow and flaky, so I use consumer-driven contract testing. The consumer publishes a contract describing exactly what it expects from the API; the provider verifies against that contract in its own pipeline. So if a team changes a response in a breaking way, their build fails immediately — I catch integration breakage at build time, per service, instead of in a fragile end-to-end suite or in production."*

---

## SonarQube (asked)
**Plain English:** A **static code analysis** platform — it scans source code (without running it) for **bugs, vulnerabilities, code smells, duplication, and coverage**, and enforces a **quality gate** in CI.
- **Quality gate** = pass/fail rules (e.g. "no new critical issues, coverage on new code ≥ 80%, no new security hotspots"). A failing gate blocks the merge/deploy.
- Sits in CI/CD; complements (doesn't replace) tests — it finds *patterns* (null derefs, injection risks, complexity), tests verify *behavior*.
- Related to **SAST** (Static Application Security Testing — analyze code) vs **SCA** (Software Composition Analysis — scan dependencies for known CVEs, e.g. your Veracode work) vs **DAST** (test the running app).

🎤 *"SonarQube is static analysis in the pipeline — it reads the code without running it and flags bugs, security vulnerabilities, code smells, duplication, and tracks coverage. The mechanism that matters is the quality gate: I set rules like 'no new critical issues and 80% coverage on new code,' and a failing gate blocks the merge. It's a shift-left safety net that makes quality a build property rather than a review afterthought — which pairs well with how I've used hooks to run checks automatically. It complements tests: Sonar catches risky patterns, tests verify behavior."*

---

## ⚠️ Pitfalls & seniority signals
- Inverted pyramid (too many E2E) → slow, flaky CI.
- Testing implementation details / over-mocking → brittle tests that break on refactor. Test behavior, not internals.
- Chasing 100% coverage as a vanity metric — coverage ≠ good tests; assert meaningfully.
- **Seniority signal:** test pyramid reasoning, contract testing for microservices, quality gates in CI, and "I test behavior not implementation" + Testcontainers for honest integration tests.

---

*Next topic, or drill deeper on this one?*
