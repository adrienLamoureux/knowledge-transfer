---
title: Worked case — an AI assistant from PoC to production (hypothetical)
id: poc-to-production-case
doc_type: concept
summary: Track J. One hypothetical case that applies the whole set end to end — join an AI assistant prototype to a multi-tenant production platform — as two moves with one unknown each, written criteria and exits, the bridge from PoC to production, six gaps a prototype usually has, the bottleneck at each stage, and four reversible waves.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [case-study, poc, production-readiness, ai-assistant, integration, multi-tenant, multi-cloud, waves, success-criteria, golden-set]
aliases: [PoC to production, two moves, golden set parity, readiness bar, prototype gaps, rollout waves, hypothetical case, worked example]
related: [engineering-principles-map.md, fde-delivery-approach.md, ai-agent-engineering.md, architecture-principles.md, cloud-fluency-and-coding.md]
group: AI and delivery
order: 110
html_twin: poc-to-production-case.html
---

# Worked case — an AI assistant from PoC to production (hypothetical)

## TL;DR
- Hypothetical case, not a roadmap: an AI assistant prototype runs on its own; a multi-tenant platform in production holds the records. Users should ask, discover, analyse and later act on platform data in one conversation.
- The assistant uses the platform only through its APIs, with the user's token; it never touches the database.
- The PoC is two moves, one unknown at a time: rebuild the assistant on its target runtime (proved by golden-set parity), then integrate it read-only for one tenant (proved by seven written targets).
- "A PoC proves it works; production proves it keeps working": six things widen, and three of a prototype's usual gaps close only with real users.
- Four waves, one change each, each with an exit test and a rollback.
- Notes are private study notes and carry no ref; Practice sources are public links.

## J1 Where we are, where we are going
- Priority: P2 depth. Label: Case.
- Today: two decoupled systems. The platform runs in production: query, command and recommendation APIs over one record of truth, and a change feed to a warehouse. The assistant prototype runs alone: a web front end, one function, one model, its own small store for sessions, memory and caps.
- Target: the assistant joins the platform through three links: the existing front door routes assistant calls; the existing identity provider stays the only user pool; a private path carries the user's token to the platform's APIs. A new read-only reporting API fronts the warehouse.
- The four verbs map to four platform APIs: ask → the query API, filtered per tenant; discover → the recommendations the platform already builds; analyse → the reporting API, each answer dated; act → the command API, only after the user says yes, with an idempotency key on every write.
- Principle applied: the seam between systems is the API; data stays where it lives (`architecture-principles`, B2).

## J2 Two moves, one unknown at a time
- Priority: P2 depth. Label: Case.
- Move 1, rebuild: the prototype becomes a production-shaped service on its target runtime: its own service identity, the chosen model behind a gateway, its own state store, traces and logs. Proved by parity with today's assistant on the golden set. No tenant data is involved.
- Move 2, integrate: a private link; keyless identity; tools behind MCP called with the user's token; a read-only reporting API; no direct database access. Proved by the PoC's criteria, on one tenant, read-only.
- If the new model misses parity, keep the old model and move only the runtime: runtime and model are separate decisions.
- Principle applied: prove the riskiest change with no tenant data involved, then integrate once, on the target, rather than twice.

## J3 Variant: the assistant runs on another cloud
- Priority: P3 reference. Label: Case.
- When the assistant runs on a different cloud from the platform, only the assistant moves: it holds no platform data, so moving it moves none.
- Costs, named before anyone asks: a cross-cloud round trip on every tool call (so pick the region nearest the platform), egress (small, because responses are small), and two clouds to run.
- The link: HA VPN or a dedicated interconnect; keyless federation, where a role on one cloud trusts the other cloud's token, so no key is stored on either side (`security-identity-principles`, G5).
- Why not move the platform: data gravity; its problems are code and process, not the cloud.

## J4 Criteria first
- Priority: P2 depth. Label: Case.
- Seven targets, agreed with the sponsor before the build, as proposals [proposal]:

| Criterion | Target | Measured by |
|---|---|---|
| Golden-set accuracy | ≥ 90%, and ≥ today's assistant | 50–100 fixed turns, code checks plus judged samples |
| Valid replies | ≥ 99% | JSON that parses the first time |
| Time to first words | ≤ 2 s | Pilot measurement |
| Full turn | ≤ 8 s p95, every hop included | Pilot measurement |
| Tenant isolation | 0 cross-tenant results | A security test set that asks for another tenant's data |
| Cost | ≤ 1¢ per turn | The assistant's cost counters |
| Usefulness | ≥ 4/5 | Pilot users, weekly |

