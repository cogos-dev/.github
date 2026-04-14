# cogos-dev

Infrastructure for AI agents that need persistent memory, intelligent context, and cross-device coordination.

CogOS is a local-first cognitive daemon written in Go with a Python voice channel. It gives AI tools like Claude Code and Ollama-backed agents the things they can't give themselves: workspace memory that persists across sessions, a trained retrieval model that surfaces the right context automatically, multi-provider inference routing with sovereignty-aware scheduling, and a real-time voice channel. Everything runs on your hardware. Everything stays yours.

The kernel defines typed interfaces in Go. Modality servers implement them in any language. The wire protocol between them is the contract — language doesn't matter, shape compatibility does.

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
│                     │  Desktop │  Codex    │                             │
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
│  skills — SKILL.md definitions for Claude Code agents                   │
│  charts — Helm charts for Kubernetes deployment (experimental)          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Repositories

### Active

| Repo | Description | Language | Version |
|------|-------------|----------|---------|
| [**cogos**](https://github.com/cogos-dev/cogos) | The kernel — Go daemon with reconciliation framework (8 providers), foveated context assembly, multi-provider inference routing, modality pipeline, BEP sync, coordination protocol. 150K LOC, 1,130 tests. | Go | v0.1.0 |
| [**mod3**](https://github.com/cogos-dev/mod3) | Model Modality Modulator — multi-model TTS (Kokoro, Voxtral, Chatterbox, Spark) with queue-aware output, barge-in detection, and browser dashboard. MCP server for Claude Code. | Python | v0.1.0 |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed identity POC — ECDSA P-256 identity, EMA trust scoring (0.7/0.4/0.2 thresholds), 3-layer coherence validation, hash-chained event ledgers. 89 tests. | Go | v0.1.0 |
| [**skills**](https://github.com/cogos-dev/skills) | Portable skill definitions (SKILL.md files) for Claude Code and compatible agents | Markdown | — |
| [**charts**](https://github.com/cogos-dev/charts) | Helm charts for Kubernetes deployment (experimental) | Helm | — |

### Archived

| Repo | Reason |
|------|--------|
| desktop | Early stage, not representative |
| openclaw-plugin | External dependency, not maintained |
| research | Content folded into kernel docs |

---

## What the kernel does

**Reconciliation framework.** 8 providers (agent, component, discord, mcp-tools, service, openclaw-gateway, openclaw-agents, openclaw-cron) with a Terraform-style plan/apply/snapshot lifecycle. Topological ordering via Kahn's algorithm. Atomic state writes with lineage tracking and serial numbers. Continuous reconciliation via 5-minute serve loop with watch mode and field-reactive conditions.

**Foveated context assembly.** On every prompt, a `UserPromptSubmit` hook fires and the context engine scores workspace documents via a trained Mamba SSM (Tiny Recursive Model, 0.878 mean NDCG). Injects a zone-ordered context window optimized for KV cache reuse.

**Multi-provider inference routing.** OpenAI-compatible and Anthropic Messages-compatible HTTP APIs. Routes to the best available provider based on a sovereignty gradient: local inference preferred, cloud escalation when needed. Works with Ollama, LM Studio, Claude, GPT, Codex, and any OpenAI-compatible endpoint — including remote nodes on the LAN.

**Modality pipeline.** Bus, router, pipeline, and supervisor for text/voice channels with a full D2 wire protocol. The Go kernel defines the modality interface; language-specific servers (like Mod3 in Python) implement it. Text module complete. Voice module connects to Mod3 via wire protocol.

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
