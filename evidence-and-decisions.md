---
title: How we work — evidence, decisions and gates
id: evidence-and-decisions
doc_type: concept
summary: Track A. Badge every claim as measured, derived or target; expect prose to drift from code and make it fail a check; record lasting decisions as ADRs with the rejected option; gate merges on more than tests; communicate limits first.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [evidence, provenance, adr, documentation, docs-as-code, drift, quality-gates, code-review, ci, communication]
aliases: [evidence ladder, provenance badges, architecture decision record, decision log, docs drift, CI gate, PR checklist, conventional commits, honest communication]
related: [engineering-principles-map.md, architecture-principles.md, devops-cloudops-principles.md, fde-delivery-approach.md]
group: Foundations
order: 10
html_twin: evidence-and-decisions.html
---

# How we work — evidence, decisions and gates

## TL;DR
- Every number gets a badge: measured (from a run), derived (from versioned configuration), or target (intended). A target is never presented as achieved.
- Prose drifts from code because code has a gate and prose does not. Generate the facts a doc states, or make a check read them.
- A lasting decision is an ADR: context, decision, consequences, the option that lost, a revisit trigger and how to undo it.
- A merge gate is lint, a format check, tests and structural rules; "the tests pass on my machine" is not the gate.
- Name your limits before anyone asks, narrate trade-offs, and end on three ordered next steps.
- Notes are private study notes and carry no ref; Practice sources are public links.

## A1 Evidence ladder: measured, derived, target
- Priority: P1 core. Sources: Notes, Lesson.
- Principle: a claim is only as strong as what produced it. Three badges cover every number:
  - Measured — it came out of a run: a load test, a spend journal, a production metric.
  - Derived — it was read from configuration under version control, at a named revision. It describes what is declared, which is not always what runs.
  - Target — intended, with nothing behind it yet.
- Principle: scope every absence. "Not found in what we searched" never means "does not exist".
- Lesson: a review that searched only one infrastructure tool missed a scaling capability, because a second tool declared it after the first tool's apply. Search every tool that writes infrastructure before claiming something is absent.
- Lesson: a p95 latency from a one-user smoke run of a few dozen requests is measured, and tells you nothing about load. Ask for the load, the sample size, the environment and the p99.
- Exercise: badge ten claims taken from a README or a slide deck your team uses.
- Check: "Our p95 is 148 ms." Which badge, and what do you ask next?

