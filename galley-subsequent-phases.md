# Galley — Subsequent Phases Roadmap

**Version:** 1.0  
**Status:** Greenfield roadmap — planning document, not an implementation commitment  
**Date:** 2026-09-07  
**Starting point:** Phase 1 / v0.1 will be implemented from the accompanying Phase 1 Specification v1.0

------------------------------------------------------------------------

## 1. Purpose

This roadmap grows Galley from a manual, bounded AI-assisted workstation into an increasingly automated software factory.

The roadmap preserves one central architectural idea introduced in Phase 1:

> **Every Galley agent has five functional layers — role/instructions, tools/actions, reasoning/planning, knowledge/memory, and evaluation/feedback. Later phases deepen those layers; they do not replace the model with a giant agent framework.**

Phase progression is intentionally evidence-driven. Each later phase is built using Galley itself and may change order when actual usage shows that another capability is more valuable.

------------------------------------------------------------------------

### 1.1 Specification formatting convention

This roadmap uses GitHub-Flavored Markdown (GFM). Tables are kept narrow; detailed capability lists use headings and bullets rather than oversized matrices. Phase specifications derived from this roadmap SHOULD follow the same convention.

## 2. Cross-phase principles

- **Adopt proven pieces; keep the architectural contract Galley-owned; delete anything that does not produce measurable value.**
- Every phase is independently shippable.
- Every phase after v0.1 is built using the prior Galley release.
- Security/integrity controls arrive no later than the autonomy they constrain.
- The five-layer agent contract remains stable while implementations deepen.
- **Long-term knowledge and memory are external to Galley.** Vault is a separate project with its own specification and runtime. Galley integrates with Vault only when a later phase has a defined integration contract.

### 2.1 Deployment profiles

| Profile        | Analysis             | Implementation | Review                                | First-class from |
|:---------------|:---------------------|:---------------|:--------------------------------------|:-----------------|
| Local          | local model family A | local family B | local, procedurally independent       | v0.2             |
| Cloud-assisted | cloud reasoning      | local coding   | cloud/different family                | v0.1             |
| Cloud-heavy    | cloud                | cloud          | different provider/model where useful | v0.3             |

### 2.2 Dogfooding metrics

Every phase after v0.1 reports:

- `self_hosting_percentage >= 80%`;
- at least 5 real Galley PRs shipped through the workflow;
- `manual_escape_rate <= 10%`;
- `critical_escapees = 0`;
- evaluation/adversarial regressions introduced by the phase = 0 unresolved at ship.

These are targets and gates, not marketing claims; phase specs may tighten them when evidence warrants.

## 2. Roadmap at a glance

| Phase    | Goal                          | Agent-layer evolution                                                                          | Automation                              |
|:---------|:------------------------------|:-----------------------------------------------------------------------------------------------|:----------------------------------------|
| **v0.1** | Assisted Workstation          | Five layers defined, minimal implementations                                                   | Manual                                  |
| **v0.2** | Governed Workflow Platform    | Better knowledge packaging, persistent workflow state, evaluation datasets, rule memory        | Manual with deterministic control plane |
| **v0.3** | Full-fleet Agent Runtime      | Adapters, explicit memory tiers, event telemetry, specialized evaluators, credential isolation | First automated invocations             |
| **v0.4** | Self-hosting                  | Automated planning/execution loops, workflow checkpoints, bounded evaluator loops              | End-to-end workflow automation          |
| **v0.5** | Observable Agent Factory      | Evaluation + feedback become measurable system capabilities                                    | Continuous evaluation                   |
| **v0.6** | Earned Autonomy               | Agent authority expands only from measured evidence                                            | Selective autonomous execution          |
| **v0.7** | Autonomous Factory Foundation | Remote approval, long-running intake, privileged control paths                                 | Minimal-touch software production       |

------------------------------------------------------------------------

## 2.3 Phase-boundary decision register

These decisions are intentionally frozen unless later evidence and an ADR justify a change.

