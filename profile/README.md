# cogos-dev

Infrastructure for AI agents that need persistent memory, intelligent context, and cross-device coordination.

CogOS is a local-first cognitive daemon written in Go with a Python voice channel. It gives AI tools like Claude Code and Ollama-backed agents the things they can't give themselves: workspace memory that persists across sessions, a trained retrieval model that surfaces the right context automatically, multi-provider inference routing with sovereignty-aware scheduling, a bidirectional voice pipeline, and now an observable, mutable substrate — hash-chained ledger, live event bus, kernel slog capture, and agent-state control — all exposed through MCP and HTTP. Everything runs on your hardware. Everything stays yours.

The kernel ships 7 importable library packages (`pkg/`), a native agent harness with local model support (Gemma E4B), a 30-tool MCP surface, and protocol adapters for MCP Streamable HTTP and the Anthropic Messages API. Modality servers implement typed interfaces in any language — the wire protocol is the contract.

## Architecture

```
                 AI AGENTS  ·  TOOLS  ·  CLIENTS
       Claude Code  ·  Cursor  ·  Ollama  ·  MCP clients
                              │
                    prompts · MCP · HTTP
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │         COGOS KERNEL   —   Go daemon on :6931            │
   │                                                          │
   │   Context          Observability       State             │
   │   ───────          ─────────────       ─────             │
   │   foveated         hash-chained        config RFC 7396   │
   │   TRM-scored       ledger + bus        agent harness     │
   │   KV-cache aware   kernel slog         reconciliation    │
   │                                                          │
   │   Sessions         Inference           Library           │
   │   ────────         ─────────           ───────           │
   │   atomic claim     OpenAI-compat       7 pkg/ modules    │
   │   first-wins       Anthropic Msgs      ~10K LOC          │
   │   bus = truth      sovereign routing   190 tests         │
   │                                                          │
   │      30 MCP tools  ·  /v1/* HTTP  ·  BEP sync engine     │
   └────────┬───────────────────┬───────────────────┬─────────┘
            │                   │                   │
            ▼                   ▼                   ▼
   ┌───────────────┐  ┌──────────────────┐  ┌────────────────┐
   │     MOD3      │  │ COG-SANDBOX-MCP  │  │  CONSTELLATION │
   │    Python     │  │      Python      │  │       Go       │
   ├───────────────┤  ├──────────────────┤  ├────────────────┤
   │  Voice I/O    │  │ Multi-session    │  │ ECDSA P-256    │
   │  TTS + STT    │  │   bridge :7823   │  │ EMA trust      │
   │  Kokoro       │  │ Handoff protocol │  │ Hash-chained   │
   │  Voxtral      │  │ Session registry │  │   coherence    │
   │  Chatterbox   │  │ 12 cogos_* tools │  │ O(1) verify    │
   │  Barge-in     │  │ HTTP streamable  │  │ Behavioral     │
   └───────────────┘  └──────────────────┘  └────────────────┘

   providers:  Ollama · LM Studio · Claude · GPT · Codex · LAN peers
   ecosystem:  research  ·  skills  ·  charts
```

---

## What's new (2026-04-22)