## A2 Names lie and prose drifts
- Priority: P1 core. Sources: Notes, Lesson, Practice.
- Principle: a name is not a behaviour. Read what the code does.
- Lesson: an upload bucket was called a quarantine, yet nothing was ever quarantined: a clean object was promoted and anything else destroyed, with no copy kept.
- Principle: when docs and code disagree it is almost always the doc that is wrong, "because the code has a gate and the prose does not". The fix is to derive each figure in prose from its source so drift fails a check.
- Lesson: in one codebase, three context files written for coding agents stated wrong counts (modules, tools, directories scanned) within months of being written. Agent context files drift like any prose.
- In practice: docs as code means docs live in version control, in plain-text markup, go through review, and run automated checks (https://www.writethedocs.org/guide/docs-as-code/).
- Exercise: pick one number from a README and trace it to the line of code or config it claims to describe.
- Check: how would you make a number in a doc fail CI when it drifts?

## A3 Decisions are written down as ADRs
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: an ADR (architecture decision record) has a title, context, decision, status and consequences; a reversed decision is kept and marked superseded, not deleted (https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).
- Principle: a good ADR also names the option that lost and why; narrating the rejected option is what makes a trade-off credible.
- Lesson: two lines make ADRs useful years later:
  - A revisit trigger — a measurable condition that reopens the decision, such as "if module count exceeds 40".
  - Reversibility — the steps to undo it, written while the context is fresh.
- Lesson: supersede explicitly. When a later decision replaces part of an earlier one, link both ways, so a reader of the old ADR finds the new one.
- Exercise: write an ADR for a decision you made this week, including the option you rejected.
- Check: what makes a decision reversible, and why record that?

## A4 Docs that stay true
- Priority: P1 core. Sources: Practice, Lesson.
- Principle: a doc stays true when a tool writes it or a check reads it.
- Lesson, four habits that worked:
  - Generate diagrams from source and never hand-edit the output.
  - Let the deploy tool write the deploy ledger (what is live, which commit, when), not people. A ledger written by the tool once showed that a commit believed to be deployed never was.
  - Give coding agents a short, dense context file that says "verify against code before asserting".
  - Organise docs by audience and depth: a brief and a detailed doc for people, and the same pair for agents. This set follows that idea: an HTML page for people, an MD twin for agents.
- Lesson: when an AI assistant answers from the same docs a human reads, the two cannot drift apart.
- In practice: block merges that change behaviour without changing the doc that describes it (https://www.writethedocs.org/guide/docs-as-code/).
- Exercise: find what is live right now using the ledger, not memory.
- Check: why is a ledger written by the deploy tool better than one written by people?

## A5 Quality gates and the PR checklist
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: a merge gate is lint (errors fail), a formatter check, the tests, and structural rules such as a file-length cap. Run the full gate locally before pushing.
- Lesson: pushes went red twice while every test passed locally: once on the formatter check, once on a lint rule. The test run was green; the gate was not.
- Lesson: fix formatting with the formatter. If that pushes a file past the length cap, split the file rather than squeeze it.
- Notes: three scanners on every merge cover different risks: static analysis of code (SAST), dependency analysis (SCA) and a code-quality gate.
- In practice: approve a change once it definitely improves the overall health of the code, even if it is not perfect; "there is only better code" (https://google.github.io/eng-practices/review/reviewer/standard.html).
- In practice: commit messages as `<type>[optional scope]: <description>`; `fix` maps to a patch release, `feat` to a minor one, a breaking change to a major one (https://www.conventionalcommits.org/en/v1.0.0/).
- In practice: measure delivery with the DORA metrics: change lead time, deployment frequency, failed deployment recovery time, change fail rate and deployment rework rate (https://dora.dev/guides/dora-metrics/).
- Exercise: run the full gate on a branch, then list which step would have caught your last CI failure.
- Check: why is a green `npm test` not enough?

## A6 Honest technical communication
- Priority: P2 depth. Source: Notes.
- Principle: name your own limits before anyone asks; an audience decides early whether it is seeing evidence or marketing.
- Principle: narrate trade-offs: every choice comes with the option you rejected.
- Principle: say "I don't know; here is how I would find out" instead of guessing.
- Principle: end on three ordered next steps and stop.
- Principle: label the one claim that rests on someone's word rather than a file, so the reader sees it.
- Principle: confirm how a number was measured before you present it.
- Exercise: present one design in five minutes, opening with its weakest point and ending on a next step.
- Check: what does naming a weakness first buy you?

## Invariants
- NEVER present a target as achieved.
- NEVER state an absence without the scope it was searched over.
- NEVER hand-edit a generated artefact; change its source.

## Gotchas
- A derived number describes declared configuration; a second infrastructure tool can change what runs (A1).
- Context files for coding agents drift like any prose (A2).

## Glossary
- ADR — architecture decision record. Aliases: decision record.
- Evidence badge — measured, derived or target. Aliases: provenance badge.
- Deploy ledger — a file the deploy tool updates on every deploy. Aliases: deploy record.
- Quality gate — a check that blocks a merge. Aliases: CI gate.
- DORA — the DevOps Research and Assessment programme and its delivery metrics.

## Open questions
- Does the team have an ADR template and a home for ADRs? [unverified]

## Sources
- https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions — Nygard's ADR format.
- https://www.writethedocs.org/guide/docs-as-code/ — docs as code.
- https://google.github.io/eng-practices/review/reviewer/standard.html — the code review standard.
- https://www.conventionalcommits.org/en/v1.0.0/ — commit message convention.
- https://dora.dev/guides/dora-metrics/ — delivery metrics.
