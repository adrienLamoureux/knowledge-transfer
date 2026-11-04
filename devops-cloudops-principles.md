---
title: DevOps and CloudOps principles
id: devops-cloudops-principles
doc_type: concept
summary: Track F. Gate production on a recent real deploy and ship the tested image; treat IaC state as shared state; promote by library or GitOps knowingly; place agents by capacity model; roll out in reversible waves and canaries; know the network path; alert on symptoms against SLOs; run incidents with clear roles; troubleshoot in six steps; load-test for real; rehearse restores; talk unit cost.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [devops, cloudops, ci-cd, infrastructure-as-code, terraform, gitops, containers, kubernetes, rollouts, canary, networking, cors, observability, opentelemetry, slo, error-budget, on-call, incidents, postmortem, troubleshooting, load-testing, disaster-recovery, finops]
aliases: [delivery pipeline, release gate, IaC state, promotion, deployment strategy, blue green, progressive delivery, VPC, security groups, golden signals, USE method, trace context, error budget policy, incident commander, escalation, runbook, blameless postmortem, k6 thresholds, RPO, RTO, pilot light, warm standby, unit cost]
related: [engineering-principles-map.md, architecture-principles.md, data-distributed-systems.md, security-identity-principles.md]
group: Run
order: 60
html_twin: devops-cloudops-principles.html
---

# DevOps and CloudOps principles

## TL;DR
- Gate production on evidence that a real deploy to test recently worked, and ship the tested image, never a rebuild.
- IaC state is shared state: know which tool declares what, store state remotely with locking, and never apply two stacks that share state together.
- Roll out one change per wave, each with an exit test and a rollback; roll back first, find the root cause second.
- Collection is not alerting: define SLOs and page on the symptoms users feel; spend the error budget deliberately.
- A written restore procedure is not a rehearsed one; and talk cost per unit, not per month.
- Notes are private study notes and carry no ref; Practice sources are public links.