| Concern                  | v0.1                                          | First later phase                                 |
|:-------------------------|:----------------------------------------------|:--------------------------------------------------|
| Agent structure          | Five-layer contract; six broad roles          | v0.2 splits roles only if measured benefit exists |
| Long-term knowledge      | External; Galley keeps only workflow artifacts | v0.2 integration readiness; v0.3 Vault integration |
| Session memory           | Harness-local, non-authoritative              | v0.3 runtime/adapter management                   |
| Workflow state           | Git + filesystem artifacts                    | v0.2 SQLite                                       |
| Tool enforcement         | Declared allowlists + deterministic checks    | v0.3 adapter enforcement                          |
| Approval                 | Host-controlled, candidate-SHA-bound          | v0.7 remote privileged channels                   |
| Credential isolation     | Known Space-level limitation                  | v0.3 per-process injection                        |
| Autonomous orchestration | Out of scope                                  | v0.4                                              |
| Earned autonomy          | Out of scope                                  | v0.6                                              |

## 3. Five-layer evolution model

### 3.1 Role / persona / instructions

- **v0.1:** `AGENT.md` defines bounded role and mandatory procedure.
- **v0.2:** role contracts become schema-validated and reusable across workflows.
- **v0.3:** adapters translate canonical Galley roles into native harness/sub-agent configuration.
- **v0.4:** roles can be invoked automatically by the workflow engine.
- **v0.5:** role definitions become versioned, evaluated components.
- **v0.6:** task-class-specific authority policies affect which roles may act autonomously.
- **v0.7:** role execution can continue asynchronously under the same control-plane contract.

### 3.2 Tools / actions

- **v0.1:** explicit per-agent tool allowlists.
- **v0.2:** tool manifests gain schemas, scopes, side-effect classes, and better error contracts.
- **v0.3:** adapter layer mediates tool use and enforces runtime permissions.
- **v0.4:** workflow engine provides tool-capability profiles per state.
- **v0.5:** tool usage is traced and evaluated.
- **v0.6:** autonomy policies restrict action classes based on task risk.
- **v0.7:** remote control-plane clients can authorize privileged actions without exposing arbitrary natural-language control.

### 3.3 Reasoning / planning

- **v0.1:** procedural skills; bounded manual reasoning.
- **v0.2:** task decomposition, deterministic context selection, state-aware planning.
- **v0.3:** automated agent invocation; richer handoffs between roles.
- **v0.4:** explicit workflow planning plus bounded retry/refinement loops.
- **v0.5:** compare reasoning strategies against evaluation outcomes.
- **v0.6:** task-class-specific planning policies; autonomy is earned, not assumed.
- **v0.7:** long-running dispatcher chooses eligible workflows from an intake queue.

Galley should not require raw chain-of-thought as a system artifact. It should capture structured decisions, tool calls, evidence, and outcomes sufficient for engineering review.

### 3.4 Knowledge / memory

Galley does not implement its own long-term knowledge or memory product. **Vault is an independent project** with its own specification, governance, agents, skills, hooks, commands, knowledge model, ingestion, and retrieval mechanisms.

Galley remains deliberately abstract about Vault internals.

| Phase | Galley / Vault relationship |
|---|---|
| v0.1 | No Vault dependency. Galley produces workflow artifacts and a local workflow archive. |
| v0.2 | Prepare and validate the Galley-side integration boundary if an independently implemented Vault is available. No Vault internals are implemented in Galley. |
| v0.3 | First target for full Vault integration, assuming the Vault project has an independently usable release and supported interface. Selected agents and skills can use Vault for long-term knowledge and memory. |
| v0.4 | Expand task-scoped Vault use across the automated workflow; record retrieval provenance in Galley events. |
| v0.5 | Evaluate Galley's use of Vault: retrieval usefulness, provenance, freshness, cost, and downstream task impact. |
| v0.6 | Use Vault-derived information in autonomy decisions only when provenance and freshness gates pass. |
| v0.7 | Long-running workflows may exchange completed Galley artifacts with Vault and use Vault as one of several knowledge sources. |

