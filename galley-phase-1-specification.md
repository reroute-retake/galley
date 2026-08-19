# Galley — Phase 1 Specification

**Version:** 2.0
**Status:** Ready for implementation
**Date:** 2026-08-19
**Delivers:** v0.1 — a container-based AI-assisted development environment
**License:** Apache 2.0
**Platforms:** macOS (arm64 + x86_64), Ubuntu Linux 22.04+ (amd64 + arm64)

---

## 1. Purpose

Galley v0.1 is a container-based, AI-assisted software development environment. The engineer enters a container, launches one of four pre-configured coding harnesses (OpenCode, Codex CLI, Antigravity, Cursor CLI), selects a model in the harness's own UI, and drives an opinionated software development lifecycle by hand — invoking skills and agents that come pre-installed and pre-configured for whichever harness is being used.

The value proposition is threefold:

- **Anti-lock-in.** Multiple provider subscriptions (ChatGPT Plus, Gemini AI Pro, cheap local models, Claude Pro if the engineer wants to install it) are usable interchangeably. No single-provider dependency.
- **ToS-compliant subscription arbitrage.** Each provider is used through *its own* native harness. Anthropic through Claude Code (external, if installed by the engineer), OpenAI through Codex CLI, Google through Antigravity, local models through OpenCode. Each subscription is honored in the way that provider's ToS permits.
- **Governance-anchored quality.** Constitution, ubiquitous language, requirements, invariants, architecture, and ADRs are first-class artifacts that every skill and agent reads before acting. Architecture drift is prevented by convention, uniformly, regardless of which model wrote which step.

Every SDLC step is invoked manually by the engineer. Galley v0.1 does not automate the workflow — it provides the container, the content, the conventions, and just enough plumbing to make the manual SDLC pleasant and cheap.

---

## 2. What v0.1 delivers

```
1. A `galley` CLI installable on macOS (Homebrew / uv) or Ubuntu (uv / curl)
2. A single container image (galley:full) with four coding harnesses
   pre-installed and configured
3. A per-space directory at ~/Documents/galley/<space>/
4. Sixteen skills + seventeen agent contracts pre-installed in each
   harness's native config directory, at container boot
5. Seven base MCP servers running (filesystem, git, fetch, serena,
   mermaid, github-pr, test-runner)
6. Optional MCP servers pre-installed but activated only via configuration
   (context7, deepwiki, exa/tavily/brave, semgrep, snyk, playwright,
   chrome-devtools)
7. A governance-first workflow: constitution, ubiquitous language,
   requirements, invariants, architecture, ADRs — created or updated
   as part of the workflow, read at the first step of every agent
8. A context-pack watcher that automatically produces compressed
   handoff artifacts using the engineer's local model
9. Semantic code intelligence via Serena and repomap
10. A manual SDLC runbook mapping every step to a harness + agent
11. Ship via `gh pr create` invoked by the ship agent
12. Archive via `cp` invoked by the archive agent
```

---

## 3. Terminology

The vocabulary Galley uses. Chosen to be crisp and non-colliding with adjacent projects.

| Term | Meaning |
|---|---|
| **Space** | An isolated development universe on the engineer's host. One or more repositories, one vault, one `.galley/` config directory. |
| **Container** | The `galley:full` image running for a Space. Disposable. Source and vault outlive it. |
| **Harness** | A coding-assistant executable inside the container. Phase 1 ships four: OpenCode, Codex CLI, Antigravity, Cursor CLI. |
| **Workflow** | One end-to-end pass through the SDLC — from requirement to shipped PR and archived artifact bundle. |
| **Task** | A discrete step inside a workflow. Identified by `T####`. |
| **Skill** | Reusable procedural knowledge (SKILL.md format) invoked inside a harness. |
| **Agent** | A bounded engineering role (Analyst, Planner, Implementer, Reviewer, …). Backed by an AGENT.md contract file. |
| **Governance** | The five files a repository must have: `constitution.md`, `ubiquitous_language.md`, `requirements/`, `invariants.md`, `architecture/architecture.md` + `architecture/adrs/`. |
| **Artifact** | A durable file produced by an SDLC step. Lives under `works/<task-id>/`. |
| **Context Pack** | A compressed version of an artifact, produced automatically for the next agent's consumption. Filename convention: `<artifact>.pack.md`. |
| **Checkpoint** | A shape check that a required artifact exists and matches its expected structure. Phase 1 checkpoints are enforced by convention in skill text. |
| **Clarification** | An unresolved question raised by an agent when it encounters ambiguity. Lives at `works/<task-id>/clarifications/C-<n>.md`. Blocks progress until the engineer resolves it. |
| **Precondition** | Required inputs before an agent can be invoked. Named in the agent's contract. |
| **Vault** | The Space's durable evidence store at `<space>/vault/`. |
| **Inbox** | `vault/inbox/<task-id>/` — the final resting place for completed workflow evidence. |

---

## 4. The governance-first principle

Every project developed with Galley has explicit engineering governance, and every skill and agent reads it before acting. Governance is treated as active constraint, not documentation.

### 4.1 The five governance documents

Every repository managed by a Space contains:

```
<repo>/
├── constitution.md              (project engineering rules — hard constraints)
├── ubiquitous_language.md       (canonical terminology used in code, docs, ADRs)
├── requirements/                (stable-ID'd functional + non-functional + security)
│   ├── functional.md
│   ├── non-functional.md
│   └── security.md
├── invariants.md                (INV-#### properties that must remain true)
├── architecture/
│   ├── architecture.md          (current architecture — the "map")
│   └── adrs/                    (Architecture Decision Records, ADR-####-<slug>.md)
└── AGENTS.md                    (agent-facing project instructions)
```

