# Independent evaluation

Phase 2 plans two separate evaluator responsibilities:

- Product correctness: check a frozen candidate artifact against the stated task contract in a clean environment.
- Observation fidelity: verify faithful capture, supported relationships, explicit gaps and offline replay.

Keep acceptance checks, reference solutions and withheld cases inaccessible to the agent during its run. Protect evaluator code and authoritative results from candidate modification during evaluation. Agent-authored tests do not replace independent checks.

Record per-requirement outcomes and distinguish candidate failure from evaluator/infrastructure failure. Validate broken baselines, passing references and incomplete fixes. No evaluator is implemented yet.

See [Evaluation](../docs/EVALUATION.md) and [Experiment Plan](../docs/EXPERIMENT_PLAN.md).