## F1 How a change ships
- Priority: P1 core. Sources: Notes, Practice.
- Principle: gate merges on scanners and tests; gate production on a recent, real (not dry-run), successful deploy to test, and reuse that tested image.
- Notes, one pipeline: pull request → build and unit tests → SAST, dependency and quality scanners → images in a registry. Production is blocked unless a test deploy recently succeeded; dry runs do not count; an override exists, as it must. The scan stage retries transient failures and tells them apart from real findings.
- In practice: measure the pipeline with the DORA metrics, throughput (change lead time, deployment frequency, failed deployment recovery time) and instability (change fail rate, deployment rework rate) (https://dora.dev/guides/dora-metrics/).
- Exercise: follow one PR from merge to production and note every gate.
- Check: why is a recent test deploy stronger evidence than a green build?

## F2 Infrastructure as code
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: know every tool that declares infrastructure; a second tool is a second state to reason about.
- In practice: Terraform state maps real resources to configuration; keep it in a remote backend with locking and access control, never in version control, because it can hold secrets (https://developer.hashicorp.com/terraform/language/state).
- Notes: a restore stack that shares state with the live stack must never be applied at the same time as it.
- Principle: prefer looking a resource up to creating it only when someone else owns it, and know who.
- Lesson: an IaC asset-upload helper that ran in a short-lived function timed out on large assets; excluding them and syncing afterwards worked, but only with deletion of "unknown" files turned off.
- Principle: review every plan for three things: resources destroyed, permissions widened, data stores replaced.
- Exercise: read one plan and list every destroy and every IAM change.
- Check: why must the restore stack never be applied together with the live one?

## F3 Environments and promotion
- Priority: P1 core. Sources: Notes, Practice, Lesson.
- Principle: two promotion idioms, each with a cost:
  - A shared pipeline library pinned to a branch, not a tag: one fix lands everywhere at once, which is excellent or terrifying depending on your appetite for change control.
  - GitOps, one repository per environment: what is in the repository is what is deployed, which kills drift at the cost of a repository per environment.
- In practice: GitOps has four principles: declarative, versioned and immutable, pulled automatically, continuously reconciled (https://opengitops.dev/).
- Principle: a one-command deploy that verifies itself: build → deploy → sanity checks → UI smoke tests → the deploy tool writes the ledger (`evidence-and-decisions`, A4).
- Lesson: delete what you stop using. A registry listed destroyed stacks as live and missed two forgotten stacks that were still billing.
- Exercise: find which promotion idiom your service uses and what a bad library change would reach.
- Check: what does pinning a pipeline library to a branch buy, and what does it risk?

## F4 Compute: containers and serverless
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: the capacity model decides cost, patching and where agents run.
- Notes:
  - An owned fleet with a stopped warm pool (storage, no compute) and a golden image refreshed on a schedule, against serverless containers per task.
  - A monitoring agent is one daemon per host on owned instances, but a sidecar in every task on serverless containers, so its cost scales with task count.
- In practice, container images: multi-stage builds, a minimal trusted base pinned by digest, rebuilt often, a `.dockerignore`, and a non-root `USER` (https://docs.docker.com/build/building/best-practices/).
- Principle: map container services to Kubernetes objects: task → pod, service → deployment and service, target group and listener → ingress, capacity provider → node pool and autoscaler.
- Needs team input: where Kubernetes runs today [unverified].
- Exercise: map one container service onto Kubernetes objects.
- Check: why does a sidecar agent cost more on serverless containers than a daemon on owned hosts?

## F5 Deployments and rollouts
- Priority: P1 core. Sources: Notes, Practice.
- Principle: health checks and draining decide when a task joins and leaves (`architecture-principles`, B4).
- Principle, strategies and their risks: big bang (everything fails at once), waves (the default; a hybrid period to operate), parallel run (double cost, drift), blue/green (stateless only; the data layer needs its own plan).
- In practice: a canary is a partial, time-limited deployment compared against a control population, on a few user-visible metrics, with a small share of traffic, one canary at a time (https://sre.google/workbook/canarying-releases/).
- Principle: one change per wave, an exit test and a rollback per wave; feature flags with cohorts route each wave.
- Principle: roll back first, then find the root cause.
- Exercise: write the exit test and rollback for your next release.
- Check: why is blue/green weak for a database change?

## F6 Networking
- Priority: P2 depth. Sources: Notes, Lesson.
- Inside an account: a VPC with subnets across availability zones; security groups that name a source group, never an address range, on the database port; regional services reached over private endpoints.
- At the edge: listener rules by path or host; a DNS alias with health evaluation; a modern TLS policy; a CDN with origin access control.
- CORS, lessons:
  - Let preflight `OPTIONS` through before authentication.
  - Add CORS headers to the gateway's own 4xx and 5xx responses too; otherwise the browser hides the real error behind a CORS message.
  - Serving the API on the same origin as the app avoids preflight entirely.
  - A storage bucket needs CORS for `fetch` and canvas reads, while a plain `<img>` works without it.
- Cost and latency: NAT gateways and load balancers carry standing costs; egress costs money; a round trip is about 0.5 ms inside one data centre and 100–150 ms between Europe and the US.
- Across clouds: HA VPN (99.99% with two tunnels) or a dedicated interconnect; check that address ranges do not overlap.
- Needs team input: who owns the shared network and how to request changes [unverified].
- Exercise: draw the path of one request from browser to database, with every port and security group.
- Check: why name a source security group instead of an address range?

## F7 Observability
- Priority: P2 depth. Sources: Practice, Notes.
- In practice, the four golden signals: latency, traffic, errors, saturation; page on symptoms ("what is broken"), use causes ("why") for debugging (https://sre.google/sre-book/monitoring-distributed-systems/).
- In practice, signals: traces, metrics and logs, plus baggage for context (https://opentelemetry.io/docs/concepts/signals/); propagate one trace across every hop with the W3C `traceparent` header (https://www.w3.org/TR/trace-context/).
- In practice, for each resource check utilisation, saturation and errors (the USE method) (https://www.brendangregg.com/usemethod.html).
- Notes: "Collection was funded; objectives were not": logs, an agent for traces and metrics, and a log router, with no service-level objective computed anywhere.
- Exercise: follow one request through logs, metrics and a trace.
- Check: which golden signal would have caught troubleshooting case T1 first?

## F8 SLOs, error budgets and alerting
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: SLI → SLO → error budget → policy. An SLA is the contract and is set looser than the SLO: promise 99.9%, aim for 99.95%.
- What each nine allows per month: 99.9% ≈ 44 minutes, 99.95% ≈ 22 minutes, 99.99% ≈ 4.4 minutes.
- In practice: when the budget for the last four weeks is spent, halt changes other than P0 fixes and security fixes until the service is back within its SLO (https://sre.google/workbook/error-budget-policy/).
- Exercise: write one SLO and its alarm for a service you own.
- Check: why is the SLA set looser than the SLO?

## F9 On-call, escalation and incidents
- Priority: P2 depth (P1 before joining a rota). Sources: Notes, Practice; most of this topic needs team input.
- Principle, the incident loop: detect → triage by severity → mitigate (roll back first) → resolve → blameless postmortem → tracked actions.
- In practice, roles: incident command (runs it, holds what is not delegated), operations (the only people changing the system), communication (the public face), planning (bugs, handoffs, what to revert); hand off command live and announce it (https://sre.google/sre-book/managing-incidents/).
- In practice: postmortems are blameless, and their triggers are agreed in advance: user-visible downtime past a threshold, any data loss, an on-call intervention such as a rollback, a long resolution, or a monitoring failure (https://sre.google/sre-book/postmortem-culture/).
- Principle: most incidents follow a change, so check the deploy timeline first.
- To take from the team's runbooks, not to invent [unverified]: severity levels with examples; page versus ticket; the escalation ladder; acknowledgement targets; customer updates under the SLA; support tiers; vendor escalation.
- Skeletons to fill [proposal]:
  - Escalation ladder: primary on-call → secondary → service owner → incident commander → leadership and customer communication.
  - Runbook: symptom and alert text, dashboards, checks, safe actions, rollback, who to call.
  - Postmortem: timeline, impact, root cause, what went well, actions with owners.
- Exercise: run case T1 (F10) as a tabletop with an incident commander and a communicator.
- Check: why does the incident commander not fix?

## F10 Troubleshooting in six steps
- Priority: P2 depth. Source: Notes.
- Principle: define (a complaint becomes a measurement) → scope (blast radius) → bisect (client → DNS → CDN → load balancer → app → cache → database → third party) → what changed → measure (test the hypothesis) → fix and prevent (restore, then make it visible).
- Worked cases:
  - T1: a slow page after a release: an N+1 query; roll back, batch the lookup, alert on p99 and count queries in CI.
  - T2: wrong numbers after a weekend: one source sent local time across a daylight-saving change.
  - T3: an assistant got worse: a bad re-index dropped retrieval while generation was fine.
  - T4: a warehouse bill doubled: an unfiltered scheduled query every 15 minutes.
- Exercise: rewrite one case on your own stack.
- Check: in T3, why check retrieval before touching the prompt?

## F11 Load and performance testing
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: a smoke run proves the path works and nothing about load. Run the real test and keep the artefact.
- In practice: load-test thresholds such as `p(95)<200` are pass/fail criteria; a failed threshold makes the run exit non-zero, so it can gate a pipeline (https://grafana.com/docs/k6/latest/using-k6/thresholds/).
- Notes: a harness with a spike profile, a p95 threshold, an isolated generator and a guard that refuses production had never been run; the only captured run was one user and a few dozen requests.
- Exercise: run a spike profile against a non-production environment and store the result.
- Check: what does a p95 from 34 samples tell you?

## F12 Backup, restore and disaster recovery
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: RPO is the data you can lose; RTO is the time to recover.
- In practice, four strategies from cheapest to most expensive: backup and restore, pilot light (data replicated, core infrastructure provisioned but switched off), warm standby (a scaled-down copy that can take traffic at once), multi-site active/active (near-zero recovery time). Test them regularly (https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html).
- Notes, restore lessons: a snapshot before each risky change with an automatic restore on failure; a credential reset is part of the restore when the restore API does not carry the password; a shared cluster restores all tenants or none.
- Principle: a written procedure with an estimated duration is not a rehearsal with a measured one.
- Exercise: write the RPO and RTO you would promise for one service, and how you would prove them.
- Check: why is a credential reset part of the restore?

## F13 Cost and FinOps
- Priority: P2 depth. Sources: Practice, Notes, Lesson.
- In practice: FinOps runs in three phases: inform (visibility and allocation), optimise (usage and rates), operate (put changes in place across engineering, finance and the business) (https://www.finops.org/framework/phases/).
- Principle: talk unit cost (per tenant, per 1,000 requests, per agent turn), never only the monthly bill.
- Notes: levers already in code are easy wins: a stopped warm pool, per-service log retention, expiry of untagged images; tags make cost per team possible.
- Lesson: in one AI product, image generation was about 90% of spend; a separate daily cap on it bounded the worst case per user by an order of magnitude.
- Exercise: compute one unit cost for a service you own.
- Check: why is a unit cost more useful to a business owner than a monthly bill?

## Invariants
- Production ships the image that passed test, never a rebuild.
- Roll back before root-causing.
- IaC state lives in a locked remote backend (F2).

## Gotchas
- Scaling declared by a second IaC tool after the main apply is invisible to a search of the main tool.
- Missing CORS headers on gateway errors make every failure look like a CORS failure in the browser (F6).

## Glossary
- SAST, SCA — static analysis of code; analysis of dependencies.
- GitOps — the repository is the deployed state, reconciled by an agent.
- SLI, SLO, SLA — indicator, objective, agreement.
- Error budget — the unreliability an SLO allows.
- RPO, RTO — recovery point objective, recovery time objective.
- IC — incident commander.

## Open questions
- The team's severity levels, escalation ladder and runbook template [unverified].

## Sources
- https://sre.google/sre-book/monitoring-distributed-systems/ — golden signals.
- https://sre.google/workbook/error-budget-policy/ — error budget policy.
- https://sre.google/sre-book/managing-incidents/ — incident roles.
- https://sre.google/workbook/canarying-releases/ — canarying.
- https://developer.hashicorp.com/terraform/language/state — Terraform state.
- https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html — DR strategies.
- https://www.finops.org/framework/phases/ — FinOps phases.
