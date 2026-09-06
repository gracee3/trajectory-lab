# Project scenario catalog

Review started 2026-09-06. Scope: the eight repositories in the [profile's Selected work](https://github.com/gracee3/gracee3/blob/HEAD/README.md), including their accessible development branches and historical work. This replaces the narrower adapter-first discovery checklist.

The goal is to identify turning points and engineering lessons, then develop detailed scenario prompts. A scenario may be a code repair, a diagnosis from supplied evidence, an experiment-design exercise, or a question with an objectively checkable answer. No adapter architecture or scenario implementation is prescribed here.

## Reading the evidence

- **Demonstrated:** a historical run/test report accompanies the relevant implementation. These are repository-recorded results; this review does not rerun them.
- **Code/test-backed:** the fix and relevant tests are visible, but a fresh execution has not been performed here.
- **Implemented / evidence limited:** a concrete change exists, but its broader claimed outcome is not fully demonstrated by the inspected artifacts.
- **Negative result:** an experiment failed its gate or was abandoned. Preserve the finding and the successful diagnosis separately from the unsuccessful objective.
- Branch status, merge status, and production use do not determine whether an issue was solved.
- Git diffs establish changes and chronology. They do not reconstruct missing conversations, failed commands, or private reasoning, and do not establish which agent made every decision.

Prompt seeds and proposed acceptance checks below are new scenario ideas. They are not descriptions of tests already run unless explicitly labeled as evidence. Follow-up notes collect related ideas without narrowing the work prematurely.

## Review progress

- [x] WhisperX-batch
- [ ] native-asr
- [ ] qwen38-int8-lab
- [ ] gpt-oss-rs, including heterogeneous/Tiger Lake work
- [ ] supermicro-observability
- [ ] digital-liquid-light-lab
- [ ] Mirabile
- [ ] Magnolia

## How to develop a selected idea later

- Preserve a starting snapshot, relevant evidence, and the historical solution separately.
- Write a task that states the outcome and constraints without leaking the fix.
- Accept alternative correct solutions; do not grade by exact patch matching.
- Preserve fresh attempts, resumes, hints, failures, and evaluator failures as distinct records.
- Keep task success, capture fidelity, and calibration suitability separate.
- The downstream calibration interest is quantization calibration, not fine-tuning. Export design and corpus experiments remain later work; no calibration benefit is claimed here.
- Keep scenario families and close variants separated from held-out evaluation.

## WhisperX-batch

Reviewed main at `b019546`, all ten main-history commits, the two listed development branches (`codex/offline-foundation`, `codex/publish-agent-guidance`), both PR descriptions, the stack-transition diff, fallback changes, Dockerfile, and offline tests. The branch implementation is represented by [PR #1](https://github.com/gracee3/whisperX-batch/pull/1), which reports 38 offline tests passing and explicitly says GPU/image/benchmark validation was not performed for that PR.

### WX-1 — Resolve a compatibility problem at the whole-stack level

- **Turning point:** the project moved from a Python 3.10 / CUDA 12.1 / Torch 2.4.1 stack with manually managed WhisperX dependencies to Python 3.11 / CUDA 12.8 / Torch 2.8, pyannote 4, and CTranslate2 4.7.1. The old Dockerfile used `--no-deps` and documented a Torch/torchvision mismatch; the new multi-stage image installs an aligned stack and performs import/version checks.
- **Evidence:** [stack transition `96850e2`](https://github.com/gracee3/whisperX-batch/commit/96850e2ecc11e4b06796524ef82b6833050a077f), following `4da36ef`; [current Dockerfile](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/Dockerfile.whisperx-torch280-cu128).
- **Status:** implemented, with owner-reported historical success. The inspected repository does not preserve a complete immutable GPU build/run bundle establishing every compatibility claim. Do not describe this as freshly reproduced.
- **Prompt seed:** Given the old image recipe, dependency metadata, and representative installation/runtime failures, determine which constraints conflict. Propose a coherent stack migration, explain the ABI and packaging boundaries, and specify what installation checks cannot establish about GPU execution.
- **Checkable outcome:** a supplied compatibility inventory is internally consistent; required packages cannot silently replace the intended core stack; the response distinguishes resolver success, imports, model initialization, and GPU execution.
- **Big-picture lesson:** repeated package pin changes may be treating symptoms of an incompatible stack. A diagnosis/report task could preserve this lesson without downloading a model.

### WX-2 — Recover an optional stage without carrying its broken configuration forward

- **Turning point:** the original diarization retry was made more complete: removing only `--diarize` left its model argument and embedding flag behind. `without_diarization_args` now removes the dependent options together.
- **Evidence:** [initial retry `7082d92`](https://github.com/gracee3/whisperX-batch/commit/7082d92ad32d70ac748bc5b5d64d11778f77b7c6), [corrective diff `b019546`](https://github.com/gracee3/whisperX-batch/commit/b019546fbae413711615577c387906ea06af198f), and [fallback tests](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/tests/test_transcribe.py).
- **Status:** code/test-backed; the PR reports passing offline tests.
- **Prompt seed:** An optional analysis stage fails, and retrying with it disabled still fails. Trace the full option dependency chain, preserve the mandatory operation, and make the degraded outcome visible.
- **Checkable outcome:** no orphaned option values remain; unrelated arguments survive; failure without the optional feature does not cause an endless retry.
- **Lesson:** graceful degradation is a configuration/state transition, not merely removing one flag.

### WX-3 — Make experimental intent survive nested configuration layers

- **Turning point:** the benchmark launcher now explicitly sends either `--no-diarize` or `--diarize`. Omitting a flag was not equivalent to requesting false when a downstream config could enable the feature. Device tracing also resolves precedence across sweep values, invocation, and config.
- **Evidence:** [launcher change](https://github.com/gracee3/whisperX-batch/commit/b019546fbae413711615577c387906ea06af198f); [benchmark command and device-precedence tests](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/tests/test_benchmark.py).
- **Status:** code/test-backed, with reported offline validation.
- **Prompt seed:** Two supposedly equivalent benchmark runs use different effective settings. Reconstruct the effective command/configuration and fix the precedence contract so the recorded experiment matches what actually executes.
- **Checkable outcome:** explicit false overrides an inherited true; device lists remain one value; effective settings and telemetry device selection agree.
- **Lesson:** a valid benchmark can be invalidated before inference starts.

### WX-4 — Turn “offline ready” into a concrete resource contract

- **Turning point:** preflight checks require the large-v3 model layout, cache preparation checks more than a marker directory, and Docker command construction preserves read-only model/input mounts and offline environment settings. Home expansion was corrected to occur before path resolution.
- **Evidence:** [`b019546` path fix and tests](https://github.com/gracee3/whisperX-batch/commit/b019546fbae413711615577c387906ea06af198f); [resource/mount tests](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/tests/test_transcribe.py); [cache completeness test](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/tests/test_helpers.py).
- **Status:** code/test-backed; offline flags and file checks are not proof that every upstream component never accesses the network.
- **Prompt seed:** A job works on its author's machine but fails in a disconnected container. Diagnose path expansion, incomplete resources, and mount visibility from a supplied filesystem inventory and launch command.
- **Checkable outcome:** distinguish missing artifacts from inaccessible artifacts; preserve immutable inputs; fail before expensive work when required resources are absent.
- **Lesson:** portable execution requires a resource inventory, not just a cache directory.

### WX-5 — Reconstruct a trustworthy benchmark from messy outputs

- **Turning point:** the benchmark harness adds reference manifests, WER normalization, speaker-label removal, prediction parsing, GPU summaries, and explicit sweep parsing. Offline tests anchor representative calculations.
- **Evidence:** [harness introduction `96850e2`](https://github.com/gracee3/whisperX-batch/commit/96850e2ecc11e4b06796524ef82b6833050a077f); [scoring/manifest/trace tests](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/tests/test_benchmark.py); [publication limits](https://github.com/gracee3/whisperX-batch/blob/b019546fbae413711615577c387906ea06af198f/docs/BENCHMARKING.md).
- **Status:** bookkeeping is code/test-backed. The documentation explicitly limits the older 200-file tuning observation because commands, repetitions, immutable image, and failure inventory were not retained.
- **Prompt seed:** Given a manifest, reference/prediction records, and GPU samples, calculate the defensible accuracy and throughput findings; identify which claimed speedups the evidence cannot support.
- **Checkable outcome:** fixed numeric answers for supplied records; account for excluded/missing outputs; do not mistake independently sharded jobs for a proven scaling curve.
- **Lesson:** “we found good defaults” and “we proved a performance improvement” are different outcomes.

### Additional ideas and owner follow-up

- Ask the owner for the exact dependency failure logs and successful image inventory; these would strengthen WX-1 without requiring reconstruction from memory.
- Preserve the progression from shell orchestration to Python control logic as a migration scenario: retain behavior while improving inspectability.
- Duplicate basenames and resume-output checks suggest a separate artifact-identity problem; current tests cover representative naming but not every possible collision.
- A successful uninterrupted agent sequence is valuable if its actual transcript survives. The commit chain alone cannot tell us whether it was uninterrupted.
