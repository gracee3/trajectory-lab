# Phase 2: controlled SWE experiments

Status: planned and authorized direction, recorded 2026-09-06. This document specifies the next phase; no task fixtures, runner, evaluator, or experiment results are implemented by this documentation update.

## Objective

Give one Codex CLI agent deterministic but nontrivial software-engineering tasks, let it work autonomously, capture its observable trajectory, and independently evaluate the final artifact. Learn how exposed execution activity relates to an objectively checked result without inferring private reasoning or causality.

The environment, inputs, and acceptance criteria should be reproducible. The agent's implementation and sequence of actions need not be identical across runs. A successful run is not required to match a reference patch.

This extends the source-based [Research Report](RESEARCH_REPORT.md) with task-outcome evaluation. The original [Research Plan](RESEARCH_PLAN.md) remains the historical research-only handoff. Its no-implementation boundary applied to that deliverable, not to all future phases.

## Experiment contract

Each task must specify the following before an agent run:

| Component | Required definition |
|---|---|
| Identity | Task ID and revision; exact starting repository commit or fixture digest |
| Environment | Runtime/toolchain versions, dependency lockfiles, image digest if used, fixed data and seeds, locale/timezone where relevant |
| User-facing task | Behavioral requirements, constraints, allowed resources, and expected deliverable; enough information to finish without clarification |
| Workspace | Fresh disposable checkout; no access to previous solutions or run artifacts |
| Execution budget | Declared wall-time/process/resource limits; available usage limits where enforceable; no fabricated token accounting |
| Interaction policy | One agent, no delegation; define handling of approvals, clarification requests, pauses and human intervention before execution |
| Capture | CLI version, model/provider and relevant configuration, selected surface, provenance and known coverage gaps |
| Final artifact | Frozen final workspace or patch plus all relevant new files; base revision, changed-file manifest and content hashes |
| Evaluator | Versioned checks, environment, fixtures and expected behaviors maintained outside the agent's editable workspace |
| Result record | Per-requirement results, run termination, evaluator health, capture fidelity and explicit unavailable evidence |

Record the initial prompt separately when the selected stream does not emit it. Keep evaluator output distinct from agent-run tool output. Do not rely on the agent's final message as proof that an artifact passes.

## Initial three tasks

These are design briefs, not ready-to-run tasks. Exact interfaces, fixtures, budgets and acceptance checks must be finalized before the first run. Prefer Rust where it suits the task; do not require a single language across all domains.

### 1. Rust streaming parser repair

Starting fixture: a small crate with a documented framed-message format, a public incremental-feed API, baseline tests, and deliberately faulty handling of partial input.

Goal: repair parsing without breaking the documented API. Support multiple frames in one input, a frame split across calls, and explicit errors for invalid or oversized frames.

Acceptance: a fixed corpus of valid and invalid frames; every two-chunk split position for each short fixture; representative multi-chunk partitions; seeded fragmentation cases; multiple concatenated frames; bounded buffering; no panic on malformed input; existing caller compatibility. Specify how errors affect subsequent input and how end-of-input is signaled.

Why it is meaningful: requires understanding existing code, reproducing a fault, managing retained state, implementing error semantics, and checking regressions.

### 2. Data-preserving database migration

Starting fixture: a small application using a pinned database engine, an existing schema, and seeded legacy databases.

Goal: introduce a documented uniqueness constraint and migrate existing rows without unintended data loss. The task must state the policy for pre-existing duplicates and conflicts; the evaluator must not invent one afterward.

Acceptance: upgrade each supported starting schema; compare preserved records and relationships; test the specified duplicate policy, constraint enforcement, transaction rollback on an injected failure, and safe repeat invocation. Reopening the database must preserve the result. Require downgrade support only if explicitly part of the task.

Why it is meaningful: combines schema changes, application compatibility, data handling, transactional behavior, and failure recovery.

### 3. Process-supervision bug repair

Starting fixture: a supervisor with a documented child lifecycle and a reproducible shutdown or restart-accounting defect, plus local fixture child processes.

Goal: correct graceful termination, child reaping and bounded restarts under a specified policy. Define which signals are forwarded, the grace interval, escalation behavior and ownership of child processes.

Acceptance: controlled children acknowledge readiness, exit with selected statuses, ignore graceful termination, or crash on instruction. Check signal handling, restart limits, reaping and cleanup. Use handshakes/barriers and generous outer deadlines rather than arbitrary short sleeps. Use virtual time for restart-policy logic where practical; actual OS-process tests still need bounded timeouts.

Why it is meaningful: requires lifecycle reasoning expressed through code, asynchronous event handling, error reporting, and resource cleanup.

## Broader domain backlog

Expand after the initial tasks establish that capture and evaluation work.

