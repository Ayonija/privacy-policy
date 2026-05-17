# Week 23 Review — Days 155–161

## Phase
Phase 4 — Final Sprint | Month 6 (Days 155–160: HLD Component Reviews | Day 161: OA Simulation begins)

## Patterns Covered This Week

### DSA Revision Patterns (Days 155–160)
- **Day 155** — Advanced Graphs (Dijkstra, Bellman-Ford, 0-1 BFS, MST)
- **Day 156** — Heaps & Tries (Top-K, Streaming Median, Prefix Trees)
- **Day 157** — Hashing & Greedy (HashMap Frequency, Interval Scheduling)
- **Day 158** — DP Part 1 (1D DP, 2D DP, LCS, LIS with Patience Sort)
- **Day 159** — DP Part 2 (Interval DP, Game Theory DP, LCS)
- **Day 160** — Backtracking (DFS + Pruning + Memoization)

### OA Contest Pattern (Day 161)
- Sliding Window + Binary Search on Sorted Array

## System Design Topics Covered

| Day | Topic | Key Concept |
|-----|-------|-------------|
| 155 | CAP Theorem in Practice + Distributed Systems Fundamentals | CP vs AP trade-off: CP systems (ZooKeeper) refuse requests during partition; AP systems (Cassandra) serve stale data. Real choice is C vs A since P is non-negotiable. |
| 156 | Rate Limiting Algorithms | Five algorithms: Token Bucket (burst-tolerant), Leaky Bucket (steady rate), Fixed Window Counter (simple, boundary burst risk), Sliding Window Log (accurate, memory-expensive), Sliding Window Counter (hybrid, ~3% error, O(1) memory). |
| 157 | Monitoring, Observability & Alerting (Three Pillars) | Metrics (Prometheus + Grafana) → what is happening. Logs (ELK stack) → what exactly happened. Traces (Jaeger/Zipkin) → which service caused the slowdown. Alert on SLO burn rate, not raw thresholds. |
| 158 | Authentication & Authorization Patterns | JWT (stateless, 15-min access / 30-day refresh), OAuth2 flows (Authorization Code + PKCE, Client Credentials, Device Flow), RBAC vs ABAC, refresh token rotation for theft detection. |
| 159 | Microservices Architecture Patterns | Service Discovery (Consul/Eureka), Circuit Breaker (CLOSED → OPEN → HALF-OPEN), API Gateway vs BFF, Saga Pattern (choreography vs orchestration), Service Mesh (Istio/Linkerd with sidecar proxies). |
| 160 | Full HLD Synthesis — Master Reference | 10-component master reference table: Redis, Kafka, PostgreSQL, Cassandra, CDN, Load Balancer, Elasticsearch, API Gateway, SQS, S3 — each with use-when, key numbers, and top trade-off. |
| 161 | Parking Lot Class Diagram (LLD) | Singleton (ParkingLot), Strategy (FeeCalculator), Factory (VehicleFactory). Concurrent slot reservation via optimistic locking with version column on ParkingSlot row. |

## Problems to Revisit
| Problem | LC # | What blocked me | Retry date |
|---------|------|-----------------|------------|
| (learner fills in after each session) | | | |

## Strength / Gap Assessment
| Pattern | Comfort (1–5) | Action |
|---------|--------------|--------|
| Advanced Graphs (Dijkstra, Bellman-Ford, 0-1 BFS, MST) | | Drill: trigger condition — when to use each algorithm |
| Heaps & Tries (Top-K, Streaming Median, Prefix Trees) | | Drill: two-heap balancing logic from memory |
| Hashing & Greedy (HashMap Frequency, Interval Scheduling) | | Drill: why sort by END not START for interval scheduling |
| DP Part 1 (1D DP, 2D DP, LCS, LIS with Patience Sort) | | Drill: patience sort — tails array semantics |
| DP Part 2 (Interval DP, Game Theory DP) | | Drill: interval DP loop order (length → start → split) |
| Backtracking (DFS + Pruning + Memoization) | | Drill: Word Break II base case ("") and memoization key |
| Sliding Window + Binary Search on Sorted Array | | Drill: deduplication step before sliding window |
| CAP Theorem + Distributed Systems Fundamentals | | Drill: 5 consistency models weakest-to-strongest |
| Rate Limiting Algorithms | | Drill: sliding window counter formula |
| Monitoring, Observability & Alerting | | Drill: SLI vs SLO vs SLA definitions |
| Authentication & Authorization (JWT, OAuth2, RBAC, ABAC) | | Drill: refresh token rotation theft detection logic |
| Microservices Architecture Patterns | | Drill: circuit breaker state transitions and triggers |
| Full HLD Synthesis | | Speed drill: recite all 10 components in under 5 min |
| Parking Lot LLD | | Drill: optimistic locking UPDATE query with version check |

