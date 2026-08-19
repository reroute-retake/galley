# Galley — Subsequent Phases Roadmap

**Version:** 1.0
**Status:** Roadmap — not a spec. Phase-2+ details are planned when we get there, using Galley itself.
**Date:** 2026-08-19
**Companion:** `galley-phase-1-specification.md` (v0.1 — the shippable milestone this roadmap builds on)

---

## 1. Purpose

Galley v0.1 (the Assisted Workstation) is Phase 1 — the manually-driven container-based development environment. This document lays out the six subsequent milestones (v0.2 → v0.7) that progressively add capabilities toward the eventual **autonomous software factory** vision, and — critically — names the existing open-source projects we will adapt from rather than build from scratch.

Two ground rules:

1. **Every phase after v0.1 is built using Galley itself.** The v0.1 manual workflow surfaces the actual gaps. We plan each phase's details when we get there.
2. **Adopt over invent.** Where a mature open-source project has already solved a piece of what we need, we adapt it (respecting licenses) rather than writing from zero.

---

## 2. Guiding principles for every phase

These stay true from v0.1 through v0.7. Any new capability that violates one of these is out of scope.

- **Governance-first.** Constitution, ubiquitous language, invariants, requirements, architecture, ADRs — read at Step 1 of every agent, uniformly across all providers.
- **Anti-lock-in.** Multiple providers usable interchangeably through their native harnesses. Each provider's ToS respected via its own subscription.
- **Cost-bounded.** Flat monthly subscription cost + local model. No pay-as-you-go surprises.
- **Deterministic control plane.** Transitions between SDLC steps are code, not LLM decisions. LLMs decide *what to write*; code decides *what happens next*.
- **Just-enough context.** Governance is relevance-filtered. Code context comes from Serena + repomap. Compressed handoff artifacts (context packs) move between steps.
- **Simplicity.** Every capability earns its keep. Deferred anything is deferred honestly.
- **Local-first.** Core development is possible with only local models. Cloud subscriptions are accelerators.

---

## 3. The roadmap at a glance

| Version | Milestone | Built how | Rough scope |
|---|---|---|---|
| **v0.1** | Assisted Workstation | Manually (spec: `galley-phase-1-specification.md`) | Substantive |
| **v0.2** | State + capture | Using v0.1 | Small |
| **v0.3** | Full-fleet automation | Using v0.2 | Substantive |
| **v0.4** | Self-hosting | Using v0.3 | Substantive |
| **v0.5** | Observability + cost governance | Using v0.4 | Medium |
| **v0.6** | Earned autonomy | Using v0.5 | Medium |
| **v0.7** | Autonomous factory foundation | Using v0.6 | Substantive |

---

## 4. v0.2 — State + capture

The manual workflow of v0.1 starts being *recorded*. Nothing gets automated yet; every step still runs through a harness by hand. Galley now knows what happened, so downstream phases have provenance to build on.

### 4.1 Capabilities

- SQLite state database at `.galley/state.db` — workflows, tasks, artifacts, clarifications, governance-touched-per-workflow
- CLI additions:
  - `galley workflow start|list|inspect|advance`
  - `galley task add|update|list`
  - `galley artifact record`
  - `galley clarification list|resolve`
- Manual state transitions — engineer runs `galley workflow advance <state>` after each SDLC step
- Governance-touch tracking — the DB records which requirements / invariants / ADRs each workflow read
- Retrospect skill runs against the recorded state instead of manual notes
- `galley workflow report <id>` — human-readable summary of a completed workflow

### 4.2 Projects to reference

