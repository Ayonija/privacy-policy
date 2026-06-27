# 06 — Messaging: Apache Kafka

> Kafka is on the JD and underpins event-driven design (and your review-system answer). Know the mental model, delivery semantics, ordering, and back-pressure cold.

---

## What Kafka is
**Plain English:** Kafka is a **distributed, durable, append-only log** for streaming events. Producers append records to topics; consumers read at their own pace. It decouples systems, absorbs spikes, and lets many consumers replay the same data.

**Why it exists:** point-to-point integrations (N services × M services) explode in complexity and couple systems tightly. A log in the middle = producers and consumers don't know about each other, data is durable + replayable, and you can add consumers later without touching producers.

---

## Core concepts (define each)
- **Topic** — a named stream of records (e.g. `reviews`).
- **Partition** — a topic is split into partitions; each partition is an **ordered, append-only** sequence. **Partitions are the unit of parallelism and ordering.**
- **Offset** — a record's position in a partition (monotonic). Consumers track "where I've read."
- **Producer** — appends records; chooses partition by **key hash** (same key → same partition → ordered) or round-robin (no key).
- **Consumer** — reads records; commits offsets to mark progress.
- **Consumer group** — a set of consumers sharing a `group.id` that **split the partitions** among themselves: each partition is consumed by **exactly one** consumer in the group. This is how you scale out *and* preserve per-partition order. Different groups each get the *full* stream (pub/sub).
- **Broker** — a Kafka server; a **cluster** is many brokers. Partitions are **replicated** across brokers (`replication.factor`); one replica is **leader** (handles reads/writes), others are **followers**. **ISR** = in-sync replicas.
- **Retention** — records persist for a time/size window (or compacted), so consumers can **replay** — unlike a traditional queue that deletes on consume.

🎤 **Say it like this:** *"Kafka is best understood as a distributed, durable, append-only log, not a traditional queue. A topic is split into partitions, and a partition is the unit of both ordering and parallelism — records are ordered within a partition but not across them. Consumers in a consumer group divide the partitions so each partition has exactly one reader in the group, which is how you scale horizontally while keeping per-key order. Because data is retained on disk and replicated across brokers, multiple independent consumers can read the same stream and even replay history — that's what makes it the backbone of event-driven systems rather than just messaging."*

---

## Delivery semantics (the classic question)
- **At-most-once** — commit offset *before* processing. If you crash after commit but before processing, the message is lost. No duplicates, possible loss. (Rarely wanted.)
- **At-least-once** — process *then* commit. If you crash after processing but before commit, you reprocess on restart → **duplicates possible, no loss**. **The common default.** → Make consumers **idempotent**.
- **Exactly-once (EOS)** — no loss, no duplicates. Kafka supports it *within Kafka* via **idempotent producers** (dedup by producer id + sequence number) + **transactions** (atomic write across partitions + offset commit). End-to-end exactly-once to an *external* system still needs idempotent writes on your side.

🎤 **Say it like this:** *"There are three delivery guarantees and they're really about when you commit the offset relative to processing. Commit before processing is at-most-once — you can lose messages. Commit after is at-least-once — you can get duplicates on a crash-retry — and that's the pragmatic default. So the real engineering move isn't chasing exactly-once; it's designing idempotent consumers so a duplicate is harmless — usually a dedup key or an upsert. Kafka does offer exactly-once within Kafka via idempotent producers and transactions, but the moment you write to an external system, you still need idempotency on your side."*

---

## Ordering, idempotent producers, back-pressure
- **Ordering** — guaranteed **only within a partition**. To keep related events ordered (all events for one user/order), use a **partition key** so they hash to the same partition. Global ordering = single partition = no parallelism (avoid).
- **Idempotent producer** (`enable.idempotence=true`) — broker dedups retried sends via producer-id + per-partition sequence number, so a network retry doesn't create a duplicate record. (Default-on in modern Kafka.)
- **Back-pressure** — *plain English:* when a consumer can't keep up with the producer's rate. Kafka handles this naturally: it's a **pull** model — consumers fetch at their own pace, and the log buffers on disk, so a slow consumer just **lags** (its offset falls behind the log end) rather than overwhelming anyone. You monitor **consumer lag** (end offset − committed offset). Fix lag by adding consumers (up to #partitions), speeding processing, or batching. *Consumer lag is the #1 Kafka health metric.*

🎤 **Say it like this:** *"Ordering is per-partition only, so I get ordering where it matters by keying on the entity — all of one order's events hash to one partition and stay ordered, while different orders parallelize. Back-pressure mostly takes care of itself because Kafka is pull-based and disk-backed: a slow consumer just accumulates lag rather than crashing the system. So the metric I watch is consumer lag — committed offset versus the log end — and I scale by adding consumers up to the partition count, since partitions cap parallelism. That's also why partition count is a capacity decision you make early."*

---

## When Kafka vs not
- **Kafka:** high-throughput event streaming, many consumers, replay, decoupling, event sourcing, log/metrics pipelines.
- **Not Kafka:** simple task queue with low volume (RabbitMQ/SQS simpler), request/reply RPC (use REST/gRPC), tiny apps (over-engineering). Kafka shines at *streams*, not per-message routing/priorities.

**Spring snippet:**
```java
@KafkaListener(topics = "reviews", groupId = "aggregator")
public void onReview(ReviewEvent e) {
    // at-least-once: make this idempotent — upsert by e.reviewId
    aggregateService.applyIdempotent(e);
}   // offset auto-commits after successful return (ack mode matters)
```

---

## ⚠️ Pitfalls & seniority signals
- Thinking Kafka guarantees **global** ordering — it's per-partition.
- Chasing exactly-once instead of **idempotent consumers** — the senior move is idempotency.
- More consumers than partitions — extras sit **idle** (partition is the parallelism cap).
- Ignoring **consumer lag** as the health signal.
- Using Kafka as a DB or an RPC channel.
- **Seniority signal:** "I key by entity for ordering, default to at-least-once + idempotent consumers, size partitions for target parallelism, and alert on consumer lag." That sentence alone reads senior.

---

*Next topic, or drill deeper on this one?*