#### Integration boundary

The Vault project owns:

- its own specification and governance;
- knowledge representation and storage;
- ingestion and maintenance workflows;
- agents, skills, hooks, and commands;
- indexes and semantic retrieval;
- Vault-specific evaluation and lifecycle.

Galley owns:

- workflow state and SDLC control;
- agent permissions;
- which agents and skills may use Vault;
- the Galley-side access/authentication boundary;
- workflow-level provenance for information used from Vault;
- failure behavior when a workflow depends on Vault.

The exact Vault API, tool names, protocol, file layout, frontmatter, indexing strategy, and internal maintenance behavior belong to the Vault specification and the future Galley integration ADR. They are intentionally not defined here.

#### Vault is one knowledge source

Future Galley agents may use:

- governance documents;
- repository source and history;
- web research;
- books and imported documents;
- meeting transcripts;
- workflow artifacts;
- Vault;
- other approved external knowledge systems.

Presence in Vault does not make information authoritative. Authority depends on provenance, freshness, and the governance rules applicable to the task.

#### Knowledge flow

```text
multiple knowledge sources
(book / web / meeting / repository / workflow artifacts / ...)
                              |
                              v
                            Vault
                              |
                              v
                     supported retrieval interface
                              |
                              v
                         Galley agent
```

Vault's internal wiki structure, semantic index, ingestion loop, and maintenance agents are out of scope for Galley's roadmap.

### 3.5 Evaluation / feedback

- **v0.1:** role-specific exit criteria, tests, reviews, deterministic release verification, adversarial suite.
- **v0.2:** reusable evaluation cases, feedback capture, rule promotion/retirement.
- **v0.3:** evaluator agents + structured event telemetry + independent evidence auditing.
- **v0.4:** automated checkpoint evaluation and bounded evaluator-optimizer loops where justified.
- **v0.5:** centralized evaluation/observability layer, trend analysis, cost/error attribution.
- **v0.6:** statistical evidence determines autonomy promotion/demotion.
- **v0.7:** remote approvals and operational monitoring become part of the feedback loop.

This evolution follows current practice: agent frameworks increasingly treat evaluation, guardrails, tracing, tools, memory, and human-in-the-loop as first-class capabilities rather than relying on a single prompt. OpenAI’s Agents SDK, Microsoft Agent Framework, and Anthropic’s current agent guidance all reflect versions of this decomposition.

------------------------------------------------------------------------

## 4. v0.2 — Governed Workflow Platform

Phase 0.2 remains engineer-led. The goal is to convert Phase 1’s manual conventions into reliable control-plane primitives.

### 4.1 Core additions

**Skill catalog**

Add the next skills only where actual Phase 1 workflows demonstrate demand:

- `brainstorming`
- `systematic-debugging`
- `adr-authoring`
- `security-review`
- `documentation-review`
- `runbook-authoring`
- `retrospective`
- `constitution-amendment`
- `threat-modeling`

Avoid a skill-count target for its own sake.

**Agent catalog**

Potentially split Phase 1 composite roles into:

- tester;
- architecture-gate;
- architecture-reviewer;
- security-reviewer;
- quality-reviewer;
- documentation-reviewer.

Each split requires measured benefit. The five-layer contract remains mandatory for every new role.

### 4.2 Runtime control plane

Introduce a SQLite state DB:

``` text
.galley/state.db
```

Track:

- workflows;
- tasks;
- artifacts;
- clarifications;
- candidate SHAs;
- governance touched;
- agent invocations;
- tool usage summaries.

The state DB is authoritative for workflow state, but Git remains authoritative for source code identity.

### 4.3 Governance-read runtime enforcement

Phase 1 uses prompt contract + inspection. v0.2 adds deterministic validation:

- required governance files must exist;
- task metadata determines relevant governance;
- exception paths are explicit;
- skill invocation is rejected when mandatory governance inputs are unavailable.

