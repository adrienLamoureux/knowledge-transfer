---
title: AI agent engineering principles
id: ai-agent-engineering
doc_type: concept
summary: Track H, part 1. The orchestrator sits in the middle, not the model; use workflows before agents; tools are prompt text whose class decides control flow, and code owns every write; design for blast radius against prompt injection; reasoning patterns, memory layers and context as a budget; state graphs with checkpoints; MCP versus A2A.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [ai, agents, llm, orchestration, workflows, tools, function-calling, prompt-injection, excessive-agency, react, memory, context-engineering, langgraph, checkpoints, human-in-the-loop, mcp, a2a]
aliases: [agent loop, orchestration ladder, prompt chaining, routing, evaluator optimizer, tool calling, tool classes, intent tools, confirm before write, lethal trifecta, self-injection, plan and execute, reflection, working memory, episodic memory, compaction, state graph, checkpointer, interrupt, Model Context Protocol, Agent2Agent, agent card]
related: [engineering-principles-map.md, ai-retrieval-evals-cost.md, security-identity-principles.md, fde-delivery-approach.md]
group: AI and delivery
order: 80
html_twin: ai-agent-engineering.html
---

# AI agent engineering principles

## TL;DR
- An AI system is seven boxes with the orchestrator in the middle; the model is the easiest part to change.
- Use the simplest pattern that works: a workflow follows code paths you wrote, an agent directs itself. If you can draw the flowchart, build the flowchart.
- A tool's name, description and schema are prompt text; the tool's class decides control flow, and code owns every write.
- Design for blast radius, not detection: identity from the edge, reads scoped, writes behind approval, never private data plus untrusted content plus a way out in one agent.
- Most agents need only working and episodic memory; context is a budget you spend.
- Notes are private study notes and carry no ref; Practice sources are public links.

## H1 The mental model: five layers, seven boxes
- Priority: P1 core. Source: Notes.
- Principle, five layers: models, knowledge, control, quality, governance.
- Principle, seven boxes of a reference architecture: ingress (identity, rate limit, trace id), orchestrator, retrieval, model gateway, tools, state store, observability. The orchestrator sits in the middle, not the model: "That is what you actually own and operate."
- Four pairs that get confused: RAG versus fine-tuning; vector database versus RAG; agent versus chain; knowledge graph versus vector search.
- Two failure patterns: the eternal pilot (good enough to keep hope alive, never shipped) and the wall at the security review (it works, then someone asks who can see what).
- Exercise: draw the seven boxes for one AI feature you know and mark which are missing.
- Check: why is the orchestrator, not the model, the centre of the design?