## Weekly Flashcard Deck (35 cards — Days 155–161)

### Day 155 Flashcards (5 cards)
| Q | A |
|---|---|
| When do you use 0-1 BFS instead of Dijkstra? | When edge weights are only 0 or 1; 0-1 BFS runs in O(V+E) vs Dijkstra's O((V+E) log V) — use a deque: cost-0 edges push to front, cost-1 edges push to back |
| What is the key trick in Bellman-Ford for "cheapest flights within K stops"? | Relax exactly K+1 times and copy the distances array before each round (temp = Arrays.copyOf(prices, n)) so you don't chain relaxations within the same round |
| Name the 5 consistency models from weakest to strongest. | Eventual → Monotonic reads → Read-your-writes → Causal → Linearizability |
| What is the difference between CP and AP systems? Give one example of each. | CP (ZooKeeper, HBase): refuse requests during partition to stay consistent. AP (Cassandra, DynamoDB): serve stale data during partition; eventually consistent |
| What is the stale-entry optimization in Dijkstra and why is it necessary? | When popping from the PQ, check if d > dist[u]; if yes, skip — this entry was added before a shorter path was found; without this check you process nodes multiple times unnecessarily |

### Day 156 Flashcards (5 cards)
| Q | A |
|---|---|
| How do you maintain a streaming median with two heaps? | maxHeap (lower half) + minHeap (upper half); always push to maxHeap first, then balance by pushing maxHeap.poll() to minHeap; if minHeap grows larger, push back to maxHeap; median = maxHeap.peek() (odd) or average of both tops (even) |
| What is the key difference between token bucket and leaky bucket? | Token bucket allows bursts up to bucket capacity B while enforcing average rate R. Leaky bucket processes at a constant rate with no burst tolerance — excess requests are dropped immediately |
| What is the boundary burst problem with fixed window counters? | At window boundary (e.g., 11:59:59 → 12:00:00), a user can make full-quota requests at end of old window + full-quota at start of new window, effectively getting 2× the rate limit in a short period |
| How does a Trie differ from a HashMap<String, Boolean> for prefix queries? | Trie supports O(L) prefix lookup (startsWith) natively. HashMap would require iterating all keys or hashing prefixes separately. Trie also shares common prefixes, saving memory for large dictionaries |
| What is the time complexity of The Skyline Problem solution and why? | O(N log N) — sorting events is O(N log N); each event does one TreeMap add/remove which is O(log N); total N events = O(N log N) overall |

### Day 157 Flashcards (5 cards)
| Q | A |
|---|---|
| What is the key insight for the Non-overlapping Intervals greedy solution? | Sort by END time (not start). Always keep the interval with the earliest end — it leaves the most room for future intervals. Count every interval you skip as one removal. This is the Activity Selection Problem. |
| How does Course Schedule III use a max-heap? | Sort by deadline. Maintain a max-heap of chosen course durations and a running total. If adding a course exceeds its deadline, swap it with the longest course in the heap (if current course is shorter) — same count, smaller total duration. |
| What is the difference between SLI, SLO, and SLA? | SLI = actual measured metric (e.g., p99 latency today). SLO = target (e.g., p99 < 200ms). SLA = contract with customer/penalty if violated. |
| Name the three pillars of observability and the primary tool for each. | Metrics → Prometheus + Grafana. Logs → ELK stack (Elasticsearch + Logstash + Kibana). Traces → Jaeger or Zipkin. |
| Why sort by end time (not start time) for the greedy interval scheduling algorithm? | Sorting by end maximizes future flexibility — taking the earliest-ending interval leaves the largest remaining time window for subsequent intervals. Sorting by start does not guarantee this. |

### Day 158 Flashcards (5 cards)
| Q | A |
|---|---|
| Why do you sort equal-width envelopes by height DESC in Russian Doll Envelopes? | So that two envelopes of the same width cannot both appear in the LIS — since heights within the same width are decreasing, the LIS algorithm can only pick at most one per width group |
| What does the "tails" array in patience sort LIS actually represent? | tails[i] is the smallest possible tail element of all increasing subsequences of length i+1 seen so far; it is NOT the actual LIS sequence, just a structure for binary search |
| What is the coin change DP recurrence and its base case? | dp[0] = 0; dp[i] = min(dp[i - coin] + 1) for all coin <= i; initialize dp[1..amount] = amount+1 (sentinel for impossible) |
| What is the difference between RBAC and ABAC? When do you use each? | RBAC: users have roles, roles have permissions — simple, bounded. ABAC: policies on user+resource+environment attributes — flexible, complex. Use RBAC for internal tools; ABAC for multi-tenant SaaS with per-tenant rules |
| How does refresh token rotation detect token theft? | On each refresh, old token is invalidated and a new one is issued. If the old (already-invalidated) token is used again, it signals reuse — the server invalidates the entire token family, forcing re-login |

