---
title: AI retrieval, evaluation and cost principles
id: ai-retrieval-evals-cost
doc_type: concept
summary: Track H, part 2. RAG as two pipelines, retrieval quality measured before prompt edits, hybrid search with re-ranking and contextual chunks, when a knowledge graph earns its cost, four layers of evaluation cheapest first, cost and latency arithmetic, the model gateway, failure triage, AI agents in the engineering workflow, governance and a readiness checklist.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [ai, rag, retrieval, chunking, hybrid-search, bm25, reranking, contextual-retrieval, knowledge-graph, evaluation, golden-set, llm-judge, cost, latency, model-gateway, guardrails, governance, eu-ai-act, nist-ai-rmf]
aliases: [retrieval augmented generation, reciprocal rank fusion, RRF, HyDE, recall@k, lost in the middle, GraphRAG, evals, LLM as judge, prompt caching, model routing, hallucination triage, dev agents, AI governance, readiness checklist]
related: [engineering-principles-map.md, ai-agent-engineering.md, security-identity-principles.md, poc-to-production-case.md]
group: AI and delivery
order: 90
html_twin: ai-retrieval-evals-cost.html
---

# AI retrieval, evaluation and cost principles

## TL;DR
- RAG is two pipelines: offline ingest and online query. Most "hallucination" is a retrieval miss, so measure recall@k before editing a prompt.
- Hybrid search (vectors plus BM25) fused by rank, then re-ranked, beats vectors alone; more context is not better.
- Evaluate in four layers, cheapest first: deterministic checks, a golden set grown from real failures, a calibrated judge, online signals. Look at the data.
- Do the cost arithmetic aloud; the first lever is not calling the model at all.
- Uptime is not correctness: pin model versions and run evals in CI.
- Notes are private study notes and carry no ref; Practice sources are public links.

## H9 RAG: two pipelines
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: offline ingest is parse → chunk → enrich (metadata, access control, titles) → embed → index; online query is rewrite → hybrid search → fuse → re-rank → assemble → generate → verify citations.
- Principle: a new embedding model means a full re-index, because vectors from different models are not comparable.
- Principle: identity flows into the retrieval filter; a user must never retrieve a chunk they could not open as a document.
- Lesson: at small scale, a database's built-in vector index is enough; a dedicated vector service with always-on nodes pays off only past roughly 100k vectors or sustained high query rates.
- Exercise: draw both pipelines for one document set you know.
- Check: why does changing the embedding model force a full re-index?

