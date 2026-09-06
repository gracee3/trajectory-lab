# Controlled SWE tasks

This directory will contain versioned task fixtures and behavioral contracts for [Phase 2](../docs/EXPERIMENT_PLAN.md). No fixtures are implemented yet.

Start with a Rust streaming parser repair, a data-preserving database migration, and a process-supervision repair. Implement and validate the parser experiment first.

Each task fixes its starting revision, environment, inputs, requirements, allowed resources, budget, autonomy policy and expected deliverables. It must require meaningful engineering work while supporting objective checks. Different correct implementations are acceptable.

Protect reference solutions and withheld evaluator cases from agent access, including accessible Git history. Validate broken baselines and reference solutions before using a task. See [Evaluation](../docs/EVALUATION.md).
