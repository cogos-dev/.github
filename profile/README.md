# cogos-dev

Infrastructure for AI agents that need persistent memory, intelligent context, and cross-device coordination.

CogOS is a local-first cognitive daemon written in Go with a Python voice channel. It gives AI tools like Claude Code and Ollama-backed agents the things they can't give themselves: workspace memory that persists across sessions, a trained retrieval model that surfaces the right context automatically, multi-provider inference routing with sovereignty-aware scheduling, a bidirectional voice pipeline, and now an observable, mutable substrate — hash-chained ledger, live event bus, kernel slog capture, and agent-state control — all exposed through MCP and HTTP. Everything runs on your hardware. Everything stays yours.

The kernel ships 7 importable library packages (`pkg/`), a native agent harness with local model support (Gemma E4B), a 20+ tool MCP surface, and protocol adapters for MCP Streamable HTTP and the Anthropic Messages API. Modality servers implement typed interfaces in any language — the wire protocol is the contract.

## Architecture

```
                        ┌─────────────────────────────┐
                        │      AI AGENTS / TOOLS       │
                        │  Claude Code · Ollama agents  │
                        └──────────┬──────────────────┘
                                   │ prompts · MCP · HTTP
                                   ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                     COGOS KERNEL  [Go daemon :6931]                      │
│                                                                          │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────────────┐   │
│  │ FOVEATED CONTEXT │  │  MULTI-PROVIDER  │  │   RECONCILIATION      │   │
│  │   ASSEMBLY       │  │    ROUTING       │  │    FRAMEWORK          │   │
│  │                  │  │                  │  │                       │   │
│  │ TRM scoring      │  │ OpenAI-compat    │  │ 8 providers           │   │
│  │ Zone-ordered     │  │ Anthropic-compat │  │ plan/apply/snapshot   │   │
│  │ KV-cache aware   │  │ Sovereignty      │  │ Kahn's topo sort     │   │
│  │                  │  │   gradient       │  │ Atomic state +       │   │
│  │ Hook: on every   │  │                  │  │   lineage tracking   │   │
│  │   user prompt    │  │ Routes to best   │  │                       │   │
│  └────────┬─────────┘  │   available      │  │ Providers: agent,    │   │
│           │             │   provider       │  │  component, discord, │   │
│           ▼             └───────┬──────────┘  │  service, openclaw×3,│   │
│     [context window]           │              │  mcp-tools           │   │
│                                ▼              └───────────────────────┘   │
│                     ┌──────────────────────┐                             │
│                     │  INFERENCE PROVIDERS  │                             │
│                     ├──────────┬───────────┤                             │
│                     │  Local   │  Cloud    │                             │
│                     │  Ollama  │  Claude   │                             │
│                     │  LM Stu. │  GPT      │                             │
│                     │  Gemma   │  Codex    │                             │
│                     │  (LAN)   │           │                             │
│                     └──────────┴───────────┘                             │
│                                                                          │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────────────┐   │
│  │ OBSERVABLE       │  │  MODALITY        │  │  COORDINATION         │   │
│  │   SUBSTRATE      │  │    PIPELINE      │  │    PROTOCOL           │   │
│  │                  │  │                  │  │                       │   │
│  │ Event bus broker │  │ Bus + Router     │  │ claim, release,       │   │
│  │   + SSE stream   │  │ Text module  ✓   │  │ checkpoint, handoff,  │   │
│  │ Hash-chained     │  │ Voice module     │  │ broadcast             │   │
│  │   ledger         │  │   → Mod³ (D2)    │  │                       │   │
│  │ Trace query      │  │ Salience ranking │  │ File-based storage    │   │
│  │ Config API       │  │ Wire protocol    │  │ 12 CLI subcommands    │   │
│  │   (RFC 7396)     │  │                  │  │                       │   │
│  │ Kernel slog sink │  │                  │  │                       │   │
│  │ Agent-state API  │  │                  │  │                       │   │
│  │ Tool-call trace  │  │                  │  │                       │   │
│  └────────┬─────────┘  └────────┬─────────┘  └───────────────────────┘   │
│           │ MCP + HTTP           │                                         │
│           ▼                      ▼                                         │
│    20+ MCP tools            ┌──────────────────────┐                     │
│    /v1/bus · /v1/ledger     │  BEP SYNC ENGINE      │                     │
│    /v1/config · /v1/traces  │  TLS + ECDSA · wire   │                     │
│    /v1/agents · /v1/*       │  Phase 1 · fsnotify   │                     │
│                             └──────────────────────┘                      │
└──────────────────────────────────────────────────────────────────────────┘
            │                     │
            ▼                     ▼
┌───────────────────┐  ┌──────────────────────────────────────────────────┐
│  MOD³              │  │  CONSTELLATION [identity & trust]                │
│  [Python MCP]      │  │                                                  │
│                    │  │  ECDSA P-256 identity    EMA trust scoring       │
│  Multi-model TTS   │  │  Hash-chained ledger     3-layer coherence      │
│  Kokoro · Voxtral  │  │  Heartbeat protocol      Behavioral trust       │
│  Chatterbox · Spark│  │                                                  │
│  Voice STT         │  │  ┌──────────────────────────────────────────┐   │
│  Barge-in detect   │  │  │  SOVEREIGNTY BOUNDARY                    │   │
│  Dashboard UI      │  │  │  .cogpublic manifest gates what          │   │
│                    │  │  │  crosses constellation → public          │   │
│  Go defines the    │  │  │                                          │   │
│  interface;        │  │  │  Public ← Org ← Constellation ← Node    │   │
│  Python implements │  │  └──────────────────────────────────────────┘   │
└────────────────────┘  └──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  MULTI-SESSION SUBSTRATE                                                 │
│  cog-sandbox-mcp bridge :7823  ·  12 cogos_* native tools                │
│  HTTP streamable transport · session registry · handoff protocol         │
│  Siblings in docker compose: bridge-{primary,secondary},                 │
│                               tailscale-{primary,secondary}              │
└──────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────┐
│  EXTENSIONS                                                              │
│  research  — LoRO training pipeline, Mamba SSM re-ranker, eval framework│
│  skills    — SKILL.md definitions for Claude Code agents                │
│  charts    — Helm charts for Kubernetes deployment (experimental)       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## What's new (2026-04-21)

**MCP surface 8 → 20+ tools.** Seven of eight critical MCP gaps closed. New tools cover the ledger (`cog_read_ledger`), trace search (`cog_search_traces`), event bus (`cog_tail_events`, `cog_query_events`), conversations (`cog_read_conversation`), tool calls (`cog_read_tool_calls`, `cog_tail_tool_calls`), agent state (`cog_list_agents`, `cog_get_agent_state`, `cog_trigger_agent_loop`), kernel slog (`cog_tail_kernel_log`), and live config (`cog_read_config`, `cog_write_config`, `cog_rollback_config`).

**Event bus broker is live.** The broker is hooked inside `AppendEvent`; real SSE stream at `/v1/bus/:id/events/stream`. `cog bus send` added for write symmetry.

**Ledger exposure.** Hash-chained events queryable via MCP and HTTP `/v1/ledger`.

**Config mutation API.** RFC 7396 merge-patch over HTTP `/v1/config`, with rollback.

**Trace search.** Unified JSONL trace query via `cog_search_traces` and `/v1/traces`; the existing `/v1/proprioceptive` endpoint is preserved byte-compatible.

**Conversation persistence.** `turn.completed` ledger events plus a sidecar; readable via `cog_read_conversation` and `/v1/conversation`.

**Kernel slog capture.** JSONL sink at `.cog/run/kernel.log.jsonl`, with `cog_tail_kernel_log` + `/v1/kernel-log`.

**Ops.** `--bind` flag and `bind_addr` YAML (defaulting 127.0.0.1). Windows Makefile targets, install docs, and linux/arm64 cross-compilation unblocked.

**Multi-session MCP substrate.** cog-sandbox-mcp v0.4.1 ships HTTP streamable transport, session + handoff bridge tools (12 tools total), and a HANDOFF_PROTOCOL spec. Kernel on :6931, bridge on :7823, twelve `cogos_*` native tools. Docker compose now has `bridge-{primary,secondary}` and `tailscale-{primary,secondary}` siblings.

---

## Repositories

### Active

| Repo | Description | Language | Version |
|------|-------------|----------|---------|
| [**cogos**](https://github.com/cogos-dev/cogos) | Cognitive daemon for AI agents. Foveated context, learned retrieval (TRM), persistent memory, multi-provider routing. 7 importable library packages (`pkg/`), native agent harness, event bus + hash-chained ledger, config mutation API, trace query, kernel slog capture, 20+ MCP tools, Anthropic Messages proxy. | Go | v0.1.0 |
| [**cog-sandbox-mcp**](https://github.com/cogos-dev/cog-sandbox-mcp) | Multi-session MCP bridge. HTTP streamable transport, session registry, handoff protocol, 12 native `cogos_*` tools. The substrate that lets multiple sessions share one kernel. | Go | v0.4.1 |
| [**mod3**](https://github.com/cogos-dev/mod3) | Model Modality Modulator. Multi-model TTS (Kokoro, Voxtral, Chatterbox, Spark) with queue-aware output, barge-in detection, and turn-taking. Bidirectional voice pipeline, MCP shim, browser dashboard. | Python | v0.1.0 |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed identity protocol. O(1) verification, hash-chained coherence, stolen keys insufficient for impersonation. | Go | v0.1.0 |
| [**research**](https://github.com/cogos-dev/research) | Research foundations. EA/EFM thesis, LoRO framework (unified Low-Rank Observer abstraction), Mamba SSM training pipeline, evaluation methodology. | Python | — |
| [**skills**](https://github.com/cogos-dev/skills) | Portable skill definitions (SKILL.md files) for Claude Code and compatible agents. | Markdown | — |
| [**charts**](https://github.com/cogos-dev/charts) | Helm charts for deploying CogOS nodes to Kubernetes. | Helm | — |

### Archived

| Repo | Reason |
|------|--------|
| desktop | Functionality absorbed by kernel dashboard |
| openclaw-plugin | Foveated context engine for OpenClaw (archived) |

---

## What the kernel does

**Foveated context assembly.** On every prompt, a `UserPromptSubmit` hook fires and the context engine scores workspace documents via a trained Mamba SSM (Tiny Recursive Model, 0.878 mean NDCG). Injects a zone-ordered context window optimized for KV cache reuse.

**Observable substrate.** Every meaningful event (turns, tool calls, config changes, agent state transitions) flows through a hash-chained ledger, a live event-bus broker with SSE streaming, and a JSONL trace sink. All of it is queryable from outside — ledger, traces, events, conversation, kernel slog, tool calls — via MCP and HTTP `/v1/`.

**Mutable config and agent-state control.** Live configuration over `/v1/config` with RFC 7396 merge-patch and rollback. List agents, read per-agent state, and trigger loops via `/v1/agents` and matching MCP tools.

**Library extraction.** 7 importable Go packages under `pkg/` — context assembly, inference routing, coordination, modality, reconciliation, and more. The kernel is a daemon *and* a library.

**Native agent harness.** Built-in agent runner with local model support (Gemma E4B via Ollama). Agents execute within the kernel process with full access to memory and context primitives.

**Multi-provider inference routing.** OpenAI-compatible and Anthropic Messages-compatible HTTP APIs. Routes to the best available provider based on a sovereignty gradient: local inference preferred, cloud escalation when needed. Works with Ollama, LM Studio, Claude, GPT, Codex, and any OpenAI-compatible endpoint — including remote nodes on the LAN.

**MCP protocol support.** 20+ tool MCP Streamable HTTP server and Anthropic Messages API proxy. The kernel speaks the protocols that AI tools already use.

**Reconciliation framework.** 8 providers (agent, component, discord, mcp-tools, service, openclaw-gateway, openclaw-agents, openclaw-cron) with a Terraform-style plan/apply/snapshot lifecycle. Topological ordering via Kahn's algorithm. Atomic state writes with lineage tracking and serial numbers.

**Bidirectional voice pipeline.** Modality bus with text and voice channels. The Go kernel defines the interface; Mod3 (Python) implements it. Queue-aware TTS output, barge-in detection, and turn-taking via wire protocol.

**BEP sync engine.** TLS transport with mutual ECDSA authentication, wire protocol with protowire encoding. Phase 1 implementation (fsnotify-based file watching). Designed for multi-node workspace synchronization.

**Coordination protocol.** 5 primitives (claim, release, checkpoint, handoff, broadcast) with file-based storage and 12 CLI subcommands. Enables multi-agent workspace sharing without central coordination.

**Multi-session substrate.** Kernel on `:6931`, `cog-sandbox-mcp` bridge on `:7823`, 12 native `cogos_*` tools, and a handoff protocol that lets sessions pass state through the ledger. Docker compose siblings provide primary/secondary bridge and tailscale pairs.

**Constellation identity.** ECDSA P-256 cryptographic identity with behavioral trust scoring. Identity is not a static credential — it's a dynamical property that emerges from consistent self-validating behavior over time. O(1) mutual verification per peer via signed heartbeats.

---

## The interface boundary

CogOS is a polyglot system. The kernel is Go. The voice server is Python. Future modality servers can be in any language. The contract between them is a wire protocol — not shared code.

The Go kernel defines typed interfaces (what blocks a module accepts and emits). The Python SDK provides the corresponding implementation. Together they form a matched pair that documents the boundary by existing on both sides of it.

```
Go (cogos/pkg/modality/)     ←  wire protocol  →     Python (mod3/)
Defines the interface                                 Implements it
```

---

## Quick start

```sh
# Build the kernel
git clone https://github.com/cogos-dev/cogos.git
cd cogos
make build

# Start the daemon (localhost by default)
./cogos serve --workspace ~/my-project
# http://localhost:6931/health

# Bind to a LAN address
./cogos serve --workspace ~/my-project --bind 0.0.0.0:6931
```

Requirements: Go 1.25+, macOS, Linux, or Windows (linux/arm64 cross-compilation supported).

For voice, add [Mod3](https://github.com/cogos-dev/mod3) as an MCP server in your `.mcp.json`. For multi-session work, run [cog-sandbox-mcp](https://github.com/cogos-dev/cog-sandbox-mcp) on `:7823`.

---

## CI

Shared reusable workflows in [`.github`](https://github.com/cogos-dev/.github):

| Workflow | What it does |
|----------|-------------|
| `go-ci.yml` | `go vet`, build, `go test -race`, golangci-lint |
| `py-ci.yml` | ruff lint + format, pytest |
| `pr-checks.yml` | CHANGELOG reminder, required files validation |

---

## Contributing

All repositories are MIT licensed. Issues and PRs welcome.
