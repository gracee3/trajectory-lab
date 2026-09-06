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
- [x] native-asr
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

## native-asr

Reviewed main at `b5b08cb` (31 visible commits), all 11 listed branches, all six returned PR records, and separate histories for bounded adjudication, tie-only adjudication, the experimental two-pass cascade, and the unmerged Tromso companion. Source diffs and acceptance notes were inspected for the areas below.

### ASR-1 — Detect silent batch failure and recover without corrupting accounting

- **Turning point:** a successful process exit could still return empty Parakeet hypotheses. The fix recursively splits failing groups, tries VAD for a singleton, then balanced PCM chunks; every attempt remains in measured runtime and the original utterance mapping is preserved.
- **Evidence:** [`44f2eaf` implementation and regression assertions](https://github.com/gracee3/native-asr/commit/44f2eafd6a6513617ead992714dab26c120b9bef), particularly `scripts/lib/batch_adapter.py` and `tests/foundation-unit`.
- **Status:** code/test-backed. The tests distinguish recovered output, unrecoverable output, and failure despite parseable partial output.
- **Prompt seed:** A batch process exits zero, but the evaluation reports pathological deletions. Determine whether this is recognition error, missing output, or orchestration failure. Repair recovery while retaining each input identity and all retry costs.
- **Checkable outcome:** no false success for empty/missing payloads under the stated fixture contract; no dropped siblings; bounded recovery; complete cost accounting.
- **Lesson:** the meaningful unit of success may be an input record, not a process.

### ASR-2 — Reconcile disagreement while keeping uncertainty visible

- **Turning point:** the deterministic ensemble turns insertions/deletions into real voting columns, applies 2-of-3 consensus, and explicitly preserves unresolved primary fallback. Word-level and coarse segment timing remain distinct.
- **Evidence:** [`fe4b675` ensemble implementation and tests](https://github.com/gracee3/native-asr/commit/fe4b6752544abe91a95785fca7fbaa4a2b24aa23); [PR #3](https://github.com/gracee3/native-asr/pull/3).
- **Status:** code/test-backed; algorithmic behavior is established by fixtures, not a universal claim that ensembles improve accuracy.
- **Prompt seed:** Given three transcripts with conflicting insertions, deletions, punctuation, and timing granularity, produce a deterministic consensus and an audit of unresolved decisions.
- **Checkable outcome:** exact expected columns and selected tokens for fixed fixtures; null/deletion votes count; missing word times are not fabricated.
- **Lesson:** disagreement is useful evidence; forcing every column into a confident answer destroys it. This can be a pure code-and-data or answer-only task.

### ASR-3 — Recover the product boundary when one workload invalidates another

- **Turning point:** the interactive cascade was accepted on short, paced phrases after broader long-utterance stress exposed mid-sentence endpointing and worse corrections. The project separated long-form processing from interactive acceptance and recorded the rejected workload.
- **Evidence:** [`9b7730b` implementation and T14 acceptance report](https://github.com/gracee3/native-asr/commit/9b7730bd585afe73d8855fb7a98e437f405de595). Experimental work also survives at [`388dcfa`, source-boundary attribution](https://github.com/gracee3/native-asr/commit/388dcfaba819b9fcf1bd92a087910b42d60a4a3e).
- **Status:** demonstrated in repository-recorded T14 aggregates: two paced 100-utterance fixtures passed the stated gates; raw local audit data is not in Git. The report explicitly notes dirty-tree provenance and identifies image/fixture hashes.
- **Prompt seed:** Review apparently contradictory latency and accuracy results from a two-stage streaming system. Explain which workloads support the interactive claim, diagnose boundary/clock errors, and define an honest acceptance contract without hiding the failed stress case.
- **Checkable outcome:** distinguish paced latency from unpaced throughput; do not generalize phrase acceptance to arbitrary long-form input; preserve provisional-to-committed ordering and correction deadlines.
- **Lesson:** stepping back from a narrow implementation problem can reveal two different products and two different validity domains.

### ASR-4 — Bound event delivery without losing terminal lifecycle evidence

- **Turning point:** the Tromso branch replaced an unbounded UI queue with a finite channel. Saturation cancels the process group and still delivers overflow plus exit. It also replaced a timing workaround for executable-busy errors with a targeted retry and a test that deliberately holds the executable open.
- **Evidence:** [`189a794` code and supervisory repair notes](https://github.com/gracee3/native-asr/commit/189a7946d8c005cec02e080970df49bd0a022011); [unmerged PR #6](https://github.com/gracee3/native-asr/pull/6).
- **Status:** demonstrated in recorded offline verification. The notes report 24 unit tests plus a PTY test at this step, repeated verification, and a separate missing-FFmpeg baseline limitation.
- **Prompt seed:** A live application grows memory when its UI lags, and occasionally loses the final exit event. Define bounded behavior and implement it without silent canonical-event loss or generic retry-on-any-error.
- **Checkable outcome:** capacity-one saturation deterministically terminates with one overflow report and an exit; unrelated spawn errors fail promptly.
- **Lesson:** bounded memory, event integrity, and process lifecycle must be solved together.

### ASR-5 — Publish an audit only when provenance and completion are valid

- **Turning point:** audited sessions gained verified model provenance before staging; successful publication is atomic and no-overwrite. Graceful cancellation cleans staging, while an abrupt kill may leave private recoverable evidence without a falsely published result.
- **Evidence:** [`f06a627` provenance and crash-path changes](https://github.com/gracee3/native-asr/commit/f06a6270372c9bb7965ff3f87f702c5acaf2a4c5); [independent closure acceptance `f2edf64`](https://github.com/gracee3/native-asr/commit/f2edf64d36f574a86d2b28f58ab5ea09fa50a528).
- **Status:** demonstrated by recorded offline checks on an unmerged branch. Closure records 131 Rust unit tests plus one PTY test and root checks; live PipeWire/model operation was not run.
- **Prompt seed:** Design or repair a journal publisher that must not expose partial results or misattribute a session to unverified models. Explain crash, cancellation, and restart states.
- **Checkable outcome:** invalid provenance creates no published destination; existing output is never overwritten; incomplete staging cannot be mistaken for a committed audit.
- **Lesson:** useful progress can be proven on a branch even when the final hardware gate and merge remain outstanding.

### Negative trajectory worth keeping — narrowed LLM adjudication still did not help

The [tie-only experiment `f16e8c7`](https://github.com/gracee3/native-asr/commit/f16e8c739b924d8ebab7a26a09a2e062d4016361) records two candidates run twice on calibration and disjoint held-out snapshots. Both were deterministic but did not improve the baseline and violated zero-fallback requirements. The long-form follow-on was correctly not run.

The useful lesson is not another attempt to constrain the adjudicator more tightly by default. Ask whether an adjudicator is necessary, what failure costs are acceptable, and whether deterministic consensus plus explicit uncertainty already serves the product better. A scenario could ask the agent to make a go/no-go decision from the published result JSON; the correct answer may be to stop.

### Additional ideas for later review

- Cache identity across paired experiments: [`8b1af11`](https://github.com/gracee3/native-asr/commit/8b1af11580fc60f87965e90f407be0f625239071) is a further lead, not yet analyzed at patch level here.
- Nonfinite confidence serialization: [`797eb65`](https://github.com/gracee3/native-asr/commit/797eb65c3216702457b551f9308125203cc2b331) could become a small contract-repair exercise.
- The companion handoff explicitly records supervisory repair and independent acceptance. Preserve it as process evidence, while avoiding reconstruction of missing Qwen/Codex conversations.
