# Month 6 Review — Phase 4: Final Sprint & Real Applications

## Phase

**Phase 4 — Final Sprint & Real Applications**

**Goal:** Enter real interviews. Full mock sessions daily, targeted gap closure, behavioral story bank finalized, all 11 system designs deliverable cold in 45 minutes.

**Context:** Phase 4 spans Slots 16–18 (Days 151–180). This is the capstone month — execution over learning.

- **Slot 16 (Days 151–160):** Full DSA pattern revision across all 18 patterns / HLD component deep dives (Load Balancing, Caching, DB Design, Kafka, CAP, Rate Limiting, Monitoring, Auth, Microservices, HLD Synthesis)
- **Slot 17 (Days 161–170):** Timed OA simulation + contest-style problems / LLD design for 8 systems (Library, BookMyShow, Hotel Booking, ATM, LRU/LFU, Social Media Feed, REST API, Elevator/E-commerce)
- **Slot 18 (Days 171–180):** Intensive mock + weak area drill / End-to-end SD mocks (Zoom, Spotify, Google Maps, Amazon, Instagram, WhatsApp, Google Drive, Airbnb) / 6-month capstone synthesis

---

## DSA Patterns Mastered (comfort ≥ 4/5)

Rate yourself on each pattern. Mark those you can cold-solve in under 25 minutes. This is the complete master list across all 6 months.

### Slot 16 Revision — Core Patterns (Days 151–160)