### 4.4 Code intelligence

Phase 1 uses Serena for symbol-aware repository inspection. v0.2 may add an Aider-style repository map only if measurement shows it improves context selection or handoffs. Any adaptation is pinned to a source revision and attributed.

### 4.5 Deterministic context pack

Introduce a deterministic context selector rather than an LLM-based compressor.

Default pack contains:

- the artifact being handed off;
- the files actually changed or directly relevant;
- applicable governance principles;
- canonical `AGENTS.md`.

LLM summarization may be introduced later only when a specific measured workload shows benefit.

### 4.6 Rules accumulation and retirement

Human-approved rules may be added from accepted review findings.

Lifecycle:

``` text
proposed → approved → active → deprecated → retired
```

Every rule records scope, provenance, owner, review date, applicability, and test case.

Rules have a token budget and an active-count budget.

### 4.7 Galley↔Vault integration readiness

v0.2 does not implement Vault. It only prepares and validates the Galley side of a future integration boundary.

Galley MAY consume an externally installed Vault only after the Vault project provides a stable, supported integration interface.

Galley-side acceptance criteria:

- Vault remains independently installable and governable;
- Galley can run normally with Vault absent;
- Vault availability is detectable without making workflow control dependent on it;
- selected skills/agents declare whether Vault access is optional or required;
- retrieved knowledge carries source/provenance metadata into the Galley workflow;
- Vault failures degrade according to the agent's declared policy rather than silently changing authority;
- no Vault-specific storage implementation is copied into Galley.

### 4.8 Evaluation assets

Create a durable evaluation corpus from Phase 1 failures:

``` text
evals/
  tasks/
  expected/
  adversarial/
  regressions/
  scoring/
```

Every recurring failure should answer:

> Can this failure be converted into a deterministic test, a review checklist item, a governance rule, or a measured evaluation case?

### 4.9 Replay

Provide **L1 interaction replay**:

- model identifier;
- prompt hash;
- model response;
- tool definitions;
- tool invocation metadata.

Label it honestly: L1 replay is not full workflow determinism.

### 4.10 MCP expansion

Only add new MCPs when a real workflow needs them.

Candidates include:

- Mermaid;
- GitHub PR operations;
- test-runner;
- Semgrep MCP;
- Context7;
- DeepWiki;
- Playwright;
- browser-devtools.

Every additional MCP must have a tool manifest, version pin, permissions review, provenance, and recovery procedure.

------------------------------------------------------------------------

## 5. v0.3 — Full-fleet Agent Runtime

This is the first phase where Galley automatically invokes agents instead of asking the engineer to copy prompts between harnesses.

### 5.1 Harness adapter contract

Define a provider-neutral adapter interface with:

- process lifecycle;
- session lifecycle;
- event parsing;
- tool-call capture;
- artifact capture;
- model/provider identity;
- error normalization;
- credential injection.

Build the adapter against the minimum viable harness first, then add additional harnesses.

### 5.2 Tool/action enforcement

Adapters become the runtime enforcement point for tool permissions.

The agent contract says what an agent needs. The adapter/control plane decides what can actually be given to it.

This is where Galley moves from declarative least-agency intent toward enforceable runtime permissions.

### 5.3 Vault integration adapter

v0.3 is the first target for full Vault integration, assuming the separate Vault project has reached an independently usable release.

The Galley adapter should expose only the capabilities needed by Galley agents. The exact operations, tool names, and wire protocol are defined jointly through the Vault/Galley integration contract, not invented by this roadmap in isolation.

Every retrieval visible to an automated workflow records enough provenance to answer:

- what was requested;
- what Vault returned;
- which source/version was used;
- which agent used it;
- whether the result was treated as authoritative, contextual, or advisory.

If Vault is unavailable, each agent's contract determines whether it may continue using non-Vault context or must stop and request clarification.

### 5.4 Baseline event log

Introduce:

``` text
.galley/events.jsonl
```

Capture:

