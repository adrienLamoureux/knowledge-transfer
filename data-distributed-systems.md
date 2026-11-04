---
title: Data and distributed systems principles
id: data-distributed-systems
doc_type: concept
summary: Track E. Know your record of truth and what it gives up under a partition; design keys from access patterns; make integrations replayable with an outbox and dead letters; stream changes with CDC; assume retries and make writes idempotent; prevent races atomically; decide fail-open versus fail-closed per control; reason about CAP, state, time and completeness.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [data, distributed-systems, consistency, replication, nosql, key-design, outbox, dead-letter-queue, cdc, idempotency, retries, concurrency, cap, caching, stateless, time, reconciliation]
aliases: [record of truth, read your writes, single-table design, access patterns, change data capture, at-least-once, exactly-once, idempotency key, backoff with jitter, race condition, lost update, fail open, fail closed, circuit breaker, CAP theorem, sticky sessions, UTC, effective dating]
related: [engineering-principles-map.md, backend-api-principles.md, devops-cloudops-principles.md, architecture-principles.md]
group: Build
order: 50
html_twin: data-distributed-systems.html
---

# Data and distributed systems principles

## TL;DR
- Know which store is the record of truth, what it gives up under a partition, and which flows must read their own writes.
- Retries are certain: every consumer and every mutating call takes an idempotency key; at-least-once plus idempotent is the practical default.
- Record every outbound call (outbox) and keep what failed (dead-letter queue); without both, a half-failed integration leaves no trace.
- Prevent lost updates with atomic increments and transactions: the database decides the winner, not request order.
- Fail open on rate limits and flags; fail closed on authentication, authorisation and tenancy, always.
- Notes are private study notes and carry no ref; Practice sources are public links.

## E1 The record of truth and its guarantees
- Priority: P1 core. Sources: Notes, Practice.
- Principle: for the store everything derives from, know its replication, what it gives up under a partition, and which reads may be stale.
- Notes, one document store:
  - A replica in production only; elsewhere a single instance.
  - It refuses rather than diverges under a partition (CP).
  - Reads prefer a secondary, which buys read availability and gives up read-your-writes.
  - Retryable writes are disabled by an engine requirement, so write idempotency becomes the application's job.
