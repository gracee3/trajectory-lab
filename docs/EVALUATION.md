# Evaluating observation fidelity

This phase evaluates the accuracy of the lab's observations and explanations. Model ranking, solution scoring, trajectory acceptance, and training-data selection are outside scope.

## Research quality

Each claimed event or boundary must be supported by versioned documentation, source evidence, or a labeled runtime observation. Separate internal implementation findings from user-accessible behavior and mark unknowns.

## Future capture and replay checks

After research and schema design, verify that:

- Exported records map back to their source without silently inventing or dropping information.
- Capture order and source ordering guarantees are distinguished.
- Calls and results are connected only through supported evidence.
- Partial output, incomplete sessions, and unknown event types remain explicit.
- JSONL is parseable and schema version/provenance are recorded.
- Replay displays the recorded activity without reexecuting commands.
- Redacted examples identify their omissions.

There is no numerical scoring rubric in this phase. Success means a reader can follow what was observed and understand its limits.
