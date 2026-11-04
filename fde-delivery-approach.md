---
title: The FDE approach — from problem to production
id: fde-delivery-approach
doc_type: concept
summary: Track I. What a forward-deployed engineer does, and how they turn a customer problem into something that ships — discover before designing, structure a design answer in timed phases, scope a PoC with written criteria and exits, phase autonomy crawl-walk-run, re-engineer prototypes into features, present architecture, and answer gaps honestly.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [fde, forward-deployed-engineering, ai-engineering, discovery, solution-design, poc, phasing, presentation, stakeholders, consulting]
aliases: [forward deployed engineer, FDSE, deployment engineer, solutions engineer, discovery questions, PoC scoping, proof of concept, crawl walk run, success criteria, exit criteria, architecture presentation]
related: [engineering-principles-map.md, poc-to-production-case.md, ai-agent-engineering.md, evidence-and-decisions.md]
group: AI and delivery
order: 100
html_twin: fde-delivery-approach.html
---

# The FDE approach — from problem to production

## TL;DR
- A forward-deployed engineer (FDE) embeds with a customer, builds and ships in the customer's real systems, owns the outcome, and feeds what they learn back into the product.
- Discover before you design: six questions in order, plus three most people never ask; play it back before proposing anything.
- A design answer runs in timed phases: clarify, draw, walk one request, harden, phase it; state non-functionals as numbers.
- A PoC that ships has written success criteria, a 4–6 week time-box, a narrow scope on real data, go/narrow/stop exits and a path to production.
- Phase autonomy crawl, walk, run; a demo is a small fraction of the work, so carry identity from the first line.
- Notes are private study notes and carry no ref; Practice sources are public links.

