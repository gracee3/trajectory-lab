# Evaluating artifacts and observation fidelity

Phase 2 adds independent task-outcome evaluation to the original observation-fidelity work. See [Experiment Plan](EXPERIMENT_PLAN.md). Evaluators and result schemas are planned, not implemented.

## Two separate questions

| Track | Question | Evidence |
|---|---|---|
| Product correctness | Does the frozen final artifact satisfy the stated task contract? | External acceptance checks, per-requirement outcomes, evaluator logs |
| Observation fidelity | Did the capture faithfully preserve the selected surface's exposed activity? | Native records, provenance, coverage checks, loss/truncation indicators |

A correct artifact can have incomplete capture. A failed artifact can have useful, faithful observations. Preserve both dimensions and all attempts.

## Product evaluation

- Define observable requirements before execution; do not grade against an exact reference patch.
- Keep evaluator code and withheld fixtures outside the agent's access during the run, not merely outside its working directory.
- Freeze the submitted artifact, including relevant untracked files, and evaluate in a clean environment with pinned dependencies.
- Prevent candidate code from modifying the authoritative evaluator or result records.
- Validate each task: broken baseline fails the intended checks, reference solution passes, and plausible incomplete fixes fail.
- Agent-authored tests and claims are evidence of activity, not independent acceptance.
- Withheld cases test the written contract. Report per-requirement pass/fail/not-run results and diagnostic output before considering any numerical summary.
- Record timeouts, resource limits, human interventions and evaluator failures explicitly.

Keep run termination (completed, interrupted, timed out, blocked, or infrastructure failure), product outcome (pass, fail, or not evaluated), evaluator health, and capture fidelity distinct. An agent process exiting successfully does not establish product correctness. An evaluator crash is not a candidate test failure.

## Capture and replay checks

- Each captured event maps back to its selected source with version and invocation provenance.
- Capture order is distinguished from source order and causal claims.
- Calls and results are joined only through supported identifiers.
- Partial output, unknown records, event loss, redactions and incomplete sessions remain explicit.
- JSONL parses under the agreed envelope while preserving unknown native fields.
- Missing native IDs/timestamps remain unavailable; collector metadata is labeled.
- Replay displays captured evidence without running tools or contacting a model.
- Recovered history and evaluator events are labeled separately from live agent events.

No trace is presumed complete simply because it parses or the product passes.

## Reproducibility and limits

Use fixed data, local services, fake clocks where appropriate, and controlled concurrency. Verify actual OS behavior with bounded tests and explicit synchronization. Avoid fragile performance thresholds on shared hardware.

Repeat from clean starts to expose variation or flakiness. Record task, environment, evaluator and agent versions for each attempt. Correctness is established only against the declared checks; untested behavior remains unknown.

## Research quality and deferred work

Continue distinguishing runtime observations, versioned source findings, documented behavior and proposals. The [Research Report](RESEARCH_REPORT.md) is source-backed and did not execute runtime experiments.

Model ranking, trajectory ranking, training-data selection, learned verifiers and fine-tuning remain outside this phase. Task acceptance checks are now in scope.
