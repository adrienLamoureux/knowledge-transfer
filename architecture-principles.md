---
title: Architecture principles — boundaries, records and readiness
id: architecture-principles
doc_type: concept
summary: Track B. One record of truth with every other path given a job relative to it; integrate through APIs, not shared databases; accounts as the boundary that contains failure; the edge as a chain; tenancy enforced in the query; boring technology by default; divergent decisions reconciled explicitly; production readiness as a checklist with evidence.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [architecture, system-of-record, integration, blast-radius, multi-account, edge, load-balancer, tenancy, multi-tenant, boring-technology, production-readiness]
aliases: [record of truth, system of record, data gravity, account boundary, landing zone, listener rules, tenant isolation, tenant filter, innovation tokens, unreconciled decisions, production bar, well-architected]
related: [engineering-principles-map.md, data-distributed-systems.md, security-identity-principles.md, devops-cloudops-principles.md]
group: Foundations
order: 20
html_twin: architecture-principles.html
---

# Architecture principles — boundaries, records and readiness

## TL;DR
- Put one record of truth at the centre and give every other path a job relative to it: keep it true, serve it, derive from it, report from it.
- Systems meet at APIs, never at each other's database; data stays where it lives.
- The account (or project) is the boundary that contains a mistake. Subnets, security groups and tags organise; they contain nothing.
- In a shared store, tenant isolation lives in the query, so it depends on every call site: identity from the token, never from the URL.
- Default to boring technology; reconcile divergent decisions on purpose; define "production-ready" as a list of checks with evidence.
- Notes are private study notes and carry no ref; Practice sources are public links.

## B1 One record of truth, read many ways
- Priority: P1 core. Sources: Notes, Lesson.
- Principle: name the one store everything else is derived from, then give each other path a clear job relative to it: keep it true (ingest), serve it (APIs), derive from it (search, recommendations), report from it (a change feed to the warehouse).
- Principle: keep derived data next to its source when you can. Vectors stored in the same cluster as the records they describe make recommendations a reader of the platform, not a second platform.
- Lesson: every extra copy needs an owner, a freshness budget and a way to rebuild it from the record of truth.
- Exercise: draw your system's record of truth and every copy of its data, with who keeps each copy fresh.
- Check: why does storing vectors beside the records matter for consistency and access control?

## B2 Integrate through APIs, not shared databases
- Priority: P1 core. Sources: Notes, Lesson.
- Principle: the seam between two systems is an API with a contract. A consumer that reads another system's database couples to its schema, bypasses its authorisation and cannot be audited.
- Principle: data has gravity: move the small, stateless part (an agent, a UI, a worker) to the data, not the data to it.
- Principle: a new consumer that needs reporting data gets a read-only API in front of the warehouse, not database credentials.
- Lesson: call the API as the user, with the user's token, so the system that owns the data keeps enforcing who may see it.
- Exercise: list every system that reads your database directly, and the API that should replace each one.
- Check: what does a direct database read bypass that an API call does not?

## B3 Accounts are the blast-radius boundary
- Priority: P1 core. Sources: Practice, Notes.
- Principle: the cloud account (a project on Google Cloud) is the boundary that contains a mistake. Everything inside it organises the estate without containing anything.
- In practice: an AWS account is an isolation boundary for identity and access; no access crosses accounts unless explicitly allowed. AWS recommends several accounts per organisation rather than one (https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html).
- Notes, patterns from a platform review:
  - Each production environment alone in its own account, so a mistake elsewhere stops at the account line.
  - Non-production environments sharing accounts are contained from production, not from each other.
  - One account that holds no environment runs every pipeline and assumes a role into the target: one audit trail for every apply.
  - A shared-services account holds the image registry and the hardened machine image.
  - When the code looks a network up instead of creating it, the network boundary is inherited; know who owns it.
- Exercise: for a mistake in one development environment, list everything it can reach.
- Check: why is a single pipeline-execution account good for audit?

## B4 The edge is a chain
- Priority: P1 core. Sources: Notes, Practice.
- Principle: know every link from the client to the service, and how each one fails.
- Notes, a typical chain: a DNS alias with target-health evaluation → one HTTPS listener with a modern TLS policy and a fixed 404 by default → listener rules by priority → target group → service.
- Principle: rule priorities act as a service registry: reading the routing table shows every service, including reserved, unbuilt ones.
- Principle: health checks and draining decide when a task joins and leaves; write them down per service.
- In practice: know your orchestrator's rollout arithmetic. On ECS, the minimum healthy count is `desiredCount × minimumHealthyPercent / 100` rounded up, and the maximum is rounded down (https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_definition_parameters.html).
- Principle: one task means no redundancy during a deploy or a host failure; a second task costs money and buys availability.
- Exercise: find one service's routing rule, health check and draining delay.
- Check: what does a default action of "fixed 404" give you that a redirect does not?