**v0.3.0 released.** Pre-1.0 reset tag. Consolidates Track 5 dead-code removal (-28,922 LoC net from the kernel root) and the full 8-gap Agent F critical MCP surface buildout. First tag in the reset series; the prior aspirational `v2.x` numbering was never tagged. See the [cogos CHANGELOG](https://github.com/cogos-dev/cogos/blob/main/CHANGELOG.md).

**All 8 critical MCP gaps closed.** The final gap — session management — landed as a hybrid: kernel owns session/handoff invariants with bus-level first-wins atomic claim (not just in-memory); bridge keeps the Python `cogos_*` ergonomics layer. Two MCP surfaces coexist by design: 8 `cogos_*` via the Python bridge, 8 new `cog_*_session|handoff_*` via the engine. Bus stays ground truth; registries are derived views rebuilt from seq-sorted replay on startup. `handoff.claim_rejected` observability event emits on every rejection.

**MCP surface 8 → 30 tools.** In addition to sessions/handoffs: ledger (`cog_read_ledger`), trace search (`cog_search_traces`), event bus (`cog_tail_events`, `cog_query_events`), conversations (`cog_read_conversation`), tool calls (`cog_read_tool_calls`, `cog_tail_tool_calls`), agent state (`cog_list_agents`, `cog_get_agent_state`, `cog_trigger_agent_loop`), kernel slog (`cog_tail_kernel_log`), and live config (`cog_read_config`, `cog_write_config`, `cog_rollback_config`).

**Track 5 refactor shipped.** Root package went from 185 non-test files / 80K LoC to 132 files / 51K LoC. Installed binary is now engine-built (`go build ./cmd/cogos/`); `/v1/bus/*` and `/v1/sessions` routes ported to the engine byte-compat; 45 dormant root files swept.

**Event bus broker is live.** Hooked inside `AppendEvent`; real SSE stream at `/v1/bus/:id/events/stream`. `cog bus send` for write symmetry.

**Ledger exposure.** Hash-chained events queryable via MCP and HTTP `/v1/ledger` (with optional `?verify_chain=true`).

**Config mutation API.** RFC 7396 merge-patch over `/v1/config`, atomic writes + rotating backups + rollback.

**Trace search and kernel slog.** Three non-overlapping observability lanes now have MCP + HTTP + on-disk artifacts: ledger (durable hash-chained), traces (client metabolites), kernel slog (runtime logs via `teeHandler`, JSONL sink at `.cog/run/kernel.log.jsonl`).

**Ops.** `--bind` flag + `bind_addr` YAML (loopback default; CORS relaxed on non-loopback bind). Windows Makefile targets, install docs, and linux/arm64 cross-compilation unblocked.

**Multi-session substrate.** `cog-sandbox-mcp` v0.4.1 ships HTTP streamable transport, session + handoff bridge tools, and the HANDOFF_PROTOCOL spec. Docker compose now has `bridge-{primary,secondary}` and `tailscale-{primary,secondary}` siblings.

---

## Repositories

### Active

| Repo | Description | Language | Version |
|------|-------------|----------|---------|
| [**cogos**](https://github.com/cogos-dev/cogos) | Cognitive daemon for AI agents. Foveated context, learned retrieval (TRM), persistent memory, multi-provider routing. 7 importable library packages (`pkg/`), native agent harness, event bus + hash-chained ledger, config mutation API, trace query, kernel slog capture, kernel-native session management with atomic handoff claim, 30 MCP tools, Anthropic Messages proxy. | Go | v0.3.0 |
| [**cog-sandbox-mcp**](https://github.com/cogos-dev/cog-sandbox-mcp) | Multi-session MCP bridge. HTTP streamable transport, session registry, handoff protocol, 12 `cogos_*` tools layered over the kernel's canonical session/handoff routes. The substrate that lets multiple sessions share one kernel. | Python | v0.4.1 |
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

**MCP protocol support.** 30-tool MCP Streamable HTTP server and Anthropic Messages API proxy. The kernel speaks the protocols that AI tools already use.

**Reconciliation framework.** 8 providers (agent, component, discord, mcp-tools, service, openclaw-gateway, openclaw-agents, openclaw-cron) with a Terraform-style plan/apply/snapshot lifecycle. Topological ordering via Kahn's algorithm. Atomic state writes with lineage tracking and serial numbers.

**Bidirectional voice pipeline.** Modality bus with text and voice channels. The Go kernel defines the interface; Mod3 (Python) implements it. Queue-aware TTS output, barge-in detection, and turn-taking via wire protocol.

**BEP sync engine.** TLS transport with mutual ECDSA authentication, wire protocol with protowire encoding. Phase 1 implementation (fsnotify-based file watching). Designed for multi-node workspace synchronization.

**Coordination protocol.** 5 primitives (claim, release, checkpoint, handoff, broadcast) with file-based storage and 12 CLI subcommands. Enables multi-agent workspace sharing without central coordination.

**Multi-session substrate.** Kernel on `:6931` owns canonical session/handoff state via `SessionRegistry` + `HandoffRegistry` (derived views over `bus_sessions` / `bus_handoffs`; atomic first-wins claim at the bus boundary, not just in-memory). Python bridge on `:7823` exposes 12 `cogos_*` ergonomic shims layered over the kernel routes. Docker compose siblings provide primary/secondary bridge and tailscale pairs.

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