- agent invocation;
- model/provider/harness;
- tool calls;
- latency;
- usage;
- state transitions;
- artifact lineage;
- failures.

This is the first real observability layer.

### 5.5 Independent evidence auditor

Add an `evidence-auditor` agent that runs from a fresh detached checkout of `candidate_commit_sha` and independently recomputes the release evidence.

Its result is a second evidence path, not a replacement for deterministic Git verification.

### 5.6 Credential isolation

Replace container-wide credentials with per-process or per-role injection.

Example:

- provider credential → only the provider-authenticating process;
- GitHub credential → only ship tooling;
- SSH agent → preferred over private-key material.

Acceptance criterion:

> A compromised non-ship agent cannot enumerate the credentials of unrelated roles simply because it shares a container.

### 5.7 Governance graph

Expose graph-shaped queries over Git-managed governance through an MCP interface.

Do not introduce a graph database unless real workload data shows that flat Git-backed queries cannot satisfy the use case.

Example queries:

``` text
principles_citing(id)
terms_defining(term)
contradicts(a,b)
supersedes(adr)
principle_load(task)
```

### 5.8 Evaluation layer

Add specialized evaluator roles only where needed:

- evidence auditor;
- retrospect agent;
- triage agent.

The evaluator does not become authoritative by virtue of being another LLM. Its output is advisory unless a deterministic control uses it.

------------------------------------------------------------------------

## 6. v0.4 — Self-hosting and automated workflow

This is the self-hosting milestone.

### 6.1 Workflow architecture

Galley’s workflow state machine is the architecture. A framework such as LangGraph is an implementation substrate, not the architectural contract.

Required capabilities:

- deterministic state transitions;
- persisted checkpoints;
- explicit human interrupts;
- resumability;
- bounded retries;
- failure recovery;
- task cancellation.

LangGraph is a valid candidate because current documentation supports persistence, human-in-the-loop interruption, time travel, and fault tolerance through checkpointing. It remains replaceable if Galley’s state-machine semantics are preserved.

### 6.2 Automated agent loop

A typical automated run becomes:

``` text
requirement
  ↓
analysis
  ↓
planning
  ↓
implementation
  ↓
evaluation
  ↓
review
  ↓
verification
  ↓
human approval
  ↓
ship
```

The control plane decides transitions. Agents do not decide that their own output is approved.

### 6.3 Evaluator-optimizer use

Evaluator-optimizer loops may be used for tasks where:

- the output has measurable criteria;
- iterative feedback improves results in evaluation;
- retries have bounded cost;
- a deterministic stopping condition exists.

Examples may include specification refinement, documentation quality, or targeted code repair.

Do not introduce evaluator loops into every task merely because a framework supports them.

### 6.4 Draft → staging → production

Introduce explicit state transitions:

``` text
draft
  → staging-eligible
  → staging-validated
  → production-eligible
  → human-approved
  → merged
```

GitHub labels are presentation. Internal Galley state is authoritative.

Promotion policy is risk-based rather than a universal fixed delay.

### 6.5 L2 workflow replay

Extend L1 replay with:

- workspace snapshot;
- dependency versions;
- image digest;
- relevant environment configuration;
- captured network responses;
- clock seed;
- RNG seed;
- external Git state.

L2 must still declare what is not reproduced.

------------------------------------------------------------------------

## 7. v0.5 — Observable and measurable Agent Factory

The system now has enough automation to justify a dedicated evaluation/observability layer.

### 7.1 External observability

Add OpenTelemetry-compatible tracing and one default backend, such as Langfuse.

The external backend is a consumer of Galley’s native event model, not the canonical workflow state.

### 7.2 Evaluation framework

Evaluate agents and workflows across:

- task completion;
- requirement adherence;
- tool selection and tool-call correctness;
- groundedness;
- regression rate;
- security findings;
- retry rate;
- latency;
- cost.

Use both:

- deterministic evaluators for objective properties;
- model-based evaluators for properties that genuinely require semantic judgment.

