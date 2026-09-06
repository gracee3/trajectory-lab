# Vision

## Why this lab exists

Trajectory Lab is a focused educational study of one mature coding agent: Codex CLI. It should help a user understand the observable execution lifecycle through concrete evidence: session boundaries, messages, tool requests, tool results, and other activity that the examined version actually exposes.

Learning comes first. Start with the existing implementation and user-accessible interfaces, establish what is observable, and only then design a small recorder and replay experience.

## Observe, don't infer

An observed record is evidence of what a surface emitted. It is not evidence of hidden reasoning, a complete model input, or why the model chose an action.

Every finding should distinguish:

- Directly observed behavior, with a reproducible example.
- Behavior documented or located in source, with a versioned citation.
- Questions that remain unverified or unavailable through the inspected surface.

Do not fill gaps with reconstructed chain of thought, presumed messages, inferred causal links, or invented timestamps. Absence from a log does not prove absence from execution.

A source-code event is not automatically a user-accessible event. Research must trace whether and how it reaches an exposed surface.

## Scope

Focus exclusively on Codex CLI. Research first; then map verified events into a simple schema, export JSONL, and replay one session as an inspectable timeline.

Replay displays recorded activity. It does not execute tools again, regenerate responses, or reproduce internal model state.

## Non-goals

Hidden reasoning access, chain-of-thought reconstruction, private-state inference, multi-agent orchestration, additional agent adapters, model comparisons, training-data production, fine-tuning, and learned verifiers are outside this phase.

## What success looks like

A reader can explain where an observed session begins and ends, follow a tool request to its visible result when identifiers support that relationship, identify gaps, and relate each displayed record to its original source.

The lab should make the limits of observation as clear as the observations themselves.

## Direction

This vision supersedes the initial general-purpose trajectory-generation proposal. The immediate next step is the [research-only handoff](RESEARCH_PLAN.md).
