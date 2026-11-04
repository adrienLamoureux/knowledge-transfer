---
title: Backend and API principles
id: backend-api-principles
doc_type: concept
summary: Track C. Middleware as an ordered chain, one composition root, small modules, written API contracts with one error shape, pagination and idempotency from day one, long work as jobs, write-ahead logging, SOLID and patterns only where a symptom calls for them, configuration outside the code, tests with injected fakes that assert call shapes, and serverless limits.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [backend, api, rest, middleware, dependency-injection, composition-root, error-handling, pagination, idempotency, async-jobs, write-ahead-log, solid, design-patterns, configuration, testing, serverless]
aliases: [request pipeline, DI, IoC, router modules, API contract, problem details, RFC 9457, cursor pagination, idempotency key, 202 Accepted, async request-reply, WAL, outbox, SOLID, chain of responsibility, strategy, registry, twelve-factor, test pyramid, contract tests]
related: [engineering-principles-map.md, data-distributed-systems.md, security-identity-principles.md, frontend-react-principles.md]
group: Build
order: 30
html_twin: backend-api-principles.html
---

# Backend and API principles

## TL;DR
- Middleware is an ordered chain: attach identity, require a user (401), require a role (403). The order is the security model.
- Build every dependency once, at startup, in one place, and pass it in; tests pass fakes, and "which implementation runs?" has one answer.
- Write the API contract down: one error shape, status codes that mean something, pagination and idempotency keys from the first version, and long work as a job.
- Record intent before acting (write-ahead logging): an outbox, the job row before the provider call, a checkpoint per step.
- Name a pattern only when its trigger appears; a registry beats a growing `if` chain.
- Notes are private study notes and carry no ref; Practice sources are public links.

