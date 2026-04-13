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
│   skills · openclaw-plugin · research                │
├─────────────────────────────────────────────────────┤
│                   Deployment                         │
│   desktop (macOS) · charts (Helm/k8s)                │
└─────────────────────────────────────────────────────┘
```

### Quick Start

- **Run locally:** Install [cogos](https://github.com/cogos-dev/cogos), add [Mod³](https://github.com/cogos-dev/mod3) for voice
- **Deploy to k8s:** Use [charts](https://github.com/cogos-dev/charts) with Helm
- **Build plugins:** Browse [skills](https://github.com/cogos-dev/skills) or create an [OpenClaw plugin](https://github.com/cogos-dev/openclaw-plugin)

---

## Repositories

| Repo | Description | Language |
|------|-------------|----------|
| [**cogos**](https://github.com/cogos-dev/cogos) | The kernel -- Go daemon with foveated context assembly, trained retrieval (TRM), hash-chained ledger, multi-provider routing | Go |
| [**mod3**](https://github.com/cogos-dev/mod3) | Model Modality Modulator -- gives agents a voice. Multi-model TTS, queue-aware output, barge-in detection. MCP server for Claude Code | Python |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed identity protocol -- trust earned through temporal consistency, not granted by authority. Hash-chained event ledgers, O(1) mutual verification | Go |
| [**research**](https://github.com/cogos-dev/research) | Research foundations -- EA/EFM thesis, LoRO (Low-Rank Observer) framework | Docs |
| [**openclaw-plugin**](https://github.com/cogos-dev/openclaw-plugin) | OpenClaw digestion adapter for Claude Code | Go |
| [**skills**](https://github.com/cogos-dev/skills) | Plugin marketplace -- skills, agents, and modality integrations for Claude Code | YAML/MD |
| [**desktop**](https://github.com/cogos-dev/desktop) | Native macOS app -- Wails + React, kernel management, integrated terminal. No Electron | Go/TS |
| [**charts**](https://github.com/cogos-dev/charts) | Helm charts for deploying CogOS nodes to Kubernetes | Helm |

---

## What makes this different

**Learned retrieval, not keyword search.** The kernel includes a 2.3M-parameter Mamba SSM (Tiny Recursive Model) trained over 631 experiments from 0.424 to 0.900 NDCG. It scores workspace documents by temporal salience, edit recency, and semantic relevance, then assembles a focused context window -- before the model sees anything. Runs inference locally in ~6KB of state.

**Foveated context assembly.** On every prompt, a `UserPromptSubmit` hook fires, the context engine scores all workspace documents, and injects a zone-ordered context window optimized for KV cache reuse. The model gets a pre-focused window instead of everything-or-nothing.

**Multi-provider routing.** OpenAI-compatible and Anthropic Messages-compatible HTTP API. Works with Ollama, LM Studio, Claude, GPT, and any OpenAI-compatible endpoint. Local models preferred by default.

**Content-addressed storage.** Every routing decision, context assembly, and state transition is recorded in an append-only, hash-chained ledger (SHA-256, RFC 8785 canonical JSON). Full audit trail, tamper-evident.

**Real-time voice.** Mod³ runs four TTS engines on Apple Silicon (Kokoro 82M, Voxtral 4B, Chatterbox ~1B, Spark 0.5B) with adaptive buffering, sentence-boundary chunking, and barge-in detection. Non-blocking -- the agent keeps working while it speaks.

**Distributed mesh.** Constellation uses BEP-based sync and content-addressed blocks for cross-device coordination. Identity is a dynamical property -- coherence with history -- not a static credential.

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

For voice, add [Mod³](https://github.com/cogos-dev/mod3) as an MCP server (`./setup.sh` and add to `.mcp.json`).

For deployment, see [charts](https://github.com/cogos-dev/charts) for Helm and Docker Compose options.

---

## Research

The design is grounded in two ideas documented in the [research](https://github.com/cogos-dev/research) repo:

- **Externalized Attention (EA)** -- deciding what information is relevant *before* the model sees it. Implemented as the foveated context engine and TRM scorer.
- **Executive Function Modulation (EFM)** -- shaping model behavior through provider routing, tool-call validation, and a process state machine, rather than prompt engineering alone.

The thesis: agent quality is a function of boundary quality. Better context in, better output out. The kernel handles the cognitive overhead so the model can focus on generation.

---

## CI

All repositories share reusable workflows defined in [`.github`](https://github.com/cogos-dev/.github):

| Workflow | What it does |
|----------|-------------|
| `go-ci.yml` | `go vet`, build, `go test -race`, golangci-lint, optional e2e |
| `py-ci.yml` | ruff lint + format, optional pyright, pytest, optional smoke test |
| `pr-checks.yml` | CHANGELOG reminder, required files validation |

Repos call these via `workflow_call` -- see any repo's `.github/workflows/ci.yml` for an example. Dependabot is configured org-wide for Go modules, pip, and GitHub Actions.

---

## Contributing

All repositories are MIT licensed. Issues and PRs are welcome.

If you're interested in the retrieval model, context engine, or voice channel, start with the individual repo READMEs -- each has build instructions and architecture docs.