## B5 Tenancy: isolation lives in the query
- Priority: P1 core. Sources: Practice, Notes; see also `security-identity-principles` G3.
- Principle: in a shared store, the boundary is whatever filters every read. Identity comes from a validated token claim, never from a URL.
- In practice: crossing the tenant boundary "in any form" is treated as a potentially unrecoverable event for a SaaS business (https://docs.aws.amazon.com/whitepapers/latest/saas-tenant-isolation-strategies/saas-tenant-isolation-strategies.html).
- In practice: broken object-level authorisation is the first item of the OWASP API Security Top 10 (https://api-security.owasp.org/editions/2023/en/0x11-t10).
- Notes, lessons from one shared store:
  - The tenant in the path is a log field, never a boundary.
  - A hook copies the tenant onto every read filter, but is skipped when a caller supplies its own filter, so correctness depends on every call site.
  - A noisy tenant shares capacity with all others; a restore brings back all tenants or none.
  - Onboarding a tenant is a pipeline run, not a console action.
- Exercise: write a test that tries to read another tenant's record through a call site that passes its own filter.
- Check: what happens to isolation when a caller passes its own filter?

## B6 Choose boring technology
- Priority: P2 depth. Sources: Practice, Notes.
- Principle: default to mature technology whose failure modes are known. Each unproven choice spends one of a few "innovation tokens"; spend them where the business is different (https://mcfunley.com/choose-boring-technology).
- Principle: every added tool costs the whole organisation in operations and attention, not only the team that chose it (https://mcfunley.com/choose-boring-technology).
- Principle: add complexity in order, and only when a measurement demands it: measure; cache and add read replicas; scale out stateless services; queue slow work; partition data; go multi-region last.
- Exercise: list the technologies in your service and mark which ones spent an innovation token.
- Check: when is a new technology worth its token?

## B7 Two builds, decisions to reconcile
- Priority: P2 depth. Source: Notes.
- Principle: two competent teams working at different times reach different answers. Treat each difference as a decision to make explicitly, not as a defect.
- Notes, eight differences found in one platform:

| Decision | One side | Other side | Cost of picking one |
|---|---|---|---|
| Edge routing | URL path | Host header | Certificates, CORS, every client base URL |
| Capacity | Owned fleet, warm pool | Serverless containers per task | Patch surface vs per-task cost and cold start |
| Scaling declaration | Main IaC tool | A second tool, after apply | One state to reason about, or two |
| Backup retention | A week | A day | Recovery point |
| Destroy protection | Final snapshot required | Final snapshot skipped | Whether a mistake is recoverable |
| Log retention | One month | Three months | Storage vs investigation window |
| IAM scoping | Named resources | Wildcards in places | Least privilege |
| Promotion unit | A version file | A repository per environment | Merge model and audit trail |

- Principle: some differences have a clearly better side (backup retention, destroy protection, IAM scoping); others are genuine trades. Edge routing is the hardest to reverse once clients exist.
- Exercise: pick one decision and argue it both ways.
- Check: which one would you unify first, and why?

## B8 Production-ready is a checklist with evidence
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: "production-ready" is a list of checkable properties, each with its evidence and its kind (derived from config, or confirmed by a person), not a feeling.
- Notes, one such list:
  - Foundation: production alone in its account; customer-managed keys; three scanners per merge; only the tested image ships.
  - Platform: two availability zones; a tenant filter on every read; the database port open to a named group only; malware scanning on upload; a snapshot and automatic restore around the riskiest change; every outbound call recorded for replay.
  - Data pipeline: at-least-once delivery with idempotent upserts; retries, then a dead-letter queue.
  - Company: ISO 27001, a recent external pen test, on-call with runbooks, a customer SLA. In that list these were confirmed by the team rather than read from configuration.
- In practice: the AWS Well-Architected Framework reviews a workload against six pillars: operational excellence, security, reliability, performance efficiency, cost optimisation and sustainability (https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html).
- Exercise: for one service you own, tick the list and attach evidence to each tick.
- Check: which ticks are derived from configuration, and which rest on someone's word?

## Invariants
- NEVER treat a URL path segment as a tenant boundary.
- Production environments share an account with nothing else.
- A consumer never reads another system's database; it calls the owning system's API (B2).

## Gotchas
- A caller-supplied filter can skip tenant injection.
- A shared cluster restores all tenants or none.
- A single-task service has no redundancy during a deploy or a failure (B4).

## Glossary
- Record of truth — the one store every other copy is derived from. Aliases: system of record, operational store.
- Blast radius — what a single mistake can reach.
- Listener rule priority — the number that decides which load-balancer rule wins.
- Tenant filter — the condition added to every read so a tenant sees only its own data.
- Innovation token — a unit of the limited budget for unproven technology.

## Open questions
- Who owns the shared network in each account, and how are changes requested? [unverified]
- Product context: who uses which product, at which customers? [unverified]

## Sources
- https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html — multiple accounts as isolation boundaries.
- https://docs.aws.amazon.com/whitepapers/latest/saas-tenant-isolation-strategies/saas-tenant-isolation-strategies.html — tenant isolation.
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_definition_parameters.html — ECS rollout limits.
- https://mcfunley.com/choose-boring-technology — innovation tokens.