## H2 Workflows before agents
- Priority: P1 core. Sources: Practice, Notes.
- In practice: workflows orchestrate models and tools through predefined code paths; agents let the model direct its own process and tool use. Find the simplest solution and add complexity only when needed (https://www.anthropic.com/engineering/building-effective-agents).
- In practice, five workflow patterns: prompt chaining, routing, parallelisation (sectioning or voting), orchestrator-workers, evaluator-optimiser (https://www.anthropic.com/engineering/building-effective-agents).
- Principle, the orchestration ladder from most to least constrained: single call → chain → router → parallel → evaluator-optimiser → agent loop. Real systems compose them.
- Principle: every unit of autonomy costs predictability, latency, money and debuggability on every request. The test: can you draw the flowchart? If so, build it.
- Principle: do not build an agent when the workflow is deterministic, one step, latency-critical, high blast radius, or impossible to evaluate.
- Exercise: take one proposed agent and redraw it as the lowest rung that works.
- Check: what makes a task genuinely need an agent loop?

## H3 Tool design and tool classes
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: the model sees only the tool's name, description and schema; edit them like prompt text.
- In practice (https://www.anthropic.com/engineering/writing-tools-for-agents):
  - A few purpose-built tools for high-impact workflows, not one tool per API endpoint.
  - Namespace related tools so the model can tell them apart.
  - Return high-signal fields (names, not opaque ids), with a concise or detailed mode.
  - Paginate, filter and truncate by default; make error messages say how to fix the input.
  - Describe each tool as you would to a new colleague; use unambiguous parameter names such as `user_id`, not `user`.
  - Evaluate tools on realistic multi-step tasks and read the transcripts.
- Principle: say when not to use a tool; constrain in the schema, not the prose; separate reads from writes in code; idempotency keys and per-run caps on writes.
- Lesson, tool classes decide control flow:

| Class | What happens | Example |
|---|---|---|
| Server-dispatch | The backend runs it, then the model gets a short closing turn | Search, generate an image, list my items |
| Client-action | The browser applies it | Change the theme |
| Intent (confirm) | Returns a proposal, never a write; the user approves; the ordinary API runs | Delete, share, continue a story |
| Delegation | Handed to another agent | Ask a critic agent for a review |
| Static | A canned answer, no model call | "What can you do?" |

- Lesson: code owns every write. The confirm path calls the same REST endpoint a human would, so authorisation lives in one place.
- Lesson: cap the closing turn after a server-dispatched tool to a small token budget; the result is already on screen.
- Lesson: two tools with similar outputs are told apart only by their descriptions; say in each when to use the other.
- Exercise: add a read tool, then a confirm-gated write tool.
- Check: why does the confirm path reuse the human REST endpoint?

## H4 Agent security: design for blast radius
- Priority: P1 core. Sources: Notes, Practice, Lesson.
- Principle, five control points:
  1. Identity: resolved at the edge and carried through.
  2. Data boundary: no training on your data, minimal retention, region pinning.
  3. Injection: indirect injection arrives with the evidence (a ticket, a PDF, a tool description); design for blast radius, not detection.
  4. Tool gate: reads allow-listed and scoped; writes behind approval.
  5. Audit: keep the retrieved document ids, "the field teams forget and auditors ask for".
- In practice: prompt injection is LLM01 and excessive agency LLM06 in the OWASP Top 10 for LLM applications, 2025 edition (https://genai.owasp.org/llm-top-10/).
- In practice, the lethal trifecta: access to private data, exposure to untrusted content, and a way to communicate externally. Never combine all three in one agent (https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/).
- In practice: treat tool descriptions and annotations from a server you do not trust as untrusted, and get the user's consent before a tool runs (https://modelcontextprotocol.io/specification/latest).
- Lesson, self-injection: stored state that re-enters the prompt (preferences, memories, summaries) is untrusted. Validate it against an allow-list on read and on write, and drop what fails.
- Notes: "A record that says 'ignore your rules' must stay a record".
- Exercise: list every string that enters your prompt and mark who controls it.
- Check: why validate stored preferences on read as well as on write?

## H5 Reasoning patterns
- Priority: P2 depth. Sources: Notes, Practice.
- Principle:
  - ReAct (reason → act → observe → repeat): adapts mid-task; many calls; can loop. From the 2022 paper that interleaves reasoning traces with tool actions (https://arxiv.org/abs/2210.03629).
  - Plan-and-execute: plan once, then run; cheaper and predictable; weak if the world changes.
  - Reflection (evaluator-optimiser): a second pass critiques the first; a real quality gain at about double the cost; needs a hard stop.
  - Router: classify, then hand off; often the right answer, and often not "an agent".
- Principle: "agentic RAG" only changes who decides when to retrieve.
- Exercise: pick a pattern for three tasks and justify each in one sentence.
- Check: when does reflection pay for its doubled cost?

## H6 Memory layers and context management
- Priority: P2 depth. Sources: Notes, Practice, Lesson.
- Principle, four layers: working (the current step, in memory), episodic (what happened, an append-only store), semantic (facts and preferences, a vector store), procedural (prompts and tools, versioned config). Most enterprise agents need working and episodic only.
- In practice: context is a finite resource with diminishing returns, an attention budget. For long tasks use compaction (summarise and restart), structured note-taking outside the context, and sub-agents with clean contexts that return summaries (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents).
- Lesson: compact on a threshold: past a fixed number of turns, fold the oldest messages into a summary, and bill the summariser's tokens to the same cost record.
- Lesson: keep short-lived task memory apart from long-lived identity memory, so task context does not pollute what the assistant remembers about the person.
- Lesson: a session id that doubles as the memory namespace gives per-conversation history with no schema change.
- Exercise: list what your agent remembers, in which layer, and for how long.
- Check: why keep task memory apart from identity memory?

## H7 State graphs and durable agents
- Priority: P2 depth. Sources: Notes, Practice.
- Principle, the six pieces of a state graph: state (a typed record every node reads and updates); reducer (how a field merges: messages append, others overwrite); node (state → partial update); edge (static or conditional); checkpointer (saves state after each step); interrupt (pause before a node until a human approves).
- Principle: reducers resolve concurrent writes within a step. Checkpoints enable resume, replay and fork.
- In practice: LangGraph is a low-level orchestration framework and runtime for long-running, stateful agents, with durable execution, human-in-the-loop and memory (https://docs.langchain.com/oss/python/langgraph/overview); checkpointers keep a thread's state for continuity, human review, time travel and fault tolerance (https://docs.langchain.com/oss/python/langgraph/persistence).
- Principle: human approval has three verbs: approve, edit then approve, reject with a reason; gate everything at first, then auto-approve the categories that prove reliable.
- Principle: a loop needs three exits: the model finishes, the step cap trips, or a no-progress check fires. Budget tokens and wall-clock time too.
- Exercise: draw one agent as a state graph with its checkpoints and one interrupt.
- Check: what does a checkpointer give you that a retry does not?

## H8 Protocols: MCP and A2A
- Priority: P2 depth. Sources: Practice, Lesson.
- In practice: MCP (Model Context Protocol) connects applications to context and tools over JSON-RPC 2.0. Servers offer resources, prompts and tools; hosts must get the user's consent before invoking a tool (https://modelcontextprotocol.io/specification/latest).
- In practice: A2A (Agent2Agent) is an open protocol for agents to discover each other and delegate tasks; an agent card is a JSON document describing an agent's skills, endpoint and authentication; MCP connects an agent to tools, A2A connects agents to agents (https://a2a-protocol.org/latest/, https://a2a-protocol.org/latest/specification/).
- Principle, rule of thumb: if the far side needs its own model to do its job, use A2A; otherwise it is a tool, so use MCP.
- Lesson: A2A between two agents in the same process is pure overhead; call the function.
- Lesson: do not expose an agent over A2A when its tools spend one user's budget; another agent could spend it.
- Lesson: an agent card must advertise the public URL, not the internal one it was built with; a protocol server behind a load balancer must run stateless (`data-distributed-systems`, E9).
- Exercise: decide MCP or A2A for three integrations and justify each.
- Check: why is A2A the wrong choice inside one process?

## Invariants
- The model never supplies identity; the user comes from the verified session (H4, `security-identity-principles` G3).
- The model proposes irreversible actions and never performs them (H3).
- Guardrails and limits run before the model (`security-identity-principles`, G9).

## Gotchas
- Tool errors with no detail stall the loop; make them actionable.
- Stored state that re-enters the prompt can steer it (H4).

## Glossary
- Orchestrator — the code that decides what happens next. Aliases: control layer.
- Workflow — models and tools on code paths you wrote; contrast with an agent, which directs itself.
- Tool class — server-dispatch, client-action, intent, delegation or static.
- Self-injection — stored state that re-enters the prompt and steers it.
- Reducer — how a state field merges in a graph.
- Checkpointer — saves graph state after each step.
- MCP, A2A — Model Context Protocol; Agent2Agent.

## Open questions
- Which of the team's internal agents can be shown as examples? [unverified]

## Sources
- https://www.anthropic.com/engineering/building-effective-agents — workflows and agents.
- https://www.anthropic.com/engineering/writing-tools-for-agents — writing tools for agents.
- https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — context engineering.
- https://genai.owasp.org/llm-top-10/ — OWASP Top 10 for LLM applications.
- https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/ — the lethal trifecta.
- https://modelcontextprotocol.io/specification/latest — the MCP specification.
- https://a2a-protocol.org/latest/ — the A2A protocol.
