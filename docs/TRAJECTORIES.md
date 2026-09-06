# Observable trajectories and run bundles

A trajectory is the observable record of one Codex CLI run on a versioned task, as exposed by the selected surface. It is not a complete execution trace or complete model context.

## Evidence and identity

The [Research Report](RESEARCH_REPORT.md) establishes source-based event mappings and gaps. Validate the installed CLI before finalizing schemas. Record source version/surface, native payload, capture order and available correlation identifiers. Source timestamps and native IDs remain distinct from collector receipt times and run IDs.

The [Experiment Plan](EXPERIMENT_PLAN.md) associates a trajectory with its task revision, starting state, initial prompt, model/configuration, diagnostics, process outcome, frozen final artifact, evaluator revision and results. An artifact must include relevant new files; a tracked-files-only diff can be insufficient.

Run IDs identify independent attempts. Continued/resumed invocations must retain their association with the originating run, and invocation-local item IDs must not be joined across captures without evidence.

## Separate outcomes

A task pass does not certify capture completeness. A failed task can have a faithful trajectory. Retain unsuccessful, interrupted, assisted and infrastructure-failed attempts with explicit outcomes. Evaluator output belongs to a separate evaluation stage, not the agent's native event stream.

Never synthesize messages, relationships, timestamps or reasoning to make a trace appear complete. Already-exposed reasoning summaries are unnecessary for the current task suite; document any content exclusions rather than implying full capture.

## JSONL and replay

Future capture exports one record per line under an agreed envelope and preserves unknown native data. Document partial lines, loss, truncation, missing terminal events and redaction. Source bytes and parsed/normalized representations have different fidelity guarantees.

Replay presents recorded activity and separately labeled evaluator results. It cannot reconstruct omitted content or establish the model's reasons for an action, and must not rerun tools.

## Schema and publication status

The files in [schemas](../schemas/) are legacy drafts. Phase-2 task/run/event/result schemas remain to be designed from validated evidence and must not inherit unsupported assumptions.

Use local benign fixtures and inspect artifacts before publication. Keep secrets and personal/private content out of public examples. Identify redactions and omissions; a sanitized excerpt is not an untouched raw record.
