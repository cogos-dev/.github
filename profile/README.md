# cogos-dev

Infrastructure for AI agents that need persistent memory, intelligent context, and cross-device coordination.

CogOS is a local-first cognitive daemon written in Go with a Python voice channel. It gives AI tools like Claude Code and Ollama-backed agents the things they can't give themselves: workspace memory that persists across sessions, a trained retrieval model that surfaces the right context automatically, multi-provider inference routing with sovereignty-aware scheduling, and a bidirectional voice pipeline. Everything runs on your hardware. Everything stays yours.

The kernel ships 7 importable library packages (`pkg/`), a native agent harness with local model support (Gemma E4B), and protocol adapters for MCP Streamable HTTP and the Anthropic Messages API. Modality servers implement typed interfaces in any language — the wire protocol is the contract.

## Architecture

```
                        ┌─────────────────────────────┐
                        │      AI AGENTS / TOOLS       │
                        │  Claude Code · Ollama agents  │
                        └──────────┬──────────────────┘
                                   │ prompts
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
│  │ MODALITY         │  │  BEP SYNC        │  │  COORDINATION         │   │
│  │  PIPELINE        │  │   ENGINE         │  │   PROTOCOL            │   │
│  │                  │  │                  │  │                       │   │
│  │ Bus + Router     │  │ TLS transport    │  │ claim, release,       │   │
│  │ Text module  ✓   │  │ ECDSA mutual     │  │ checkpoint, handoff,  │   │
│  │ Voice module     │  │   auth           │  │ broadcast             │   │
│  │   → Mod³ (D2)    │  │ Protowire        │  │                       │   │
│  │ Salience ranking │  │   encoding       │  │ File-based storage    │   │
│  │ Wire protocol    │  │ Phase 1          │  │ 12 CLI subcommands    │   │
│  └────────┬─────────┘  └────────┬─────────┘  └───────────────────────┘   │
└───────────┼─────────────────────┼────────────────────────────────────────┘
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
│  EXTENSIONS                                                              │
│  research  — LoRO training pipeline, Mamba SSM re-ranker, eval framework│
│  skills    — SKILL.md definitions for Claude Code agents                │
│  charts    — Helm charts for Kubernetes deployment (experimental)       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Repositories

### Active

| Repo | Description | Language | Version |
|------|-------------|----------|---------|
| [**cogos**](https://github.com/cogos-dev/cogos) | Cognitive daemon for AI agents. Foveated context assembly, learned retrieval (TRM), persistent memory, multi-provider routing. 7 importable library packages (`pkg/`), native agent harness, MCP Streamable HTTP, Anthropic Messages API proxy. | Go | v0.1.0 |
| [**mod3**](https://github.com/cogos-dev/mod3) | Model Modality Modulator. Multi-model TTS (Kokoro, Voxtral, Chatterbox, Spark) with queue-aware output, barge-in detection, and turn-taking. Bidirectional voice pipeline, MCP shim, browser dashboard. | Python | v0.1.0 |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed identity protocol. O(1) verification, hash-chained coherence, stolen keys insufficient for impersonation. | Go | v0.1.0 |
| [**research**](https://github.com/cogos-dev/research) | Research foundations. EA/EFM thesis, LoRO framework (unified Low-Rank Observer abstraction), Mamba SSM training pipeline, evaluation methodology. | Python | — |
| [**skills**](https://github.com/cogos-dev/skills) | Portable skill definitions (SKILL.md files) for Claude Code and compatible agents | Markdown | — |
| [**charts**](https://github.com/cogos-dev/charts) | Helm charts for deploying CogOS nodes to Kubernetes | Helm | — |

### Archived

| Repo | Reason |
|------|--------|
| desktop | Functionality absorbed by kernel dashboard |
| openclaw-plugin | Foveated context engine for OpenClaw (archived) |

---

## What the kernel does

**Foveated context assembly.** On every prompt, a `UserPromptSubmit` hook fires and the context engine scores workspace documents via a trained Mamba SSM (Tiny Recursive Model, 0.878 mean NDCG). Injects a zone-ordered context window optimized for KV cache reuse.

**Library extraction.** 7 importable Go packages under `pkg/` — context assembly, inference routing, coordination, modality, reconciliation, and more. The kernel is a daemon *and* a library.

**Native agent harness.** Built-in agent runner with local model support (Gemma E4B via Ollama). Agents execute within the kernel process with full access to memory and context primitives.

**Multi-provider inference routing.** OpenAI-compatible and Anthropic Messages-compatible HTTP APIs. Routes to the best available provider based on a sovereignty gradient: local inference preferred, cloud escalation when needed. Works with Ollama, LM Studio, Claude, GPT, Codex, and any OpenAI-compatible endpoint — including remote nodes on the LAN.

**MCP protocol support.** MCP Streamable HTTP server and Anthropic Messages API proxy. The kernel speaks the protocols that AI tools already use.

**Reconciliation framework.** 8 providers (agent, component, discord, mcp-tools, service, openclaw-gateway, openclaw-agents, openclaw-cron) with a Terraform-style plan/apply/snapshot lifecycle. Topological ordering via Kahn's algorithm. Atomic state writes with lineage tracking and serial numbers.

**Bidirectional voice pipeline.** Modality bus with text and voice channels. The Go kernel defines the interface; Mod3 (Python) implements it. Queue-aware TTS output, barge-in detection, and turn-taking via wire protocol.

**BEP sync engine.** TLS transport with mutual ECDSA authentication, wire protocol with protowire encoding. Phase 1 implementation (fsnotify-based file watching). Designed for multi-node workspace synchronization.

**Coordination protocol.** 5 primitives (claim, release, checkpoint, handoff, broadcast) with file-based storage and 12 CLI subcommands. Enables multi-agent workspace sharing without central coordination.

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

# Start the daemon
./cogos serve --workspace ~/my-project
# http://localhost:6931/health
```

Requirements: Go 1.25+, macOS or Linux.

For voice, add [Mod3](https://github.com/cogos-dev/mod3) as an MCP server in your `.mcp.json`.

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