## H10 Retrieval quality
- Priority: P2 depth. Sources: Notes, Practice.
- Chunking: split on structure and keep the heading; 300–800 tokens with 10–20% overlap as a starting point; small-to-big retrieval.
- In practice, contextual retrieval: prepend a short chunk-specific context before embedding and indexing. On the published benchmark, failed retrievals fell 35% with contextual embeddings, 49% adding contextual BM25, 67% adding re-ranking (https://www.anthropic.com/news/contextual-retrieval).
- Hybrid search: vectors plus BM25, because identifiers and acronyms need exact matches; scores have no shared unit, so fuse ranks with Reciprocal Rank Fusion, `k = 60`.
- Re-ranking: retrieve 50–100 candidates broadly, re-rank to 5–10.
- Query rewriting and HyDE.
- In practice, assembly: models use information best at the start or end of a long context and worst in the middle (https://arxiv.org/abs/2307.03172). More context is not better.
- Metrics: recall@k first, then MRR, nDCG@k and context precision. "Was the evidence retrieved?" comes before any prompt change.
- Exercise: measure recall@k on ten real questions.
- Check: why can a vector score and a BM25 score not be added directly?

## H11 Knowledge graphs
- Priority: P3 reference. Source: Notes.
- Principle: vector search cannot answer multi-hop questions, global themes or counts. Three levels of adoption: graph as metadata, graph as retriever, graph as the knowledge. Entity resolution is the hard part; design the schema first; access control must hold on every hop; reuse graphs you already own (a CMDB, an org chart).
- Exercise: write one question your users ask that vector search cannot answer.
- Check: why does a multi-hop query that returns nothing usually point to entity resolution?

## H12 Evaluation: four layers
- Priority: P2 depth. Sources: Notes, Practice.
- Principle, cheapest first:
  1. Deterministic checks: valid JSON, required fields, citations present and actually retrieved, a latency budget. Aim for 60–70% of the eval surface; run them in CI and in production.
  2. A golden set of 50–100 cases, grown from real failures; every bug report becomes a test.
  3. An LLM judge, calibrated against human scores before it is trusted.
  4. Online signals, including how much the user edits a draft before sending it.
- In practice: start with cheap assertions per feature, then human and model review of logged traces with simple pass/fail labels, then A/B tests for mature products; never stop looking at the data (https://hamel.dev/blog/posts/evals/).
- In practice, judges: strong model judges reach over 80% agreement with human preferences, but show position, verbosity and self-enhancement bias (https://arxiv.org/abs/2306.05685). Prefer pairwise comparison, give a rubric with examples, and never make a judge the only gate.
- RAG metrics: context recall, context precision, faithfulness, answer relevance.
- Principle: evals run in CI on every prompt change; pin model versions; "a system can be 100% available and 40% wrong".
- Exercise: write ten golden cases from real failures and one deterministic check.
- Check: why calibrate a judge before trusting it?

## H13 Cost and latency
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: do the arithmetic aloud: requests × tokens × price.
- Where the cost is: input tokens dominate in RAG; output is priced several times higher; calls per request multiply; retries hide.
- Where the latency is: time to first token versus total time; streaming improves the first, not the second; run independent steps concurrently.
- Levers in order: do not call the model; cache the stable prefix; route by difficulty; send less context; batch offline work; cap output.
- Principle: speak cost per resolved task, not per token.
- Lesson: a short agent turn on a small, fast model costs about a tenth of a cent; media generation, not text, usually dominates an AI product's bill.
- Exercise: cost one feature per 1,000 requests, then apply the first two levers.
- Check: why does streaming change what users feel but not what you pay?

## H14 Model choice and the gateway
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: a model gateway handles routing, version pinning, retries, fallback, caching and per-tenant quotas; small models do routing and extraction; fine-tuning changes behaviour, not facts; distillation is safe only after evals exist.
- Lesson: let an admin override the model through config with a short cache, so a model change is a config change with an eval run, not a deploy.
- Lesson: "being on a cloud is not an argument for" that cloud's model; choose on the eval, the price and the terms.
- Exercise: list which steps of one feature could move to a smaller model.
- Check: why does fine-tuning not fix missing facts?

## H15 Guardrails, limits and failure triage
- Priority: P2 depth. Sources: Notes, Practice.
- Failure modes: hallucination, injection, tool failure, runaway loops, context overflow, provider outage, silent drift, and unbounded consumption (https://genai.owasp.org/llm-top-10/).
- Triage table:

| Symptom | Check first |
|---|---|
| Confident, wrong answer | Whether the evidence was retrieved (recall@k) |
| Right facts, wrong emphasis | Context ordering; key evidence buried in the middle |
| Great in testing, poor in production | The golden set; rebuild it from real logs |
| Quality dropped with no deploy | Model alias moved, re-ingested index, changed data, provider default |
| Agent loops | Step cap, progress check, actionable tool errors |
| Works for you, fails for a colleague | Permissions or a stale index; reproduce as them |

- Principle: degrade, don't disappear: name the degraded behaviour for provider down, retrieval empty and tool timeout.
- Principle: screen what goes into and comes out of the model with a managed filter when records from other systems enter the prompt.
- Exercise: write the degraded behaviour for each failure of one feature.
- Check: a colleague gets a wrong answer you cannot reproduce. What do you check?

## H16 AI agents in the engineering workflow
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle, two layers of agents: machine agents run the system and cannot change it; AI agents change the system and cannot run it. Each AI agent declares what it may write, so the worst bad run is a pull request a human declines.
- Notes: a review agent produced false approvals, so its output is a comment, never a merge.
- Lesson, working with coding agents: planner → workers → integrator; freeze contracts before implementation; investigate read-only before fixing a bug; keep a dense context file for agents and verify it against code (`evidence-and-decisions`, A4).
- Needs team input: which team agents can be shared as examples, and the current AI-governance guidelines [unverified].
- Exercise: write the "may write" declaration for one agent you would build.
- Check: why is a comment-only review agent safe to leave unattended?

## H17 AI governance and regulation
- Priority: P3 reference. Sources: Notes, Practice.
- Principle: under the EU AI Act most internal assistants are limited risk, so the main duty is transparency; GDPR still applies; ISO/IEC 42001 is the AI analogue of ISO 27001; sector rules; five questions to ask at the start of any engagement.
- In practice: the NIST AI Risk Management Framework 1.0 was released in January 2023, with a generative AI profile, NIST AI 600-1, in July 2024; NIST states it is being revised (https://www.nist.gov/itl/ai-risk-management-framework).
- These dates and names move; re-check before quoting [unverified].

## H18 The readiness checklist
- Priority: P3 reference. Source: Notes.
- The field guide's tick-boxes for retrieval, evaluation, operations and governance, for example: every retrieved item carries a stable id cited in the answer; deterministic checks run in production; the audit record holds retrieved ids and the exact model version; someone has answered in writing what happens on an erasure request.

## Invariants
- Measure retrieval before editing a prompt.
- Pin model versions; an alias that moves is an unannounced deploy.

## Gotchas
- A golden set written by the team is cleaner than real traffic; rebuild it from logs.
- A judge rating 1–5 clusters around 4; use pairwise comparison.

## Glossary
- RAG — retrieval-augmented generation.
- BM25 — a keyword ranking function. Aliases: sparse retrieval.
- RRF — reciprocal rank fusion of several rankings.
- HyDE — hypothetical document embeddings: embed a model-written answer to search with.
- recall@k — share of questions whose evidence is in the top k results.
- Golden set — fixed cases with expected outputs.

## Open questions
- Current EU AI Act dates, re-checked at publication [unverified].

## Sources
- https://www.anthropic.com/news/contextual-retrieval — contextual retrieval.
- https://arxiv.org/abs/2307.03172 — lost in the middle.
- https://arxiv.org/abs/2306.05685 — LLM-as-a-judge and its biases.
- https://hamel.dev/blog/posts/evals/ — levels of evaluation.
- https://www.nist.gov/itl/ai-risk-management-framework — NIST AI RMF.
