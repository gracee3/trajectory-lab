# Architecture direction

Phase 2 is specified in [Experiment Plan](EXPERIMENT_PLAN.md), informed by the source-backed [Research Report](RESEARCH_REPORT.md). Components below are planned.

| Component | Responsibility |
|---|---|
| Task fixture | Versioned starting code, data, environment and behavioral contract |
| Runner | Fresh workspace, one autonomous Codex CLI agent, declared resources and termination policy |
| Capture | Existing user-accessible events plus separately labeled invocation, stderr and process metadata |
| Artifact snapshot | Freeze final candidate changes and relevant new files before evaluation |
| Independent evaluator | Check the submitted artifact in a clean environment; protect tests and authoritative results from candidate modification |
| Run bundle | Associate task/run/artifact/evaluator identities, evidence, outcomes and gaps |
| Replay | Inspect recorded activity and separately labeled evaluation results offline |

The evaluator is a separate stage, not another agent. Keep its code and withheld cases inaccessible during the agent run. Keep final results outside candidate control during evaluation as well.

Validate the installed CLI and selected surface before implementing its adapter. Exec JSON is the initial recommendation, with documented omissions; app-server is a larger option if required events justify it. Do not assume interactive transcript, exec output and persisted history have identical coverage.

Preserve native payloads, source/version provenance, and explicit unavailable values. Collector identifiers and receipt times are not source identifiers or execution timestamps. The task's final artifact is independent of whatever file-change details a log happens to expose.

Replay never reruns tools or requests model responses. Private-state inference, multi-agent orchestration and training export remain outside the architecture.