`galley init` (via the initializer agent's guidance) scaffolds these files from templates when a repository is first added.

### 4.2 Every skill and agent declares its governance touchpoints

Every SKILL.md frontmatter names the governance the skill needs:

```yaml
---
name: architecture-review
governance_read:
  - constitution.md
  - ubiquitous_language.md
  - invariants.md
  - architecture/architecture.md
  - architecture/adrs/*
governance_write:
  - architecture/adrs/*         # this skill may propose new/superseding ADRs
---
```

Every AGENT.md frontmatter names the governance the agent needs:

```yaml
---
id: architecture-reviewer
governance_read:
  - constitution.md
  - ubiquitous_language.md
  - invariants.md
  - architecture/architecture.md
  - architecture/adrs/*
governance_write:
  - architecture/adrs/*
  - invariants.md               # may propose new invariants; engineer approves
---
```

### 4.3 Every agent's Step 1 is "read governance"

Every agent contract's body opens with:

```markdown
## Mandatory Step 1: Read governance

Before doing ANYTHING else, read every file listed under
governance_read in this contract.

If any is missing, STOP. Do not attempt to proceed. Instead, output:
  "Governance missing: <files>. Please run the initializer agent or
   manually create the file."
```

This is enforced by convention in Phase 1. Skill/agent authoring discipline ensures every contract carries this block.

### 4.4 Governance loading is relevance-filtered

Each task's YAML front matter declares which governance items apply specifically to it. Skills use this to load only relevant items rather than everything:

```yaml
# in a task's own metadata
governance:
  requirements: [REQ-0012, REQ-0018]
  invariants: [INV-DATA-004]
  adrs: [ADR-0003, ADR-0007]
```

An agent invoked for this task loads:

- Constitution (always — small)
- Ubiquitous language (always — small)
- All invariants (small)
- Only the requirements listed
- Only the ADRs listed (plus their supersession chains)
- Architecture sections relevant to the task's scope

For a typical implementation task this brings governance load from ~15-25K tokens down to ~3-5K tokens.

### 4.5 Governance write requires human approval for constitutional / invariant changes

| Governance document | Written by | Human approval required? |
|---|---|---|
| `constitution.md` | initializer (create); constitution-amendment skill (updates) | **Yes for updates** — engineer explicitly confirms |
| `ubiquitous_language.md` | initializer (create); any agent introducing a new term | No — additions only |
| `requirements/*.md` | analyst (propose); any agent producing new REQ IDs | **Yes for adding/removing REQ IDs** |
| `invariants.md` | initializer (create); architecture-reviewer proposes | **Yes for adding INV IDs** |
| `architecture/architecture.md` | initializer (create); architecture-reviewer (updates) | No — updates reflecting approved ADRs |
| `architecture/adrs/ADR-####.md` | architecture-gate proposes; implementer may propose superseding ADR | No — ADRs are proposable freely; supersession must cite |
| `AGENTS.md` | initializer | Rarely touched afterward |

### 4.6 Architecture drift is prevented by construction

Because every agent reads relevant governance at Step 1 and cites it in outputs:

- **Analysis** classifies claims as FACT / INFERENCE / ASSUMPTION / UNKNOWN, citing governance where applicable.
- **Planning** decomposes tasks with explicit REQ / INV / ADR links.
- **Architecture Gate** blocks tasks that would silently invalidate an ADR.
- **Implementation** cites relevant invariants and ADRs in commit messages when relevant.
- **Review** classifies BLOCKER findings by which governance document a violation cites.

Different models writing different steps of the same workflow (Antigravity for analysis, Codex for planning, OpenCode for implementation) stay aligned because they all anchor on the same governance files.

---

## 5. Space model

A **Space** is an isolated development universe on the host.

### 5.1 Directory layout

All Galley Spaces live under a single parent directory: `~/Documents/galley/` (macOS/Linux default; overridable via `$GALLEY_HOME`).

```
~/Documents/galley/                    (parent — all Galley Spaces)
├── <space-1>/                          (e.g., galley, payments, commerce)
│   ├── .galley/
│   │   ├── space.yaml                  (Space configuration — §7)
│   │   ├── .env                        (per-Space credentials — gitignored, §8)
│   │   └── custom/
│   │       ├── skills/                 (per-Space skill overrides)
│   │       └── agents/                 (per-Space agent overrides)
│   ├── .gitignore                      (covers .galley/.env*, logs/, works/)
│   ├── repos/                          (Git repositories owned by this Space)
│   │   └── <repo-name>/
│   ├── vault/
│   │   └── inbox/                      (completed workflow bundles — §19)
│   ├── works/                          (per-task manual artifacts)
│   │   └── <task-id>/
│   │       ├── analysis.md
│   │       ├── analysis.pack.md
│   │       ├── plan.md
│   │       ├── plan.pack.md
│   │       ├── ...
│   │       └── clarifications/
│   │           └── C-<n>.md
│   └── logs/                           (container logs)
│
├── <space-2>/
│   └── ...
│
└── .galley/                           (host-level Galley metadata)
    ├── config.yaml
    └── spaces.yaml                     (registry of known Spaces)
```

### 5.2 Space invariants

- **INV-SPACE-001** — A container has filesystem access only to resources associated with its own Space. Cross-space access is prohibited.
- **INV-SPACE-002** — Space name matches `[a-z][a-z0-9-]{0,63}`.
- **INV-SPACE-003** — Spaces live under `$GALLEY_HOME` (default `~/Documents/galley/`). One installation, one parent.

### 5.3 Multiple repositories per Space

A Space may host one or more repositories. Cross-repository work is a per-workflow choice; the engineer decides which repos are in scope by adding them to the Space.

---

## 6. Host CLI (`galley`)

Thin wrapper around Docker plus a small Python core for Space management. Installed via `uv tool install galley` (macOS + Linux) or `brew install galley` (macOS convenience).

### 6.1 Command surface

```bash
# Space lifecycle
galley init <space>                            # create ~/Documents/galley/<space>/
galley list                                    # list all Spaces
galley destroy <space>                         # remove a Space (with confirmation)

# Repository management (per Space)
galley <space> repo add <git-url> [--name <n>] [--branch <b>]
galley <space> repo remove <name>
galley <space> repo list
galley <space> repo status

# Container lifecycle
galley build                                   # build galley:full image
galley start <space>                           # start the Space's container
galley stop <space>                            # graceful stop (SIGTERM 30s → SIGKILL)
galley restart <space>                         # stop + start
galley shell <space>                           # exec into container (primary UX)
galley exec <space> <command> [args...]        # run one command non-interactively
galley logs <space> [--follow] [--tail N]      # stream / tail container logs

# Diagnostics
galley status                                  # which Spaces up, image versions
galley ps                                      # detailed container state
galley doctor                                  # health: Docker, image, MCPs, envs
galley version                                 # galley + image version
galley update                                  # update galley itself; suggests rebuild

# Manual override for context-pack watcher (§16)
galley pack <artifact-path> [--for <next-step>]
```

### 6.2 Exit codes

- `0` — success
- `1` — user error (bad args, unknown Space, missing file)
- `2` — runtime error (Docker not running, container refuses to start)
- `3` — build failure
- `4` — health-check failure (container up but doctor found issues)

### 6.3 `galley stop` — graceful shutdown

```bash
galley stop <space>
```

Behavior:

1. Send `docker stop --time 30` — SIGTERM to the container. s6-overlay has 30 seconds to gracefully stop each supervised service (base MCPs, watcher, etc.).
2. If the container is still running after 30 seconds, `docker stop` escalates to SIGKILL.
3. Container state (`~/Documents/galley/<space>/`) is unaffected — the container is disposable; the Space is not.
4. `galley start` cleanly resumes: same volumes, same env from `.galley/.env`, MCPs come back up, watcher rescans.

### 6.4 What `galley init <space>` produces

Running `galley init galley` creates:

```
~/Documents/galley/galley/
├── .galley/
│   ├── space.yaml                      (filled with sensible defaults)
│   ├── .env                            (skeleton with placeholders + comments)
│   └── custom/
│       ├── skills/                     (empty)
│       └── agents/                     (empty)
├── .gitignore                          (covers .galley/.env*, works/, logs/)
├── repos/                              (empty until `galley repo add`)
├── vault/
│   └── inbox/
├── works/
└── logs/
```

Additionally, the first `galley init` on a host creates `~/Documents/galley/.galley/`:

```
~/Documents/galley/.galley/
├── config.yaml
└── spaces.yaml
```

### 6.5 What `galley repo add` produces

`galley <space> repo add git@github.com:<owner>/<repo>.git`:

1. Clones the repository into `repos/<name>/`.
2. Checks whether the five governance files exist.
3. If any are missing, offers to scaffold them from template. Engineer answers yes/no. If yes, templates are copied and the engineer is prompted to fill in project specifics.
4. Registers the repo in `.galley/space.yaml`.

---

## 7. Space configuration — `.galley/space.yaml`

The authoritative Space configuration file.

```yaml
space:
  name: galley
  schema: 1
  created: 2026-08-19T10:00:00Z

repositories:
  - name: galley
    url: git@github.com:<owner>/galley.git
    path: repos/galley
    default_branch: main

runtime:
  image: galley:full
  uid_remap: true

# Env-var names each provider expects. Values live in .galley/.env.
credentials:
  anthropic: ANTHROPIC_API_KEY
  openai:    OPENAI_API_KEY
  google:    GEMINI_API_KEY
  github:    GITHUB_TOKEN
  local_model: LOCAL_MODEL_KEY           # optional; only if endpoint requires auth

# Local model — served on another machine, typically fronted by a LiteLLM proxy.
local_model:
  enabled: true
  base_url: http://model-machine.local:4000/v1
  api_key_ref: local_model               # references credentials.local_model
  default_model: qwen3.8-27b             # names the model the packer + local
                                          # SDLC steps default to

# Which pre-installed harnesses to expose.
harnesses:
  enabled:
    - opencode
    - codex-cli
    - antigravity
    - cursor-cli

# Which MCP servers are active beyond the base set.
mcp:
  enabled:
    - context7                            # requires CONTEXT7_API_KEY
    - exa                                 # requires EXA_API_KEY
    - playwright                          # no auth
    # semgrep, snyk, deepwiki, chrome-devtools, tavily, brave — available;
    # add here to activate

# Context-pack watcher configuration (§16).
pack:
  enabled: true
  debounce_seconds: 30
  overrides:
    # Per-file custom targets (optional).
    # "works/*/custom-analysis.md": "planner"

# Governance discipline settings (§4).
governance:
  scaffold_on_repo_add: true              # offer template scaffolding
  require_governance_read: true           # skills MUST read governance at Step 1
```

---

## 8. Credentials — `.galley/.env`

Per-Space `.env` file at `~/Documents/galley/<space>/.galley/.env`. Never committed to any Git repository (top-level `.gitignore` covers `.galley/.env*`).

Skeleton written by `galley init`:

```dotenv
# Galley credentials — Space: <space-name>
#
# This file is NEVER checked into Git. It is read at container start and
# passed to the container as env vars via `docker run --env-file`.
# Vendor SDKs (anthropic, openai, google-genai) auto-load the standard names.

# ─── Model provider keys ─────────────────────────────────────────────
ANTHROPIC_API_KEY=sk-ant-...                # https://console.anthropic.com/
OPENAI_API_KEY=sk-...                       # https://platform.openai.com/
GEMINI_API_KEY=...                          # https://aistudio.google.com/

# ─── Version control ─────────────────────────────────────────────────
GITHUB_TOKEN=ghp_...                        # PR creation via `gh` CLI

# ─── Local model (optional) ──────────────────────────────────────────
LOCAL_MODEL_KEY=                            # only if remote LiteLLM proxy requires auth

# ─── Optional MCP servers ────────────────────────────────────────────
CONTEXT7_API_KEY=                           # https://context7.com/dashboard
EXA_API_KEY=                                # https://exa.ai/
TAVILY_API_KEY=
BRAVE_API_KEY=
SNYK_TOKEN=
```

**Rules:**

- Values live only in this file. `space.yaml` references env var *names*, never values.
- Missing values are OK — the corresponding provider or MCP just becomes unavailable.
- Engineer discipline: do not paste secrets into agent chats. Do not commit anything from `works/` without review.

---

## 9. Container image — `galley:full`

Single build target for Phase 1.

### 9.1 Base

- `python:3.12-slim-bookworm` (Debian 12 slim) — multi-arch (amd64 + arm64)
- Non-root user `forge` (UID/GID remapped to host at container start)
- Approximate compressed image size: ~2.5 GB

### 9.2 Contents

```
System:
  - Debian bookworm slim
  - s6-overlay v3 (service supervision)
  - build-essential, curl, ca-certificates, gnupg

Languages / runtimes:
  - Python 3.12 + uv 0.5+
  - Node.js 22 LTS
  - mise (language runtime management per project)
  - git 2.45+, gh CLI (GitHub operations)

Code intelligence:
  - Serena MCP (uv tool install serena-mcp)
  - Serena's project memory redirected to .galley/serena/

Harnesses (Phase 1 — four pre-installed):
  - OpenCode                              (npm i -g @opencode-ai/opencode)
                                          → for local models + any OpenAI-compatible endpoint
  - Codex CLI                             (npm i -g @openai/codex)
                                          → for OpenAI (ChatGPT Plus / Pro subscription)
  - Antigravity CLI (agy)                 (downloaded binary from Google)
                                          → for Google (Gemini AI Pro subscription)
  - Cursor CLI                            (curl https://cursor.com/install | sh, or npm)
                                          → for Cursor's model routing (Cursor Start / Pro subscription)
  # Claude Code is NOT pre-installed. The engineer typically runs it
  # on their host. If desired inside the container:
  #    npm i -g @anthropic-ai/claude-code

MCP servers (running at boot — the base set):
  - filesystem                            (Galley built-in, stdio)
  - git                                   (Galley built-in, stdio)
  - fetch                                 (Anthropic reference, stdio)
  - serena                                (Oraios AI, HTTP :3110)
  - mermaid                               (community, stdio)
  - github-pr                             (GitHub official, stdio)
  - test-runner                           (Galley built-in, stdio)

MCP servers (installed but disabled unless space.yaml enables):
  - context7 (Upstash)
  - deepwiki (Cognition)
  - exa / tavily / brave (web search)
  - semgrep
  - snyk
  - playwright
  - chrome-devtools

Watcher:
  - galley-pack-watcher                     (s6 service — §16)

Testing utilities:
  - pytest, ruff, pyright (Python)
  - eslint, prettier (JavaScript / TypeScript)
  - curl, jq

Browser (for playwright/chrome-devtools when enabled):
  - Chromium, Firefox, WebKit (headless)
```

### 9.3 Content layer

Baked into the image at `/workspace/galley/`:

```
/workspace/galley/
├── skills/                     ← 16 SKILL.md folders (§13)
├── agents/                     ← 17 AGENT.md folders (§14)
├── AGENTS.md                   ← project-scope instructions (Codex CLI reads this natively)
├── mcp-servers.json            ← canonical MCP config (per-harness formats generated at boot)
├── setup-harnesses.sh          ← per-harness content-installation fan-out (§11)
├── templates/
│   ├── constitution.md
│   ├── ubiquitous_language.md
│   ├── invariants.md
│   ├── architecture.md
│   ├── ADR-0000-record-architecture-decisions.md
│   ├── AGENTS.md
│   └── requirements/
│       ├── functional.md
│       ├── non-functional.md
│       └── security.md
└── docs/
    ├── runbooks/
    │   └── galley-manual-workflow.md
    └── guides/
        ├── getting-started.md
        ├── writing-a-skill.md
        ├── per-harness-invocation.md
        └── governance-first.md
```

### 9.4 Mounts at `galley start`

- `~/Documents/galley/<space>/repos/` → `/workspace/repos/` (rw)
- `~/Documents/galley/<space>/vault/` → `/workspace/vault/` (rw)
- `~/Documents/galley/<space>/works/` → `/workspace/works/` (rw)
- `~/Documents/galley/<space>/logs/` → `/workspace/logs/` (rw)
- `~/Documents/galley/<space>/.galley/custom/` → `/workspace/.custom/` (rw)
- `~/.gitconfig` → `/home/forge/.gitconfig` (ro) — engineer's git identity
- SSH agent socket:
  - macOS: via socat proxy at `/tmp/ssh-agent-proxy.sock`
  - Linux: `$SSH_AUTH_SOCK` forwarded

Env vars from `.galley/.env` are passed via `docker run --env-file`.

### 9.5 `galley start` boot sequence

```
galley start <space>
   ↓
1. Verify Docker is running
2. Verify space.yaml parses cleanly
3. Ensure required directories exist under ~/Documents/galley/<space>/
4. Read .galley/.env into an env-file for docker run
5. docker run -d ... galley:full
6. Container boot:
     a. s6-overlay starts services in dependency order
     b. Base MCP servers start (filesystem → git → fetch → serena → ...)
     c. Optional MCPs start iff enabled in space.yaml AND env vars present
     d. galley-pack-watcher starts, begins watching /workspace/works/
     e. setup-harnesses.sh runs, fanning out content into each harness's
        native config directory (§11)
7. Wait for container health check (30 s ceiling)
8. Print connection summary:
     "Space 'galley' started.
      Harnesses ready: opencode, codex, agy
      MCPs running: <list>
      Enter with: galley shell galley"
```

---

## 10. Local model configuration

The local model (typically Qwen 3.8 27B or similar) runs on a separate machine — usually fronted by a LiteLLM proxy — and is reached by the container over the network.

Configuration lives in `.galley/space.yaml` under `local_model`:

```yaml
local_model:
  enabled: true
  base_url: http://model-machine.local:4000/v1
  api_key_ref: local_model
  default_model: qwen3.8-27b
```

The container connects via the `openai` Python SDK with `base_url` set to `local_model.base_url`. Any OpenAI-compatible endpoint works (LiteLLM proxy, llama.cpp's `llama-server`, Ollama, vLLM, LM Studio) — Galley treats it as an opaque endpoint.

The local model is used:

- By the harnesses configured to point at it (typically OpenCode) — engineer selects it in the harness UI.
- By the context-pack watcher (§16) for automatic compression of handoff artifacts.

---

## 11. Per-harness content installation

At container start, `setup-harnesses.sh` (~150 lines of Bash) fans out the canonical `/workspace/galley/` content into each harness's expected configuration directory. This is what makes skills and agents discoverable by whichever harness the engineer chooses.

### 11.1 The mapping

| Content | OpenCode | Codex CLI | Antigravity (agy) | Cursor CLI |
|---|---|---|---|---|
| Skills | `~/.config/opencode/skills/*` (symlinks) | invoked via `$` or slash commands from AGENTS.md | `~/.gemini/skills/*` + `~/.gemini/antigravity-cli/skills/*` (symlinks, dual location) | `~/.cursor/skills/*` (symlinks) |
| Agents | `~/.config/opencode/agents/*` (symlinks) | referenced in `AGENTS.md` | (skills-as-agents pattern) | `~/.cursor/agents/*` (symlinks) |
| Project instructions | `.opencode/AGENTS.md` (per-repo) | `~/.codex/AGENTS.md` (symlink to canonical) | `~/.gemini/AGENTS.md` (symlink) | `~/.cursor/AGENTS.md` (symlink) |
| MCP config | `~/.config/opencode/opencode.json` (generated JSON) | `~/.codex/config.toml` (generated TOML) | `~/.gemini/config/mcp_config.json` (generated JSON) | `~/.cursor/mcp.json` (generated JSON) |

All four harnesses accept SKILL.md as the skill format — one authored skill folder is symlinked into four locations. MCP configuration differs in file format (JSON vs TOML) — a small Python helper generates each format from the canonical `mcp-servers.json`.

**Note on Cursor CLI paths:** Cursor CLI shares its config directory (`~/.cursor/`) with Cursor IDE. Exact skill / agent / MCP path conventions inside `~/.cursor/` are verified as part of Spike C — Cursor's rapid release cadence may shift these.

### 11.2 `setup-harnesses.sh` behavior

```
1. Read canonical mcp-servers.json
2. For each MCP server, check required env vars are present.
   Skip servers whose required env vars are missing (silently — the
   engineer sees them as "disabled" via `galley exec <space> galley-mcp status`).
3. Fan out symlinks from /workspace/galley/ into each harness's config dirs.
4. Layer per-Space overrides from /workspace/.custom/ (shadow canonical
   entries by same name).
5. Generate per-harness MCP config files (JSON for OpenCode + Antigravity + Cursor;
   TOML for Codex).
6. Report:
     "Harness setup complete.
      Available: opencode  (~/.config/opencode)
                 codex     (~/.codex)
                 agy       (~/.gemini)
                 cursor    (~/.cursor)
      Skills: 16 canonical + <n> custom
      Agents: 17 canonical + <n> custom"
```

### 11.3 Per-Space customization

Engineer drops a customized skill at `~/Documents/galley/<space>/.galley/custom/skills/<name>/SKILL.md`. On next `galley start`, that skill shadows the canonical one by the same name. Same pattern for agents. No need to fork the Galley repo for per-project tweaks.

---

## 12. MCP servers

### 12.1 Base set (always running when their credentials permit)

Seven MCP servers form the always-available foundation:

| Server | Provider | Purpose | Transport | License |
|---|---|---|---|---|
| `filesystem` | Galley built-in | Repo-scoped file read/write | stdio | Apache 2.0 |
| `git` | Galley built-in | Git operations (log, diff, status, commit, push) | stdio | Apache 2.0 |
| `fetch` | Anthropic reference | Web content fetching, HTML→markdown | stdio | MIT |
| `serena` | Oraios AI | LSP-backed semantic code intelligence | HTTP :3110 | MIT |
| `mermaid` | community | Render Mermaid → PNG / SVG | stdio | MIT |
| `github-pr` | GitHub official | PR read/write, issues, comments | stdio | MIT |
| `test-runner` | Galley built-in | Run tests, lint, type-check | stdio | Apache 2.0 |

### 12.2 Optional set (installed; enabled in `space.yaml`)

| Server | Purpose | Requires |
|---|---|---|
| `context7` (Upstash) | Version-specific library docs | `CONTEXT7_API_KEY` |
| `deepwiki` (Cognition) | Q&A over public GitHub repos | (no auth) |
| `exa` | Neural web search | `EXA_API_KEY` |
| `tavily` | LLM-formatted web search | `TAVILY_API_KEY` |
| `brave` | Independent-index web search | `BRAVE_API_KEY` |
| `semgrep` | SAST | (no auth) |
| `snyk` | SCA, IaC, container, SBOM | `SNYK_TOKEN` |
| `playwright` | Browser automation | (no auth) |
| `chrome-devtools` | Chrome DevTools protocol | (no auth) |

Optional MCPs are pre-installed in the image but not started unless their ID appears in `.galley/space.yaml` `mcp.enabled[]` **and** their required env vars are present.

### 12.3 MCP inspection

Inside the container:

```bash
galley-mcp status                          # which MCPs are running / stopped / disabled
galley-mcp logs <server-id>                # tail that MCP's log
```

These are shell utilities inside the container; the engineer reaches them via `galley exec <space> galley-mcp status` from the host.

---

## 13. Skills — the 16 canonical set

Skills are reusable procedural knowledge invoked by an agent inside a harness. Each skill lives at `/workspace/galley/skills/<name>/SKILL.md` with YAML frontmatter and a markdown body.

### 13.1 Catalog

| # | Skill | Purpose | Used in |
|---|---|---|---|
| 1 | requirements-analysis | Turn raw requirement → REQ-#### with EARS-style notation | Analysis |
| 2 | research-first | Force research before implementation on non-trivial work | Analysis |
| 3 | brainstorming | Enumerate approaches before decomposition | Planning |
| 4 | task-decomposition | Break plan into DAG with acceptance criteria | Planning |
| 5 | architecture-review | Compare change against architecture.md + ADRs | Arch Gate + Review |
| 6 | adr-authoring | Produce a well-shaped ADR-####-<slug>.md | Arch Gate |
| 7 | tdd | RED → GREEN → REFACTOR discipline | Implementation |
| 8 | systematic-debugging | Structured bug-finding (bisect, hypothesis testing) | Implementation |
| 9 | security-review | Threat model + vulnerability scan | Review |
| 10 | documentation-review | Doc drift + link check + terminology consistency | Review |
| 11 | runbook-authoring | Produce a well-shaped runbook | Runbook update |
| 12 | git-hygiene | Commits, branches, PR messages | All |
| 13 | release-verification | Ship gate checks in one place | Ship |
| 14 | retrospective | Workflow retrospective structure | Retrospect |
| 15 | verification-before-completion | Enforce evidence-before-done discipline | All |
| 16 | constitution-amendment | Propose constitution change (requires human approval) | Governance updates |

### 13.2 SKILL.md format

```markdown
---
name: architecture-review
description: |
  Review a code change against architecture.md and existing ADRs.
  Detect architecture drift, propose superseding ADRs where needed.
version: 1.0
harness_compatibility: [any]
tools_required: [filesystem-read, git-read, serena-read, mermaid-render]
governance_read:
  - architecture/architecture.md
  - architecture/adrs/*
  - invariants.md
governance_write:
  - architecture/adrs/*
context_budget: 8000-15000
manual_invocation:
  paste_as: system_prompt
  expected_input: "The diff to review + task context"
  expected_output: "works/<task-id>/architecture-review.md — findings classified BLOCKER/MAJOR/MINOR/SUGGESTION"
---

# Architecture Review

## Mandatory Step 1: Read governance
Before reviewing:
1. Read architecture/architecture.md
2. Read every ADR in architecture/adrs/
3. Read invariants.md
If any are missing, STOP and request the engineer runs the initializer agent.

## Step 2: Read the change
<...>

## Step 3: Classify findings
Every finding is BLOCKER | MAJOR | MINOR | SUGGESTION.
Every BLOCKER MUST cite a specific ADR ID, INV ID, or constitution rule.

## Step 4: Propose superseding ADRs
If the change invalidates an existing ADR, produce a draft superseding ADR
using the adr-authoring skill. Mark it Proposed until human-approved.

## When you encounter ambiguity
If you find something you cannot resolve from governance or task context,
DO NOT GUESS.
1. Create works/<task-id>/clarifications/C-<n>.md with:
   - The specific question
   - The options you see
   - Your recommendation (if any) with rationale
2. STOP. Return control to the engineer with:
   "Clarification C-<n> needed. See works/<task-id>/clarifications/C-<n>.md"

## Failure modes
- Over-broad findings without citation
- Missing supersession proposals when ADRs are invalidated
- Scope creep into quality-review territory

## Verification
- Every BLOCKER cites a governance ID
- Every proposed ADR includes rationale + supersession reference
- Output matches expected_output structure
```

### 13.3 Skill discipline

- **Length**: skill bodies target ≤300 lines. Longer skills should decompose into sub-skills.
- **Progressive disclosure**: frontmatter (name, description, tools, governance) is always in context. Body is loaded when the skill is actually invoked. Native in Codex CLI and Claude Code. Approximated in OpenCode and Antigravity via a two-stage prompt.
- **Governance-first**: Step 1 is always "read governance." No skill omits this.
- **Ambiguity handling**: every skill that could hit ambiguity includes the "When you encounter ambiguity" block above.
- **Output convention**: skills produce artifacts at `works/<task-id>/<step>.md`. The context-pack watcher (§16) handles the compressed handoff automatically.

---

## 14. Agents — the 17 canonical roles

Agents are bounded engineering roles. Each has a machine-readable AGENT.md contract at `/workspace/galley/agents/<role>/AGENT.md`.

### 14.1 Catalog

| # | Agent | Governance read | Governance write | Preferred harness | Preferred model tier |
|---|---|---|---|---|---|
| 1 | initializer | (creates all) | ALL | any | any |
| 2 | analyst | ALL relevant | requirements/* proposals | Antigravity | long-context (Gemini 3 Pro) |
| 3 | planner | constitution, req, arch, ADRs, invariants | (none) | Codex CLI or Antigravity | reasoning (GPT-5.6-Luna) |
| 4 | architecture-gate | architecture, ADRs, invariants | ADR proposals | Codex CLI or Antigravity | reasoning-heavy |
| 5 | implementer | invariants, ADRs, ubi_lang | (proposes ADRs when needed) | OpenCode | coding (Qwen 3.8 27B) |
| 6 | tester | requirements, invariants | (none) | OpenCode | coding |
| 7 | architecture-reviewer | architecture, ADRs, invariants | ADR + invariant proposals | Codex or Antigravity | reasoning-heavy |
| 8 | security-reviewer | constitution (sec), invariants | (none) | Codex CLI | reasoning |
| 9 | quality-reviewer | ubi_lang, constitution (quality) | (none) | any | reasoning |
| 10 | documentation-reviewer | ubi_lang, existing docs | doc updates | any | reasoning |
| 11 | runbook-reviewer | operational sections | runbook updates | any | reasoning |
| 12 | triage-agent | invariants, requirements | (creates corrective tasks) | any | reasoning |
| 13 | ship-agent | requirements, ADRs | (none) | any | reasoning |
| 14 | archive-agent | (deterministic copy) | (none) | any | (any — mechanical) |
| 15 | retrospect-agent | (reads works/) | proposes constitution/skill improvements | any | reasoning |
| 16 | hindsight-agent | (reads multiple retrospectives) | proposes governance changes | any | reasoning-heavy |
| 17 | horizon-agent | (reads hindsight outputs) | proposes new skills/agents | any | reasoning-heavy |

The "preferred harness" and "preferred model tier" columns are guidance for the engineer. Actual model selection happens in the harness UI at invocation time.

### 14.2 AGENT.md format

```markdown
---
id: architecture-reviewer
version: 1
role: architecture-reviewer
skills: [architecture-review, adr-authoring]
governance_read:
  - constitution.md
  - ubiquitous_language.md
  - invariants.md
  - architecture/architecture.md
  - architecture/adrs/*
governance_write:
  - architecture/adrs/*
  - invariants.md
preconditions:
  - implementation-report.md exists in works/<task-id>/
  - diff attached
  - task acceptance criteria loaded
tools_required:
  - filesystem-read
  - git-read
  - serena-read
  - mermaid-render
manual_invocation:
  compatible_harnesses: [opencode, codex-cli, antigravity, cursor-cli]
  independence_rule: "MUST NOT be the same harness that produced the implementation"
  paste_as: system_prompt
  expected_output: "works/<task-id>/architecture-review.md classifying findings BLOCKER / MAJOR / MINOR / SUGGESTION with governance citations"
---

# Architecture Reviewer

You are Galley's Architecture Reviewer.

## Mandatory Step 1: Read governance
Before doing ANYTHING else, read:
- constitution.md
- ubiquitous_language.md
- invariants.md
- architecture/architecture.md
- every file in architecture/adrs/
If any are missing, STOP.

## Step 2: Verify preconditions
Verify each item under `preconditions` in this contract is satisfied.
If not, STOP and ask the engineer.

## Step 3: Apply the architecture-review skill
See /workspace/galley/skills/architecture-review/SKILL.md.

## Step 4: Cite every BLOCKER
Every BLOCKER finding MUST cite a specific ADR ID, invariant ID, or
constitution rule. Findings without citations are downgraded to SUGGESTION.

## When you encounter ambiguity
Create works/<task-id>/clarifications/C-<n>.md and STOP.
```

### 14.3 Cross-harness reviewer independence

Every reviewer agent contract carries an `independence_rule` field:

> **The reviewer's harness MUST NOT be the same harness that produced the implementation being reviewed.**

If the implementer wrote code in OpenCode with a local model, the reviewer must run in Codex CLI or Antigravity — a different provider entirely. This is manually enforced by the engineer in Phase 1 (documented in the manual workflow runbook).

Rationale: the same model reviewing its own work misses its own systematic biases. Different harness = different provider = independent judgment.

---

## 14A. References — where our skills and agents come from

**We adapt from established open-source projects rather than writing every skill and agent from scratch.** Each of the 16 skills and 17 agents in §13 and §14 draws on one or more existing projects for its shape, procedure, and known failure modes — then gets rewritten in Galley's terminology, integrated with our governance-first convention, and pared down to solo-engineer scale.

This section names the sources. The intent is threefold:

1. **Faster time-to-value.** Adapting a well-tested skill from Superpowers or GSD is faster and safer than authoring from zero.
2. **Provenance.** Every borrowed pattern is traceable to its source. Commit messages and skill frontmatter reference the origin so anyone reading later understands the lineage.
3. **License compliance.** We respect each project's license (mostly MIT / Apache 2.0). Attribution goes in each adapted skill/agent's frontmatter under a `derived_from:` field.

### 14A.1 Sources by concern

**Skills — methodology / workflow discipline:**

- **[Superpowers (obra/superpowers)](https://github.com/obra/superpowers)** — 224k+ GitHub stars, MIT. Agentic skills framework with mandatory-workflow discipline.
  - **Adapt for:** `tdd`, `systematic-debugging`, `brainstorming`, `verification-before-completion`, `git-hygiene`, `release-verification`, `subagent-driven-development` (v0.3+).
  - **What to change:** their "mandatory for everything" stance → our advisory-per-task-type. Remove Superpowers-specific plugin structure; keep the skill body.
- **[GSD (Get Sh*t Done)](https://docs.opengsd.net/core/introduction)** — 48k+ GitHub stars, MIT. Spec-driven development with fresh-context-per-phase.
  - **Adapt for:** `requirements-analysis`, `task-decomposition`, `release-verification`. GSD's Plan / Execute / Review shape maps cleanly to our analysis → planning → implementation → review flow.
  - **What to change:** GSD is coupled tightly to slash-command idioms in Claude Code; generalize to our harness-agnostic SKILL.md format.
- **[Compound Engineering plugin (EveryInc)](https://github.com/EveryInc/compound-engineering-plugin)** — MIT. 29 agents, 20 skills, 22 slash commands. Plan / Execute / Review 80/20 workflow.
  - **Adapt for:** `retrospective`, `research-first`, and multi-dimensional `review` skills (architecture / security / quality patterns).
  - **What to change:** their 29-agent count is too coarse; distill down to our 17 canonical agents.
- **[ECC / Everything Claude Code (affaan-m/everything-claude-code)](https://github.com/affaan-m/everything-claude-code)** — 214k+ GitHub stars, MIT. 262 skills, 64 agents.
  - **Adapt for:** `security-review`, memory / persistence concepts, cross-harness compatibility patterns.
  - **What to change:** ECC's "install everything" default is bloat; cherry-pick the specific skills we need.

**Skills — governance and spec-driven workflow:**

- **[GitHub Spec Kit (github/spec-kit)](https://github.com/github/spec-kit)** — MIT, GitHub's official spec-driven dev toolkit. Constitution → Plan → Tasks → Implement pipeline.
  - **Adapt for:** `requirements-analysis` (EARS-style notation), `adr-authoring` (structured decision records), `constitution-amendment` (the constitution concept itself).
  - **Templates for:** `constitution.md`, `requirements/functional.md` — Spec Kit's templates are directly usable with light rewrite.
- **[Kiro (AWS)](https://kiro.dev/)** — spec-first IDE using EARS notation for requirements. Requires an AWS account so we don't use the IDE, but the notation and three-artifact pattern (requirements.md / design.md / tasks.md) are adaptable.
  - **Adapt for:** `requirements-analysis` (EARS-style notation), the three-artifact bootstrap pattern.
  - **What to skip:** the VS Code fork / Bedrock coupling.

**Agent contracts and cross-harness workflow:**

- **[AI-SDLC Framework (ai-sdlc-framework/ai-sdlc)](https://github.com/ai-sdlc-framework/ai-sdlc)** — declarative resource types (Pipeline, AgentRole, QualityGate, AutonomyPolicy, AdapterBinding). Apache 2.0.
  - **Adapt for:** AGENT.md contract structure (their AgentRole → our Agent), the cross-harness reviewer independence rule (§14.3), the spec/status split pattern.
  - **What to change:** their Kubernetes-style API ceremony is enterprise-scale; simplify to solo-engineer scale. Rename QualityGate → Checkpoint (§3).
- **[MetaGPT (geekan/MetaGPT)](https://github.com/geekan/MetaGPT)** — MIT. "Software company in a box" with PM / Architect / Engineer / QA agent roles.
  - **Adapt for:** the SDLC role taxonomy (analyst, planner, implementer, tester, reviewers).
  - **What to change:** MetaGPT's tight multi-agent coupling → our loosely-coupled artifact-based handoff.

**Skills / agents for coding-specific patterns:**

- **[Aider (paul-gauthier/aider)](https://github.com/paul-gauthier/aider)** — Apache 2.0. Repomap pattern, architect+editor split.
  - **Adapt for:** `implementer` agent's architect-mode variant (planning model + editing model per task); repomap conventions.
  - **Directly used:** repomap generation (tree-sitter + PageRank) as a technique in our code-context §15.
- **[Serena (oraios/serena)](https://github.com/oraios/serena)** — MIT. LSP-backed semantic code retrieval.
  - **Directly adopted:** as our first-party code-intelligence MCP server (see §12.1). No adaptation needed — used as-is.

**Runbook + operational patterns:**

- **[PagerDuty runbook templates](https://response.pagerduty.com/before/writing_runbook/)** — structure for operational runbooks.
  - **Adapt for:** `runbook-authoring` skill's output shape (prerequisites, procedure, expected output, failure handling, rollback).
- **[Rundeck (rundeck/rundeck)](https://github.com/rundeck/rundeck)** — runbook execution engine.
  - **Adapt as reference only:** for how executable operational knowledge is structured. We don't adopt Rundeck itself.

**Container packaging patterns:**

- **[HolyClaude (CoderLuii/HolyClaude)](https://github.com/CoderLuii/HolyClaude)** — MIT. Docker-based AI coding workstation with s6-overlay + dual-volume UID remapping.
  - **Directly adopted for:** container packaging discipline, UID remapping for host-native file ownership, s6-overlay for multi-service supervision, one-step docker-compose deployment (see §9, §11).

**Layout convention:**

- **[agentic-code (shinpr/agentic-code)](https://github.com/shinpr/agentic-code)** — `.agents/{tasks,workflows,skills}` layout with progressive rule loading.
  - **Adapt for:** the `/workspace/galley/{skills,agents,templates}/` layout convention.

### 14A.2 Attribution convention

Every adapted skill and agent frontmatter carries a `derived_from` field:

```yaml
---
name: tdd
version: 1.0
derived_from:
  - project: superpowers
    url: https://github.com/obra/superpowers
    license: MIT
    original_skill: tdd
    modifications: |
      - Removed Superpowers-specific plugin structure
      - Aligned with Galley's governance-first Step 1 convention
      - Rewrote in Galley terminology (see §3 of Phase 1 spec)
      - Added `governance_read` and `governance_write` frontmatter
---
```

This makes provenance greppable and license attribution automatic when we ship the container image.

### 14A.3 What we do NOT adapt from these sources

- **The "install everything" default** from ECC / Superpowers. Our 16 skills + 17 agents are the curated minimum.
- **Kubernetes API ceremony** from AI-SDLC Framework. Solo-engineer scale.
- **Mandatory-for-every-task discipline** from Superpowers. Skills are advisory unless a task explicitly requires them.
- **Multi-agent conversation orchestration** from MetaGPT / AutoGen. Our agents hand off through artifacts, not conversations.
- **Slash-command tight coupling to Claude Code** from GSD / Superpowers. Our skills are harness-agnostic SKILL.md files.

### 14A.4 Impact on Spike A

Spike A (§22.1 — draft one skill + one agent contract, validate, iterate format) becomes concretely easier:

1. Pick the source project for the target skill (`architecture-review` → adapt from Superpowers' architecture-related skills or AI-SDLC's QualityGate patterns).
2. Copy the source's SKILL.md / equivalent as a starting point.
3. Rewrite in Galley terminology (Checkpoint / Clarification / Precondition, per §3).
4. Add `governance_read` / `governance_write` frontmatter and the mandatory Step 1 body block.
5. Add `derived_from` attribution.
6. Validate against a real ticket in each of the four harnesses.

Total time-to-first-skill drops from "author from scratch" (many hours) to "adapt-and-validate" (a couple of hours).

---

## 15. Code context — Serena + repomap

Code context is delivered to agents via two complementary mechanisms.

### 15.1 Serena (semantic, LSP-backed)

Serena runs as an HTTP MCP server on port 3110 inside the container. It exposes symbol-level operations across 20+ languages via the Language Server Protocol:

- `find_symbol(name)` — precise symbol resolution
- `find_referencing_symbols(symbol)` — real cross-references (via LSP, not embedding similarity)
- `get_symbol_body(symbol)` — pull a symbol's implementation on demand
- `replace_symbol_body(symbol, new_body)` — surgical symbol-boundary edit
- Serena project memory scoped to `.galley/serena/`

Every skill and agent that touches code lists `serena-read` (and, for implementer only, `serena-write`) in its tools.

### 15.2 Repomap (static, PageRank-based)

At the start of every code-touching agent invocation, the agent is given a compact **repomap** of the target repo — a token-budgeted (~1000-2000 tokens) summary of the most-referenced symbols in the codebase. Produced by tree-sitter parse + personalized PageRank biased toward files in the current task's scope.

Repomap gives the agent up-front orientation. Serena lets it navigate precisely once oriented. Both together are the "just-enough context" answer for code.

### 15.3 Never dump raw files

Skills instruct agents: prefer `find_symbol` and `get_symbol_body` over asking the engineer to paste full files. Full-file dumps are wasteful of tokens and degrade LLM accuracy at scale (Chroma "context rot" research shows 30–50% accuracy loss well before nominal window limits).

---

## 16. Context-pack watcher

Automatic, deterministic context compression at the SDLC handoff points.

### 16.1 What it does

A filesystem-watching service (`galley-pack-watcher`) runs as an s6-supervised service inside the container. When an SDLC-step artifact settles on disk, it produces a compressed handoff file for the next agent.

```
Agent produces:      works/T0003/analysis.md      (~5-10K tokens, human-focused)
                                    │
                     (30-second debounce for file to settle)
                                    │
                                    ▼
Watcher invokes:     local LLM via .galley/space.yaml local_model.base_url
                                    │
                     (compression using a target-specific prompt template)
                                    │
                                    ▼
Watcher writes:      works/T0003/analysis.pack.md (~500-2000 tokens, planner-focused)
Watcher logs:        works/T0003/logs/pack-analysis.log
```

### 16.2 Compression targets

The watcher determines the compression target from the artifact's filename:

| Artifact filename | Compressed for |
|---|---|
| `analysis.md` | planner |
| `plan.md` | architecture-gate |
| `architecture-gate.md` | implementer |
| `implementation-report.md` | reviewers (arch / security / quality) |
| `review.md` | triage |
| `triage.md` | ship |
| `ship-report.md` | archive |

Overridable per-file via `.galley/space.yaml`:

```yaml
pack:
  overrides:
    "works/*/custom-analysis.md": "planner"
```

### 16.3 Why watcher instead of hook or skill instruction

- **Zero tokens** in the agent's premium session (the pack is produced outside the LLM's context).
- **Deterministic** — packing happens whether the agent remembers to invoke it or not.
- **Harness-agnostic** — same behavior whether the artifact was produced by OpenCode, Codex CLI, or Antigravity.
- **Skill authoring stays clean** — skills focus on the actual work, no orchestration mixed in.

### 16.4 Configuration in `.galley/space.yaml`

```yaml
pack:
  enabled: true                              # off to disable auto-pack
  debounce_seconds: 30                       # wait for file to stop being written
  overrides:
    # (empty by default)
```

### 16.5 Manual override — `galley pack`

The engineer can regenerate a pack manually:

```bash
galley pack works/T0003/analysis.md                     # infers target from filename
galley pack works/T0003/analysis.md --for architecture  # override target
```

Same implementation as the watcher, just triggered on demand.

### 16.6 Local LLM unavailable

If the endpoint is unreachable when the watcher fires:

- The watcher writes an error to `works/<task-id>/logs/pack-<step>.log`.
- The pack file is NOT created (the engineer sees only the human artifact, and the missing pack).
- The engineer's next agent invocation uses the full human artifact instead of the pack (larger context, higher token cost, but functional).
- Once the endpoint is back, the engineer runs `galley pack <artifact>` manually.

The workflow is never blocked by a missing pack — the human artifact is always sufficient.

---

## 17. Clarifications — handling ambiguity

When any agent encounters ambiguity it cannot resolve from governance or task context, it does not guess. It creates a clarification file and stops.

### 17.1 Convention

```
works/<task-id>/clarifications/
├── C-1.md
├── C-2.md
└── ...
```

Each `C-<n>.md` follows a fixed shape:

```markdown
# Clarification C-1

**Raised by:** analyst (invoked in Antigravity)
**Raised at:** 2026-08-19T14:23:00Z

## Question
<the specific question that needs resolution>

## Options
1. <option 1 with brief explanation>
2. <option 2 with brief explanation>
3. <option 3 with brief explanation>

## Recommendation (if the agent has one)
<option N — with rationale>

## Resolution
<empty — engineer fills this in>
```

### 17.2 Behavior

- The agent writes the clarification file and stops.
- The engineer opens `C-<n>.md`, writes their resolution under `## Resolution`.
- The engineer re-invokes the same agent (or continues the session in the same harness — depending on the harness's support for resuming).
- The re-invoked agent reads `C-<n>.md`, sees the resolution, continues.
- Clarifications remain in the workflow's `works/` directory as durable audit — every guess-avoided is captured.

### 17.3 Which skills produce clarifications

Skills that raise clarifications when needed: `requirements-analysis`, `brainstorming`, `task-decomposition`, `architecture-review`, `adr-authoring`, `security-review`, `documentation-review`, `runbook-authoring`.

Skills that don't (mechanical / deterministic work): `git-hygiene`, `tdd` (once acceptance criteria are clear), `verification-before-completion`.

### 17.4 Suppression

The engineer can mark a decision-worthy item as intentional-non-decision with a comment:

```html
<!-- galley:not-a-clarification -->
```

Placed anywhere in the source artifact, this tells the agent "this ambiguity is intentional; proceed."

---

## 18. The manual SDLC workflow

Documented in full at `/workspace/galley/docs/runbooks/galley-manual-workflow.md`.

### 18.1 Prerequisites

- `galley init <space>`
- Edit `.galley/.env` with API keys and local-model settings
- `galley <space> repo add <git-url>` — accepts governance skeleton if repo lacks it
- `galley build && galley start <space> && galley shell <space>`
- Inside the container, `cd /workspace/repos/<repo>`
- Create `works/T####/` for the current task

### 18.2 The 8-step SDLC — a typical feature

| # | Step | Harness | Model tier | Agent | Output artifact | Auto-packed for |
|---|---|---|---|---|---|---|
| 1 | Analysis | Antigravity | gemini-3-pro | analyst | `analysis.md` | planner |
| 2 | Planning | Codex CLI | gpt-5.6-luna | planner | `plan.md` | architecture-gate |
| 3 | Architecture Gate | Codex or Antigravity | reasoning-heavy | architecture-gate | `architecture-gate.md` (+ possibly ADR proposals) | implementer |
| 4 | Implementation | OpenCode | qwen3.8-27b (local) | implementer | code diff + tests + `implementation-report.md` | reviewers |
| 5 | Verification | OpenCode | qwen3.8-27b | tester (or same implementer) | test logs; `verification.md` | reviewers |
| 6 | Documentation + Runbook | any | reasoning | documentation-reviewer + runbook-reviewer | doc/runbook updates | reviewers |
| 7 | Review (architecture / security / quality) | ≠ implementer's harness | reasoning-heavy | architecture-reviewer + security-reviewer + quality-reviewer | `review.md` | triage |
| 8 | Ship + Archive | any | reasoning | ship-agent then archive-agent | PR URL + `vault/inbox/<task-id>/` | (archived — no further pack) |

Note: **step 7 uses a different harness than step 4** (cross-harness reviewer independence, §14.3).

### 18.3 Governance touchpoints throughout

Every one of the 8 steps starts with Step 1: read governance. Ambiguity is captured as clarifications. Proposed governance changes (new ADRs, new invariants) travel through the same 8 steps for their own approval.

### 18.4 Ship — step 8a

The ship agent's job:

1. Read `works/<task-id>/plan.md`, `review.md`, `implementation-report.md`.
2. Verify the release-verification skill's checklist:
   - All required tests pass
   - Documentation is updated for behavior/API changes
   - Runbook is updated if operationally significant
   - No unresolved clarifications
   - No unaddressed BLOCKER review findings
   - Git state clean (working branch has commits; nothing dirty)
3. Produce `works/<task-id>/ship-report.md` with the checklist result.
4. If all green, run:
   ```bash
   gh pr create \
     --title "<from plan.md>" \
     --body "$(cat works/<task-id>/ship-report.md)" \
     --base main \
     --head workflow/<task-id>
   ```
5. Write the PR URL back to `ship-report.md`.

`GITHUB_TOKEN` from `.galley/.env` is what `gh` uses.

If any check fails, the ship agent stops and reports the failure. The engineer resolves the issue and re-invokes.

### 18.5 Archive — step 8b

The archive agent's job (mechanical, no LLM strictly required):

1. Create `vault/inbox/<task-id>/`.
2. Copy every file from `works/<task-id>/` to `vault/inbox/<task-id>/`.
3. Write `vault/inbox/<task-id>/manifest.md`:
   ```markdown
   # Workflow Manifest: T####

   **Requirement:** <from plan.md>
   **Started:** <timestamp>
   **Shipped:** <timestamp>
   **PR:** <URL from ship-report.md>

   ## Artifacts
   - analysis.md (+ analysis.pack.md)
   - plan.md (+ plan.pack.md)
   - architecture-gate.md (+ pack)
   - implementation-report.md (+ pack)
   - review.md (+ pack)
   - triage.md (+ pack) — if triage was needed
   - ship-report.md

   ## Clarifications resolved
   - C-1: <one-line summary>
   - C-2: <one-line summary>

   ## Governance touched
   - ADRs: ADR-0007 (new), ADR-0003 (superseded by ADR-0007)
   - Invariants: (none)
   - Requirements: REQ-0012 (implemented)
   ```
4. Leaves `works/<task-id>/` in place. Engineer may clean up manually if desired.

### 18.6 Governance touched during a workflow

If a workflow proposes new ADRs, new invariants, or requirement updates, those proposed files live at `works/<task-id>/proposed/`:

```
works/<task-id>/proposed/
├── architecture/adrs/
│   └── ADR-0007-adopt-litellm.md
└── invariants.md.diff              # unified diff against invariants.md
```

The engineer reviews these proposals as part of the ship step. Accepted proposals are committed to the repo as part of the workflow's PR. Rejected proposals stay in `works/` for audit.

---

## 19. Vault archive

`vault/inbox/<task-id>/` is the durable resting place for a completed workflow. Everything the workflow produced — human artifacts, context packs, clarifications, proposed governance changes, ship report — lives there.

Contents are copied by the archive-agent (§18.5) at ship time. No hashing, no manifest generation beyond the plain-markdown one, no atomic staging — the vault directory is just a well-organized filesystem archive of completed workflows.

---

## 20. Documentation deliverables

Ships in the image at `/workspace/galley/docs/`. All engineer-facing documentation is written in Phase 1 alongside the skills and agents.

| File | Purpose | Target length |
|---|---|---|
| `docs/runbooks/galley-manual-workflow.md` | The 8-step SDLC lookup table + recipes + harness cheat-sheet + troubleshooting | ≤500 lines |
| `docs/guides/getting-started.md` | First-time engineer setup | ≤200 lines |
| `docs/guides/writing-a-skill.md` | How engineers author their own skills | ≤150 lines |
| `docs/guides/per-harness-invocation.md` | How each of OpenCode / Codex CLI / Antigravity is launched, expected quirks | ≤150 lines |
| `docs/guides/governance-first.md` | The discipline of §4 in one place | ≤150 lines |
| `docs/guides/anti-lock-in.md` | Why four harnesses, why subscription-tier arbitrage, ToS-compliance rationale | ≤100 lines |
| `docs/guides/references.md` | Which open-source projects each skill and agent is adapted from, with license attributions | ≤200 lines |

---

## 21. Acceptance criteria

Phase 1 is done when every one of the following passes.

### 21.1 CLI works on macOS + Ubuntu

- `uv tool install galley` (or `brew install galley` on macOS) succeeds.
- `galley version` reports version.
- `galley doctor` reports Docker healthy.
- `galley` runs cleanly on macOS (arm64 + x86_64) and Ubuntu 22.04+ (amd64 + arm64).

### 21.2 Space lifecycle

- `galley init <space>` creates `~/Documents/galley/<space>/` with the layout in §5.1.
- `.galley/.env` is scaffolded with placeholders + explanatory comments.
- `.gitignore` covers `.galley/.env*` and other sensitive paths.
- `galley list` shows the Space.
- `galley destroy <space>` removes it (with confirmation).

### 21.3 Repo management

- `galley <space> repo add <url>` clones the repo.
- Governance skeleton offered when the target repo lacks the five governance files.
- Templates copied on acceptance; engineer prompted to fill project specifics.
- `galley <space> repo list` shows the repo.
- `galley <space> repo remove` removes it (with confirmation).

### 21.4 Container lifecycle

- `galley build` produces `galley:full` cleanly on amd64 and arm64.
- `galley start <space>` boots the container in ≤30 seconds; s6-overlay starts the 7 base MCP servers plus the context-pack watcher.
- `galley shell <space>` gives an interactive prompt as the engineer's user.
- `galley stop <space>` cleanly stops within 30 seconds.
- `galley restart <space>` cleanly cycles.
- `galley logs <space>` streams container logs.

### 21.5 Per-harness content installation

- `setup-harnesses.sh` runs at container boot without errors.
- Inside the container:
  - `opencode` (from a repo directory) discovers all 16 skills + 17 agents via `~/.config/opencode/`.
  - `codex` reads `~/.codex/AGENTS.md` (symlinked to canonical) and its MCP config from `~/.codex/config.toml`.
  - `agy` finds skills at both `~/.gemini/skills/` and `~/.gemini/antigravity-cli/skills/`, reads MCP from `~/.gemini/config/mcp_config.json`.
  - `cursor-agent` (Cursor CLI) discovers skills / agents via `~/.cursor/` (exact path convention verified in Spike C).
- `galley exec <space> ls ~/.config/opencode/skills/` lists all 16 skills.

### 21.6 MCP servers

- `galley exec <space> galley-mcp status` reports 7 base servers RUNNING.
- Serena at `http://localhost:3110/health` responds OK.
- Optional MCPs are RUNNING iff their ID is in `space.yaml` `mcp.enabled[]` AND their required env vars are present.

### 21.7 Governance-first is enforced in content

- Every skill's `SKILL.md` frontmatter declares `governance_read`.
- Every agent's `AGENT.md` frontmatter declares `governance_read` and (where applicable) `governance_write`.
- Every agent's body opens with the "Mandatory Step 1: Read governance" section that STOPS on missing files.
- Every skill that can hit ambiguity carries the "When you encounter ambiguity" block with the clarification-file convention.

### 21.8 Context-pack watcher

- `galley-pack-watcher` is running (visible in `galley exec <space> ps auxf`).
- Writing a file matching `works/<task>/analysis.md`, `plan.md`, etc., triggers packing after 30s debounce.
- Pack appears at `works/<task>/analysis.pack.md` etc.
- Pack was produced by the local model (visible in `works/<task>/logs/pack-analysis.log`).
- If the local endpoint is unreachable, the error is logged, the pack is skipped, the workflow continues.
- `galley pack <artifact> --for <target>` manual invocation works.

### 21.9 The manual SDLC completes end-to-end for a real ticket

- Engineer picks a real ticket in a real repo.
- Follows the 8 steps in the runbook using Antigravity → Codex CLI → OpenCode.
- Every step's agent reads governance at Step 1 (verified by inspecting the agent's session output).
- Cross-harness reviewer independence is respected (verified: reviewers run in a different harness than implementation).
- Ambiguities are captured as clarifications and resolved by the engineer.
- Ship agent invokes `gh pr create` successfully.
- Archive agent copies to `vault/inbox/<task-id>/`.
- The engineer reports the experience is pleasant enough to want to keep using it.

### 21.10 Documentation is complete

- All six documentation files (§20) are written and shipped in the image.
- The manual-workflow runbook covers all 8 steps + recipes + troubleshooting.

### 21.11 Spikes complete

- **Spike A** — draft one skill (architecture-review) + one agent contract (architecture-reviewer) in the format specified in §13.2 / §14.2. Validate against a real ticket end-to-end via manual invocation in each of the three harnesses. Iterate format if needed before scaling to the remaining 15 skills / 16 agents.
- **Spike B** — s6-overlay + Serena MCP startup ordering. Verify boot ordering, timeout handling, restart-on-crash.
- **Spike C** — per-harness content installation. Verify the symlink + config-generation approach works cleanly for all four harnesses (OpenCode + Codex CLI + Antigravity + Cursor CLI).
- **Spike D** — context-pack watcher. Verify inotify (Linux) + fsevents (macOS) file-watching triggers correctly, debounce works, local-LLM invocation succeeds and writes valid packs. Verify local-endpoint-unreachable is handled cleanly.

Only when all 11 criteria pass does Phase 1 ship as v0.1.

---

## 22. Spikes

Three spikes gate Phase 1 delivery. All are time-boxed and produce concrete deliverables.

### 22.1 Spike A — Adapt one skill + one agent contract; iterate format

**Time-box:** 1 day
**Approach:** adapt an existing skill from Superpowers, GSD, or Compound Engineering (per §14A) rather than authoring from scratch. This reduces time-to-first-skill dramatically and lets us validate the *adaptation pattern* itself.

**Deliverable:** `/workspace/galley/skills/architecture-review/SKILL.md` and `/workspace/galley/agents/architecture-reviewer/AGENT.md` in the final format, both carrying `derived_from` attribution. Run through a real ticket in each of the four harnesses. Confirm frontmatter reads cleanly when pasted, LLM follows the procedure reliably, output matches `expected_output`, governance-first Step 1 is respected, clarification convention is honored.

Iterate the format based on findings. Only after the two files are stable, scale to the remaining 15 skills + 16 agents by adapting from the sources named in §14A.

### 22.2 Spike B — s6-overlay + Serena MCP startup ordering

**Time-box:** 1 day
**Deliverable:** `setup-harnesses.sh` + s6 service definitions. Verify:

- Startup order is deterministic (filesystem → git → fetch → serena → mermaid → github-pr → test-runner → watcher).
- Serena's HTTP endpoint reaches ready state in ≤10 seconds cold-start on Debian slim.
- s6's dependency graph handles missing env vars gracefully (github-pr is skipped, not failed, when GITHUB_TOKEN is unset).
- Restart-on-crash works for each supervised service.

### 22.3 Spike C — Per-harness content installation

**Time-box:** 2 days
**Deliverable:** `setup-harnesses.sh` producing correct symlinks and generated MCP configs. Verify:

- Each harness discovers the symlinked skills correctly (skill invocation works end-to-end).
- Generated MCP configs are valid syntax for each harness.
- Each harness picks up all 7 base MCPs.
- Per-Space custom overrides shadow canonical entries without breaking discovery.
- Symlink approach survives harness upgrades.

If any harness rejects symlinks (unlikely), fall back to copy-on-boot for that harness.

### 22.4 Spike D — Context-pack watcher

**Time-box:** 1 day
**Deliverable:** `galley-pack-watcher` s6 service + `galley pack` CLI. Verify:

- Inotify (Linux) and fsevents (macOS) file-watching correctly detects writes under `works/<task>/`.
- Debounce logic (30-second default) waits for file to settle before firing.
- Only SDLC-step artifacts (`analysis.md`, `plan.md`, …) trigger packing; `.pack.md` files and non-SDLC files are ignored.
- The watcher calls the local model at `local_model.base_url` and writes the pack file within seconds.
- Local-endpoint-unreachable is logged and does not block the workflow.
- `galley pack <artifact> --for <target>` manual invocation produces the same output as the watcher.

---

## 23. Platforms

Officially supported for Phase 1:

- **macOS** — arm64 (Apple Silicon) and x86_64 (Intel Mac).
- **Ubuntu Linux** — 22.04 LTS or newer, amd64 or arm64.

Docker Desktop (macOS) and Docker Engine (Linux) are the container runtimes. Docker version 24+ required.

---

## 24. Anti-lock-in and cost governance

### 24.1 The four harnesses respect provider ToS

Each cloud provider is used through *its own* native harness with the engineer's subscription to that provider:

- **OpenAI** models (GPT-5.6-Luna, o-series) — through **Codex CLI** with the engineer's ChatGPT Plus / Pro subscription
- **Google** models (Gemini 3 Pro, Gemini 3.5 Flash) — through **Antigravity CLI** with the engineer's Gemini AI Pro subscription
- **Cursor's model routing** (Cursor's custom + fallback to major providers) — through **Cursor CLI** with the engineer's Cursor Start / Pro subscription
- **Local** models (Qwen 3.8 27B or similar) — through **OpenCode** pointed at the engineer's LiteLLM proxy / llama.cpp / vLLM endpoint

Anthropic Claude models are used through **Claude Code**, which the engineer typically runs on the host, not inside the container. Claude Code can optionally be installed manually inside the container (`npm i -g @anthropic-ai/claude-code`).

Galley never routes an API key from one provider through a different harness. This preserves subscription ToS across every provider.

### 24.2 Cost is bounded by subscription

Because each provider is accessed through its own subscription, the engineer's cost per workflow is bounded by flat monthly subscription costs — not by pay-as-you-go token bills. A typical setup:

- **Cursor Start:** ~$6.49 / ₹649 per month (entry tier)
- **ChatGPT Plus:** ~$20/mo
- **Gemini AI Pro:** ~$20/mo
- **Local model:** electricity only
- **Optional:** Claude Pro (~$20/mo) on host, or drop one of the subscriptions above

Total: ~$26 to $60/mo for unlimited (subject to each provider's fair-use limits) access to four provider ecosystems. The Cursor Start tier meaningfully lowers the floor — an engineer can start with just Cursor Start + local model for ~$7/mo total.

### 24.3 Context management minimizes premium-token consumption

Several Phase 1 mechanisms reduce token consumption in premium harnesses:

- **Governance loading is relevance-filtered** (§4.4) — only the specific requirements/ADRs/invariants a task touches are loaded.
- **Serena delivers symbol-level code context** (§15.1) — no raw file dumps.
- **Repomap is token-budgeted** (§15.2) — ~1000-2000 tokens for whole-repo orientation.
- **Context-pack watcher moves compression to the local model** (§16) — the next agent reads a compressed pack, not the full artifact.
- **Skill progressive disclosure** (§13.3) — skill body loads only when invoked.

---

## 25. Governance summary — the promise Galley v0.1 makes

Every Galley workflow, in Phase 1:

1. Reads the five governance documents before doing consequential work.
2. Filters governance loading by task relevance to control token cost.
3. Captures ambiguity as clarifications rather than guessing.
4. Uses different harnesses for implementation and review, so review is independent.
5. Cites governance IDs in every BLOCKER finding.
6. Produces both a human-readable artifact and a compressed pack for the next agent — the pack via the local model, deterministically.
7. Ships as a PR opened by the ship agent using `gh pr create`.
8. Archives every artifact + clarification + governance proposal to `vault/inbox/<task-id>/`.

Between the container, the pre-installed content, and these conventions, an engineer can drive a full SDLC across three different provider harnesses without architecture drift, provider lock-in, or unbounded cost.

That is v0.1.

---

*End of Phase 1 Specification.*
