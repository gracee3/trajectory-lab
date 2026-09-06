# Trajectory Lab

Trajectory Lab is an educational project studying the observable execution lifecycle of Codex CLI. Its next phase gives one agent reproducible, nontrivial software-engineering tasks, captures exposed execution activity, and independently evaluates the final artifact.

**Observe, don't infer.** An event log records exposed activity; it does not explain private reasoning or establish why an implementation worked. A product passing its acceptance checks and a capture being faithful are two separate findings.

## Start here

- [Vision](docs/VISION.md): purpose and boundaries.
- [Phase 2: Controlled SWE Experiments](docs/EXPERIMENT_PLAN.md): task contracts, initial tasks, broader domains and implementation sequence.
- [Evaluation](docs/EVALUATION.md): independent artifact checks and capture fidelity.
- [Research Report](docs/RESEARCH_REPORT.md): pinned source findings, capture surfaces and runtime gaps.
- [Research Plan](docs/RESEARCH_PLAN.md): historical research-only handoff.
- [Roadmap](docs/ROADMAP.md): status and completion criteria.

## Next phase

Start with a Rust streaming parser repair, a data-preserving database migration, and a process-supervision bug. Each task gets fixed starting code/data, a reproducible environment, a behavioral contract and protected independent acceptance checks.

The agent may inspect, implement, test and iterate autonomously within declared resources. Capture its observable trajectory, freeze its final artifact, then evaluate in a clean environment. Preserve failed, interrupted and incomplete attempts as well as successful ones. Different valid implementations and action sequences are expected.

Later task domains include HTTP services, concurrency, networking, observability, packaging, frontend and local security repairs. These are planned task designs, not implemented fixtures.

## Boundaries

- One Codex CLI agent per run; no multi-agent framework or additional agent adapters.
- No hidden reasoning access, chain-of-thought reconstruction, or private-state inference. Reasoning content is unnecessary for this phase.
- Task-outcome evaluation is in scope; model ranking, trajectory ranking, training-data production, fine-tuning and learned verifiers remain deferred.
- Observe existing interfaces without patching Codex internals.
- Replay displays recorded evidence; it never reruns commands or regenerates responses.

## Current status

Source/documentation research is published at a pinned upstream commit. Runtime experiments remain unperformed because the research environment lacked a CLI executable. Validate the target installed version before relying on its output.

This repository contains documentation and legacy draft schemas. There are no implemented task fixtures, recorder, independent evaluator or replay tool yet. The phase-2 update records the authorized next direction, not completed experiments.

| Path | Role |
|---|---|
| `docs/` | Vision, completed research, experiment plan, evaluation and roadmap |
| `tasks/` | Planned versioned SWE fixtures and task contracts |
| `runners/` | Planned autonomous Codex execution and capture |
| `evaluators/` | Planned independent artifact checks and capture-fidelity checks |
| `schemas/` | Legacy drafts; not finalized phase-2 contracts |

The initial capture recommendation is `codex exec --json`, with explicit limits on IDs, timing, tool coverage, output, cancellation and usage. App-server is a larger option if the selected experiment needs richer events. See the research report before choosing an adapter.