The current Microsoft and Google agent platforms both expose structured evaluation of agent behavior, tool use, quality, safety, and hallucination; this supports making evaluation a platform capability rather than an afterthought.

### 7.3 Vault retrieval evaluation

v0.5 evaluates Galley's **use** of Vault, not Vault's internal implementation.

Measure:

- retrieval usefulness on representative tasks;
- provenance coverage;
- stale-result rate;
- task success with and without Vault retrieval;
- retrieval latency and token cost;
- cases where Vault information conflicts with authoritative governance or primary evidence;
- failure behavior when Vault is unavailable.

The Vault project's own specification owns its internal retrieval-quality metrics, indexing benchmarks, ingestion metrics, and knowledge-maintenance evaluation.

### 7.4 Experience and decision observability

Track not only what happened, but:

- which component changed;
- what the change was expected to improve;
- whether the predicted improvement occurred;
- whether a regression occurred.

This creates a causal feedback loop for improving Galley’s own agent configuration.

### 7.5 Attribution before distillation

When learning from historical workflows:

1.  attribute outcomes to the prior configuration;
2.  identify accepted/rejected changes;
3.  only then distill the lessons into new rules, skills, or policy.

Avoid distilling unverified agent opinions into permanent memory.

### 7.6 Cost governance

Track:

- provider/model;
- workflow;
- role;
- tokens;
- latency;
- estimated cost;
- subscription usage where observable.

Do not equate subscription price with actual capacity. Provider plans can impose usage or rate limits even without per-token billing.

------------------------------------------------------------------------

## 8. v0.6 — Earned autonomy

Autonomy is a measurable capability, not a configuration flag.

### 8.1 Autonomy policy

Each task class has an autonomy level.

Example levels:

``` text
1 — human approves every step
2 — human approves before implementation
3 — human approves before ship
4 — autonomous for low-risk classes
5 — fully autonomous only for explicitly eligible routine classes
```

### 8.2 Promotion evidence

Promotion requires evidence for the specific task class:

- sufficient sample size;
- task success above threshold;
- escaped-defect rate below threshold;
- security-blocker rate below threshold;
- rollback frequency below threshold;
- incident rate below threshold;
- rule/telemetry integrity intact;
- no critical safety event in the evaluation window.

A confidence interval should accompany point estimates. A raw approval percentage is not sufficient evidence.

### 8.3 Demotion

Autonomy is revocable.

Triggers include:

- critical security incident;
- material increase in escaped defects;
- broken evidence integrity;
- telemetry corruption;
- repeated approval bypass attempts;
- sustained performance regression.

### 8.4 Agent-layer effect

At this phase, the five layers become policy-controlled:

- the same role may receive different tools by task class;
- memory visibility may change by risk class;
- planning depth may be bounded differently;
- evaluator requirements become stricter for higher-risk tasks.

------------------------------------------------------------------------

## 9. v0.7 — Autonomous factory foundation

Galley becomes a long-running software-production system with minimal human attention, but not unrestricted autonomy.

### 9.1 Intake

Potential inputs:

- local repository queue;
- GitHub Issues;
- local web UI;
- approved external task source.

External task stores remain optional. The durable task contract stays Galley-native.

### 9.2 Dispatcher

The dispatcher:

- selects eligible work;
- creates an isolated execution context;
- invokes the appropriate role;
- tracks progress;
- pauses for approval or clarification;
- resumes after explicit control-plane input.

### 9.3 Privileged notification channels

Slack, Telegram, email, or web interfaces are authentication/authorization boundaries, not convenience UX.

Every privileged action must bind to:

- task ID;
- candidate SHA;
- relevant artifact hash;
- authenticated principal;
- nonce / replay protection;
- expiration where appropriate.

Natural-language messages must never become unrestricted control-plane programs.

### 9.4 Long-running workflows with Vault

The autonomous factory may use Vault as a knowledge source and may hand completed Galley artifacts to Vault through Vault's supported ingestion interface.

The separation remains explicit:

