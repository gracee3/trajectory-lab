# Vision

## Why this lab exists

Trajectory Lab is a focused educational study of one mature coding agent: Codex CLI. It should help a user understand the observable execution lifecycle through concrete evidence: session boundaries, messages, tool requests, tool results, and other activity that the examined version actually exposes.

Learning comes first. The source investigation establishes a baseline for what is observable. The next phase uses reproducible, nontrivial software-engineering tasks to observe autonomous work and independently evaluate its final artifacts.

## Observe, don't infer

An observed record is evidence of what a surface emitted. It is not evidence of hidden reasoning, a complete model input, or why the model chose an action.

Every finding should distinguish:

- Directly observed behavior, with a reproducible example.
- Behavior documented or located in source, with a versioned citation.
- Questions that remain unverified or unavailable through the inspected surface.

Do not fill gaps with reconstructed chain of thought, presumed messages, inferred causal links, or invented timestamps. Absence from a log does not prove absence from execution.

A source-code event is not automatically a user-accessible event. Research must trace whether and how it reaches an exposed surface.

## Scope

Focus exclusively on one Codex CLI agent per run. Build versioned SWE task fixtures with reproducible environments and objective acceptance criteria; allow autonomous implementation, capture exposed activity, freeze the resulting artifact, and evaluate it independently. Map verified events into a simple schema, export JSONL, and inspect runs as timelines.

Determinism belongs to starting conditions and acceptance criteria, not a prescribed patch or sequence of agent actions. Product correctness and observation fidelity are separate evaluations. Retain unsuccessful and incomplete attempts alongside successful ones.

Replay displays recorded activity. It does not execute tools again, regenerate responses, or reproduce internal model state.

## Non-goals

Hidden reasoning access, chain-of-thought reconstruction, private-state inference, multi-agent orchestration, additional agent adapters, model comparisons, training-data production, fine-tuning, and learned verifiers are outside this phase.

## What success looks like

A reader can explain where an observed session begins and ends, follow a tool request to its visible result when identifiers support that relationship, identify gaps, and relate each displayed record to its original source.

A reader can also identify the submitted artifact, understand the task requirements, and inspect independent evidence of which requirements passed or failed. An observed sequence does not establish why a solution worked.

The lab should make the limits of observation and evaluation as clear as their findings.

## Direction

This vision supersedes the initial general-purpose trajectory-generation proposal. The [Research Report](RESEARCH_REPORT.md) preserves the completed source investigation. The authorized next phase is [Controlled SWE Experiments](EXPERIMENT_PLAN.md), starting with a Rust parser repair, a database migration, and a process-supervision repair. Task-outcome evaluation is now in scope; model comparisons and training-data production remain deferred.
