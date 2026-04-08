# cogos-dev

**Externalized attention and executive function modulation for intelligent systems.**

CogOS is the substrate between observers and generators. It provides the cognitive infrastructure that AI models can't give themselves: persistent memory, foveated context assembly, multi-provider routing, distributed identity, and a voice — running on your hardware, working with any model, keeping everything yours.

The model doesn't think. The substrate thinks. The model generates.

## Architecture

CogOS externalizes two functions that models do poorly and expensively:

**Externalized Attention (EA)** — deciding what information is relevant *before* the model sees it. Not retrieval. Not augmentation. Attention: selective amplification of what matters, suppression of what doesn't. Implemented as foveated context assembly — a scoring pipeline (TRM, a [Tiny Recursive Model](https://arxiv.org/abs/2510.04871) implemented as a Mamba SSM for temporal context scoring, + salience field) that selects and zone-orders documents for the model's context window.

**Executive Function Modulation (EFM)** — deciding how the model should behave *before* it generates. Not prompting. Modulation: shaping the computational trajectory through local-first provider routing, tool-call validation, a process state machine (Active/Receptive/Consolidating/Dormant), and consolidation policy.

The result: a 4B parameter model on a phone produces quality comparable to much larger models in standard agent loops, because the cognitive overhead is externalized into the substrate.

## Repositories

| Repo | What it is |
|------|-----------|
| [**cogos**](https://github.com/cogos-dev/cogos) | The kernel — continuous process daemon with foveated context, Mamba TRM, hash-chained ledger, multi-provider routing |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed trust — identity as temporal coherence, O(1) mutual verification, stolen keys insufficient |
| [**mod3**](https://github.com/cogos-dev/mod3) | Modality bus — translates between thinking and acting, voice-first with multi-model TTS on Apple Silicon |
| [**skills**](https://github.com/cogos-dev/skills) | Plugin marketplace — 17 Agent Skills across workflow, research, voice, and dev tools |
| [**research**](https://github.com/cogos-dev/research) | Theory — EA/EFM thesis, LoRO framework, cognitive architecture papers |
| [**desktop**](https://github.com/cogos-dev/desktop) | Native macOS app — Wails + React, kernel management, integrated terminal |
| [**charts**](https://github.com/cogos-dev/charts) | Deployment — Helm charts + Docker Compose for single-node through multi-node topologies |

## The Bet

Every other approach says "make the model smarter." More parameters, more training data, more RLHF.

CogOS says: make the environment more structured so that *any* intelligence — human or machine — can operate more effectively within it.

The model is a dependent variable. The substrate is the independent variable.

## License

All repositories are MIT licensed.
