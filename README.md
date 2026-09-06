# Trajectory Lab

Trajectory Lab is an educational project focused on understanding the observable execution lifecycle of Codex CLI by instrumenting events that are already exposed to the user.

The goal is to learn where a session begins, how tool calls and results move through it, what the user can observe, and how those observations can be recorded and replayed. **Observe, don't infer.** An event log describes exposed execution activity; it does not explain the model's private reasoning.

## Start here

- [Vision](docs/VISION.md): why this lab exists and its boundaries.
- [Research Plan](docs/RESEARCH_PLAN.md): questions and the research-only handoff for the next Codex agent.
- [Research Report](docs/RESEARCH_REPORT.md): source-backed lifecycle map, event inventory, capture recommendation, and runtime-validation gaps.
- [Roadmap](docs/ROADMAP.md): milestones and completion criteria.

## Non-goals

- Accessing hidden reasoning or reconstructing chain of thought.
- Inferring private model state, intent, or unexposed context from visible activity.
- Building a multi-agent framework or supporting multiple agents at this stage.
- Training or fine-tuning models, generating training datasets, ranking models, or building a benchmark platform.
- Reimplementing Codex CLI or adding instrumentation to its internals before understanding existing user-accessible surfaces.

## Milestones

1. **Map the Codex CLI lifecycle.** Research session and turn boundaries, tool execution, completion, interruption, and resumption where supported.
2. **Identify observable events.** Inventory existing user-accessible surfaces, their payloads, relationships, limitations, and version dependencies.
3. **Design a simple event schema.** Use the research findings to represent observed records without inventing missing information.
4. **Export JSONL logs.** Capture one record per line with provenance and explicit handling of incomplete or unavailable data.
5. **Replay a session.** Display a recorded session in order, including visible tool activity and results. Replay means inspecting the log, not rerunning commands.

Milestones 1–2 are documented in the research report at a pinned upstream commit. Runtime observation remains unperformed because a local Codex CLI executable was unavailable. Milestones 3–5 remain planned; no capture or replay features are implemented.

## Current status and layout

The repository contains documentation and preliminary schemas from an earlier, broader trajectory-generation proposal. There is no implemented recorder, exporter, or replay tool yet.

| Path | Current role |
|---|---|
| `docs/` | Vision, research handoff, roadmap, and provisional design notes |
| `schemas/` | Legacy draft task/trajectory schemas; not the new event contract |
| `runners/` | Notes for future Codex CLI capture work |
| `tasks/` | Notes for future small observation scenarios |
| `evaluators/` | Notes on capture and replay fidelity |

The current direction supersedes the original multi-model training-data pipeline. Existing schemas are retained for reference and must not determine the new event design. The research report recommends a bounded `codex exec --json` capture adapter, with explicit limits on IDs, timing, output, cancellation, and usage. Review that recommendation and validate the target CLI version before beginning the schema and implementation milestones.
