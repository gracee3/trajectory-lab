# Roadmap

The current scope is an educational study of Codex CLI's observable execution lifecycle. This roadmap supersedes the earlier training-data pipeline.

## Foundation — documentation

- [x] State the educational purpose and non-goals.
- [x] Add Vision and a research-only handoff.
- [x] Mark legacy schemas and broader pipeline plans as superseded.

## Milestone 1 — map the Codex CLI lifecycle

Research session/turn entry, tool execution, completion, interruption, and resumption where supported. Pin the examined version and cite evidence.

Done when: `docs/RESEARCH_REPORT.md` explains verified boundaries and explicitly marks unknowns.

## Milestone 2 — identify observable events

Inventory user-accessible surfaces and native events. Trace tool calls and results, identifiers, ordering, output transformations, and visibility limits.

Done when: the research report includes an evidence-backed inventory and recommends one minimal capture surface. Milestones 1 and 2 form the research handoff; no implementation is required.

## Milestone 3 — design a simple event schema

After the research report is reviewed, define a small envelope for verified observations. Preserve native payloads, source/version provenance, correlation identifiers where available, and distinctions between source and capture metadata.

Done when: a documented schema and sanitized fixtures express known events and missing data without invented information. The original task/trajectory schemas do not fulfill this milestone.

## Milestone 4 — export JSONL logs

Capture through the selected existing surface and export one event record per line. Document ordering, partial records, interruption, sensitive-data handling, and coverage limits.

Done when: a small session produces parseable JSONL whose records can be traced to their source.

## Milestone 5 — replay a session

Display recorded activity in order and show tool relationships only when supported by captured identifiers. Make incomplete sessions and unknown events visible.

Done when: a user can inspect the recorded session without contacting a model or rerunning tools.

## Deferred scope

Multi-agent frameworks, other agent adapters, benchmarking, trajectory ranking, dataset generation, and model training are outside this roadmap.