- In practice: every read preference except primary may return stale data, because secondaries replicate asynchronously (https://www.mongodb.com/docs/manual/core/read-preference/).
- Exercise: list the user flows that write then immediately read.
- Check: where would secondary-preferred reads break a user flow?

## E2 Key design from access patterns
- Priority: P2 depth. Sources: Practice, Lesson, Notes.
- In practice: in a key-value store, do not design the schema until you know the questions it must answer; keep related data together, use sort order, spread load across partitions, and keep as few tables as possible (https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-general-nosql-design.html).
- Lesson: one partition per owner (`USER#<id>`) with sort-key namespaces per entity (`IMG#`, `JOB#`, `SESSION#<id>#MSG#`) served every pattern as a key lookup, with no secondary index.
- Lesson: use atomic counters (`ADD`) instead of read-modify-write.
- Lesson: over-fetch, then act on a slice: a compaction job reads the oldest window plus what it keeps, and deletes only the head.
- Notes: an outbox table keyed tenant-first, billed on demand, with selective expiry and point-in-time recovery.
- Exercise: add a new access pattern without a secondary index.
- Check: why does an atomic `ADD` remove the lost-update race?

## E3 Outbox, dead letters and replay
- Priority: P1 core. Sources: Notes, Practice.
- Principle: record every outbound call so it can be replayed, and keep what could not be delivered at the longest retention available. Poison messages go to a dead-letter queue, not an infinite retry.
- Notes: "Without them an integration that fails halfway leaves no trace and no way back".
- In practice: the transactional outbox stores the message in the same transaction as the business change; a separate relay publishes it, and may publish it twice (https://microservices.io/patterns/data/transactional-outbox.html).
- Exercise: replay one dead-letter message safely and prove it ran once.
- Check: what makes a replay safe? (E5)

## E4 Change data capture and the warehouse pipeline
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: stream changes from the database's log instead of polling it; keep the forwarder generic, so it knows nothing about the warehouse model.
- In practice: CDC tools read the database's transaction or replication log and publish each change to a stream (https://debezium.io/documentation/reference/stable/architecture.html).
- Notes, one pipeline:
  - Change streams → one forwarder per collection → an ingestion API (OAuth per tenant) → a topic per tenant → a stream processor with registered schemas → open table format files in object storage.
  - At-least-once delivery; the warehouse upserts on tenant, entity and key, and the latest event wins. Failures retry on a ladder, then go to a dead-letter queue.
  - Known gap: deletes were dropped, because a change-stream delete event carried no document body.
- Exercise: trace one field from the source document to its warehouse column.
- Check: a record is deleted. What does the warehouse show a week later?

## E5 Delivery guarantees and idempotency
- Priority: P1 core. Sources: Notes, Practice.
- Principle, three guarantees: at most once (may lose, never duplicates), at least once (never loses, may duplicate), exactly once (costly). At-least-once means consumers must be idempotent.
- Principle: retries are certain, so every mutating call takes an idempotency key; deduplicate within a bounded window and decide what "bounded" means.
- In practice: the caller supplies a unique request token; a repeat returns a response equivalent to the first, and the same token with different parameters is rejected (https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/).
- Notes: every write an AI agent makes carries an idempotency key, because a model will retry.
- Exercise: make one POST idempotent and test a duplicate.
- Check: with at-least-once delivery plus idempotency, what does the consumer store?

## E6 Concurrency: races and atomic updates
- Priority: P2 depth. Source: Lesson.
- Principle: read-modify-write loses updates; use atomic increments or conditional writes.
- Principle: check-then-act races. Two requests both see 49 of 50 and both proceed. One transaction that increments and reads, then decides, lets the database pick the winner.
- Principle: optimistic concurrency: store a version number and write only if it is unchanged.
- Exercise: write a test with two concurrent requests at 49 of 50 and show only one wins.
- Check: why is check-then-consume unsafe even with fast code?

## E7 Failure handling: fail open or fail closed
- Priority: P2 depth. Sources: Notes, Practice, Lesson.
- Principle, the building blocks: a timeout on every call; backoff with jitter; circuit breakers; backpressure (a clear "retry after", not a timeout); dead-letter queues; bulkheads per tenant; graceful degradation (serve less, honestly).
- In practice: set a timeout on every remote call; retry with capped exponential backoff and jitter; retry at one layer only, because three retries at each of five layers multiply load on a failing database 243 times (https://builder.aws.com/content/3EumjoZascWd1oZiEgL8ORlv3qE/timeouts-retries-and-backoff-with-jitter).
- Principle: decide fail-open or fail-closed per control, deliberately:
  - Fail open: rate limiters and feature flags. A datastore hiccup must not take the site down; daily caps backstop abuse.
  - Fail closed: authentication, authorisation, tenancy. When the link to a system of record is down, a tool call fails closed: the assistant says so and writes nothing.
- Exercise: list every control in one service and mark each open or closed.
- Check: name one control that must never fail open, and why.

## E8 Consistency, CAP, replication and caching
- Priority: P2 depth. Sources: Notes, Practice, Lesson.
- Principle: strong versus eventual consistency; under a partition you keep consistency or availability, so say which. Synchronous replication is slow; asynchronous replication can lose recent writes or serve stale reads.
- In practice: the "two out of three" reading of CAP misleads; partitions are rare, and the choice between consistency and availability arises only during one, and can differ per operation (https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/).
- Principle: caching patterns are cache-aside, write-through, a CDN and TTLs; invalidation is the hard part.
- Lesson: a per-instance cache with a 60-second TTL converges within one TTL; when the use case tolerates that, a shared cache (and its standing cost) is not needed.
- Exercise: pick one cache and write down its staleness budget.
- Check: when is a per-instance cache enough instead of a shared one?

## E9 State and scale
- Priority: P2 depth. Sources: Practice, Lesson.
- In practice: processes are stateless and share nothing; sticky sessions should never be relied on; anything that must persist goes to a backing service (https://12factor.net/processes).
- Lesson: an agent framework's in-memory session store lost conversations between turns once the service scaled to zero or past one instance.
- Lesson: a protocol server that kept per-session state broke behind a load balancer; running it stateless was a correctness requirement, not an optimisation.
- Exercise: find every in-memory structure in one service and say what happens at two instances.
- Check: why does scale-to-zero break in-memory state even with one instance?

## E10 Time, ordering and clocks
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: store timezone-aware UTC everywhere. A source that sends local time across a daylight-saving change gives a duplicated or missing hour.
- Principle: ordering needs keys: per-key ordering in a topic, or sessions in a queue.
- Lesson: daily caps that roll over at UTC midnight are simple to reason about; say so in the user-facing message.
- Exercise: find one timestamp in your data that is not UTC.
- Check: why does local time break an hourly aggregate twice a year?

## E11 Completeness and reconciliation
- Priority: P3 reference. Source: Notes.
- Principle: events give real time; scheduled sweeps give completeness. You need both to prove nothing was missed.
- Principle: during a migration, run CDC with verification counts.

## E12 Distributed patterns and the scaling order
- Priority: P3 reference. Source: Notes.
- Five patterns: sharding (by hash, range or tenant), replication, write-ahead log, data versus metadata, load distribution. Scale on the real signal: queue depth, not CPU.
- Scaling order: measure; cache and add read replicas; scale out stateless services; queue slow work; partition data; go multi-region last.
- Availability multiplies: three 99.9% services in a chain give about 99.7%.

## E13 Audit-grade data
- Priority: P3 reference. Source: Notes.
- Principle: effective dating answers "what did it look like on 30 June?"; evidence is write-once and hashed; reproducibility stores inputs, logic version and timestamps. For AI, also store the index, prompt and model versions and the retrieved ids.

## E14 Data platforms you will hear about
- Priority: P3 reference. Source: Notes.
- Vectors beside relational data (PGVector): one transaction and one access model.
- Graph databases suit traversal questions.
- Governance layers: Databricks (Delta time travel, Unity Catalog lineage) and Cloudera (Ranger, Atlas).
- Streaming and lakehouse vocabulary: Kafka topics, Flink jobs, Iceberg tables in Parquet, a schema registry.

## Invariants
- NEVER let authentication, authorisation or tenancy fail open (E7).
- Every mutating call that can be retried carries an idempotency key (E5).
- Retry at one layer only (E7).

## Gotchas
- Secondary-preferred reads concede read-your-writes.
- A change-stream delete without a document body can make the warehouse miss deletions.
- In-memory state silently breaks at two instances, and at scale-to-zero (E9).

## Glossary
- CDC — change data capture: streaming changes from a database's log.
- Outbox — a table of outbound messages written with the business change, for publishing and replay.
- DLQ — dead-letter queue: where failed messages wait.
- Idempotency key — a client-chosen id that makes a repeated call return the first result.
- CAP — consistency, availability, partition tolerance.
- Effective dating — storing when a fact was true, not only its latest value.

## Open questions
- None.

## Sources
- https://www.mongodb.com/docs/manual/core/read-preference/ — read preference and staleness.
- https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-general-nosql-design.html — NoSQL design from access patterns.
- https://microservices.io/patterns/data/transactional-outbox.html — transactional outbox.
- https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/ — idempotent APIs.
- https://builder.aws.com/content/3EumjoZascWd1oZiEgL8ORlv3qE/timeouts-retries-and-backoff-with-jitter — timeouts, retries and jitter.
- https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/ — CAP twelve years later.
- https://12factor.net/processes — stateless processes.