```text
Galley owns workflow execution and control
                ↓
        completed artifacts
                ↓
       Vault ingestion boundary
                ↓
Vault owns long-term knowledge / memory
                ↓
       retrieval back to agents
```

Galley does not become responsible for Vault's internal maintenance loop. A future integration may invoke Vault's own agents, skills, hooks, and commands through its supported interface.

### 9.5 NIST interoperability watch

NIST’s 2026 AI Agent Standards Initiative identifies security, identity/authorization, and interoperability as active areas. Galley should track emerging standards and adopt stable interfaces only when they improve interoperability without weakening the core control-plane model.

------------------------------------------------------------------------

## 10. Phase gates and newly opened attack surfaces

Each phase gate includes the new risk introduced by that phase:

| Phase | New attack surface to test                                                                       |
|:------|:-------------------------------------------------------------------------------------------------|
| v0.2  | integration-boundary mistakes, untrusted external knowledge, malformed provenance, replay-data privacy |
| v0.3  | untrusted/stale Vault retrieval, adapter credential leakage, automated tool execution |
| v0.4  | orchestration/state-transition bugs, automated evaluator loops, staging promotion errors         |
| v0.5  | telemetry leakage, metric gaming, evaluator drift                                                |
| v0.6  | unsafe autonomy promotion, feedback-loop corruption                                              |
| v0.7  | remote approval/control-channel compromise, unattended long-running execution                    |

Every phase after v0.1 is gated on evidence from the previous phase.

### v0.2 gate

- real Phase 1 workflows identify repeated failure modes;
- state DB survives restart;
- evaluation corpus contains representative failures;
- rules do not grow without retirement;
- deterministic context packing demonstrates useful token reduction or comparable quality at lower context cost.

### v0.3 gate

- automated agent invocation works for the core workflow;
- event log provides sufficient traceability;
- credential isolation is demonstrated;
- evidence auditor catches injected/missing evidence;
- adapter contract works across at least two harnesses.

### v0.4 gate

- end-to-end self-hosting works on real Galley changes;
- every automated transition is deterministic;
- approval cannot be forged by the agent;
- retry and recovery are bounded and observable.

### v0.5 gate

- evaluation metrics show stable baselines;
- architecture/tool/model changes trigger re-evaluation;
- costs and failure rates are attributable to specific roles/components.

### v0.6 gate

- autonomy promotion evidence is task-class specific;
- demotion works in practice;
- safety and integrity signals cannot be disabled by the agent.

### v0.7 gate

- privileged notification channels have authenticated approval semantics;
- long-running recovery works;
- remote approval replay is prevented;
- the factory can operate for a sustained period without hidden manual intervention.

------------------------------------------------------------------------

## 11. Things Galley should continue to avoid

Unless real measurements change the conclusion, Galley should not add:

- a large multi-agent framework merely to coordinate six roles;
- Galley-owned persistent memory for every interaction;
- implementing Vault itself inside Galley;
- vector RAG for code that is better retrieved through symbol/LSP-aware tooling;
- LLM-based code compression without measured benefit;
- parallel execution without a demonstrated bottleneck;
- a graph database for governance merely because graph queries are useful;
- unrestricted external MCP marketplaces;
- natural-language remote commands that bypass the control plane;
- permanent autonomy without revocation and evaluation.

The default is still:

> **Adopt proven pieces, keep the architectural contract Galley-owned, and delete anything that does not produce measurable value.**

------------------------------------------------------------------------

## 12. Research basis — September 2026

### Agent architecture and primitives

- OpenAI Agents SDK — instructions, tools, handoffs, guardrails, sessions, human-in-the-loop, tracing.  
  <https://openai.github.io/openai-agents-python/>
- Microsoft Agent Framework — tools, context/knowledge, planning, hooks, observability, evaluation.  
  <https://learn.microsoft.com/en-us/agent-framework/agents/>

### Workflow and evaluation patterns

