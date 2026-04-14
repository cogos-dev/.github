# cogos-dev

Infrastructure for AI agents that need persistent memory, intelligent context, and cross-device coordination.

CogOS is a local-first cognitive daemon written in Go. It gives AI tools like Claude Code, Cursor, and Ollama-backed agents the things they can't give themselves: workspace memory that persists across sessions, a trained retrieval model that surfaces the right context automatically, multi-provider inference routing, and a real-time voice channel. Everything runs on your hardware. Everything stays yours.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    CogOS Ecosystem                   │
├──────────────┬──────────────┬───────────────────────┤
│   Kernel     │  Modalities  │    Identity & Trust    │
│   cogos      │  mod3        │    constellation       │
│              │              │                        │
│  Inference   │  Voice TTS   │  Behavioral trust      │
│  Memory      │  Voice STT   │  Hash-chained          │
│  Context     │  Barge-in    │  coherence proofs       │
│  Reconcile   │  Dashboard   │                        │
├──────────────┴──────────────┴───────────────────────┤
│                   Extensions                         │
│   skills · charts (experimental)                     │
└─────────────────────────────────────────────────────┘
```

---

## Repositories

### Active

| Repo | Description | Language | Status |
|------|-------------|----------|--------|
| [**cogos**](https://github.com/cogos-dev/cogos) | The kernel — Go daemon with reconciliation framework, foveated context assembly, trained retrieval (TRM), multi-provider inference routing, modality pipeline, BEP sync engine | Go | v0.1.0 |
| [**mod3**](https://github.com/cogos-dev/mod3) | Model Modality Modulator — multi-model TTS with queue-aware output and barge-in detection. MCP server for Claude Code | Python | v0.1.0 |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed identity POC — ECDSA P-256 identity, EMA trust scoring, hash-chained event ledgers | Go | v0.1.0 |
| [**skills**](https://github.com/cogos-dev/skills) | Portable skill definitions (SKILL.md files) for Claude Code and compatible agents | Markdown | — |
| [**charts**](https://github.com/cogos-dev/charts) | Helm charts for Kubernetes deployment (experimental, not production-tested) | Helm | — |

### Archived

| Repo | Reason |
|------|--------|
| desktop | Early stage, not representative |
| openclaw-plugin | External dependency, not maintained |
| research | Architectural essays, no runnable code |

---

## What the kernel actually does

**Reconciliation framework.** 8 providers (agent, component, discord, mcp-tools, service, and 3 OpenClaw providers) with a Terraform-style plan/apply loop, topological ordering via Kahn's algorithm, and atomic state writes with lineage tracking.

**Foveated context assembly.** On every prompt, a `UserPromptSubmit` hook fires, the context engine scores all workspace documents via a 2.3M-parameter Mamba SSM (Tiny Recursive Model, trained over 631 experiments to 0.900 NDCG), and injects a zone-ordered context window optimized for KV cache reuse.

**Multi-provider routing.** OpenAI-compatible and Anthropic Messages-compatible HTTP API. Works with Ollama, LM Studio, Claude, GPT, and any OpenAI-compatible endpoint.

**Modality pipeline.** Bus, router, pipeline, and supervisor for text/voice channels with a full D2 wire protocol. Text module complete; voice module connects to Mod3.

**BEP sync engine.** TLS transport with mutual ECDSA authentication, wire protocol with protowire encoding. Phase 1 (fsnotify-based provider).

**Coordination protocol.** 5 primitives (claim, release, checkpoint, handoff, broadcast) with file-based storage and 12 CLI subcommands.

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

Requirements: Go 1.24+, macOS or Linux.

For voice, add [Mod3](https://github.com/cogos-dev/mod3) as an MCP server.

---

## CI

All repositories share reusable workflows defined in [`.github`](https://github.com/cogos-dev/.github):

| Workflow | What it does |
|----------|-------------|
| `go-ci.yml` | `go vet`, build, `go test -race`, golangci-lint |
| `py-ci.yml` | ruff lint + format, pytest |
| `pr-checks.yml` | CHANGELOG reminder, required files validation |

---

## Contributing

All repositories are MIT licensed. Issues and PRs are welcome.
