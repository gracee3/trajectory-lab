# Architecture direction

Phase 2 is specified in [Experiment Plan](EXPERIMENT_PLAN.md), informed by the source-backed [Research Report](RESEARCH_REPORT.md). The [Environment Lineage and Reuse Plan](ENVIRONMENT_PLAN.md) maps existing project Dockerfiles, proposed image families and offline dependencies. Components below are planned.

- **Task fixture:** versioned starting code, data, environment and behavioral contract.
- **Runner:** fresh workspace, one autonomous Codex CLI agent, declared resources and termination policy.
- **Capture:** existing user-accessible events plus separately labeled invocation, stderr and process metadata.
- **Artifact snapshot:** freeze final candidate changes and relevant new files before evaluation.
- **Independent evaluator:** check the submitted artifact in a clean environment; protect tests and authoritative results from candidate modification.
- **Run bundle:** associate task/run/artifact/evaluator identities, evidence, outcomes and gaps.
- **Replay:** inspect recorded activity and separately labeled evaluation results offline.

New environments may extend a common Ubuntu/Codex base. Existing project images may retain their original parents and receive the same versioned Codex support payload. Both paths implement the same run contract; neither assumes that a slim application runtime contains development tools.

Share immutable image layers and installer collections. Give each attempt a fresh workspace and writable state. The runner selects allowed installer mounts and service access; the agent does not require Docker socket access. Keep fault-injection scripts and answer-bearing history outside the candidate environment.

The evaluator is a separate stage, not another agent. Keep its code and withheld cases inaccessible during the agent run. Keep final results outside candidate control during evaluation as well. Existing reference images can remain separate evaluator services.

Validate the installed CLI and selected surface before implementing its adapter. Exec JSON is the initial recommendation, with documented omissions; app-server is a larger option if required events justify it. Do not assume interactive transcript, exec output and persisted history have identical coverage.

Preserve native payloads, source/version provenance, and explicit unavailable values. Collector identifiers and receipt times are not source identifiers or execution timestamps. The task's final artifact is independent of whatever file-change details a log happens to expose.

Replay never reruns tools or requests model responses. Private-state inference, multi-agent orchestration and training export remain outside the architecture.