### Day 159 Flashcards (5 cards)
| Q | A |
|---|---|
| What is the recurrence for Interval DP and what does each variable represent? | dp[i][j] = min/max over k in [i, j-1] of (dp[i][k] + dp[k+1][j] + cost(i,k,j)); i = subarray start, j = end, k = split point; outer loop iterates lengths 2 to n |
| How does Stone Game III model the game theory subproblem? | dp[i] = max net stones current player collects from index i onward, trying taking 1, 2, or 3 stones; dp[i] = max(sum_of_taken - dp[i+t]) for t in {1,2,3}; opponent's optimal play is subtracted |
| Why does Alice always win Divisor Game when n is even? | Even n: take 1 (divides any n) → Bob gets odd. Odd n: all divisors are odd; odd minus odd = even → any move from odd gives the opponent an even number. So even is always a winning position. |
| What are the three states of a circuit breaker and the transition triggers? | CLOSED (normal) → OPEN (N consecutive failures; fail fast). OPEN → HALF-OPEN (after timeout; send one probe). HALF-OPEN → CLOSED (probe succeeds) or OPEN (probe fails). |
| What is the difference between Saga choreography and orchestration? | Choreography: services emit events, others react — decentralized, harder to trace. Orchestration: central saga coordinator sends commands and waits for replies — easier to debug and monitor overall state. |

### Day 160 Flashcards (5 cards)
| Q | A |
|---|---|
| What is the base case for Word Break II memoized DFS and why is it critical? | When start == s.length(), return a list containing one empty string "". This seeds the recursive sentence construction — each caller prepends its matched word to "" (yielding the last word) or to a non-empty suffix sentence |
| Name 3 differences between Kafka and SQS. | Kafka: replay (messages retained 7 days), multiple independent consumer groups, ordering within partition. SQS: simpler/managed, at-least-once delivery, no replay, single consumer per message |
| When should you shard a PostgreSQL table? | When write throughput consistently exceeds ~10K writes/sec after vertical scaling, or when a single table exceeds ~1TB and query plans become expensive |
| What is the difference between L4 and L7 load balancing? | L4 (TCP/UDP): < 0.1ms overhead, no app context, any protocol. L7 (HTTP): ~1ms overhead, enables URL routing, SSL termination, cookie-based stickiness, header inspection |
| How does a CDN presigned URL allow direct client-to-S3 upload? | Server generates a presigned URL with embedded credentials and expiry (15 min). Client POSTs file directly to S3 using the URL. Backend is never in the data path — reduces latency and backend load. |

### Day 161 Flashcards (5 cards)
| Q | A |
|---|---|
| What is the key preprocessing step for LC 2009 and why is deduplication necessary? | Sort the array and remove duplicates into a new array. Deduplication is necessary because duplicate values can never both appear in a valid continuous array of distinct integers — each duplicate is guaranteed to require a replacement operation. |
| How do you detect duplicates in an array in O(n) time and O(1) extra space (LC 442)? | For each element num, go to index abs(num)-1 and negate the value there. If it is already negative, abs(num) is a duplicate. This uses the array itself as a visited hash map via sign flipping. |
| In LC 1498, why do you add pow(2, right-left) for each valid left pointer? | When nums[left]+nums[right]<=target, every subset of the elements between left+1 and right (inclusive) forms a valid subsequence with nums[left] as the minimum. There are 2^(right-left) such subsets. |
| What is the Strategy design pattern and how does FeeCalculator use it? | Strategy encapsulates a family of algorithms behind a common interface and makes them interchangeable at runtime. FeeCalculator is the interface; HourlyFeeCalculator and FlatFeeCalculator are concrete strategies; ParkingLot holds a FeeCalculator reference and calls calculate(ticket) without knowing the pricing algorithm. |
| How does optimistic locking prevent a double-booking in a parking lot without holding a long database lock? | Each ParkingSlot row has a version integer. The UPDATE includes WHERE version=currentVersion. If another transaction committed first, the version has incremented and 0 rows are updated, signaling a conflict. The application then retries with the next available slot — no lock is held during the retry. |

## Mock / Contest Log
| Date | Platform | Score / Result | Key mistake | Fixed? |
|------|----------|---------------|-------------|--------|
| | | | | |

## Week 23 Debrief
- Which OA pattern felt weakest this week?
- Which LLD/HLD component do you need another 30-min review on?
- Log your answers in `knowledge-base/revision-log.md`.