- Exits: go when every target holds for two weeks; narrow by dropping a function that misses; stop on any cross-tenant result, or under 80% accuracy after two iterations. Time-box 4–6 weeks.
- Scope: one tenant, the admin cohort, read-only functions (ask, discover, analyse). Act follows once reads pass.
- Secure from day one: the user's token on every call, a keyless link, a read-only tool allow-list, no personal data in logs, a weekly security test set.

## J5 The bridge from PoC to production
- Priority: P2 depth. Label: Case.
- "The PoC proves it works. Production proves it keeps working". Six things widen:

| What widens | In the PoC | In production |
|---|---|---|
| Tenants | One | Every tenant |
| Users | The admin cohort | Everyone, via a cohort flag |
| Functions | Ask, discover, analyse | Plus act, after the user's yes, with an idempotency key |
| Traffic | A few pilot users | The peak, load-tested first |
| Running it | Watched by the team | SLOs, alarms, on-call |
| Security | Day-one basics | Plus input screening, audit, pen test |

- Narrative rule: say what the PoC was built to prove; never call the assistant "not ready".
- Most PoCs should stop; day-one security is not negotiable, and the rest is paid for only once the PoC passes.

## J6 The bar and six gaps a prototype usually has
- Priority: P2 depth. Label: Case.
- The bar is the platform's own (`architecture-principles`, B8); the assistant joins it.
- Six gaps a prototype assistant usually has, and when each closes:
  1. Identity: one role shared by every route → its own service identity, allowed only its tools. Closed by the PoC.
  2. Quality: unit tests but no golden set → 50–100 golden turns. Closed by the PoC.
  3. Safety: platform records enter the prompt as text → input and output screening before writes arrive. Before production.
  4. Reliability: several chained model calls under a gateway timeout → streaming and a time budget per turn. Closed by the PoC.
  5. Operations: logs only → SLOs, alarms, on-call. Before production.
  6. Load and cost: never load-tested → a load test with real turns and a daily spend alarm. Before production.
- Keep what is already right: data per user and calls as the user; every write waits for a yes; the model proposes and code writes; per-user caps; a retry that rolls back to a snapshot instead of overwriting.

## J7 Build, scale, break; then four waves
- Priority: P2 depth. Label: Case.
- What breaks first, by stage: PoC — answer quality and the round trip per tool call; beta — turn latency under traffic, chained calls and model quotas; production — the runtime's instance and concurrency limits, and the platform's own APIs, because every turn makes several tool calls.
- Security grows at the same pace: day-one basics → input screening, per-tenant rate limits, a data perimeter → an audit record per tool call (user, tenant, tool), alarms on unusual tool use, the pen test extended.
- Four waves:

| Wave | What changes | Exit test | Rollback |
|---|---|---|---|
| 1 · Move 1 | The rebuilt assistant, on its own | Golden-set parity | Keep using today's assistant |
| 2 · Move 2, the PoC | The link and read-only tools, one tenant, admin cohort | The seven targets | Switch the tools off |
| 3 · Beta | A few tenants; act behind the user's yes | Service levels hold | Flip the cohort flag |
| 4 · Everyone | Every tenant; managed runtime features one at a time | Service levels for a full window | One change at a time |

- Measured every wave: golden-set score, p95 turn latency, cost per turn, error rate.
- Each wave ends on its exit test, not on a date.
- First three steps: agree the criteria in writing; write the golden set and score today's assistant; start move 1.

## Invariants
- The assistant never reads the platform's database; it calls APIs with the user's token.
- Tool calls fail closed when the link is down: the assistant says so and writes nothing.

## Gotchas
- All targets are proposals to agree with a sponsor, not measurements.
- A wave planned by date rather than by exit test ships unproven changes.

## Glossary
- Parity — the new assistant scores at least as high as today's on the same golden set.
- Cohort flag — a feature flag per user group (`all`, `admin`, `beta`) that routes each wave.
- Wave — one change, one exit test, one rollback.

## Open questions
- None; the case is hypothetical by design.

## Sources
- Notes: a private hypothetical case deck of twelve slides and annexes; no ref.
