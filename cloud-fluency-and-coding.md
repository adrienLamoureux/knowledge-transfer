---
title: Cloud fluency and coding fundamentals
id: cloud-fluency-and-coding
doc_type: concept
summary: Tracks K and L, reference. Map the services you run across AWS, Google Cloud and Azure; learn the few concepts that differ; design for zero idle cost; work across clouds; migrate with the 7 Rs, the strangler fig and waves; landing zones; and the Python, complexity and algorithm fundamentals behind everyday code.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [cloud, aws, google-cloud, azure, migration, landing-zone, finops, multi-cloud, python, complexity, algorithms]
aliases: [service map, AWS to GCP, GCP for AWS engineers, standing cost, scale to zero, Workload Identity Federation, cutover, 7 Rs, 6 Rs, strangler fig pattern, Shared VPC, VPC Service Controls, cloud foundation, big-O, coding drills]
related: [engineering-principles-map.md, devops-cloudops-principles.md, poc-to-production-case.md, backend-api-principles.md]
group: Reference
order: 120
html_twin: cloud-fluency-and-coding.html
---

# Cloud fluency and coding fundamentals

## TL;DR
- Learn each cloud by mapping the services you already run, then the few concepts that differ: projects, a global VPC, IAM granted to principals.
- Zero idle cost is a design rule: avoid the services that bill while nobody uses them until load justifies them.
- Migrate in phases (assess, plan, migrate, optimise), choose per workload with the 7 Rs, modernise with the strangler fig, and cut over in waves.
- Price code by multiplying loops and pricing every line inside them; most algorithm problems are one of a dozen patterns.
- Product names and regulatory dates move; re-check them before quoting.
- Notes are private study notes and carry no ref; Practice sources are public links.

## K1 A service map for what you run
- Priority: P3 reference. Sources: Notes, Practice.
- Principle: map AWS → Google Cloud → Azure for the services you run, each with a one-line use case. Start from containers and functions, databases and queues, streaming, object storage, AI model hosting, keys and secrets, monitoring, DNS and load balancing, the registry and CI.
- Notes, the rows most teams need first:

| Need | AWS | Google Cloud | Azure |
|---|---|---|---|
| Containers | Fargate, EKS | Cloud Run, GKE | Container Apps, AKS |
| Functions | Lambda | Cloud Run functions | Azure Functions |
| Messaging | SNS + SQS, Kinesis | Pub/Sub | Service Bus, Event Hubs |
| Object storage | S3 | Cloud Storage | Blob Storage |
| Keys, secrets | KMS, Secrets Manager | Cloud KMS, Secret Manager | Key Vault |

