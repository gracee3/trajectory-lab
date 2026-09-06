# Evaluators

Evaluators score a completed attempt. Deterministic outcome checks come first; process metrics are secondary ranking signals.

Initial evaluator families:

- target behavior / regression tests
- build and compiler checks
- lint and formatting checks
- diff scope / unrelated-change detection
- tool-use and repeated-work metrics
- recovery behavior after failed commands or tests
- final-response validation against actual results

Hard correctness failures should normally make a trajectory ineligible regardless of soft process scores. Evaluator versions must be recorded with every score artifact.
