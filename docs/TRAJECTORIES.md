# Observable session records

For this phase, a trajectory means the observable record of a Codex CLI session as exposed by the selected surface. It does not imply a complete execution trace or complete model context.

## Research before schema

The [Research Plan](RESEARCH_PLAN.md) must establish native event names, fields, identifiers, timing, ordering, and coverage before a canonical event schema is designed.

Potential envelope concerns include schema version, source surface and Codex version, capture order, native event type, native payload, and available correlation identifiers. These are design questions, not confirmed Codex fields.

Record source timestamps separately from capture timestamps. Preserve unavailable values as unavailable. Never synthesize tool relationships, messages, or reasoning to make a trace appear complete.

## JSONL and replay

A later milestone will export one event record per JSONL line and document how partial output, unknown events, gaps, and interrupted sessions are handled.

Replay presents captured observations in order. It cannot reconstruct omitted content, hidden reasoning, private state, or the model's reasons for taking an action.

## Existing schemas

The files in [schemas](../schemas/) are legacy drafts from the original dataset-generation scaffold. They are not the current event contract and should not constrain the research findings.

## Publishing examples

Use benign sessions and sanitize secrets or personal/private content before committing examples. Keep redaction explicit so a sanitized excerpt is not mistaken for an untouched raw record.
