# Roadmap

Trajectory Lab studies one Codex CLI agent through observable execution and reproducible SWE tasks. [Phase 2](EXPERIMENT_PLAN.md) adds independent artifact evaluation; it does not introduce model rankings or a training-data pipeline.

## Foundation and phase 1 — delivered documentation

- [x] State educational purpose and observable-evidence boundaries.
- [x] Publish [Research Plan](RESEARCH_PLAN.md) and [Research Report](RESEARCH_REPORT.md).
- [x] Map lifecycle and inventory surfaces at a pinned upstream commit.
- [x] Document capture recommendation and gaps.
- [x] Record the authorized controlled-experiment phase.

The research report is source/documentation evidence. Runtime validation was not performed because no local CLI executable was available. No task pack, recorder, evaluator or replay tool is implemented yet.

## Phase 2.1 — define and validate initial tasks

Write contracts for a Rust parser repair, data-preserving migration and process-supervision repair. Implement the parser fixture/evaluator first, then extend to the other two.

Done when each fixture is versioned and its evaluator rejects the broken baseline and representative incomplete fixes while accepting a reference solution. Reference solutions and withheld tests must not be available to the agent.

## Phase 2.2 — validate capture and define schemas

Observe a real run on the installed CLI; document version/configuration and selected surface. Define minimal task, run, event and result contracts with native provenance and explicit missing data.

Done when observed fixtures support the schemas and known gaps are recorded. Legacy schemas do not satisfy this milestone.

## Phase 2.3 — capture and evaluate one autonomous run

Run the parser task in a fresh workspace, capture existing exposed events, freeze its final artifact, and evaluate externally.

Done when a run bundle includes prompt/provenance, native records, diagnostics, termination, artifact manifest, per-requirement results and separate capture-fidelity assessment. Validate failure/interruption handling too.

## Phase 2.4 — extend across the initial three domains

Apply the same workflow to migration and process supervision. Repeat clean runs to assess reproducibility and variability.

Done when every task has a validated evaluator and at least one attempted run with an honest result bundle. Do not discard unsuccessful attempts or require identical agent trajectories.

## Phase 2.5 — offline inspection and replay

Display captured activity with supported call/result links, artifact references and clearly separated evaluator outcomes.

Done when a reader can inspect evidence and gaps without executing tools or contacting a model.

## Later domain expansion

HTTP, concurrency, networking, observability, packaging, frontend and local security-repair tasks are documented in the [domain backlog](EXPERIMENT_PLAN.md#broader-domain-backlog). Expand after the initial three establish a useful capture/evaluation workflow.

## Deferred scope

Multi-agent frameworks, additional agent adapters, model comparison/leaderboards, trajectory ranking, training-data production, fine-tuning and learned verifiers.
