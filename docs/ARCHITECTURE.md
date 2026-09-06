# Architecture

## Design principles

1. **Raw first.** Capture complete trajectories before converting them to any training-specific format.
2. **Verifier-first selection.** Teacher/model identity is metadata, never the acceptance criterion.
3. **Reproducibility.** Every run should identify the task, repository, base commit, environment image, model, agent scaffold, prompt/configuration, and evaluator version.
4. **Training/eval isolation.** Held-out evaluation tasks must never enter accepted training data.
5. **Outcome before process.** Executable correctness dominates stylistic or efficiency preferences.
6. **Model-agnostic interfaces.** Runners should normalize different agents into a common event stream.

## Components

### Tasks
A task defines the initial repository state, user instruction, environment, validation commands, and split (`train`, `dev`, or `held-out`). Dataset adapters may import SWE-style tasks while custom task packs can focus on Rust, Linux, shell, networking, containers, and observability.

### Environments
Each attempt should start from a clean, reproducible environment. Container images are preferred where practical. A run must not inherit mutable state from an earlier attempt.

### Runners
A runner launches an agent/model and emits normalized events such as assistant messages, tool calls, tool results, patches, and final responses. Native agent transcripts should also be retained when possible.

### Trajectory store
The canonical trajectory is richer than an SFT message list. It includes provenance, execution events, resource usage, final diff, validation outputs, and evaluator results.

### Evaluators
Evaluators operate in layers:

- deterministic outcome gates: build, test, target behavior
- regression gates
- diff/scope checks
- process metrics: repeated reads, command count, failed hypotheses, recovery behavior
- optional learned verifier scores later

### Acceptance
Acceptance policies consume evaluator outputs and produce a decision plus reason. Policies should be versioned so historical datasets remain reproducible.

### Exporters
Exporters transform accepted raw trajectories into downstream formats without mutating the originals. Planned targets include Qwen-style SFT chat data, preference pairs, and verifier datasets.

## Data flow

```text
Task Manifest
    |
    v
Environment Builder ---> clean workspace
    |
    v
Runner / Agent ---> normalized event stream ---> raw trajectory
                                             |
                                             v
                                         evaluators
                                             |
                                             v
                                      score + verdict
                                             |
                                  +----------+----------+
                                  |                     |
                               reject                accept
                                                        |
                                                        v
                                                    exporters
```

## Reproducibility identifiers

A run should eventually be reproducible from at least:

- `task_id`
- repository URL and immutable base commit
- environment/container digest
- runner and agent version
- model identifier/checkpoint
- system/developer/user prompt hashes
- sampling/inference configuration
- evaluator version
- acceptance-policy version

## Non-goals for the initial scaffold

- training framework implementation
- learned reward-model training
- large-scale orchestration
- online reinforcement learning
- storing giant generated corpora directly in Git

The first useful vertical slice is deliberately small: one task format, one runner, one reproducible environment, deterministic evaluation, and a lossless trajectory artifact.