- **[SQLite is All You Need for Durable Workflows](https://obeli.sk/blog/sqlite-is-all-you-need-for-durable-workflows/)** — architectural precedent for state persistence on SQLite alone
- **[Cloudflare Workflows V2](https://blog.cloudflare.com/durable-objects-alarms-and-workflows/)** — reference for SQLite-backed workflow state at scale
- **[Vibe Kanban](https://vibekanban.com/)** ([source](https://sourceforge.net/projects/vibe-kanban.mirror/)) — Kanban-style task-state UI patterns; adapt the concept, not the code
- **[Agent Kanban](https://agent-kanban.dev/)** — task-with-status recording; leader/worker separation ideas
- **[Aider's git integration](https://github.com/paul-gauthier/aider)** — commit hygiene patterns; how the tool interacts with git history
- **[Anthropic MCP fetch server](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)** — pattern for extension points

**Adopt:** SQLite step-log pattern, workflow-as-record model, minimal CLI surface. **Skip:** framework overkill (Temporal, Prefect) — a hand-rolled state DB is enough.

---

## 5. v0.3 — Full-fleet automation (Harness Adapter + all five adapters)

First automated agent invocation. `galley task run <task-id>` invokes an agent in a harness without the engineer copy-pasting anything. All five harnesses (OpenCode, Claude Code, Codex CLI, Antigravity, Cursor CLI) get automated adapters in this milestone.

### 5.1 Capabilities

- Harness Adapter interface — subprocess management, event stream parsing, session lifecycle
- Canonical Event schema (message / tool-call / artifact-written / usage / error / session events) as Pydantic v2 models
- Context assembly automation — governance filtering + repomap + Serena results collated into the agent's prompt
- Agent + Skill loader with schema validation
- **Five adapters** delivered incrementally:
  1. OpenCode adapter first — shapes the interface against the minimum-feature-surface harness
  2. Then Codex CLI, Antigravity, Cursor CLI in parallel (each ~300 LOC once interface stabilizes)
  3. Claude Code adapter last — validates against the maximum-feature-surface harness
- `galley task run <id>` — engineer selects a task; Galley invokes the appropriate agent

### 5.2 Projects to reference

**Adapter architecture:**

- **[OpenHands SDK v1](https://github.com/All-Hands-AI/OpenHands)** — event-stream + append-only EventLog architecture. Their `Conversation` + stateless `Agent` pattern is the direct model for our Harness Adapter.
  - **Adopt:** event-stream design, append-only log, stateless-agent + stateful-session pattern.
  - **Skip:** their runtime and specific tool schemas.
- **[Aider Coder taxonomy](https://github.com/paul-gauthier/aider)** — patterns for different edit formats (EditBlockCoder, ArchitectCoder, etc.).
  - **Adopt:** the "architect+editor" split as a task-level option.
  - **Skip:** the coder-per-format proliferation.

**Per-harness native docs (each adapter's implementation source):**

- **[Claude Code docs](https://docs.claude.com/en/docs/claude-code)** — skills, subagents, hooks, plugins, output formats
- **[Codex CLI docs](https://developers.openai.com/codex/cli)** and [customization stack guide](https://codex.danielvaughan.com/2026/04/12/codex-cli-customisation-stack-unified-system/) — AGENTS.md, skills, MCP, subagents, plugins
- **[Antigravity CLI docs](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/)** — successor to Gemini CLI
- **[OpenCode docs](https://open-code.ai/en/docs/config)** — `opencode.json` config, agents/, commands/, plugins/, skills/, tools/
- **[Cursor CLI docs](https://cursor.com/docs/cli/overview)** and [headless mode](https://cursor.com/docs/cli/headless) — `--print`, `--force`, `--yolo`, agent mode

**Model access (in-adapter):**

- **[anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python)** — for Anthropic-hosted models (only if adapter needs direct model access)
- **[openai Python SDK](https://github.com/openai/openai-python)** — for OpenAI + any OpenAI-compatible endpoint (local model via LiteLLM proxy)
- **[google-genai Python SDK](https://github.com/googleapis/python-genai)** — for Google models

**Note:** Galley's own `ModelClient` should mostly be irrelevant in this phase — harnesses own their model calls per §24.1 in Phase 1. Direct SDK use is only for utility calls (retrospective summarization, etc.).

**Adopt:** OpenHands' event-stream architecture as the canonical Event schema shape. Per-harness docs as the implementation reference for each adapter.

---

## 6. v0.4 — Self-hosting (full SDLC automation via LangGraph)

The whole SDLC runs automatically end-to-end. Galley develops Galley through the workflow engine. **This is the self-hosting milestone.**

### 6.1 Capabilities

- **LangGraph** as the state-machine substrate — deterministic transitions, `interrupt()` for approvals, checkpointed replay, time-travel debug
- Workflow orchestrator that walks the SDLC steps automatically
- Deterministic Checkpoint evaluator (presence + shape checks — no LLM judgement in transitions)
- Approval queue (file-drop mechanism) — engineer approves gates asynchronously
- Automated: initializer, analyst, planner, architecture-gate, implementer, tester
- Automated multi-dimensional review — architecture-reviewer + security-reviewer + quality-reviewer run in parallel using **different harnesses than the implementer** (enforced by construction, not convention)
- Automated: triage-agent, ship-agent, archive-agent
- `galley workflow start --from-arg "requirement text"` — engineer types a requirement; Galley takes it end-to-end

### 6.2 Projects to reference

**Orchestration substrate:**

- **[LangGraph](https://github.com/langchain-ai/langgraph)** — 126k+ stars, mainstream OSS state-machine framework. **This IS our substrate.** Native `interrupt()` for approvals, checkpointer for persistence, time-travel debug. Anthropic's engineering guidance calls this "the winning 2026 approach."
  - **Adopt:** the framework directly. Use standalone (LangGraph can be used without LangChain-full).
  - **Skip:** LangChain-full's other primitives; stay lean.
- **["A Deterministic Control Plane for LLM Coding Agents" (arxiv 2606.26924)](https://arxiv.org/pdf/2606.26924)** — the reference implementation `Rel(AI)Build` uses SHA-256 content addressing, HMAC-stamped lockfiles, tiered permissions, phase state machine.
  - **Adopt:** the architectural pattern — deterministic gates, code-checked transitions.
  - **Skip:** their HMAC-signing complexity for solo use.

**Workflow shape and SDLC discipline:**

- **[GSD (Get Sh*t Done)](https://docs.opengsd.net/core/introduction)** — 48k+ stars, spec-driven dev for AI coding agents. Plan / Execute / Review workflow with fresh context per phase. Works across 12+ harnesses.
  - **Adopt:** the fresh-context-per-phase model (avoids context rot), slash-command-per-workflow-step pattern, the discipline of "if a session gets long, split it."
  - **Skip:** GSD's tight coupling to Claude Code idioms; we've already generalized.
  - **Direct reference for:** planner, architect, implementer skill bodies.
- **[Superpowers (obra/superpowers)](https://github.com/obra/superpowers)** — 224k+ stars, agentic skills framework.
  - **Adopt:** brainstorming, subagent-driven-development, verification-before-completion, systematic-debugging patterns.
  - **Skip:** mandatory-for-everything stance.
- **[Compound Engineering plugin (EveryInc)](https://github.com/EveryInc/compound-engineering-plugin)** — 29 agents, 22 slash commands, 20 skills. Plan/Execute/Review 80/20 workflow.
  - **Adopt:** the compounding-improvement principle, review-heavy 80/20 workflow ratio.
  - **Skip:** their specific 29-agent count.
- **[AI-SDLC Framework](https://github.com/ai-sdlc-framework/ai-sdlc)** — declarative resource types (Pipeline, AgentRole, QualityGate, AutonomyPolicy, AdapterBinding).
  - **Adopt:** the spec/status split pattern (user declares `spec`, system writes `status`); the progressive-enforcement model (advisory → soft-mandatory → hard-mandatory) for our Checkpoints; the cross-harness review independence rule (already in Phase 1).
  - **Skip:** Kubernetes-style API maturity ceremony; compliance postures (EU AI Act, ISO 42001).

**Reference architecture (not adopted, studied):**

- **[Anthropic engineering guidance on state-machines-over-chatbots](https://www.anthropic.com/engineering)** — for hybrid pattern rationale.

**Adopt:** LangGraph as substrate; GSD / Superpowers / Compound Engineering skill patterns as content sources; AI-SDLC's spec/status split for our Checkpoint resource.

---

## 7. v0.5 — Observability + cost governance

Understanding what workflows actually cost and how they're performing.

### 7.1 Capabilities

- OTel tracing with **Langfuse** as the default backend
- Per-workflow cost accounting — subscription-tier consumption tracked per provider
- Per-model pricing table refreshed quarterly
- Cost budgets with warning thresholds (`space.yaml` `cost_policy`)
- Alerts when a subscription is approaching its monthly cap
- Evaluation harness — `galley eval report` for trends across workflows
- Retrospect agent automated (per-workflow retrospective)
- Metrics: task success rate, retry rate, model win-rate per role, cost per workflow, time-to-decision

### 7.2 Projects to reference

**Observability backends:**

- **[Langfuse](https://langfuse.com/)** — open-source, self-hostable, most widely-deployed OSS LLM observability platform. OTel-compatible ingestion. **This is our default backend.**
  - **Adopt:** Langfuse as the primary tracing target. Self-hosted deployment matches local-first.
- **[LangSmith](https://smith.langchain.com/)** — trace-as-primary-object pattern; excellent prompt-iteration UX.
  - **Adopt as reference:** trace shape, eval-experiment structure.
  - **Skip as backend:** LangChain coupling; hosted-only.
- **[Braintrust](https://www.braintrust.dev/)** — evaluation workflow model; prompt-centric evals.
  - **Adopt as reference:** eval-loop shape.
- **[Helicone](https://helicone.ai/), [Arize Phoenix](https://phoenix.arize.com/), [W&B Weave](https://weave-docs.wandb.ai/), [Portkey](https://portkey.ai/), [Laminar](https://laminar.sh/), [Latitude](https://latitude.so/)** — alternatives supported behind the OTel exporter.

**Cost governance:**

- **[AI-SDLC Framework CostPolicy resource](https://github.com/ai-sdlc-framework/ai-sdlc)** — per-execution hard limits + budget with alert thresholds + abort actions.
  - **Adopt:** the resource shape directly (relabeled to our terminology).
- **[LiteLLM pricing tables](https://github.com/BerriAI/litellm/blob/main/litellm/model_prices_and_context_window_backup.json)** — reference for per-model pricing data.
  - **Adopt:** as a source-of-truth for updating our own ~15-row cost table quarterly.
  - **Skip:** LiteLLM the library (see ADR-017 in the main engineering spec).

**Evaluation:**

- **[LangSmith evaluations](https://docs.smith.langchain.com/evaluation)** — dataset + eval-experiment pattern.
- **[Braintrust eval workflow](https://www.braintrust.dev/docs/guides/evals)** — prompt-centric evaluation.
- **[Chroma Research "Context Rot" study](https://www.trychroma.com/research/context-rot)** — reference for what our own evals should measure.

**Adopt:** Langfuse as OTel backend; AI-SDLC's CostPolicy shape; LiteLLM's public pricing data (without the library dependency).

---

## 8. v0.6 — Earned autonomy

Progressive relaxation of human-in-the-loop. Not all tasks require manual approval forever.

### 8.1 Capabilities

- **AutonomyPolicy** resource — declares autonomy levels (1 = engineer approves every step; 5 = fully autonomous for routine tasks)
- Task-class detection — workflows get classified (e.g., "single-file refactor with tests" vs "cross-cutting architecture change")
- Promotion criteria — task-class N graduates from Level 2 to Level 3 after 50 workflows at 95%+ approval rate
- Demotion triggers — any critical-security-incident demotes the affected task-class for a cooldown period (e.g., 4 weeks)
- Hindsight agent runs periodically, proposes autonomy adjustments
- Engineer role narrows from "approve every step" to "approve unusual work + trigger promotions"

### 8.2 Projects to reference

**Autonomy models:**

- **[AI-SDLC Framework AutonomyPolicy resource](https://github.com/ai-sdlc-framework/ai-sdlc)** — the direct source for earned-promotion + demotion-triggers pattern. Their five-level structure (Junior → Mid → Senior → Lead → Autonomous) with quantitative promotion criteria is exactly what we adopt.
  - **Adopt:** the resource shape; promotion-criteria semantics; demotion-trigger cooldowns.
  - **Skip:** the "engineering-manager approval" specifics — solo engineer doesn't have that role structure.

**Learning loops:**

- **[Compound Engineering plugin](https://github.com/EveryInc/compound-engineering-plugin)** — the compounding-improvement principle: every completed engineering unit should improve the system's ability to perform future engineering work.
  - **Adopt:** the retrospect / hindsight / horizon loop pattern.
- **[ECC / Everything Claude Code](https://github.com/affaan-m/everything-claude-code)** — continuous-learning patterns (skills evolved from past workflows).
  - **Adopt:** the "learned pattern extraction" flow, feed into hindsight agent.
- **[Superpowers verification-before-completion skill](https://github.com/obra/superpowers)** — reference for how to enforce evidence-based advancement even when autonomy is higher.

**Agent memory (for cross-workflow learning):**

- **[Mem0](https://github.com/mem0ai/mem0)** — layered memory (conversation / session / user / org). Consider if our own file-based memory becomes insufficient.
- **[Zep](https://www.getzep.com/)** — temporal knowledge graph for reasoning about how facts change over time.
- **[Letta (formerly MemGPT)](https://www.letta.com/)** — OS-style memory paging.
- **All three are optional.** Our file-based `vault/.memory/` under `patterns/` and `retrospectives/` is likely sufficient.

**Autonomy inspiration (proprietary — study, don't adopt):**

- **[Devin (Cognition)](https://cognition.ai/blog/introducing-devin)** — autonomous mode with parallel subtask coordination. Not a code source; conceptual reference for how graduated autonomy might feel.

**Adopt:** AI-SDLC's AutonomyPolicy resource verbatim (with terminology adjusted). Compound Engineering's retrospect/hindsight/horizon loop pattern.

---

## 9. v0.7 — Autonomous factory foundation

The container can be given ideas and produce shipped software with minimal engineer attention.

### 9.1 Capabilities

- Idea intake — a repository (or GitHub Issues, or a local web app) is the input queue
- Long-running container mode — Galley's dispatcher watches the intake, picks up new ideas, runs the SDLC to completion
- Notification channels:
  - Local web app (FastAPI + a small React UI) — approve / reject / clarify from a browser
  - Telegram bot integration — receive notifications and reply on your phone
  - Slack integration for team spaces
  - Email fallback
- Approval back-channel — engineer replies to a Telegram / web notification; Galley resumes the workflow with the decision
- Horizon agent proposes skill/agent improvements based on hindsight patterns
- The autonomous-factory vision: engineer adds ideas to a repo, opens Telegram once a day to answer clarifications and approve merges

### 9.2 Projects to reference

**Multi-platform notification gateway:**

- **[NousResearch Hermes Agent](https://github.com/NousResearch/hermes-agent)** — has native gateways for Telegram / Discord / Slack / WhatsApp / Signal / Email. Persistent memory (FTS5 + LLM summarization).
  - **Adopt as reference:** the multi-platform gateway pattern.
  - **Skip:** their agent-as-orchestrator model (violates INV-CORE-001 — LLM cannot change authoritative workflow state). We use it purely as a design source.

**Telegram / messaging integrations:**

- **[python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)** — MIT, actively maintained. Handles polling, webhook mode, inline keyboards.
  - **Adopt directly.**
- **[slack-sdk-python](https://github.com/slackapi/python-slack-sdk)** — official Slack SDK.
  - **Adopt directly.**
- **[Twilio API](https://www.twilio.com/docs/messaging)** — WhatsApp Business API + SMS.
  - **Adopt directly** for WhatsApp / SMS.
- **stdlib `smtplib`** — email fallback. Zero dependencies.

**Local web app (browser-based intake / approval):**

- **[FastAPI](https://fastapi.tiangolo.com/)** — async Python web framework.
  - **Adopt:** as the backend for the local intake / approval web app.
- **[HTMX](https://htmx.org/)** or **small React app** — pick one for the UI layer. HTMX is lighter, React is more familiar to most engineers.

**Autonomous-workflow inspiration:**

- **[Devin (Cognition)](https://cognition.ai/blog/introducing-devin)** — autonomous mode; opens PRs; iterates on feedback.
  - **Adopt as reference:** the "runs subtasks and coordinates" pattern for future parallelism (not v0.7 itself).
- **[GitHub Copilot Coding Agent](https://docs.github.com/en/copilot/using-github-copilot/coding-agent)** — GA September 2025. Assign a GitHub Issue → autonomous PR.
  - **Adopt as reference:** the issue-to-PR workflow shape.
- **[Baton (mraza007/baton)](https://github.com/mraza007/baton)** — polls GitHub Issues, runs Claude Code in isolated worktrees.
  - **Adopt as reference:** the polling / worktree isolation pattern.
- **[Emdash (YC W26)](https://emdash.dev/)** — parallel agents in isolated worktrees, desktop UI.
  - **Adopt as reference:** desktop UI patterns (if we build one).
- **[Conductor](https://conductor.build/)** — macOS app; parallel Claude Code / Codex worktrees dashboard.
  - **Adopt as reference:** dashboard shape.
- **[Claude Squad](https://github.com/smtg-ai/claude-squad)** — detached background sessions with worktree isolation.
  - **Adopt as reference:** background-session lifecycle.

**Task queue / dispatcher:**

- **[Vibe Kanban](https://vibekanban.com/)** — Kanban UI for AI coding agents, MCP-native. Company shut down April 2026 but OSS continues.
  - **Adopt as reference:** the intake board pattern.
- **[Agent Kanban](https://agent-kanban.dev/)** — leader agent + worker agents pattern.
  - **Adopt as reference:** the intake-queue shape (but not the LLM-driven leader).

**Layout convention:**

- **[agentic-code (shinpr/agentic-code)](https://github.com/shinpr/agentic-code)** — `.agents/{tasks,workflows,skills}` layout with progressive rule loading.
  - **Adopt as reference:** intake queue layout under `.galley/intake/`.

**Adopt:** `python-telegram-bot` + `slack-sdk` + Twilio + FastAPI directly for the notification gateway. NousResearch Hermes Agent's multi-platform pattern as design source. Baton / Emdash / Conductor as design sources for the dispatcher.

---

## 10. What we will NOT add (across all phases)

Naming these explicitly so scope creep has an argument.

- **Credential vault** — `.galley/.env` on an encrypted-disk laptop is functionally equivalent for a solo engineer. Team scenarios: use existing team-secret tools (1Password, Bitwarden), drop values into `.env` at container start.
- **Kubernetes-native architecture** — enterprise ceremony; not our fit. AI-SDLC Framework's k8s style is a design source, not an implementation model.
- **Compliance postures** (EU AI Act, NIST AI RMF, ISO 42001) — enterprise-facing; solo engineer irrelevant.
- **Parallel task execution with worktrees** — added *only* if serial execution becomes a real bottleneck in real usage.
- **External task-store adapters** (GitHub Issues, Plane, Linear, Jira) — `works/<task-id>/` is a valid task store. Adapters added on demand when a specific project needs them.
- **Vault rotation** — added when vault has enough content to warrant it (probably a long time — this is a v1.x concern at earliest).
- **Foundry integration** — Foundry doesn't exist yet. The inbox contract is stable; whenever Foundry exists, it can consume `vault/inbox/`.
- **Multi-agent parallel review with DSSE attestation** — the cross-harness reviewer independence rule (Phase 1 §14.3) provides most of the benefit without crypto.
- **Full-featured operator TUI** — a simple `galley workflow status <id>` + log tailing covers ~95% of the UX at ~5% of the effort. Add a TUI only if real usage demands it.
- **A2A (Agent-to-Agent) protocol** — over-engineering for our single-container architecture.
- **Complex prompt-compression libraries** (LLMLingua, etc.) — the two-artifact + Serena approach gets us most of the benefit without new dependencies.
- **Vector RAG over the codebase** — Serena's LSP-backed symbol retrieval outperforms vector similarity for code (2026 consensus). Not adopted.
- **Cross-session agent memory backends** (Mem0, Zep, Letta) — deferred; our file-based `vault/.memory/` is sufficient until it demonstrably isn't.

Every one of these has been considered and rejected for either "not needed at our scale" or "would add complexity without justifying value" reasons. Reconsider only when specific concrete usage argues otherwise.

---

## 11. The important asterisk

This roadmap is a **plan, not a commitment.** The strong constraints are:

1. Every phase must be shippable — never a "half-implemented next phase" state.
2. Every phase after v0.1 is built *using Galley* — the previous version dogfoods the next.
3. Governance-first stays load-bearing through every phase.
4. Anti-lock-in / subscription-arbitrage / cost-bounded stay first-class through every phase.
5. Autonomous factory (v0.7) is the destination — every prior phase is a step toward it, not just a feature we like.

If the real experience of v0.1 says "actually, we need observability before automation" — we do that. The phases as listed are the *default* order. They are not law.

---

## 12. Reference registry consolidated

A single place to see every project we adapt from, grouped by concern. See the individual phase sections above for what we adopt from each.

### Skills + methodology frameworks
- [Superpowers (obra/superpowers)](https://github.com/obra/superpowers) — 224k stars, agentic skills framework
- [Compound Engineering plugin (EveryInc)](https://github.com/EveryInc/compound-engineering-plugin) — plan/execute/review, compounding-improvement
- [GSD (Get Sh*t Done)](https://docs.opengsd.net/core/introduction) — 48k stars, spec-driven dev with fresh-context-per-phase
- [ECC / Everything Claude Code](https://github.com/affaan-m/everything-claude-code) — 214k stars, skill-first architecture
- [GitHub Spec Kit](https://github.com/github/spec-kit) — Constitution/Plan/Tasks/Implement pipeline
- [Kiro (AWS)](https://kiro.dev/) — EARS-style requirements notation
- [agentic-code (shinpr/agentic-code)](https://github.com/shinpr/agentic-code) — `.agents/` layout convention

### Orchestration / state machine
- [LangGraph](https://github.com/langchain-ai/langgraph) — v0.4 substrate (adopt)
- [AI-SDLC Framework](https://github.com/ai-sdlc-framework/ai-sdlc) — declarative resource types (adopt shape)
- ["A Deterministic Control Plane for LLM Coding Agents"](https://arxiv.org/pdf/2606.26924) — architectural pattern reference
- [OpenHands SDK v1](https://github.com/All-Hands-AI/OpenHands) — event-stream architecture (adopt)
- [Apache Burr](https://burr.apache.org/) — considered; rejected in favor of LangGraph
- [Temporal / DBOS / Inngest / Restate](https://temporal.io/) — durable-execution concepts (reference only)

### Coding harnesses (adapters in v0.3)
- [OpenCode](https://open-code.ai/) — local + OpenAI-compatible harness
- [Claude Code](https://docs.claude.com/en/docs/claude-code) — Anthropic (external in v0.1, adapter in v0.3+)
- [Codex CLI](https://developers.openai.com/codex/cli) — OpenAI
- [Antigravity CLI](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/) — Google (successor to Gemini CLI)
- [Cursor CLI](https://cursor.com/docs/cli/overview) — Cursor
- [Aider](https://github.com/paul-gauthier/aider) — repomap + architect+editor pattern (design source)

### Model access
- [anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python)
- [openai Python SDK](https://github.com/openai/openai-python) — also used with custom `base_url` for local endpoints
- [google-genai Python SDK](https://github.com/googleapis/python-genai)
- [LiteLLM](https://github.com/BerriAI/litellm) — pricing-data reference only; not adopted as library (ADR-017)

### Code intelligence
- [Serena MCP (Oraios AI)](https://github.com/oraios/serena) — LSP-backed semantic retrieval (adopted in v0.1)
- [Aider repomap](https://github.com/paul-gauthier/aider) — tree-sitter + PageRank pattern (adopted in v0.1 conceptually)
- [Sourcegraph](https://sourcegraph.com/) — whole-repo indexing pattern reference

### MCP servers
- [Anthropic MCP official servers](https://github.com/modelcontextprotocol/servers)
- [Context7 (Upstash)](https://github.com/upstash/context7) — library docs
- [Semgrep MCP](https://github.com/semgrep/mcp) — SAST
- [Snyk agent-scan](https://github.com/snyk/agent-scan) — SCA / IaC / SBOM
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) — browser automation
- [Chrome DevTools MCP](https://github.com/GoogleChromeLabs/chrome-devtools-mcp)
- [Mermaid MCP](https://github.com/peng-shawn/mermaid-mcp-server)
- [DeepWiki MCP (Cognition)](https://docs.devin.ai/work-with-devin/deepwiki-mcp)
- [Exa MCP](https://github.com/exa-labs/exa-mcp-server), [Tavily MCP](https://github.com/tavily-ai/tavily-mcp), [Brave Search MCP](https://github.com/brave/brave-search-mcp)
- [GitHub official MCP server](https://github.com/github/github-mcp-server)

### Observability
- [Langfuse](https://langfuse.com/) — v0.5 default backend (adopt)
- [LangSmith](https://smith.langchain.com/) — reference
- [Braintrust](https://www.braintrust.dev/) — reference
- [Helicone, Arize Phoenix, W&B Weave, Portkey, Laminar, Latitude] — alternatives via OTel

### Agent memory
- [Mem0](https://github.com/mem0ai/mem0), [Zep](https://www.getzep.com/), [Letta / MemGPT](https://www.letta.com/) — optional v0.6+ integrations if file-based memory becomes insufficient

### Notifications (v0.7)
- [NousResearch Hermes Agent](https://github.com/NousResearch/hermes-agent) — multi-platform gateway design source
- [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) — Telegram (adopt directly)
- [slack-sdk-python](https://github.com/slackapi/python-slack-sdk) — Slack (adopt directly)
- [Twilio](https://www.twilio.com/docs/messaging) — WhatsApp / SMS (adopt directly)
- [FastAPI](https://fastapi.tiangolo.com/) — local web app

### Container / infrastructure
- [HolyClaude (CoderLuii)](https://github.com/CoderLuii/HolyClaude) — Docker packaging + s6-overlay + UID remapping (adopted in v0.1)
- [Anthropic dev container patterns] — Docker + bind-mounted repo

### Autonomous / dispatcher (v0.7)
- [Baton (mraza007/baton)](https://github.com/mraza007/baton) — GitHub Issues polling → PR
- [Emdash (YC W26)](https://emdash.dev/) — parallel worktrees + desktop UI
- [Conductor](https://conductor.build/) — macOS parallel agents dashboard
- [Claude Squad](https://github.com/smtg-ai/claude-squad) — background session lifecycle
- [Vibe Kanban](https://vibekanban.com/) / [Agent Kanban](https://agent-kanban.dev/) — Kanban UI patterns
- [GitHub Copilot Coding Agent](https://docs.github.com/en/copilot/using-github-copilot/coding-agent) — issue-to-PR workflow reference
- [Devin (Cognition)](https://cognition.ai/blog/introducing-devin) — autonomous mode reference

---

## 13. How to use this document

When planning a phase (starting with v0.2):

1. Re-read the phase's section (§4–9).
2. For each capability, open the referenced projects — read their source, understand their patterns.
3. Extract the specific patterns / conventions we adopt. Rewrite in Galley's terminology and simplify to solo-engineer scale.
4. Cite the source in the resulting spec / code / ADR — so the provenance of every borrowed pattern is traceable.

This document is deliberately not a spec. It's a **starting point for the phase's spec**, which gets written when we actually start the phase — using Galley itself.

---

*End of Subsequent Phases Roadmap.*