## C1 Request pipeline and middleware
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: middleware is a chain of responsibility. Each step handles the request, passes it on, or rejects it.
- Principle: order matters: attach identity → require a user (401) → require a role (403). A role check placed before the user check reads a user that does not exist.
- Principle: let CORS preflight (`OPTIONS`) through before any auth check; guards respond and return early, so handlers stay thin.
- In practice: in Express, error-handling middleware takes four arguments `(err, req, res, next)` and is registered after every other `app.use()` and route; in Express 5 a rejected promise from an async handler reaches it automatically (https://expressjs.com/en/guide/error-handling.html).
- Lesson: a shared error helper that returned `error.message` in every 500 body exposed internal details to clients. Log the detail; return a generic message and a correlation id.
- Exercise: add a guarded admin route and test both rejections: 401 when anonymous, 403 when not an admin.
- Check: why must the admin check come after the user check?

## C2 Composition root and dependency injection
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: build every dependency (clients, stores, helpers, middleware) in one place, as close as possible to the entry point, and pass it in. That place is the composition root (https://blog.ploeh.dk/2011/07/28/CompositionRoot/).
- Principle: modules never `require` or import a concrete client themselves; they receive it. Tests then pass a fake object, with no module mocking.
- Lesson: a flat dependency object grows large; keep a revisit trigger (a property count or a line count) rather than restructuring early, because renaming every destructuring site costs more than it saves.
- Lesson: allow at most one mode switch (local or cloud, real or stub) and branch on it in the composition root only.
- Exercise: add a new client through the composition root, with a test that passes a fake.
- Check: why is "one place builds everything" easier to test than import-time singletons?

## C3 Module structure and file size
- Priority: P1 core. Source: Lesson.
- Principle: each feature module exports a factory that returns a router; one hub mounts them all. Gains: namespaces stay apart, each router can be tested alone, the mount map reads like a table of contents.
- Lesson: changing a mount prefix silently breaks every client that hard-coded the old path; treat mount paths as part of the contract.
- Lesson: cap file length mechanically in CI. One route file grew to thousands of lines before a cap existed; split into a helpers file or a sub-module when the cap trips.
- Exercise: split one long route file by sub-domain.
- Check: what does a file-length cap prevent, and what can it cost?

## C4 API contracts and design
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: write the contract down: every endpoint, its auth tier (public, user, admin), input, output and every error. Response shapes are contracts even when they look like details; a field renamed from `sessionId` to `session.id` breaks clients.
- Principle, one error shape: use problem details, `application/problem+json`, with `type`, `title`, `status`, `detail` and `instance` (https://www.rfc-editor.org/rfc/rfc9457).
- Principle, status codes carry meaning: 400 bad input, 401 not signed in, 403 not allowed, 404 not found, 409 conflict, 429 too many requests with an optional `Retry-After` (https://www.rfc-editor.org/rfc/rfc6585).
- Principle, paginate from the first version: adding pagination later is a breaking change, because a new default page size silently truncates old clients. Use opaque page tokens and a documented default size (https://google.aip.dev/158).
- Principle, idempotency keys on every POST that creates or charges: the client sends a unique key; a repeat with the same key returns the same result; the same key with different parameters is a validation error (https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/, https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/).
- Principle, deprecate with a date: announce removal with the `Sunset` header and a timestamp (https://www.rfc-editor.org/rfc/rfc8594).
- Principle, contract tests: consumers state what they rely on; the provider runs those expectations as tests before it deploys (https://martinfowler.com/articles/consumerDrivenContracts.html).
- Lesson: a function allowed 120 seconds sat behind a gateway with a 29-second integration timeout; users hit the gateway's limit, not the function's. Know every timeout on the path and design long work around the shortest.
- Needs team input: API standards, naming and versioning conventions [unverified].
- Exercise: document one endpoint fully, including every error it can return.
- Check: why is adding pagination to an existing list endpoint a breaking change?

## C5 Long work is a job
- Priority: P2 depth. Sources: Practice, Lesson.
- Principle: work that outlives a request becomes a job: the API validates, records the job, returns `202 Accepted` with a `Location` to poll and an optional `Retry-After`; the status resource reports a documented set of states; when finished it points to the result (https://learn.microsoft.com/en-us/azure/architecture/patterns/asynchronous-request-reply).
- Principle: a client may ask to wait a few seconds with `Prefer: wait=N`; the server answers synchronously if the work finishes in time, otherwise with 202 (https://www.rfc-editor.org/rfc/rfc7240).
- Principle: one job record and one status vocabulary for every kind of long work; provider ids stay internal; the server, not a browser cache, is the source of truth for what is running.
- Lesson: in one codebase the same "start, then poll" idea had been built about eleven times with five different status vocabularies. Replacing them with one job record let clients resume in-flight work from one server feed and deleted a client-side cache.
- Exercise: add a new job type end to end, including its terminal states.
- Check: why is a server feed a better source of truth than a client cache?

## C6 Write-ahead logging and durable work
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: record the intent durably first, then apply it, so you can replay after a crash. The cost is one extra write.
- In databases: the write-ahead log; change data capture reads it (`data-distributed-systems`, E4).
- In services:
  - Outbox: store the outgoing message in the same transaction as the business change; a relay publishes it later (https://microservices.io/patterns/data/transactional-outbox.html).
  - Persist before acting: write the job record before calling the provider.
  - Checkpoint each step, so a failure at step 7 does not re-run and re-pay for steps 1–6.
  - Snapshot and roll back: a bad result can be reverted, not only overwritten.
- Principle: a relay may publish a message more than once, so every consumer must be idempotent (https://microservices.io/patterns/data/transactional-outbox.html).
- Notes: an outbox table plus a dead-letter queue, and a snapshot before the riskiest change with an automatic restore.
- Exercise: draw a crash at each step of one long operation and say what survives.
- Check: why write the job row before the provider call, not after?

## C7 OOP and SOLID
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: each SOLID letter prevents one smell:
  - Single responsibility — a class that both fetches and formats.
  - Open/closed — a registry instead of a growing `if` chain.
  - Liskov substitution — a subclass that raises on an inherited method.
  - Interface segregation — small protocols, not one large one.
  - Dependency inversion — depend on the abstraction; inject the concrete thing (C2).
- Principle: composition over inheritance; in Python, `Protocol` for structural seams and frozen dataclasses for value objects.
- Principle: no hard-coded values or magic numbers; name them in a config or constants module.
- Lesson: the same list of allowed values defined separately in the frontend and the backend drifts; keep one source and generate or share the other.
- Exercise: find one SOLID violation in your code and propose the smallest fix.
- Check: in Python, why is a registry usually better than inheritance for open/closed?

## C8 Design patterns where they show up
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: reach for a pattern only when its trigger appears.

| Pattern | Trigger | Typical backend example |
|---|---|---|
| Chain of responsibility | Steps that may handle, pass on or reject | Middleware (C1) |
| Strategy and registry | A conditional that keeps growing | One handler per provider, looked up by name |
| Adapter | One interface must fit another | Running a web framework inside a function runtime |
| Repository | Hide storage behind an interface | A store with a real and an in-memory implementation |
| Command | Undo, replay, audit or a queue | A proposed action stored, approved, then executed |
| Observer | One event, several reactions | A "session expired" event the UI reacts to |
| State machine | Behaviour changes over an object's life | A job's status transitions (C5) |
| Circuit breaker | A dependency that fails repeatedly | Fail fast while open, try one call when half-open (https://martinfowler.com/bliki/CircuitBreaker.html) |

- Deciding question for wrappers: adds behaviour → Decorator; hides or defers → Proxy; converts → Adapter.
- Exercise: name the pattern in five snippets from your service.
- Check: Decorator, Proxy or Adapter: what is the deciding question?

## C9 Class and domain design
- Priority: P2 depth. Source: Notes.
- Principle, a six-step checklist: name the nouns and their lifecycles; draw the boundary and keep the domain free of SDK imports; choose the seam where things vary and make it a protocol; state invariants and enforce them in the constructor; default to immutable; say how you would test it.
- Building blocks: frozen value objects, string enums for clean serialisation, timezone-aware UTC, state machines as a transition table.
- Exercise: design one of a scheduler, a write-once hashed evidence store, bounded-window deduplication, or a findings workflow.
- Check: where should an invariant be enforced, and why not in the callers?

## C10 Configuration and environments
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: configuration lives outside the code. The test: could the codebase be made public right now without exposing a credential? (https://12factor.net/config)
- Principle: validate configuration at startup and fail fast on a bad value, rather than failing on the first request that needs it.
- Principle: one build artefact serves every environment; the environment supplies the values. Runtime config served to a browser holds no secrets.
- Lesson: gate development-only paths on the server's own mode, never on a header the client can send.
- Lesson: feature flags read from a store with a short cache may fail open when the store is unavailable; a check that guards identity never does (`data-distributed-systems`, E7).
- Exercise: move one literal into config, with a startup check and a test.
- Check: when may a flag fail open, and when must a check fail closed?

## C11 Testing the backend
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: many fast unit tests, fewer integration tests, very few end-to-end tests, plus contract tests between services (https://martinfowler.com/articles/practical-test-pyramid.html).
- Principle: unit tests inject hand-made fakes through the composition root (C2).
- Principle: a fake must assert the call shape. A fake that accepts any arguments hides real bugs.
- Lesson: a data-store helper took one options object; callers passed positional arguments and got an empty list back, with no error. History, compaction and a list view silently did nothing in production while hundreds of tests passed, because the fakes accepted any call. Tests now assert the argument shape, and the helper rejects a wrong one.
- Principle: locally, keep the real loop and fake only the slow or paid edge (a model call, a provider); a bug that then reproduces locally is a wiring bug.
- Exercise: write a fake that would have caught the wrong-shaped call.
- Check: why did hundreds of green tests not catch it?

## C12 Serverless specifics
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: serverless is cheap when idle and constrained when busy: know the timeouts, payload limits, cold starts and concurrency of your platform.
- Lesson: on request-billed container platforms, CPU is billed for the whole request, so waiting on a third party costs money and holds a concurrency slot. A short `Prefer: wait` plus client polling (C5) beats one long synchronous call.
- Lesson: up to four chained model calls under a 29-second gateway timeout is a reliability gap; the fix is streaming, a time budget per turn that drops optional calls, and a queue for work that can wait.
- Lesson: avoid services that bill while idle (a NAT gateway, an always-on database, provisioned concurrency) until the load justifies them (`cloud-fluency-and-coding`, K3).
- Exercise: estimate the cost of one 60-second request against a 5-second one on your platform.
- Check: why does a short wait plus polling beat one long synchronous call?

## C13 Python service patterns
- Priority: P3 reference. Sources: Notes, Practice.
- Principle: `Protocol` seams with an in-memory twin for every provider; dataclasses with `slots`; dependency resolution in the framework's dependency system.
- Gotchas: mutable default arguments; building strings with `+=` in a loop is O(n²).

## Invariants
- The role check runs after the user check (C1).
- Every POST that creates or charges accepts an idempotency key (C4).
- Error responses never carry internal exception messages (C1).

## Gotchas
- A helper that returns an empty result on wrong-shaped arguments fails silently; validate arguments and throw (C11).
- Renaming a mount prefix or a response field breaks clients that the compiler cannot see (C3, C4).

## Glossary
- Composition root — the one place dependencies are built and wired.
- Guard — a function that responds and returns early when a precondition fails.
- Problem details — the RFC 9457 error format.
- Poll recipe — what a client needs to re-check a job: the status URL and a suggested interval.
- Write-ahead log — record intent before applying it. Aliases: WAL.

## Open questions
- The team's API standards: versioning, pagination, error shape [unverified].

## Sources
- https://www.rfc-editor.org/rfc/rfc9457 — problem details for HTTP APIs.
- https://google.aip.dev/158 — pagination.
- https://learn.microsoft.com/en-us/azure/architecture/patterns/asynchronous-request-reply — async request-reply.
- https://microservices.io/patterns/data/transactional-outbox.html — transactional outbox.
- https://martinfowler.com/articles/practical-test-pyramid.html — the test pyramid.
- https://12factor.net/config — configuration in the environment.