| Domain | Candidate task | Objective checks |
|---|---|---|
| HTTP services | Idempotent job-submission endpoint | Duplicates, conflicting payloads, concurrent requests and restart persistence |
| Concurrency | Queue that loses jobs during shutdown | Controlled scheduling/barriers; no lost or duplicate jobs under defined delivery semantics |
| Networking | Client reconnection and retry repair | Local scripted server, injected disconnects, fake clock and bounded retries |
| Observability | Correct metrics counters and labels | Exact expected samples from fixed events; cardinality and reset behavior |
| Build and packaging | Repair clean-environment build and distribution | Locked/offline dependencies, artifact startup, expected file layout |
| Frontend | Accessible filterable table with URL state | Keyboard interactions, refresh/back behavior, fixed data and browser checks |
| Security engineering | Repair archive extraction path traversal | Local adversarial fixtures and verification that writes stay inside the destination |

These extend the same single-agent experiment contract. They do not introduce model ranking, production-system testing, or a public benchmark leaderboard.

## Reproducibility and autonomy

- Reset to the same task revision and fresh state for each independent run. Pin dependencies and pre-provision them so external network availability does not decide the result.
- Use local fixture services and deterministic inputs. Fix seeds and record them; a seed alone does not make concurrent execution deterministic.
- Evaluate behavior and required properties, not exact code, patch text, number of tool calls, or one preferred approach.
- Allow the agent to inspect files, edit, build, test and iterate within the declared boundary. Record its own tests as trajectory evidence.
- Define how unsupported approvals and requests for help are handled. Do not silently auto-approve more authority or add helpful prompts midway. Record intervention as an assisted run.
- Reset between retries and preserve each attempt. A resume is a continuation associated with the original run, not an independent fresh trial.
- Use robust resource limits to terminate runaway work. If a timeout prevented evaluation, record that explicitly; do not claim incorrectness that the evaluator never established.

## Independent evaluation

Use two separate tracks: [product correctness and observation fidelity](EVALUATION.md). Never combine them into an unexplained aggregate score.

Keep acceptance checks outside the agent's editable workspace. Withheld tests exercise stated requirements and must be inaccessible during the autonomous run if described as withheld. Merely placing them in a sibling directory is insufficient when the agent can read that directory.

After the run ends, freeze the artifact and evaluate it in a fresh environment. The candidate code must not be able to alter the evaluator or the authoritative result record. Treat external evaluation as a separate stage with its own logs, process result and provenance.

Before evaluating agent attempts, demonstrate that the starting broken fixture fails the relevant checks, a reference solution passes, and representative incomplete fixes fail. The reference solution validates the task; it is not a patch-matching oracle and must not leak into the starting checkout or its accessible Git history.

Checks provide evidence against a declared contract, not a proof of universal correctness. Document untested behavior and any flakiness.

## Capture and trajectory evidence

Start by validating the installed CLI and the report's proposed `codex exec --json` surface on a real task. Record stdout events, stderr, invocation metadata and process outcome separately. The report's source finding is not a runtime verification.

Known gaps include omitted streaming chunks, interactive approval exchanges, native turn IDs/timing, compaction, output truncation, and possible event loss. Missing evidence stays missing. Where an interactive transcript is inspected, label it as a human-facing corroborating view; do not assume parity with exec JSON.

If the first experiment requires richer live events, document the decision to use app-server and its client responsibilities. Do not patch Codex internals or silently change task permissions to obtain more observations.

A run bundle should associate:

- The task/environment/evaluator revisions and initial prompt.
- CLI/model/configuration provenance, with secrets omitted.
- Captured native events, capture metadata, diagnostics and termination evidence.
- The frozen final artifact and an exact manifest of what was submitted for evaluation.
- Independent evaluator output and per-requirement results.
- A capture-fidelity assessment, gaps, redactions and any human interventions.

Preserve successful, unsuccessful, interrupted and infrastructure-failed attempts. An unsuccessful solution can be an informative trajectory; a successful solution can have incomplete capture. Reasoning-summary content is unnecessary for this phase, and hidden reasoning/chain-of-thought reconstruction remains out of scope.

## Implementation sequence and completion criteria

1. Finalize the three task contracts; implement and validate the parser fixture/evaluator first.
2. Confirm the installed Codex version and capture surface with a real benign run. Resolve the minimal task/run/event/result schemas from that evidence.
3. Implement the smallest runner and independent evaluation stage, then complete one parser run end to end.
4. Add the migration and supervisor tasks using the same contract. Repeat from clean starts and record variability rather than claiming identical trajectories.
5. Add offline replay that presents captured activity alongside separately labeled evaluator results without executing tools.
6. Expand domains only after the first three tasks meet the criteria below.

Phase completion requires three versioned tasks with validated independent checks; at least one attempted run and an honest result bundle per task; explicit handling of failed/interrupted/incomplete captures; and a reader able to follow an observed tool request/result through to the submitted artifact and external evaluation where evidence permits. Passing agent solutions are desirable, but a failed agent attempt does not invalidate a correctly designed experiment.

This plan authorizes the direction for subsequent implementation work. This update itself delivers documentation only.
