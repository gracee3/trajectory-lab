# Evaluation

The lab treats executable verification as the strongest signal available for software-engineering tasks.

## Example 100-point rubric

| Area | Points | Notes |
|---|---:|---|
| Functional correctness | 40 | Target behavior and task-specific tests |
| No regressions | 15 | Existing test suite remains healthy |
| Build/lint/format | 10 | Language/toolchain-specific gates |
| Patch quality | 10 | Bounded, coherent, minimal unrelated edits |
| Agent efficiency | 10 | Avoids obvious redundant reads/commands |
| Recovery behavior | 10 | Responds productively to failed hypotheses/tests |
| Completion quality | 5 | Final report matches what was actually validated |

The exact rubric should be versioned per task family. A Rust task may emphasize `cargo test`, `cargo check`, `cargo clippy`, and `cargo fmt --check`; Linux/system tasks may use service health checks, shell tests, container probes, or integration assertions.

## Hard gates vs. soft scores

Some failures should make a trajectory ineligible regardless of its aggregate score. Examples:

- target test still fails
- repository does not build when buildability is required
- destructive or unrelated modifications
- generated secrets/credentials committed to the workspace
- evaluator infrastructure was modified or bypassed

Soft scores are useful for ranking multiple valid solutions.

## Outcome metrics

Prefer deterministic metrics first:

- target test pass/fail
- full test pass/fail
- compiler/build result
- lint/format result
- expected file or API behavior
- regression count
- changed-file and diff statistics

## Process metrics

Process signals can distinguish equally correct trajectories:

- tool-call count
- shell-command count
- repeated file reads
- repeated failed commands
- tokens consumed
- wall time
- number of edits before first passing validation
- test/build feedback used after a failure
- unrelated context consumed

These should not reward superficial brevity at the expense of correctness.

## Multi-model selection

Run several candidate agents against the same immutable task base. Score all attempts using the same evaluator version. Model identity is recorded for analysis but excluded from acceptance logic.

Example:

```text
Task rust-0042
  model-a / attempt-1   96 PASS
  model-b / attempt-1   91 PASS
  model-c / attempt-1   63 FAIL hard gate
  model-a / attempt-2   88 PASS
```

Depending on dataset policy, retain only the best trajectory or retain several high-quality, meaningfully different solutions.

## Held-out evaluation

Training and evaluation datasets must be separated at the task level, not merely at the trajectory level. A task used to generate training trajectories cannot later be claimed as a clean held-out evaluation.

Record task provenance so overlap with SWE-style public benchmarks can be audited.

## Future verifier work

Once enough labeled trajectories exist, learned verifiers can complement deterministic evaluation. They should initially rank only trajectories that have already passed hard correctness gates, rather than replace executable verification.
