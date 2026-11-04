---
title: Engineering principles — the map
id: engineering-principles-map
doc_type: concept
summary: Start here. What the knowledge-transfer set teaches, where its material comes from, how each source is labelled, which principles recur in every doc, and a reading path for each role.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [knowledge-transfer, principles, learning-path, index, evidence]
aliases: [learning plan, engineering handbook, principles index, reading list, knowledge transfer]
related: [evidence-and-decisions.md, architecture-principles.md, backend-api-principles.md, frontend-react-principles.md, data-distributed-systems.md, devops-cloudops-principles.md, security-identity-principles.md, ai-agent-engineering.md, ai-retrieval-evals-cost.md, fde-delivery-approach.md, poc-to-production-case.md, cloud-fluency-and-coding.md]
group: Start here
order: 0
html_twin: engineering-principles-map.html
---

# Engineering principles — the map

## TL;DR
- This set is knowledge transfer: general engineering principles for architecture, APIs, frontend, data, operations, security, AI and the forward-deployed (FDE) way of delivering. It is not a guide to any one codebase or product.
- Material comes from three places: published practice, private study notes, and lessons from building and reviewing real systems.
- Every principle carries its source label: Practice, Notes or Lesson; the one worked example is labelled Case.
- Twelve docs, one per theme (tracks A–L). Every section states the principle, how to apply it, then an exercise and a check question.
- Every section carries a priority: P1 core (every engineer), P2 depth (whoever owns the area), P3 reference.
- Notes are private, so they carry no ref; each point is restated so it stands alone.

## What this set is and is not
- It is: principles that hold across stacks, each with its reason and its source, written for engineers joining the team.
- It is not: onboarding onto a specific codebase, a product tour, or the team's internal standards. Where only the team's own standards can answer, the doc says "Needs team input".
- Examples are generic. A lesson learned on a real system is stated as a general rule, without naming the system.

## The twelve docs
| Track | Doc | Teaches | Sections |
|---|---|---|---|
| A | [evidence-and-decisions](evidence-and-decisions.md) | Evidence badges, drifting docs, ADRs, docs that stay true, quality gates, honest communication | A1–A6 |
| B | [architecture-principles](architecture-principles.md) | Record of truth, integrating through APIs, accounts as blast radius, the edge, tenancy, boring technology, reconciling decisions, production readiness | B1–B8 |
| C | [backend-api-principles](backend-api-principles.md) | Middleware, composition root, modules, API contracts, jobs, write-ahead logging, SOLID, patterns, domain design, config, testing, serverless | C1–C13 |
| D | [frontend-react-principles](frontend-react-principles.md) | Provider tree, Atomic Design, tokens, state, effects and hooks, the API client, browser auth, conversational UI, accessibility, delivery, testing | D1–D13 |
| E | [data-distributed-systems](data-distributed-systems.md) | Record of truth, key design, outbox, CDC, idempotency, races, failure handling, CAP, state, time, reconciliation | E1–E14 |
| F | [devops-cloudops-principles](devops-cloudops-principles.md) | Shipping a change, IaC, promotion, compute, rollouts, networking, observability, SLOs, incidents, troubleshooting, load tests, DR, FinOps | F1–F13 |
| G | [security-identity-principles](security-identity-principles.md) | AuthN and AuthZ, OAuth 2.0, OIDC, JWT, PKCE, object-level authorisation, secrets, service identity, supply chain, uploads, abuse limits, compliance, privacy | G1–G11 |
| H (part 1) | [ai-agent-engineering](ai-agent-engineering.md) | The AI mental model, workflows before agents, tool design, agent security, reasoning patterns, memory and context, state graphs, MCP and A2A | H1–H8 |
| H (part 2) | [ai-retrieval-evals-cost](ai-retrieval-evals-cost.md) | RAG, retrieval quality, knowledge graphs, evals, cost and latency, the gateway, failure triage, AI in the workflow, governance | H9–H18 |
| I | [fde-delivery-approach](fde-delivery-approach.md) | The FDE role, discovery, design answers, PoC scoping, phasing, prototype to production, presenting, honest gaps | I1–I8 |
| J | [poc-to-production-case](poc-to-production-case.md) | One hypothetical case, end to end: two moves, criteria first, the bridge, the gaps, the waves | J1–J7 |
| K, L | [cloud-fluency-and-coding](cloud-fluency-and-coding.md) | Service maps across clouds, zero-idle design, cross-cloud work, migration, landing zones, Python and algorithm fundamentals | K1–K7, L1–L4 |

## Source labels
| Label | What it is | How it is cited |
|---|---|---|
| Practice | Published guidance: IETF and W3C standards, OpenID, OWASP, the Google SRE books, cloud vendors' architecture guidance, framework documentation, well-known engineering articles | A URL; each one was opened and checked on 2026-10-26 |
| Notes | Private study notes: cloud architecture and system design with practice cases, an anonymised platform review, a hypothetical case deck, a GenAI field guide, audit-ready engineering, Python references and drills | No ref: the notes are private, so each point is restated so it stands alone |
| Lesson | Learned building or reviewing real systems | Stated as a general rule; the system is not named; no ref is given |
| Case | The hypothetical worked case in track J | No ref: adapted from a private hypothetical case deck |

