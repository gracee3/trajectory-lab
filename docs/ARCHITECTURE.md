# Architecture status

Architecture is provisional until the [research report](RESEARCH_PLAN.md) establishes which Codex CLI events are exposed and how they can be observed.

The intended lab has three small responsibilities:

1. Observe an existing user-accessible Codex CLI surface.
2. Preserve its records in a simple, documented JSONL representation.
3. Replay those records for educational inspection.

The research phase must select the surface before choosing an implementation. Do not assume all execution modes expose identical events or that an internal event type is publicly available.

Preserve source/version provenance and native payloads. Distinguish source identifiers and timestamps from identifiers or timestamps added by the collector. Document gaps, transformations, and any redaction. A normalized record must not imply access to unobserved model inputs or internal state.

Replay is a display operation over recorded data; it must not rerun tools or request new model responses.

The previous multi-model runner, scoring, acceptance, and training-export architecture is superseded. See [Vision](VISION.md) for scope and [Roadmap](ROADMAP.md) for sequencing.