- In practice: Google publishes a category-by-category comparison of AWS and Azure services with Google Cloud (https://docs.cloud.google.com/docs/get-started/aws-azure-gcp-service-comparison).
- Exercise: write the Google Cloud and Azure twin for each service your team runs.

## K2 Concepts that differ on Google Cloud
- Priority: P3 reference. Source: Notes.
- Principle: resources live in projects under folders and an organisation; the VPC is global and subnets are regional; firewall rules target tags or service accounts; IAM roles are granted to principals, and service accounts are identities; every service is an API enabled per project; sustained-use discounts apply automatically on many VM families.

## K3 Standing-cost traps and zero-idle design
- Priority: P3 reference. Source: Lesson.
- Principle: every component should cost about nothing when nobody uses it, until load justifies more.
- Lesson, services that bill while idle, and what replaced them at small scale:

| Trap | Why it bills while idle | Used instead at small scale |
|---|---|---|
| A dedicated vector-search service | Always-on index nodes | The database's built-in vector index |
| A global load balancer plus CDN | A monthly charge before any traffic | Managed hosting with rewrites |
| A managed SQL database for session state | An instance runs all month | A serverless document store |
| A managed Redis for a shared cache | A node runs all month | A per-instance cache with a short TTL |
| Self-hosted GPUs | Machines reserved by the hour | A provider that bills GPU time per second |
| NAT gateways, provisioned concurrency | Hourly charges | No private egress, on-demand concurrency |

- Principle: quote cloud prices as order-of-magnitude anchors with a date, never as quotes.
- Exercise: list every component in one service that bills while idle.

## K4 Working across clouds
- Priority: P3 reference. Sources: Notes, Practice.
- Principle: keyless identity with Workload Identity Federation (https://docs.cloud.google.com/iam/docs/workload-identity-federation); HA VPN (99.99% with two tunnels) or a dedicated interconnect; egress costs money.
- Principle: under the EU Data Act, cloud switching charges end on 12 January 2027 [unverified: re-check the date].
- Lesson: hosting rewrites are first-match-wins; a catch-all placed first makes "the API return HTML".

## K5 Migration
- Priority: P3 reference. Sources: Notes, Practice.
- Phases: assess, plan, migrate, optimise.
- In practice, the 7 Rs per workload: retire, retain, rehost (lift and shift), relocate, repurchase, replatform, refactor or re-architect. For large migrations, rehost, relocate or replatform first and modernise after (https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html).
- In practice, the strangler fig: build the new system alongside the old and move behaviour over in small pieces, then remove the old code (https://martinfowler.com/bliki/StranglerFigApplication.html).
- Principle: "Move first, modernize where it pays"; lift-and-shift is a step, not the destination.
- Cutover: big bang, waves, parallel run, blue/green; one tenant at a time is the unit for low-risk moves; move databases with CDC and verification counts.
- Principle: operations move too: dashboards, alerts, runbooks and on-call.

## K6 Landing zones
- Priority: P3 reference. Sources: Practice, Notes.
- In practice: a landing zone (a cloud foundation) is the configuration an organisation needs before it deploys enterprise workloads; its core design areas are identity, resource hierarchy, network and security controls (https://docs.cloud.google.com/architecture/landing-zones).
- Notes: organisation → folders → projects; org policies as guardrails (allowed regions, no public IPs, no key creation); a Shared VPC so one network team serves many projects; VPC Service Controls as a perimeter round Google APIs.

## K7 Product renames
- Priority: P3 reference. Source: Notes.
- Gemini Enterprise Agent Platform (formerly Vertex AI); Agent Runtime (formerly Agent Engine); Knowledge Catalog (formerly Dataplex). Checked September 2026; re-check before publishing [unverified].

## L1 Python essentials
- Priority: P3 reference. Source: Notes.
- Topics: the object model; instance versus class attributes; the three kinds of method; properties; inheritance, MRO and mixins; composition over inheritance; dunder methods; dataclasses; ABCs versus Protocols; the built-in collections; EAFP; context managers and generators; gotchas.

## L2 Pricing a solution
- Priority: P3 reference. Source: Notes.
- Principle: nested loops multiply and sequential loops add; price every line inside a loop, because `x in a_list`, a slice, a `sort`, `pop(0)` and string `+=` each hide a loop; total = times the innermost line runs × its cost. Use `deque` for a queue.

## L3 Algorithm patterns and their triggers
- Priority: P3 reference. Source: Notes.
- Principle, a thirty-second triage decides the pattern: what must be returned (one number or all the answers separates dynamic programming from backtracking); whether the input is sorted (two pointers, binary search); and the size of n sets the allowed complexity (about 10^5 to 10^6 allows `O(n log n)`).

## L4 Practical drills that match real work
- Priority: P3 reference. Source: Notes.
- Five drills shaped like everyday backend tasks:
  - Rate limiter: at most N calls per user in any sliding window; the caller passes the clock in.
  - Flatten a config: nested keys to dotted keys and back; unflatten(flatten(d)) must equal d.
  - Diff records: two snapshots in, added, removed and changed fields out.
  - Endpoint report: from an access log, the endpoints with the worst server-error rate.
  - Parse a CSV line by hand: quoted fields, literal commas, doubled quotes.

## Invariants
- Quote cloud prices as dated order-of-magnitude anchors, never as quotes (K3).

## Gotchas
- A hosting catch-all placed first swallows the API silently (K4).
- A shared cache is not always needed: a per-instance cache with a short TTL converges within one TTL (`data-distributed-systems`, E8).

## Glossary
- Landing zone — the organisation, folder, project, network and policy baseline. Aliases: cloud foundation.
- Strangler fig — replace a system one capability at a time.
- 7 Rs — retire, retain, rehost, relocate, repurchase, replatform, refactor.
- WIF — Workload Identity Federation.
- VPC-SC — VPC Service Controls.
- EAFP — "easier to ask forgiveness than permission": try, then handle the exception.

## Open questions
- Re-check product names and EU dates at publication [unverified].

## Sources
- https://docs.cloud.google.com/docs/get-started/aws-azure-gcp-service-comparison — service comparison.
- https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html — the 7 Rs.
- https://martinfowler.com/bliki/StranglerFigApplication.html — the strangler fig.
- https://docs.cloud.google.com/architecture/landing-zones — landing zones.