## Evidence badges and doc tags
- Measured — the number came out of a run.
- Derived — the number was read from configuration under version control, at a named revision.
- Target — intended, with nothing behind it yet. Never present a target as achieved.
- Doc tags in these MD files: an untagged fact was seen in its source; `[inferred]` was deduced; `[unverified]` needs an owner to confirm; `[proposal]` is not built.

## Priorities
| Tag | Means | Read it |
|---|---|---|
| P1 core | Every engineer should know it before making a change in their area | First |
| P2 depth | Needed to own a service, a feature or an area | When you take ownership |
| P3 reference | Read it when the work needs it | As needed |

## Principles that recur across the set
| Principle | In one line | Where it is taught |
|---|---|---|
| Evidence before claims | Badge every number; scope every absence ("not found in what we searched") | A1, A6 |
| Prose drifts, code has a gate | Generate the facts in docs, or make a check read them | A2, A4 |
| One place to look | One composition root, one API client, one job record, one navigation source | C2, C5, D6 |
| Record intent before acting | Outbox, the job row before the provider call, a checkpoint per step | C6, E3 |
| Retries are certain | Every consumer and every mutating call takes an idempotency key | C4, E5, H3 |
| Identity comes from the verified token | Never from the path, the body or the model | B5, G3, H4 |
| Fail open on limits, closed on identity | A datastore hiccup must not take the site down; auth must never pass by default | E7, G9 |
| Code owns every write | The model proposes; code validates, the user confirms, the normal API writes | H3, D8 |
| Systems meet at APIs | Never at each other's database | B2, J1 |
| The smallest thing that works | Boring technology; a workflow before an agent; scale in order | B6, H2, E12 |
| No standing bill by default | Avoid services that cost money while idle | C12, K3 |

## Learning paths by role
| Role | First (P1) | Then (P2) | Reference (P3) |
|---|---|---|---|
| Everyone | A1–A5, B1–B5, G1–G3, F1 | A6 | — |
| Backend | C1–C4, C10–C11, E1, E3, E5 | C5–C9, C12, E2, E4, E6–E10, G4–G9 | C13, E11–E14 |
| Frontend | D1–D3, D6–D7, D11, C4 | D4–D5, D8–D10, D12, G8, G11 | D13 |
| AI | H1–H4, I1 | H5–H10, H12–H16, I2–I7, J | H11, H17–H18 |
| DevOps and cloud | F1–F3, F5 | F4, F6–F13, E7–E10, G4–G7 | K |
| Architecture and delivery | A6, B6–B8, I1–I4 | I5–I7, J | I8, K |

## How to run a learning session
1. Read one section's principle aloud (5 minutes).
2. Open one of its Practice links and find the sentence the principle rests on (10 minutes).
3. Do the exercise in pairs (30 minutes).
4. Answer the check question without notes; if the answer needs the doc, re-read the principle.
5. Write one sentence on where the principle applies in your own service.

## Where the material came from
| Source | Label | Feeds |
|---|---|---|
| Standards, OWASP, SRE books, vendor and framework documentation, engineering articles | Practice | Every doc |
| Cloud architecture and system-design notes, with practice cases | Notes | B, E, F, G, H, I, K |
| An anonymised platform review | Notes | A, B, C6, E, F, G, H16 |
| A hypothetical case deck | Notes, Case | B2, B8, E5, E7, I7, J |
| A GenAI field guide and four one-pagers | Notes | G11, H, I |
| Audit-ready engineering notes | Notes | A3, A6, C7, C9, D13, E11, E13, E14, G10, I8 |
| Python references and drills | Notes | C1, C8, C13, L |
| Building and reviewing real systems | Lesson | Most docs, stated without naming the system |

## Verify before sharing
- Dated external facts: cloud product names (checked September 2026), the EU AI Act and EU Data Act dates.
- The platform review is anonymised and undated; date any number taken from it before quoting it.
- The platform review says a minimum healthy of 50% against one task "rounds down to none"; the ECS documentation says the minimum healthy count rounds up. This set follows the documentation (`architecture-principles`, B4); re-check the review's claim before reusing it.
- The case deck's company-level ticks were confirmed by a team, not read from configuration.
- Public pages move; the URLs were checked on 2026-10-26.

## Gaps only the team can fill
- Escalation policy, severity levels and the on-call ladder (F9) [unverified: owner to name].
- API standards and naming conventions (C4) [unverified].
- Design system, component library and accessibility standard (D2, D9) [unverified].
- Identity provider and SSO standards for customers (G2) [unverified].
- Where Kubernetes and Angular are used today (F4, D13) [unverified].
- Who owns the shared network and how to request changes (B3, F6) [unverified].
- ISO 27001 control owners and how controls are evidenced (G10) [unverified].
- Current AI-governance guidelines (H16) [unverified].

## Glossary
- Track — one theme of the set, lettered A–L.
- Label — where a principle comes from: Practice, Notes, Lesson or Case.
- Evidence badge — measured, derived or target. Aliases: provenance badge.
- P1, P2, P3 — core, depth, reference.
- FDE — forward-deployed engineer (`fde-delivery-approach`, I1).

## Open questions
- Should the set move to an internal repository and gain internal examples once they are agreed? [unverified]

## Sources
- Practice: each track doc lists its own public links under its Sources.
- Notes: private study notes; no ref (see Source labels).