- [ ] Two Pointers — sort + shrink; 3Sum deduplication; First Missing Positive index placement
- [ ] Sliding Window — variable window; character frequency diff (`nonZero` counter); deduplication before sliding
- [ ] Prefix Sum / Suffix Product — output array as accumulator; O(1) extra space; modular prefix for divisibility
- [ ] Linked List — iterative K-group reversal; fast/slow pointer (Nth from end gap = n+1); cycle detection
- [ ] Monotonic Stack — next greater element (decreasing stack); largest rectangle (increasing stack with sentinel); daily temperatures
- [ ] Trees DFS — post-order arm pattern (return single arm, update global max with left+right); LCA null propagation; two-pass DFS (precompute subtree height, propagate complement downward)
- [ ] Trees BFS — level-order with size snapshot; deepest leaves sum; second minimum BFS with two distance arrays
- [ ] Graphs BFS/DFS — reverse multi-source BFS (Pacific Atlantic); multi-source simultaneous expansion (rotting oranges, shortest bridge); BFS with implicit/dynamic adjacency + group deletion (Jump Game IV)
- [ ] Union-Find — path compression + union by rank; cycle detection (order.size() != n in Kahn's); O(α(n)) amortized
- [ ] Dijkstra — stale-entry skip (d > dist[u]); Dijkstra on grid (min-heap, Math.max cost aggregator for LC 778); 0-1 BFS deque variant for {0,1} edge weights
- [ ] Topological Sort — Kahn's BFS (cycle detected when order.size() < n); DFS post-order; Bellman-Ford K-step copy-before-relax trick
- [ ] SCC — Kosaraju (two-pass DFS + transpose graph); Tarjan (disc/low/stack); bridges and articulation points
- [ ] Heaps — top-K (min-heap of size k); streaming median (two heaps: maxHeap lower half, minHeap upper half); course schedule III (max-heap swap for deadline); Skyline Problem (TreeMap / maxHeap by endpoint)
- [ ] Tries — isEnd flag for root replacement; passCount for prefix score aggregation; reversed-word Trie for suffix encoding; Trie + grid backtracking (Word Search II)
- [ ] Hashing — frequency map patterns; anagram/permutation check via nonZero counter; first missing positive index negation trick; modular prefix for continuous subarray sum
- [ ] Greedy — interval scheduling (sort by END, Activity Selection); Non-overlapping Intervals; queue reconstruction by height (sort DESC + insert at k); greedy min-heap (furthest building / ladders)
- [ ] DP 1D/2D — coin change 0/1 knapsack (inner loop high→low); LCS standard recurrence; LIS patience sort (tails array semantics); Russian Doll Envelopes (sort by width ASC, height DESC)
- [ ] DP Interval — recurrence `dp[i][j] = min/max over k in [i,j-1] of (dp[i][k] + dp[k+1][j] + cost)`; loop order: length → start → split; Burst Balloons last-balloon trick
- [ ] DP Bitmask — TSP-style subset states; Bitmask DP on assignment; shortest path visiting all nodes
- [ ] DP State Machine — circular structure (Freedom Trail); stock with cooldown; buy/sell with transaction limit
- [ ] Backtracking — N-Queens three-set O(1) constraint; Sudoku DFS with row/col/box sets; Trie-augmented grid search (Word Search II, shared DFS + Trie pruning + delete found word)

### Slot 17 OA Contest Patterns (Days 161–170)

- [ ] Sweep Line + Binary Search on Event Arrays — count active intervals at query time; upperBound starts vs lowerBound ends
- [ ] Heap Enumeration of Ranked Sums — seed heap with (arr1[i] + arr2[0]); extend vs swap push to enumerate k smallest pairs
- [ ] Prefix/Suffix DP with Heap-Maintained Running Optimum — max-heap for prefix (evict largest), min-heap for suffix (evict smallest); valid split range [n-1, 2n-1]
- [ ] Binary Search on Answer Value (Feasibility Check) — lo=max(weights), hi=sum(weights) for ship capacity; lo=1, hi=max(piles) for Koko; binary search on k + canAssign check
- [ ] Sliding Window on Positions (Median Cost) — collect positions array, prefix sum, offset term to subtract positional spread
- [ ] Two-Pass DFS (Precompute + Propagate) — pass 1: depth + subtree height; pass 2: propagate max height without current subtree; track top-2 child heights
- [ ] Offline Queries + Min-Heap — sort queries; sweep with interval pointer; expire stale entries; O(n log n) total
- [ ] Game Theory DP — Stone Game (dp[i] = net stones from index i; subtract opponent's dp); Divisor Game (even = winning position)
- [ ] Patience Sort / LIS — tails array, binary search for lower_bound replacement; Russian Doll Envelopes (sort width ASC, height DESC to prevent same-width grouping)

### Slot 18 Weak Area Drill (Days 171–180)

- [ ] Median of Two Sorted Arrays — Binary Search on Partition: ensure nums1 shorter; partitionX in [0,m]; partitionY = (m+n+1)/2 - partitionX; check maxLeftX≤minRightY && maxLeftY≤minRightX; O(log(min(m,n)))
- [ ] Dijkstra on Grid — dist[r][c] = min max-elevation; Math.max cost aggregator for LC 778; min-heap replaces BFS queue; stale-entry skip (d > dist[r][c])
- [ ] Binary Lifting (LCA / k-th Ancestor) — up[n][LOG]; up[node][0]=parent; up[node][j]=up[up[node][j-1]][j-1]; query: decompose k in binary, jump through set bits; O(n log n) precompute, O(log n) per query
- [ ] DP String Parsing — dp[i] = ways for s[0..i-1]; iterate j backward from i-1; prune on leading zero (break) or value > k (break); at most log10(k) chars per valid segment; O(n log k)
- [ ] Backtracking with Aggressive Pruning — topmost-leftmost uncovered cell first (canonical search order); try largest square first (fast upper bound); prune when count >= result; skip duplicate bucket sums in partition problems
- [ ] Merge Sort + BIT (Count Inversions / Smaller After Self) — track (value, originalIndex) pairs; during merge, count right-half elements placed before each left-half element; result[pair[i][1]] += (j - mid)
- [ ] Trie passCount — passCount at each node = words sharing that prefix; sum passCount along word's path for prefix score; O(total_chars) build, O(L) per query
- [ ] BFS with Implicit/Dynamic Adjacency + Group Deletion — value→indices map; process neighbors i-1, i+1, and all same-value indices; DELETE group from map after first processing to prevent O(n²) re-enqueue
- [ ] Monotonic Stack — Largest Rectangle: increasing stack of indices; sentinel 0-height at end; width = i-(stack.empty?-1:peek())-1; Decode String: stack of (repeatCount, builtString); Remove K Adjacent Duplicates: stack of (char, count)

---

## Patterns Needing Drill

List patterns where your comfort is ≤ 3 after self-assessment. Schedule a targeted 90-minute session per pattern before each real interview.

| Pattern | Comfort (1–5) | Target retry date | Notes |
|---------|--------------|------------------|-------|
| (learner fills in) | | | |
| | | | |
| | | | |
| | | | |

---

## System Designs Mastered

### All 11 End-to-End Systems

Deliver each of the following cold in 45 minutes: requirements → scale → architecture → deep dive → trade-offs. The numbers below are the anchor facts — memorize them.

---

#### 1. Twitter / X (Month 5, Slot 13)

**Architecture:** API Gateway → Tweet Service → Fanout Service → Redis sorted set per user for timeline → Cassandra (tweet store). Hybrid fanout: push for users with < 1M followers (fan-out on write to all follower feeds), pull for celebrities (fan-out on read, merge at query time). Tweet IDs: Snowflake (globally unique, time-sortable, no central coordinator). Search: Elasticsearch with real-time index. Trending: `ZINCRBY` sliding window in Redis. Rate limiting: token bucket per user_id.

**Key numbers:** 300M DAU, 500M tweets/day, 5K write QPS, 28K average / 100K peak read QPS. Read:write = 20:1. Avg 100 followers → 50K fanout writes/sec. Timeline stored in Redis sorted set; TTL for inactive users.

---

#### 2. Uber / Lyft (Month 5, Slot 14)

**Architecture:** Location Service (driver heartbeat 1/sec → Redis Geo ZADD) → Matching Service (geohash radius search → scoring → ETA from routing graph) → Ride Service (state machine: REQUESTED → ACCEPTED → ARRIVED → IN_RIDE → COMPLETED). WebSocket for real-time driver location to rider. Surge pricing: supply/demand ratio per geohash cell. ETA: precomputed road graph + Dijkstra; Kafka events for state transitions; driver location stream → Kafka → Geo update worker.

**Key numbers:** 5M trips/day, 1M active drivers, driver location = 1M writes/sec, matching latency target < 5s, ETA accuracy ±30s, geohash precision 6 ≈ 1.2km cells. WebSocket: 1 long-lived connection per driver + per active rider.

---

#### 3. YouTube / Netflix (Month 5, Slot 15)

**Architecture:** Upload → chunked S3 + async Kafka → transcode workers (FFmpeg, 15-20 renditions: 360p/480p/720p/1080p/4K) → CDN origin. Streaming → signed CDN URL → HLS/DASH segments from edge POP → ABR algorithm on client switches quality based on bandwidth. Discovery: Elasticsearch for search + two-tower neural network for recommendations (user vector × item vector → dot product) with Faiss ANN lookup. Comments: Cassandra (partition: video_id). Likes: Kafka → Redis INCR → Cassandra batch flush. Auth: JWT 15min + Redis refresh tokens.

**Key numbers:** 2B YouTube users, 500 hrs uploaded/min, 1B hrs watched/day. Netflix: 250M subscribers, 17,000+ OCAs (Open Connect Appliances embedded in ISP data centers), 95%+ CDN hit rate, recommendations drive 75-80% of watch time, TTFF < 2s, 15-20 transcode renditions per video.

---

#### 4. Instagram (Slot 18, Day 171)

**Architecture:** Upload Service → S3 + SQS → async thumbnail worker. Post Service: PostgreSQL for metadata (post_id, user_id, s3_url, caption, like_count). Feed Service: hybrid fan-out — push (write to follower feed tables) for users < 1M followers; pull (query latest posts at read time) for celebrities > 1M followers; merge at query time. Story Service: Cassandra with TTL=86400s per row (natively expired — no cron job). Explore: CNN image embeddings + collaborative filtering → ANN search.

**Key numbers:** 1B users, 100M DAU, 100M posts/day, 500M stories/day, 10M photos/day × 500KB avg = 5TB/day. 99% of feed reads served from Redis pre-computed cache. CDN offloads 95%+ of photo serving. Stories: ~5,800 Cassandra writes/sec.

---

#### 5. WhatsApp (Slot 18, Day 172)

**Architecture:** Connection Service → consistent hashing assigns each user_id to a Chat Server → each client holds a persistent WebSocket. Message flow: Sender → Chat Server A → Kafka → Chat Server B (if recipient online, push immediately) → Recipient. Offline delivery: Chat Server B stores to Cassandra (partition: conversation_id, cluster: message_id DESC); delivers on reconnect. Group messages: Kafka topic per group; Chat Server fan-out (not per-message DB write). Media: S3 presigned upload, share URL in message, thumbnail async. Presence: Redis key user:{id}:online TTL=30s; heartbeat every 15s; pub/sub for subscribed friends.

**Key numbers:** 2B users, 100B messages/day ≈ 1.15M messages/sec, 10TB/day text storage. Each Chat Server handles ~50K concurrent WebSocket connections → 2B / 50K = ~40,000 Chat Servers. E2EE via Signal Protocol (Diffie-Hellman ratchet, new key per message). Offline presence lag: up to 30s.

---

#### 6. Google Drive (Slot 18, Day 173)

**Architecture:** Upload Service → client splits file into 4MB chunks → SHA-256 hash per chunk → upload only new chunks (deduplication + delta sync) → presigned S3 URL per chunk. Metadata Service: PostgreSQL — Files(file_id, owner_id, name, size, version, s3_path), FileChunks(file_id, chunk_index, chunk_hash, s3_chunk_url). Sync Service: long-polling or WebSocket → server pushes change notification on file update → client downloads only changed chunks. Sharing: FilePermissions(file_id, user_id, role: VIEWER/EDITOR/OWNER) + shareable link with signed token + expiry. Versioning: new version entry per save; restoration updates metadata pointer; chunks GC'd only when all versions deleted. Conflict: LWW (simple, conflict copy for Dropbox-style) or OT (Google Docs collaborative real-time).

**Key numbers:** 1B users (Google), 700M files/day, 700TB/day avg. Chunk deduplication saves ~30% storage. Delta sync: 100MB file with 1MB change uploads 1 chunk (1MB) instead of 100MB. CDN for frequently accessed; S3 long-tail.

---

#### 7. Amazon E-commerce (Slot 18, Day 174)

**Architecture:** Product Catalog: PostgreSQL structured data + Elasticsearch full-text + DynamoDB real-time inventory. Search: query → Elasticsearch (BM25 recall) → ML CTR ranking model (precision) → results in <100ms. Cart: Redis per-user hash (TTL) → persisted to DynamoDB on checkout. Checkout: atomic `UPDATE Products SET inventory = inventory - qty WHERE product_id=? AND inventory >= qty` — 0 rows affected = out of stock; Redis DECRBY + WATCH/MULTI/EXEC for extreme scale pre-gate. Order Saga: ORDER_PLACED → Payment Service → Inventory Service → Fulfillment Service via Kafka; compensating transactions on failure (refund on inventory failure). Recommendations: collaborative filtering + "customers also bought" graph; Spark nightly batch → DynamoDB with TTL refresh.

**Key numbers:** 300M products, 1.5M orders/day normal, ×10 on Prime Day (15M/day). Two-stage search (recall + precision) under 100ms. Flash sale: Redis DECRBY pre-gate before SQL. Circuit breakers on all downstream services during Prime Day. Pre-warm caches before peak events.

---

#### 8. Airbnb (Slot 18, Day 175)

**Architecture:** Listing Service: PostgreSQL + Elasticsearch (geo-bounding-box + filters) + S3 photos + CDN delivery. Search: Elasticsearch geo query → ML ranking (reviews, price, response rate) → availability join from Cassandra via Kafka consumer (propagation lag ~2s — Elasticsearch is not source of truth). Availability: Cassandra (listing_id, date) composite key, status: AVAILABLE / HELD / BOOKED. Booking: HOLD via `INSERT IF NOT EXISTS` CAS (Paxos) → PAYMENT via Stripe (idempotency key = booking_id) → CONFIRM + host notification. Reviews: PostgreSQL; avg_rating updated via Kafka consumer. Dynamic Pricing: ML model on supply/demand/seasonality/events → host sets floor/ceiling.

**Key numbers:** 7M+ listings, 2M+ guests/night, 150+ countries. Availability writes: 6M Cassandra writes/day (trivial). Search latency: Elasticsearch geo < 50ms + Cassandra availability < 10ms + ML ranking < 20ms = < 100ms total. Double-booking prevention: Cassandra lightweight transactions (Paxos, applied=false on conflict).

---

#### 9. Zoom (Slot 18, Day 176)

**Architecture:** Signaling Server: WebSocket-based for session setup, participant events, ICE candidate exchange (WebRTC NAT traversal). Media: SFU (Selective Forwarding Unit) — each participant uploads 1 video stream to SFU; SFU selects which simulcast layer (360p/720p/1080p) to forward to each subscriber based on their bandwidth. SFU vs MCU: MCU mixes all streams server-side (1 composite stream per participant, high CPU); SFU just forwards (lower server cost, higher client bandwidth). Recording: SFU raw streams → Kafka → FFmpeg mixer → S3 (MP4) + Whisper ASR transcript. Screen share: separate high-res/low-fps track (1080p @ 5fps vs camera 720p @ 30fps). Jitter buffer: 60-200ms absorbs network jitter.

**Key numbers:** 300M daily participants, 3.3T minutes in 2020, 10M concurrent peak meetings. P2P for 4 people = 12 total streams; SFU = 4 upload + fan-out (1 upload per person vs n-1 uploads in P2P). Audio priority: 50Kbps vs video 500Kbps-2Mbps — drop video quality first under bandwidth constraint. Zoom uses Erlang/BEAM via Ejabberd for millions of concurrent lightweight processes.

---

#### 10. Spotify (Slot 18, Day 177)

**Architecture:** Music Catalog: PostgreSQL metadata + S3 audio files (96/160/320kbps) + CDN delivery. Streaming: HLS — audio segmented into 10s chunks; client buffers 3 segments ahead; presigned S3 URL per chunk (1h expiry); adaptive bitrate based on bandwidth. Playlist Service: Cassandra for user playlists (user_id, playlist_id, track_list); PostgreSQL for editorial. Recommendation: matrix factorization (NMF) via weekly Spark batch on billions of listening events → user and item vectors → ANN search via Annoy library → Discover Weekly 30 songs. Audio analysis: CNN on spectrogram → features (tempo, key, energy, danceability). Offline: encrypted blobs on device + DRM key tied to account, validated every 30 days. Podcast: separate RSS ingestion pipeline + same CDN delivery.

**Key numbers:** 600M users, 100M songs, 80M DAU, 30B streams/day ÷ 86400 ≈ 347K streams/sec. Catalog storage: 100M songs × 8MB avg = 800TB. CDN offloads 99%+ of audio. Discover Weekly: pre-computed Sunday night in Spark, stored in DynamoDB, read as a simple key-value lookup on Monday.

---

#### 11. Google Maps (Slot 18, Day 178)

**Architecture:** Map Data: road network as weighted directed graph (100B+ nodes = intersections, edges = road segments); Protocol Buffers columnar format; sharded by geographic tile. Routing Engine: Contraction Hierarchies (CH) — offline preprocessing contracts unimportant nodes, adds shortcut edges between neighbors; query-time bidirectional Dijkstra uses shortcuts, visits ~1,000 nodes instead of billions, <100ms cross-country. Real-Time Traffic: GPS probes from Android (anonymized) → Kafka → Speed Estimation Service → road segment speeds updated every ~2 minutes. Geocoding: Elasticsearch with geo_point indexing + address normalization (forward: address → lat/lng); PostGIS ST_Distance spatial index (reverse: lat/lng → nearest address). Map Tiles: pre-rendered PNG/WebGL at zoom 0-22 → object storage → CDN; tile server generates missing tiles on-demand with PostGIS. Places/POI: PostgreSQL + PostGIS for location-aware queries; Elasticsearch for name search.

**Key numbers:** 1B+ monthly users, 25M map updates/day, GPS probes from 2B Android devices. CH preprocessing: offline once per graph update; query visits ~1,000 contracted nodes vs billions raw. Traffic lag: ~2 minutes from GPS probe to route weight update. ETA = CH base + real-time segment speeds + historical patterns + ML congestion prediction.

---

## LLD Systems Mastered (Slot 17)

All 8 LLD systems covered in Slot 17 (Days 162–168). For each, you should be able to whiteboard the class diagram, key DB schema, and primary concurrency/design-pattern decision in 10 minutes.

| # | System | Key Design Pattern | Critical Concurrency / DB Trick |
|---|--------|-------------------|----------------------------------|
| 1 | Library Management ER | Repository Pattern; two partial indexes on Loans table | `SELECT ... FOR UPDATE` prevents two patrons checking out the last copy simultaneously |
| 2 | BookMyShow Ticket Booking | Schema: ShowTimes, Seats, BookingSeats; expires_at for time-boxed locks | `PRIMARY KEY (seat_id, showtime_id)` — duplicate key error = double-booking prevented atomically |
| 3 | Hotel Room Booking | Redis distributed lock `SET room:{hotel}:{type}:{date} LOCKED NX EX 30` | NX = mutual exclusion; EX = deadlock prevention if lock holder crashes |
| 4 | ATM State Machine | State Pattern — IdleState, CardInsertedState, AuthenticatedState, TransactionState | CardInsertedState is stateful (stores cardId, pinAttempts); all others are stateless singletons |
| 5 | LRU / LFU Cache | LRU: HashMap + doubly linked list with sentinel head/tail; LFU: HashMap + per-frequency LinkedHashSet | LFU: LinkedHashSet preserves insertion order within frequency bucket → O(1) LRU tie-breaking |
| 6 | Social Media Feed DB | Hybrid fan-out strategy; INDEX (user_id, created_at DESC) on Feeds table | `PRIMARY KEY (user_id, post_id)` on Likes prevents double-like at DB level |
| 7 | REST API Design Patterns | Idempotency keys (POST via Redis TTL 24h); cursor pagination vs offset pagination | Cursor: `WHERE (created_at, id) < (cursor_ts, cursor_id) ORDER BY created_at DESC LIMIT N` = O(log N) regardless of depth |
| 8 | Elevator / E-commerce | Strategy Pattern (FeeCalculator); Factory Pattern (VehicleFactory); Saga for order pipeline | Optimistic locking via version column: `UPDATE SET ... WHERE version=currentVersion`; 0 rows = conflict, retry |

---

## OA / Mock Interview Stats

Track this table weekly throughout Month 6. Review after each session and schedule targeted re-drills on any pattern where completion rate < 80%.

- OA contests attempted: ___ (fill in)
- Mock interviews completed: ___ (fill in)
- Average completion rate under time pressure: ___%
- DSA: Average problems solved per 45-min session: ___/2 hard
- System Design: Average components covered per 45-min session: ___/5 major components
- Behavioral: STAR stories polished and under 2 minutes: ___/8 target

### Problem Coverage by Slot

**Slot 16 (Days 151–160) — Pattern Revision:**
Covered all 18+ DSA patterns with revision problems. Key hard problems: LC 84 (Largest Rectangle — monotonic stack), LC 2045 (Second Minimum Time — BFS with two distance arrays), LC 1192 (Critical Connections — Tarjan bridges), LC 2246 (Longest Path with Different Adjacent Characters — post-order DFS arm), LC 778 (Swim in Rising Water — Dijkstra on grid; covered in Slot 18 drill).

**Slot 17 (Days 161–170) — OA Contest Simulation:**
- LC 2009 (Minimum Number of Operations — sort + dedup + sliding window; preprocessing step: remove duplicates before sliding window)
- LC 442 (Find All Duplicates — O(n) time O(1) space via sign-flip index marking)
- LC 2251 (Number of Flowers in Full Bloom — sweep line + binary search, upperBound vs lowerBound semantics)
- LC 2386 (Find the K-Sum of an Array — heap enumeration with extend + swap pushes)
- LC 2163 (Minimum Difference in Sums After Removal — prefix max-heap + suffix min-heap, valid split range [n-1, 2n-1])
- LC 2040 (Kth Smallest Product — binary search on value, negative divisor flips inequality)
- LC 875 (Koko Eating Bananas), LC 1011 (Capacity To Ship) — binary search on feasibility
- LC 2458 (Height of Binary Tree After Subtree Removal — two-pass DFS, track top-2 child heights)
- LC 1851 (Minimum Interval to Include Each Query — offline queries + min-heap, sort queries and process in order)
- LC 2071 (Maximum Tasks You Can Assign — binary search on k + greedy canAssign with TreeMap)
- LC 1405 (Longest Happy String — greedy max-freq-first), LC 621 (Task Scheduler — O(1) formula), LC 1642 (Furthest Building — greedy min-heap of size k)

**Slot 18 (Days 171–180) — Weak Area Drill + End-to-End Mocks:**
- LC 4 (Median of Two Sorted Arrays — binary search on partition, O(log(min(m,n))))
- LC 778 (Swim in Rising Water — Dijkstra on grid)
- LC 1483 (Kth Ancestor — Binary Lifting, up[node][j] precompute)
- LC 1416 (Restore The Array — string parsing DP, O(n log k))
- LC 1240 (Tiling a Rectangle — backtracking + topmost-leftmost canonical order + pruning)
- LC 315 (Count of Smaller After Self — merge sort with original-index tracking)
- LC 2416 (Sum of Prefix Scores — Trie passCount)
- LC 1345 (Jump Game IV — BFS + value group deletion to prevent O(n²))
- LC 84 (Largest Rectangle in Histogram — monotonic increasing stack, sentinel 0 at end)
- LC 394 (Decode String — stack of (repeatCount, builtString)), LC 1209 (Remove K Adjacent Duplicates — stack of (char, count))

---

## Month Flashcard Master Deck

Flashcard sets from each weekly review. Use spaced repetition (Anki or physical cards). Review the highest-yield cards daily in the two weeks before each interview.

- **Week 22 flashcards:** `knowledge-base/week-22/weekly-review.md` — 35 cards (Backtracking Hard capstone + Slot 16 revision begins: Two Pointers, Linked List, Monotonic Stack, Trees, Graphs, Union-Find, Kahn's, multi-source BFS)
- **Week 23 flashcards:** `knowledge-base/week-23/weekly-review.md` — 35 cards (DSA revision: Advanced Graphs, Heaps, Tries, Hashing, Greedy, DP Parts 1 & 2, Backtracking; HLD: CAP Theorem, Rate Limiting, Monitoring, Auth, Microservices, HLD Synthesis; LLD: Parking Lot)
- **Week 24 flashcards:** `knowledge-base/week-24/weekly-review.md` — 35 cards (OA Contest patterns: Sweep Line, Heap Enumeration, Prefix/Suffix Heap DP, Binary Search on Value, Sliding Window Median, Two-Pass DFS, Offline Queries; LLD: Library, BookMyShow, Hotel, ATM, LRU/LFU, Social Feed, REST API)
- **Week 25 flashcards:** *(no separate weekly-review.md — review day files)* — Days 169–175: Binary Search Partition (LC 4), Dijkstra on Grid (LC 778), Binary Lifting (LC 1483), DP String Parsing (LC 1416), Backtracking Pruning (LC 1240), Merge Sort/BIT (LC 315); Systems: Instagram, WhatsApp, Google Drive, Amazon, Airbnb
- **Week 26 flashcards:** *(no separate weekly-review.md — review day files)* — Days 176–180: Merge Sort + BIT (LC 315), Trie passCount (LC 2416), Graph BFS group-delete (LC 1345), Monotonic Stack (LC 84/394/1209), 6-month synthesis; Systems: Zoom, Spotify, Google Maps

**Highest-yield cards to review daily before interviews:**

| Q | A |
|---|---|
| How do you set up Binary Search on Partition for LC 4 Median of Two Sorted Arrays? | Ensure nums1 is shorter. Binary search partitionX in [0,m]; partitionY = (m+n+1)/2 - partitionX. Invariant: maxLeftX≤minRightY && maxLeftY≤minRightX. Adjust: maxLeftX>minRightY → hi=partitionX-1; else lo=partitionX+1. Time: O(log(min(m,n))). |
| How does Dijkstra apply to LC 778 Swim in Rising Water? | dist[r][c] = minimum possible max-elevation on any path from (0,0) to (r,c). Min-heap; for each neighbor: newCost = Math.max(currCost, neighbor_elevation); if newCost < dist[neighbor], update and enqueue. Replace BFS FIFO queue with min-heap whenever edge weights are non-uniform. |
| How do you precompute and query Binary Lifting for k-th ancestor? | up[node][0]=parent; for j 1..LOG: up[node][j]=up[up[node][j-1]][j-1]. Query: for each set bit j in k, jump node=up[node][j]; return -1 if node becomes -1. Precompute O(n log n); query O(log n). |
| How do you set up the DP recurrence for LC 1416 Restore The Array? | dp[i] = ways to restore s[0..i-1]. For each i, try substrings s[j..i-1] (j from i-1 down to max(0,i-10)): if leading zero → break; parse to long; if > k → break; else dp[i] += dp[j]. Base: dp[0]=1. Return dp[n] % (1e9+7). O(n log k). |
| How does merge sort count elements smaller than self to the right (LC 315)? | Track (value, originalIndex) pairs. During merge: when a right-half element is chosen over left-half elements still waiting, each left-half element remaining gains +1 to its count. Implement: result[arr[i][1]] += (j - mid) before placing arr[i] in merged output. |
| How does the passCount Trie enable Sum of Prefix Scores (LC 2416)? | passCount at a node = number of words that pass through it (i.e., share that prefix). Insert all words, incrementing passCount at every node visited. Score each word by summing passCount along its full path — O(L) per query after O(total_chars) build. |
| In LC 1345 Jump Game IV, why must you delete the value's index group after processing in BFS? | Without deletion, BFS re-enqueues all same-value indices every time any of them is dequeued — O(n²) worst case. Deleting after first visit ensures the group is processed exactly once; subsequent arrivals at same-value nodes can only use i±1 neighbors. |
| State the monotonic stack algorithm for LC 84 Largest Rectangle in Histogram. | Maintain increasing stack of indices. For i in [0..n] (sentinel h=0 at n): while stack top bar > current height, pop index p; height=heights[p]; width=(stack empty)?i:i-stack.peek()-1; update max. Push i. Time O(n). |
| Design Instagram's feed for celebrity accounts. | Celebrity accounts (>1M followers): fan-out on read — followers pull celebrity's latest posts at query time (no write fan-out). Regular users: fan-out on write — post asynchronously pushed to all follower feed tables. At read time, merge pre-computed push feed with fresh celebrity posts pulled in real-time. |
| How does Airbnb prevent double-booking using Cassandra? | Cassandra lightweight transaction (Paxos): `INSERT INTO availability (listing_id, date, booking_id) VALUES (?,?,?) IF NOT EXISTS`. If applied=false, another booking holds that date. Final source-of-truth check happens against Cassandra at HOLD time, not Elasticsearch (which may lag ~2s). |

---

## 6-Month Journey Summary

### Full Pattern Mastery Table

A complete reference of all DSA pattern categories across the 6-month program.

| Pattern Category | First Introduced | Deepened | Hard Problems Solved | Target Comfort |
|-----------------|-----------------|----------|---------------------|----------------|
| Two Pointers | Month 1, Slot 1 | Slot 16 revision | 3Sum, Trapping Rain Water, Container With Most Water | ≥ 4/5 |
| Sliding Window | Month 1, Slot 1 | Slot 16 revision | Minimum Window Substring, Longest Repeating Char Replacement | ≥ 4/5 |
| Prefix Sum / Hashing | Month 1, Slot 2 | Slot 16 + Slot 17 | Product Except Self, Subarray Sum Equals K, Continuous Subarray Sum | ≥ 4/5 |
| Linked List | Month 1, Slot 3 | Slot 16 revision | Reverse K-Group, Cycle Detection, LRU Cache | ≥ 4/5 |
| Binary Search | Month 2, Slot 4 | Slot 17 + Slot 18 | Median Two Arrays, Koko Bananas, Ship Capacity, Max Tasks | ≥ 4/5 |
| Trees DFS / BFS | Month 2, Slot 5 | Slot 16 revision | Binary Tree Max Path, Serialize/Deserialize, LCA, Two-Pass DFS | ≥ 4/5 |
| Heaps | Month 2, Slot 6 | Slot 16 + Slot 17 | Median Data Stream, Top K Frequent, Skyline, Find K-Sum | ≥ 4/5 |
| Graphs BFS / DFS | Month 3, Slot 7 | Slot 16 + Slot 18 | Word Ladder, Jump Game IV, Shortest Bridge, Pacific Atlantic | ≥ 4/5 |
| Union-Find | Month 3, Slot 7 | Slot 16 revision | Number of Islands II, Accounts Merge, Critical Connections | ≥ 4/5 |
| Monotonic Stack | Month 3, Slot 8 | Slot 16 + Slot 18 | Largest Rectangle, Daily Temperatures, Next Greater Element II | ≥ 4/5 |
| Tries | Month 3, Slot 9 | Slot 16 + Slot 18 | Word Search II, Replace Words, Sum of Prefix Scores, Short Encoding | ≥ 4/5 |
| Greedy | Month 3, Slot 9 | Slot 16 + Slot 17 | Non-overlapping Intervals, Queue Reconstruction, Furthest Building | ≥ 4/5 |
| DP 1D / 2D | Month 4, Slot 10 | Slot 16 + Slot 17 | Coin Change, LCS, LIS, Partition Equal Subset, Count Squares | ≥ 4/5 |
| DP Interval | Month 4, Slot 11 | Slot 16 + Slot 17 | Burst Balloons, Strange Printer, Remove Boxes | ≥ 4/5 |
| DP Bitmask | Month 4, Slot 11 | Slot 17 OA | TSP Shortest Path Visiting All Nodes, Assignment Problems | ≥ 4/5 |
| DP State Machine | Month 4, Slot 11 | Slot 16 revision | Freedom Trail, Buy/Sell Stock III/IV, Best Time with Cooldown | ≥ 4/5 |
| Backtracking | Month 4, Slot 12 | Slot 16 + Slot 18 | N-Queens, Sudoku, Word Search II, Tiling Rectangle, Partition K Subsets | ≥ 4/5 |
| Sweep Line | Month 5 → Slot 17 | Slot 17 OA | Flowers in Bloom, Meeting Rooms II, Falling Squares | ≥ 4/5 |
| Merge Sort / BIT | Month 6, Slot 18 | Slot 18 weak area | Count of Smaller After Self, Count Inversions | ≥ 4/5 |
| Binary Lifting | Month 6, Slot 18 | Slot 18 weak area | Kth Ancestor of Tree Node, LCA in O(log n) per query | ≥ 4/5 |
| DP String Parsing | Month 6, Slot 18 | Slot 18 weak area | Restore The Array, Longest String Chain | ≥ 4/5 |
| Dijkstra on Grid | Month 5 → Slot 18 | Slot 18 weak area | Swim in Rising Water, Shortest Path in Binary Matrix | ≥ 4/5 |
| Advanced Graph BFS | Month 5 → Slot 18 | Slot 18 weak area | Jump Game IV (group deletion), Shortest Bridge (multi-source BFS) | ≥ 4/5 |

---

### System Design Fluency

All 11 systems — target: cold delivery in 45 minutes, no notes, no hints. Requirements → Scale → Architecture → Deep Dive → Trade-offs.

| # | System | Core Challenge | Key Pattern | Cold Delivery Target |
|---|--------|---------------|-------------|----------------------|
| 1 | Twitter / X | Fan-out at 100M followers | Hybrid push/pull + Snowflake IDs | 45 min |
| 2 | Uber / Lyft | Real-time location + matching | Redis Geo + WebSocket + geohash | 45 min |
| 3 | YouTube / Netflix | Global video delivery + recommendations | CDN + ABR + two-tower NN + Faiss | 45 min |
| 4 | Instagram | Photo feed at 1B users | S3 + CDN + hybrid fan-out + Cassandra TTL stories | 45 min |
| 5 | WhatsApp | 1.15M messages/sec + offline delivery | WebSocket + Kafka + Cassandra + Redis presence | 45 min |
| 6 | Google Drive | Chunk dedup + delta sync + conflict resolution | SHA-256 per chunk + LWW/OT + presigned S3 | 45 min |
| 7 | Amazon | Flash sale inventory + order saga | Atomic SQL guard + Saga pattern + Elasticsearch recall | 45 min |
| 8 | Airbnb | Double-booking prevention at scale | Cassandra CAS (IF NOT EXISTS) + Elasticsearch geo | 45 min |
| 9 | Zoom | Video at 300M users + adaptive quality | SFU + simulcast layers + WebRTC signaling | 45 min |
| 10 | Spotify | Recommendations at 600M users | Matrix factorization + Annoy ANN + Spark batch | 45 min |
| 11 | Google Maps | Cross-country routing in <100ms | Contraction Hierarchies + GPS probe Kafka pipeline | 45 min |

---

### What's Next — Real Interview Mode

These 5 actions are the entire focus from Day 181 onward. Everything else is maintenance.

1. **Apply immediately and broadly.** Submit applications to all FAANG and FAANG-adjacent companies within the next 2 weeks. Tailor each resume bullet to the company's engineering blog language (use their terminology: "latency" at Google, "customer obsession" at Amazon, "impact" at Meta). Build a tracking spreadsheet: company, role, recruiter contact, OA date, phone screen date, onsite date, status.

2. **Schedule and pipeline interviews deliberately.** Aim to schedule OA → phone screen → onsite in sequence across multiple companies so you have overlapping timelines. Practice your hardest companies last (after 2-3 real interviews). Give yourself 1 week between each onsite for rest and targeted gap closure.

3. **Run full mock loops weekly.** One full mock interview loop per week (2 DSA rounds + 1 system design + 1 behavioral), timed with no hints. After each mock, identify the 1-2 patterns or components that stalled you; schedule a 90-minute targeted re-drill from the relevant day files before your next real interview.

4. **Finalize your behavioral story bank.** Write out 8-10 STAR stories covering: ownership of a hard technical decision, delivering under extreme time pressure, data-driven decision-making, cross-team collaboration, a project that failed and how you recovered, mentorship, simplifying a complex system, disagreeing with a senior engineer. Practice each story aloud to under 2 minutes. Map each story to 2-3 leadership principles so you can adapt on the fly.

5. **Stay sharp without over-preparing.** Solve 2 LeetCode Hard problems per week during active interview pipelines to maintain reflexes under timed pressure. Rotate through the 10 highest-yield flashcards daily (5 minutes). Do not start new topics — drill what you already know until it is automatic.

---

### 5 Universal Interview Principles

Derived from 180 days of deliberate practice. These are the meta-lessons that transfer across every company, every round, and every problem type.

**1. Classify the pattern before touching the keyboard.**
Spend the first 60-90 seconds identifying whether the problem is Two Pointers, BFS, DP, Backtracking, Binary Search, etc. State your pattern classification out loud before writing a single line of code. Interviewers score you on your thinking process as much as your correct answer — a confident, correct classification with a wrong first attempt beats silent coding that reaches the right answer by luck. If you cannot classify it, list what you know: "the array is sorted, which suggests binary search; there is a 2D grid, which suggests BFS or DP; I need an optimal split, which suggests DP." Elimination is also pattern recognition.

**2. Write the numbers before drawing the boxes.**
In every system design interview, the first thing on the whiteboard or document should be the scale: DAU, peak QPS, storage per day, storage over 5 years. These numbers determine every architectural decision. The same feature — a user activity feed — requires a single SQL query at 10K users, a Redis cache at 10M users, and a hybrid push/pull fan-out with CDN at 1B users. An architecture diagram without numbers is decoration. Interviewers look for candidates who derive architecture from constraints, not candidates who pattern-match to a pre-memorized design.

**3. Narrate your trade-offs, not just your choices.**
Saying "I'll use Cassandra" is table stakes. Saying "I'll use Cassandra because write throughput exceeds 10K/sec, the data doesn't need JOINs, and eventual consistency is acceptable here — but if we needed strong consistency for billing, I'd use PostgreSQL" demonstrates senior-level thinking. Every choice you make has an alternative you consciously rejected. Name it, explain why you rejected it, and move on. This applies to DSA too: "I'm using a monotonic stack instead of a nested loop because the nested loop is O(n²) and this gives O(n) — the trade-off is the non-obvious invariant I need to maintain."

**4. Comfort with imperfection is a performance multiplier.**
Real interviews are not LeetCode with infinite time. You will get stuck. You will write a bug. You will miss an edge case. The distinguishing behavior is not perfection — it is how you respond to imperfection. When you get stuck: name what you know, state your current hypothesis, ask a clarifying question, or switch to a brute-force approach and optimize from there. When you find a bug: explain it calmly, fix it, and verify with a test case. Interviewers specifically test for composure under pressure. Silence and panic are the worst responses. Talking through uncertainty is always better.

**5. The interview ends when you stop learning from it.**
After every practice mock and every real interview, run a 10-minute debrief: What pattern did I classify correctly? What pattern did I misclassify? What system design component did I forget? What STAR story felt forced? Log answers to `knowledge-base/revision-log.md` and schedule a targeted 90-minute re-drill within 48 hours. The 180 days of preparation built the foundation; the real interviews are the final calibration. Treat every interview — including offers — as a data point that tells you where to drill next. The learner who iterates on feedback fastest wins.
