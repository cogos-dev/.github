# cogos-dev

**Externalized attention and executive function modulation for intelligent systems.**

CogOS is the substrate between observers and generators. It provides the cognitive infrastructure that AI models can't give themselves: persistent memory, foveated context assembly, multi-provider routing, distributed identity, and a voice — running on your hardware, working with any model, keeping everything yours.

The model doesn't think. The substrate thinks. The model generates.

## Architecture

CogOS externalizes two functions that models do poorly and expensively:

**Externalized Attention** — deciding what information is relevant *before* the model sees it. Not retrieval. Not augmentation. Attention: selective amplification of what matters, suppression of what doesn't.

**Executive Function Modulation** — deciding how the model should behave *before* it generates. Not prompting. Modulation: shaping the computational trajectory through the sovereignty gradient, tool-call validation, process state machine, and consolidation policy.

The result: a 4B parameter model on a phone produces quality comparable to much larger models in standard agent loops, because the cognitive overhead is externalized into the substrate.

## Repositories

| Repo | What it is |
|------|-----------|
| [**cogos**](https://github.com/cogos-dev/cogos) | The kernel — continuous process daemon with foveated context, Mamba TRM, hash-chained ledger, multi-provider routing |
| [**constellation**](https://github.com/cogos-dev/constellation) | Distributed trust — identity as temporal coherence, O(1) mutual verification, stolen keys insufficient |
| [**mod3**](https://github.com/cogos-dev/mod3) | Voice modality — multi-model TTS on Apple Silicon, adaptive buffering, dual-modal output |
| [**desktop**](https://github.com/cogos-dev/desktop) | Native macOS app — Wails + React, kernel management, integrated terminal |
| [**charts**](https://github.com/cogos-dev/charts) | Deployment — Helm charts + Docker Compose for single-node through multi-node topologies |

## The Bet

Every other approach says "make the model smarter." More parameters, more training data, more RLHF.

CogOS says: make the environment more structured so that *any* intelligence — human or machine — can operate more effectively within it.

The model is a dependent variable. The substrate is the independent variable.

## License

All repositories are MIT licensed.