## I1 What a forward-deployed engineer does
- Priority: P1 core. Sources: Practice, Notes.
- In practice: an FDE is a software engineer embedded with a company's customers to design, build and ship software, today usually AI, inside the customer's real systems, and to feed lessons back into the core product. The term comes from Palantir (https://www.iit.edu/blog/forward-deployed-engineer).
- In practice: AI companies hire FDEs because the bottleneck is deployment, not model capability; FDE job postings grew sharply from 2023 to 2026 (https://tomtunguz.com/fde-arms-race/).
- Principle, how the role differs from its neighbours:
  - An FDE builds it, in the customer's context, and stays until it runs.
  - A customer or solutions engineer designs and proves it, "one notch less deep" technically and "one notch more business".
  - A product engineer builds it once, for every customer.
- Principle: the customer's business is the requirement, and the next step is the ending of every conversation.
- Principle: "the interesting engineering is around the model, not in it": data plumbing, retrieval quality, evaluation and access control.
- Exercise: describe one recent piece of your work as an FDE would: the customer's outcome, what you built, what went back to the product.
- Check: what does an FDE feed back to the product team, and why does it matter?

## I2 Discover before you design
- Priority: P2 depth. Source: Notes.
- Principle, six questions in this order, with no product names before step 3: outcome ("what changes in 12 months if this works?"), current state, pain, constraints, success criteria, decision (who signs, who could block, by when).
- Principle, three questions most people never ask:
  - Prior attempts: what was tried, and why didn't it stick?
  - The sceptic: who thinks this is a bad idea, and why?
  - Error tolerance: is it advisory, or does it act with consequences? This sets human-in-the-loop, eval thresholds and guardrails.
- Principle: map the room: the buyer wants the number, the champion wants a story to retell, the sceptical architect wants the diagram and its failure modes, the incumbent team fears losing control.
- Principle: play it back ("so what I heard is…"), keep their numbers for the business case, and say when the answer is "not us".
- Exercise: run the six questions on a real internal request, then play it back in three sentences.
- Check: why is "is it advisory, or does it act?" the most senior question in an AI discovery?

## I3 Structure a design answer
- Priority: P2 depth. Source: Notes.
- Principle, timed phases in minutes: clarify 2–3, draw 2, walk one request 5, harden 5, phase it 2. Ask before you draw: users, data and who may see it, which decisions need a human, the cost of a wrong answer, how we will know it works.
- Principle, drawing rules: left to right; boxes are nouns, arrows are verbs; draw the boundaries; 6–10 boxes; a number beside each box.
- Principle, non-functionals as numbers: latency (a p99, or time to first token), throughput, availability (an SLO), durability (RPO and RTO), residency, cost (a ceiling and a unit cost).
- Principle, four questions to answer unprompted: is it working (SLOs, and evals for AI)? where is the bottleneck? how do the parts work together (contracts, timeouts, backpressure)? does it serve everyone well (p99, no noisy neighbour)?
- Principle, seven moves on the whiteboard: clarify the goal → today → target → path in waves → risks and mitigations → value (cost, KPI) → next step (a scoped PoC).
- Exercise: answer one design prompt in 16 minutes, timing each phase.
- Check: why walk one request through the diagram before hardening it?

## I4 Scope a PoC that ships
- Priority: P2 depth. Sources: Notes, Case.
- Principle: success criteria in writing first → a 4–6 week time-box → narrow (one use case, real data) → exit criteria → a path to production (owner, platform, on-call).
- Trap: "the PoC that never ships": it demos beautifully and nothing goes live, because nobody agreed what "working" meant or who would own it.
- Principle: carry identity from line one, even while the corpus is three PDFs on a laptop; otherwise the security review becomes a wall.
- Principle: the eternal pilot is a measurement problem: agree a golden set and a threshold before tuning.
- Case: seven targets and three exits (go, narrow, stop), agreed with the sponsor before anything is built; see `poc-to-production-case`, J4.
- Exercise: write success and exit criteria for a PoC you might run.
- Check: what does a written "stop" exit buy the team?

## I5 Phase it: crawl, walk, run
- Priority: P2 depth. Source: Notes.
- Principle: crawl: one workflow, read-only, a human approves everything, weeks not quarters, measured against the golden set; walk: auto-approve the categories the data shows are reliable, add a second workflow; run: multi-step autonomy where a wrong answer is cheap, human gates where it is not.
- Principle, one box of the reference architecture grows per scenario:

| Scenario | The box that grows |
|---|---|
| Document Q&A | Knowledge: ingest, chunking, hybrid search, citations |
| Agents and workflows | Tools and orchestrator: state graph, approvals, idempotency |
| Contact centre | Ingress: streaming, speech, a hard latency budget |
| Code assistant | Tools: repository as corpus, sandboxed execution |
| Generated media | Model gateway: async jobs, storage, moderation |
| NL-to-SQL | Tools: schema in prompt, query validation, row and cost caps |
| Bulk processing | No online path: batch endpoints, throughput over latency |

- Exercise: phase one AI feature into crawl, walk and run, with the gate between each.
- Check: what evidence moves a category from human approval to auto-approval?

## I6 From prototype to production
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: a convincing demo is a small fraction of the work; the rest is data plumbing, retrieval quality, evaluation harnesses and access control, none of it visible in the demo, "which is exactly why schedules built from the demo are wrong".
- Principle: re-engineering a demo into a feature adds identity, evals, limits, contracts, observability and an owner.
- Lesson: prototypes built quickly by non-engineers are valuable specifications; keep their behaviour and their users' words, and rebuild the plumbing.
- Exercise: list what a demo you have seen would need before production.
- Check: why are schedules built from a demo wrong?

## I7 Present architecture
- Priority: P2 depth. Sources: Notes, Case.
- Principle, narrative: where we are → where we are going → how → what proves it; keep the story consistent across slides; say what a PoC was built to prove, never that something is "not ready".
- Principle, slides: diagrams first, speaker notes, annexes as backup, evidence badges.
- Principle, a seven-slide shape: context, problem, options, solution (one diagram), plan, outcome, ask.
- Principle, demo value in three minutes: pain → outcome (a number) → how → means (time, cost, risk) → next step.
- Principle, in the room: restate the question; draw first; give numbers; say "I don't know — here's how I'd find out"; end every answer with the next step.
- Exercise: present one design in seven slides and stop on the ask.
- Check: why end every answer with a next step?

## I8 Answering a gap honestly
- Priority: P3 reference. Source: Notes.
- Principle: state the boundary plainly, then show you understand why the thing exists; understanding without experience is a respectable position, and claiming otherwise is the worst move.

## Invariants
- Success criteria are written before the build.
- No product names before the pain is understood.

## Gotchas
- A PoC without a stop exit becomes an eternal pilot.

## Glossary
- FDE — forward-deployed engineer: builds with the customer, in their systems, and owns the outcome.
- PoC — proof of concept; it ends on its criteria, not on a demo.
- Golden set — fixed cases that prove a PoC's quality (`ai-retrieval-evals-cost`, H12).

## Open questions
- One internal before-and-after example of a prototype taken to production [unverified].

## Sources
- https://www.iit.edu/blog/forward-deployed-engineer — what an FDE is.
- https://tomtunguz.com/fde-arms-race/ — why AI companies hire FDEs.