- Anthropic — *Building Effective AI Agents*, including workflow vs agent distinction and evaluator-optimizer pattern.  
  <https://www.anthropic.com/engineering/building-effective-agents>
- Anthropic — *Demystifying evals for AI agents*, emphasizing continuous evaluation across the agent lifecycle.  
  <https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents>
- Microsoft Agent Framework evaluation docs — task completion, tool usage, quality, safety, and workflow evaluation.  
  <https://learn.microsoft.com/en-us/agent-framework/agents/evaluation>
- Google Vertex AI Agent Engine evaluation and memory documentation — separate evaluation datasets, traces, and scoped memory.  
  <https://docs.cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/evaluate>  
  <https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/memory-bank/fetch-memories>

### Security and trust

- OWASP Top 10 for Agentic Applications 2026 — excessive agency and related agent risks.  
  <https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/>
- OWASP Agentic Skills Top 10 — security risks in agent skill ecosystems and manifests.  
  <https://owasp.org/www-project-agentic-skills-top-10/>
- NIST AI Agent Standards Initiative — security, identity/authorization, interoperability, and agent standards work.  
  <https://www.nist.gov/artificial-intelligence/ai-agent-standards-initiative>

### MCP

- MCP July 2026 specification release notes — stateless core, extensions, tasks, authorization hardening, JSON Schema 2020-12, and trace propagation.  
  <https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/>

------------------------------------------------------------------------

## Appendix A — Implementation references by phase

These are implementation references, not mandatory dependencies.

### v0.1

- OpenAI Agents SDK — agent/tool/guardrail/session/evaluation primitives: <https://openai.github.io/openai-agents-python/>
- NIST AI Agent Standards Initiative: <https://www.nist.gov/artificial-intelligence/ai-agent-standards-initiative>
- NIST agent identity/authorization concept paper: <https://csrc.nist.gov/pubs/other/2026/02/05/accelerating-the-adoption-of-software-and-ai-agent/ipd>
- MCP 2026-07-28 specification/release notes: <https://blog.modelcontextprotocol.io/posts/2026-07-28/>
- MITRE 2025 CWE Top 25: <https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html>
- GitHub Artifact Attestations: <https://docs.github.com/en/actions/concepts/security/artifact-attestations>
- Docker bind mounts: <https://docs.docker.com/engine/storage/bind-mounts/>

### v0.2

- Aider RepoMap: <https://github.com/Aider-AI/aider>
- MCP Python SDK / FastMCP: <https://github.com/modelcontextprotocol/python-sdk>
- NIST SSDF SP 800-218: <https://csrc.nist.gov/Projects/ssdf>
- Microsoft threat modeling guidance: <https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling>

### v0.3

- OpenHands SDK/event architecture: <https://github.com/All-Hands-AI/OpenHands>
- OpenAI Agents SDK session/memory distinction: <https://openai.github.io/openai-agents-python/>
- MCP authorization guidance: <https://modelcontextprotocol.io/>
- macOS Keychain / Linux secret-service implementations as credential-backend references

### v0.4+

- LangGraph: <https://github.com/langchain-ai/langgraph>
- Thoughtworks Technology Radar: <https://www.thoughtworks.com/radar>
- Langfuse: <https://langfuse.com/>
- NIST AI Agent Standards Initiative for evolving identity/interoperability guidance

### Vault integration rule

Galley treats Vault as an external system. Vault-specific implementation decisions belong to the Vault project and are imported into Galley only through a versioned integration contract and Galley integration ADR. Vault-specific storage, indexing, Markdown schemas, and maintenance rules are intentionally outside this roadmap.

------------------------------------------------------------------------

## 13. Final roadmap statement

> **Galley should grow by strengthening the five layers of a bounded agent and the deterministic control plane around them — not by accumulating agents, tools, memory stores, or orchestration frameworks.**
>
> **Automation comes after trustworthy state, evaluation, evidence, and authorization. Autonomy is earned from measured performance and remains revocable.**

*End of Subsequent Phases Roadmap v1.0.*
