# Project scenario catalog

Review started 2026-09-06. Scope: the eight repositories in the [profile's Selected work](https://github.com/gracee3/gracee3/blob/HEAD/README.md), including their accessible development branches and historical work. This replaces the narrower adapter-first discovery checklist.

This pass covers the profile's eight Selected work repositories, not every repository on the account. Histories were inventoried and the relevant diffs and evidence were selected for deeper reading. Coverage is stated per repository; it does not include every deleted branch, unreachable commit or private local artifact.

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
- [x] qwen38-int8-lab
- [x] gpt-oss-rs, including heterogeneous/Tiger Lake work
- [x] supermicro-observability
- [x] digital-liquid-light-lab
- [x] Mirabile
- [x] Magnolia

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

Reviewed main at `b5b08cb` (32 visible commits), all 11 listed branches, all five returned PR records (PRs #2–#6), and separate histories for bounded adjudication, tie-only adjudication, the experimental two-pass cascade, and the unmerged Tromso companion. Source diffs and acceptance notes were inspected for the areas below.

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

## qwen38-int8-lab

Reviewed main at `4494971`, the returned complete 63-commit history, all eight listed branches, all 16 PR records, and targeted implementation diffs plus architecture, candidate, evaluation, and recovery reports. The current recovery report is newer than the PR #14 description and adds RTX 3090 evidence; the narrower older description must not overwrite that result.

### QINT-1 — Separate the transformation target from the graph execution boundary

- **Turning point:** treating every Linear as a sequential subgraph triggered `KeyError: forward`. Keeping Linear modules as quantization targets while using decoder layers as tracing/onloading boundaries resolved the failure. Architecture inspection also exposed 1-centered RMSNorm, GDN layers, and MTP tensors omitted by the instantiated model class.
- **Evidence:** [implementation `5bfa399`](https://github.com/gracee3/qwen38-int8-lab/commit/5bfa399f561fcd935e8af883df1c58d8afe97610); [architecture policy](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/reports/architecture-policy.md); [attempt ledger](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/reports/evaluation-and-agent-status-2026-08-29.md).
- **Status:** demonstrated by the recorded synthetic smoke and real load/trace gate: 64 decoder targets, 65 sequential subgraphs, and 256 intended quantized modules.
- **Prompt seed:** Given a module inventory, tracing failure, and transformation API, identify the correct unit for transformation versus scheduling. Produce a coverage policy and explain which architecture-sensitive paths need separate evidence.
- **Checkable outcome:** correct target counts and exclusions; no unsupported assumption that every Linear or normalization layer behaves alike.
- **Lesson:** changing the abstraction boundary can solve a failure that repeated parameter tuning cannot.

### QINT-2 — Make an apparently complete artifact independently loadable

- **Turning point:** the quantized shard/index checkpoint still needed a read-only view supplying two missing processor files. The serializer was fixed to copy and hash them inside staging before atomic publication, alongside preserved MTP tensors.
- **Evidence:** [`8589f78` serializer/preflight changes](https://github.com/gracee3/qwen38-int8-lab/commit/8589f786020a338e67ac5144e03d4c70582bfa30); [successful tiny/small/quality report](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/reports/quality-candidate-2026-08-25.md); [PR #8 negative test](https://github.com/gracee3/qwen38-int8-lab/pull/8).
- **Status:** demonstrated in recorded direct-loading gates on the final self-contained artifact. Functional loading does not establish standardized model accuracy.
- **Prompt seed:** A model loads only when its source directory is also available. Audit the output manifest and repair publication so the deliverable is complete on its own.
- **Checkable outcome:** required metadata, processor files, shard references, and preserved tensors are present and checked before publication; invalid/missing files prevent promotion.
- **Lesson:** serialization success is not deliverable completeness. The same task applies to installers, offline appliances, and signed release bundles.

### QINT-3 — Diagnose an evaluation workload that exceeds a working chat runtime

- **Turning point:** prompt-render auditing caught a 12,314-token request before inference. Later log-likelihood runs still OOMed; explicit KV allocation, text-only activation, and bounded chunked prefill allowed the maximum-request runtime gate to pass without altering the immutable checkpoint.
- **Evidence:** [`b15f181` runtime gate and configuration diff](https://github.com/gracee3/qwen38-int8-lab/commit/b15f1812e01b65c9ec3e7943aae7bb853ce494c8); [request/runtime attempt ledger](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/reports/evaluation-and-agent-status-2026-08-29.md); [PR #10](https://github.com/gracee3/qwen38-int8-lab/pull/10).
- **Status:** demonstrated engineering gate: 154,531 rendered requests with zero truncation and a complete maximum-request log-probability check. The standardized suite was paused; it does not provide a completed accuracy score.
- **Prompt seed:** A server passes chat smoke tests but fails evaluation. Use request lengths, allocation logs, and runtime settings to design a representative preflight and repair resource allocation without shortening the benchmark silently.
- **Checkable outcome:** maximum workload remains intact; log-probabilities are complete; partial evaluation is not promoted into a final score; candidate-only evidence cannot claim BF16 retention.
- **Lesson:** smoke tests must exercise the operation that actually exhausts resources.

### QINT-4 — Identify the supervisor as the source of a misleading failure

- **Turning point:** repeated late-stage `KeyboardInterrupt` failures were caused by the swap watchdog, not an unexplained operator interrupt. The patch records effective limits, prints the reason immediately, and raises `ResourceSafetyError`. A host wrapper coordinates temporary settings and restoration.
- **Evidence:** [`18554c4` fix and tests](https://github.com/gracee3/qwen38-int8-lab/commit/18554c453e61aa1a9b18557db19b1ae3e709d634); [PR #13](https://github.com/gracee3/qwen38-int8-lab/pull/13).
- **Status:** code/test-backed with reported tests for sustained triggering and restoration on normal failure/handled signals. The revised 32 GiB ceiling was an operational choice, not a proven full-run requirement.
- **Prompt seed:** An expensive job repeatedly stops near completion with a misleading exception. Correlate worker and supervisor evidence, identify the actual termination cause, and make it diagnosable without disabling protection.
- **Checkable outcome:** distinguish operator cancellation, resource abort, and worker fault; retain RAM constraints; restore temporary settings where the signal model permits.
- **Lesson:** expand the investigation beyond the failing function to the system that controls it.

### QINT-5 — Resume expensive stateful computation with exact equivalence

- **Turning point:** rolling snapshots preserve model/qparameter state, cached calibration batches, RNG state, and identity. Publication replaces the current pointer before deleting the prior generation; checksums and writer locking reject unsafe resumes.
- **Evidence:** [`5094844` implementation and manifest tests](https://github.com/gracee3/qwen38-int8-lab/commit/5094844644c4786daef8e23735739ebfb250f350); [RTX 3090 verification `5970f39`](https://github.com/gracee3/qwen38-int8-lab/commit/5970f391deddf04dffdab110a8065743d90170a6); [recovery report](https://github.com/gracee3/qwen38-int8-lab/blob/44949714ff6dde6db866532f8600129af689361e/reports/resumable-quant-2026-09-06.md).
- **Status:** demonstrated for small Qwen synthetic models, including BF16/CPU offload and actual RTX 3090 interruption/resume. W8A8 matched all 109 tensors; W4A16 matched 127. Full 27B I/O and power-loss recovery remain unvalidated.
- **Prompt seed:** Determine the minimum complete recovery state for a staged numerical pipeline. Repair interrupted-stage and final-export recovery while rejecting a changed recipe, corrupted snapshot, or second writer.
- **Checkable outcome:** fresh-process resumption matches uninterrupted output exactly on the supplied small computation; incomplete publication preserves the previous valid generation.
- **Lesson:** a recoverable job needs more than saved weights or a stage number.

### Additional ideas and unsuccessful paths

- Long-context comparison is another evidence-analysis task: [PR #12](https://github.com/gracee3/qwen38-int8-lab/pull/12) reports TP2 faster and with more balanced memory than PP2, plus a 150K retrieval result. That is a measured narrow result, not proof of all long-context quality.
- [Unmerged single-GPU fix, PR #16](https://github.com/gracee3/qwen38-int8-lab/pull/16) distinguishes a sequential quantizer's one-GPU requirement from the serving topology's TP2 requirement. Useful prompt: find an inherited preflight assumption that blocks a valid configuration.
- The preserved serialization interruption is valuable evidence of a correctly enforced guard and an unfinished artifact; do not relabel it as a corrupt completed checkpoint.
- Dataset-parser corrections for JSON-string messages and differing source schemas are leads for a later calibration-data integrity scenario.
- Keep the broader decision visible: whether another full quant or another large evaluation is justified by the evidence and available resources.

## gpt-oss-rs — CPU, Tiger Lake, and archived heterogeneous research

Reviewed main at `8b3ff46`, the latest 200 reachable commits, all 151 listed branch names/heads across two pages, all 13 PR records, targeted historical code diffs, Tiger Lake closure, final research report, and the HET retrospective/gate. This is a systematic selection of turning points, not a claim that every commit on every historical branch was read. Earlier probe/replay and multi-GPU branches were checked as distinct archived leads.

### GPT-1 — Repair blocking initialization at an asynchronous runtime boundary

- **Turning point:** Harmony encoding initialization created a blocking HTTP client whose helper-runtime lifecycle could panic inside Tokio. Initialization moved before async serving, with a process-lifetime OnceLock and a dedicated-thread lazy path for library callers.
- **Evidence:** [`b9eea9c` server/tokenizer diff](https://github.com/gracee3/gpt-oss-rs/commit/b9eea9c6ef70ecbb159bf053d18b183a70d525c6), especially `crates/gpt-oss-tokenizer/src/protocol.rs` and server `main.rs`; [PR #1](https://github.com/gracee3/gpt-oss-rs/pull/1).
- **Status:** code/test-backed with recorded closure of eight Harmony/Tokio failures and the restored non-GPT replay path.
- **Prompt seed:** A library initializes correctly in a standalone command but panics when called from an async service. Trace initialization and destruction across dependency/runtime boundaries, then make startup and lazy use safe.
- **Checkable outcome:** one shared initialization, no nested-runtime-drop panic, clear initialization errors, and preserved protocol behavior.
- **Lesson:** the failing stack frame may be a dependency's cleanup, not the service's main operation.

### GPT-2 — Preserve numerical semantics while changing packed layouts and SIMD kernels

- **Turning point:** compact MXFP4 execution gained explicit layouts and scalar/AVX2/AVX-512 paths. An E8M0 correction fixed special scale encodings that a simple exponent-bit shift mishandled.
- **Evidence:** [special-value fix `849785e`](https://github.com/gracee3/gpt-oss-rs/commit/849785e04c6df993cd3740688d787d330da78b14); [AVX-512 x8 diff `eb92640`](https://github.com/gracee3/gpt-oss-rs/commit/eb92640a82177be2e7ccc75617f5c6a686942c1b); [final exactness evidence](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/docs/research/FINAL_REPORT.md).
- **Status:** demonstrated within published shape/kernel checks; full-model greedy-token equality is a separate, weaker claim than identical logits.
- **Prompt seed:** Given a compact numeric format, scalar reference, optimized implementation, and edge-case mismatch, identify the semantic error and retain the numerical contract across layouts and supported instruction paths.
- **Checkable outcome:** correct zero/special scale semantics; exact reference agreement on prescribed cases; reject forced unavailable ISA; distinguish kernel correctness from model quality.
- **Lesson:** ordinary values can conceal a format error. A small deterministic numerical question may capture the turning point without a model runtime.

### GPT-3 — Make model and sequence updates transactional

- **Turning point:** immutable model resources, mutable sequence state, and execution scratch became separate owners. A prepared step stages K/V rows and tokens, then validates revisions and all target sequences before commit.
- **Evidence:** [`94a01a6` transactional CPU model diff](https://github.com/gracee3/gpt-oss-rs/commit/94a01a6097d830dcbd081983851016395b4ab457); [sampling/lifecycle follow-on `2247df9`](https://github.com/gracee3/gpt-oss-rs/commit/2247df9f35090cffb29762479871c454a79821d3); [research report](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/docs/research/FINAL_REPORT.md).
- **Status:** code/test-backed for the inspected model-state change; publication records bounded lifecycle/scheduling verification. No general multi-user production-readiness claim.
- **Prompt seed:** A cancelled or failed computation leaves cached state partially advanced. Define prepare/execute/commit semantics so a retry sees the same committed state, including stale revisions and duplicate sequence references.
- **Checkable outcome:** dropped/failed prepared work leaves state unchanged; all preconditions are checked before mutation; committed positions and cache contents agree.
- **Lesson:** atomicity applies to in-memory numerical state, not only databases.

### GPT-4 — Coordinate heterogeneous work with proven completion before reuse

- **Turning point:** H7 ran a real 20B continuation under static CPU/GPU0/GPU1 expert ownership, with bounded relay storage, deterministic route-rank reduction, and visibility-last commit. A recoverable post-enqueue fault drained siblings and retried; unproven drain quarantined resources and rejected reuse.
- **Evidence:** [`a9ab97a` implementation](https://github.com/gracee3/gpt-oss-rs/commit/a9ab97aef349e7f05b79dd6a1aa6eed1853dd7b4); [immutable H7 gate and linked JSON records](https://github.com/gracee3/gpt-oss-rs/blob/a9ab97aef349e7f05b79dd6a1aa6eed1853dd7b4/docs/het/evidence/implementation-2026-08/h7/README.md).
- **Status:** demonstrated historical success, retained in the archive although the published CPU tree excludes the HET runtime. Exact eight-token continuation passed twice; no controlled HET speedup was established.
- **Prompt seed:** Several devices contribute to one result and a remote operation fails after a sibling has started. Decide what can be committed, retried, released, or quarantined from supplied completion evidence.
- **Checkable outcome:** no partial visible state; no early buffer reuse; canonical reduction order; safe retry only after a proven drain; uncertain completion cannot become success.
- **Lesson:** the broad reusable problem is distributed ownership and transactional publication, not just GPU kernel plumbing.

### GPT-5 — Turn local optimization into a defensible dispatch decision

- **Turning point:** Tiger Lake's candidate matrix region won in isolation but regressed full requests, so automatic promotion was rejected. Xe residency also delivered a large repeated-projection improvement yet zero cache hits on measured full-model prefill; explicit integration remained valid while automatic acceleration stayed disabled.
- **Evidence:** [Tiger Lake closure](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/docs/TIGER_LAKE_CLOSURE.md); [PR #5](https://github.com/gracee3/gpt-oss-rs/pull/5); [explicit Xe integration `6caf274`](https://github.com/gracee3/gpt-oss-rs/commit/6caf27423744148dafb1fb2670a03f29452311f3).
- **Status:** demonstrated correctness/integration and demonstrated negative promotion result. Closure records 42/42 official generated-token comparisons and 22 live OpenCL tests. These do not erase the negative full-request performance result.
- **Prompt seed:** Given operator benchmarks, workload traces, cache hit/miss records, thermals, and full-request pairs, decide whether to promote an optimization. Explain why an impressive local result can fail at application level.
- **Checkable outcome:** reject unsupported automatic regions; account for actual reuse and conversion/upload costs; distinguish decode gains from prompt/full-request latency.
- **Lesson:** the right scenario answer can be “keep the implementation available, but do not choose it automatically.” The later Xeon publication has its own hardware/workload-specific decisions and must not be mixed with Tiger Lake evidence.

### Rabbit hole to preserve — the capacity constraint changed the question

The [HET retrospective](https://github.com/gracee3/gpt-oss-rs/blob/8b3ff46e25c213104219db8e9d390bc05dacf8bf/docs/research/HETEROGENEOUS_RETROSPECTIVE.md) separates proven H7 20B execution from incomplete H8 120B construction and the later R4 comparison. R4 eventually rejected a 13,761,300,984-byte native shard because its frozen mapping window was 10,544,040,680 bytes. Improving release ordering did not make that shard fit.

A useful evidence-only scenario asks the agent to step back: is this still a lifetime bug, an admission-policy mismatch, or a need for a different ingestion strategy? The answer must preserve the successful ownership work while recognizing that the current experiment cannot establish the larger objective. Do not quietly relax a resource contract merely to obtain a passing run.

### Additional ideas for later review

- The archived [YaRN RoPE change `bd49d35`](https://github.com/gracee3/gpt-oss-rs/commit/bd49d35432f9d532337dbcb39ed4c6a1f4f4776a) wires scaling parameters through configuration and runner/probe paths. Its code is a promising lead; this review does not claim it closed every historical parity divergence.
- [Scoped shard transactions, PR #11](https://github.com/gracee3/gpt-oss-rs/pull/11) provide a smaller source/synthetic ownership exercise with nine transaction tests.
- The 151 branch names reveal extensive numerical localization and source-attribution work. A future prompt could ask which new observation would discriminate competing causes, instead of requesting another narrowly scoped patch.
- Fresh oracle and converted-GGUF identity are further reproducibility scenarios: identical model names are not identical evidence artifacts.
- Retain successful H7 work even though it was excluded from the release tree; archival disposition is not failure.

## supermicro-observability

Reviewed main at `a1d239f`, all 13 main-history commits, all five listed branches, and all five PR records. Inspected configuration, installation and observation diffs, GPU sampler code/tests, and the dated deployment methodology. Hardware observations below belong to that recorded deployment.

### OBS-1 — Separate fresh measurements from a process that is merely alive

- **Turning point:** the Rust GPU exporter uses one persistent 250 ms sampler, retains latest values and rolling summaries, and exposes sample age and sampler health separately. Reversed GPU output order and changed indices must not attach measurements to the wrong device.
- **Evidence:** [exporter implementation and eight tests](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/gpu-exporter/src/main.rs); [generalization diff](https://github.com/gracee3/supermicro-observability/commit/997c31263dd71b355374697e587db44c9ca4ae71). Tests cover missing values, malformed rows, GPU identity changes, stale data and bounded restart backoff.
- **Status:** code/test-backed; [PR #1](https://github.com/gracee3/supermicro-observability/pull/1) reports eight Rust tests and a live check with two fresh GPU series.
- **Prompt seed:** supply a timestamped two-device stream containing reordered rows, unavailable fields and a stalled producer. Ask which values remain usable, which health signals must change, and how to preserve device identity across recovery.
- **Checkable outcome:** unavailable values are not zero; retained values carry age; stale sampling is unhealthy even if HTTP works; device reassignment does not merge histories; retry delay remains bounded.
- **Broader lesson:** successful transport does not establish current evidence. This can become an evidence-classification prompt without a running GPU.

### OBS-2 — Turn a machine-specific deployment into a reusable system without losing its constraints

- **Turning point:** checked-in host assumptions became explicit optional profiles and private generated configuration. Storage references resolve to stable host-local devices, while a neutral container path exposes only the selected SMART device. Protected devices stay excluded; monitoring consumes cached fan metrics without owning fan control.
- **Evidence:** [configuration refactor `997c312`](https://github.com/gracee3/supermicro-observability/commit/997c31263dd71b355374697e587db44c9ca4ae71); [PR #1 migration report](https://github.com/gracee3/supermicro-observability/pull/1). The report records six healthy containers, one allowed SMART device, the protected disk still read-only/unmounted and absent from metrics, and an uninterrupted fan service.
- **Status:** demonstrated for the reported deployment, with configuration and exporter tests. It does not establish portability of the original cooling calibration.
- **Prompt seed:** supply a redacted device topology, collector configuration and two deployment profiles. Ask for a migration plan that preserves the existing invariants while removing hard-coded identities.
- **Checkable outcome:** identify which device each path resolves to; select only the allowed collector targets; preserve optional features as opt-in; keep fan control external; avoid treating an empty exclusion set as a pattern matching every device.
- **Broader lesson:** generalization means separating mechanisms, local identities and policy. Merely substituting variable names is insufficient.

### OBS-3 — Diagnose a false health failure during an operational handoff

- **Turning point:** the local-first workflow separated checkout operation from optional system installation and preserved configuration, credentials and data. Live verification then found that Grafana listened on the selected private address while its health probe still queried loopback.
- **Evidence:** [`3ffe6c4` implementation](https://github.com/gracee3/supermicro-observability/commit/3ffe6c4c5136436c09b187024f0e55b78937cc64), particularly the Compose health-check address and lifecycle ownership guards; [PR #3](https://github.com/gracee3/supermicro-observability/pull/3) includes the subsequent live correction.
- **Status:** the bind/probe correction and checkout handoff are demonstrated by the historical live report. Optional system promotion has implementation/static validation evidence; the initial validation explicitly did not perform a system installation.
- **Prompt seed:** present a service reachable by the user but marked unhealthy, its bind settings, health probe and two possible lifecycle owners. Ask for the smallest justified correction and a state-preserving handoff plan.
- **Checkable outcome:** align probe and actual listener; avoid broadening the listener to hide the error; prevent competing stack owners; preserve monitoring history and credentials; distinguish uninstall from explicit data removal.
- **Broader lesson:** deployment failures can be disagreements between configuration layers, even when the underlying service is working.

### OBS-4 — Make an agent observation interface bounded and semantically honest

- **Turning point:** a single Python core added fixed-profile JSON observations and a STDIO interface, with limits on query windows, points, response bytes and labels. Disabled optional data differs from missing enabled data; absent private salt suppresses GPU dimensions.
- **Evidence:** [observation interface `8207c5c`](https://github.com/gracee3/supermicro-observability/commit/8207c5cc275f5e05d17734b52b918652851c083a); [observation tests](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/tests/test_observation.py); [PR #4](https://github.com/gracee3/supermicro-observability/pull/4) reports 27 Python tests and one successful bounded live snapshot.
- **Status:** demonstrated at those reported scopes. The live snapshot output was discarded, so it is not a retained replay fixture.
- **Prompt seed:** provide synthetic query responses with non-finite numbers, missing optional metrics, excess series, raw identifiers and malformed replies. Ask for the correct normalized observation and error classification.
- **Checkable outcome:** obey the 24-hour window, 600 query-point and 120 returned-point per-series limits, 10,000 total returned points and 4 MiB response bound; retain only permitted dimensions; produce deterministic summaries; keep protocol stdout clean.
- **Broader lesson:** a useful agent interface includes uncertainty and resource limits in its meaning. A prompt with expected JSON could test this independently of an adapter.

### OBS-5 — Evaluate the observer's cost without overstating the experiment

- **Turning point:** the project defined a monitoring budget and moved machine-specific observations into a dated case study with explicit reproduction requirements and limitations.
- **Evidence:** [methodology](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/docs/METHODOLOGY.md); [deployment report](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/docs/deployments/x11spa-tf-dual-rtx3090.md), separated from generic claims in `997c312`.
- **Status:** demonstrated as one deployment-budget observation. The five-minute run recorded 1.885% of one logical CPU for the fast exporter, 8.498% for the stack and 285.8 MiB peak aggregate container memory. This is not a comparative benchmark.
- **Prompt seed:** supply those measurements, acceptance thresholds and a tempting claim that the monitoring stack is universally negligible. Ask whether the local budget passed, what the CPU denominator means and what further evidence a general claim requires.
- **Checkable outcome:** correctly compare the local values with the 2%, 10% and 1 GiB thresholds; avoid dividing by or multiplying across cores incorrectly; refuse to derive confidence intervals or long-term disk growth from insufficient samples.
- **Broader lesson:** determining the strongest supported conclusion is itself a solved engineering task, even when the answer contains no code.

### Additional ideas and unresolved boundaries

- Dashboard iteration in [PR #5](https://github.com/gracee3/supermicro-observability/pull/5) offers a separate visual-reasoning family: mixed units need meaningful axes, and hidden legend labels can still require identity in the hover data. Review actual rendered panels before calling a visual fix proven.
- Cached fan samples provide a data-only exercise: an active controller flag does not establish fresh or safe cooling. Use sample timestamps and separate health fields from the [fan contract](https://github.com/gracee3/supermicro-observability/blob/a1d239f53b1e2eece262ab611ea4f46bf5f9e1c0/docs/FAN-METRICS.md).
- The public/private-address validator deserves boundary review before reuse: the current test accepts a documentation-range address. Define the intended address policy explicitly instead of treating a library's broad “private” classification as a complete specification.
- No inspected artifact establishes a failed fan-control trajectory here. Keep cooling-controller hypotheses separate from this repository's monitoring-only evidence.

## digital-liquid-light-lab

Reviewed all four listed branches, both unmerged PRs, the four-commit bootstrap history, the five-commit interactive history, and all 70 commits reachable from the listed CPU research branch at `c530049`. Main at `729bb99` contains only the initial README. The meaningful implementation and accepted research gates are therefore largely outside main.

### DLL-1 — Keep simulation time deterministic when presentation stalls

- **Turning point:** validated simulation contracts introduced a bounded fixed-step accumulator; the desktop pipeline then connected those contracts to continuous native GPU presentation.
- **Evidence:** [contract implementation `7467ff3`](https://github.com/gracee3/digital-liquid-light-lab/commit/7467ff359024922e4a1738a65bc41543106d603a); [native pipeline `66e91b4`](https://github.com/gracee3/digital-liquid-light-lab/commit/66e91b46146e13d7194d2bb577f903859e64288d); [clock and regression](https://github.com/gracee3/digital-liquid-light-lab/blob/66e91b46146e13d7194d2bb577f903859e64288d/crates/liquid-light-core/src/lib.rs).
- **Status:** code/test-backed scheduling; [PR #1](https://github.com/gracee3/digital-liquid-light-lab/pull/1) separately reports five frames presented on Intel Iris Xe/Vulkan. This proves a native pipeline, not fluid physics or performance.
- **Prompt seed:** give irregular presentation intervals, a fixed simulation step and a catch-up cap. Ask for the simulation advances, interpolation remainder, discarded backlog and replay implications.
- **Checkable outcome:** the existing 35 ms / 10 ms / two-step case yields two advances, 10 ms discarded and interpolation 0.5. Extend with pause, reset and multiple-frame cases; do not let presentation timing silently change the physical step.
- **Broader lesson:** a live visual system needs explicit overload semantics. The arithmetic exercise is independently useful before any rendering harness.

### DLL-2 — Preserve the conservation certificate through the actual numerical representation

- **Turning point:** the HDA-003 review showed why a mathematically equivalent rate form was not an adequate floating-point authority. The corrected contract keeps certified stored cell-volume increments and one shared integrated value per face all the way into continuity assembly.
- **Evidence:** [integrated-continuity correction `c8d32e1`](https://github.com/gracee3/digital-liquid-light-lab/commit/c8d32e1b77ec38b599ce5e3cf90c76dda5490f3a); [Review E closure](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/reviews/review-e-hda003-integrated-continuity-closure.md); later [Review N production-object mutation checks](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/reviews/review-n-h-wp4-review-m-correction-closure.md).
- **Status:** the specification finding was closed, then implementation evidence demonstrated that a prohibited divide/remultiply mutation fails native-bit regressions at two time steps.
- **Prompt seed:** provide a small closed mesh, certified increments and two algebraically equivalent assembly procedures. Ask which preserves the stated certificate, why the other can fail and which numerical diagnostics remain distinct.
- **Checkable outcome:** each face is formed once and scattered with opposite signs; the predeclared dependent-cell increment closes the stored-volume certificate; continuity does not reconstruct increments from rounded rates. Distinguish that structural identity from a later numerical sum of materialized rows and from nonlinear convergence.
- **Broader lesson:** the “same equation” can have different executable contracts. This supports a mathematical explanation or tiny numerical counterexample, not necessarily a full solver task.

### DLL-3 — Prevent a diagnostic computation from impersonating authoritative evidence

- **Turning point:** a card-backed finite-difference wrapper still accepted an arbitrary residual callback, then labeled its result authoritative. The final correction sealed the path to the actual Schedule R residual and prepared state.
- **Evidence:** [small authority-boundary diff `3f66c29`](https://github.com/gracee3/digital-liquid-light-lab/commit/3f66c29199ca32e52adb394c3692a1fe85404d39); [accepted WP1/WP2 closure review](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/reviews/review-i-h-wp1-wp2-correction-closure-2.md).
- **Status:** demonstrated by the recorded closure review, test passes and independent mutation checks. Generic callback-based differentiation remains valid as a diagnostic, with a different authority label.
- **Prompt seed:** show a provenance-rich result produced by an API that permits callers to replace its evaluator. Ask whether its evidence label is justified and how to close the gap without eliminating useful diagnostics.
- **Checkable outcome:** authoritative construction binds the resolved card, geometry, controls and actual residual internally; callers cannot substitute the measured computation while retaining the stronger label.
- **Broader lesson:** metadata cannot prove that the claimed operation happened. This generalizes to benchmark wrappers, signing pipelines and evaluation reports.

### DLL-4 — Repair an experiment whose comparisons do not measure the same thing

- **Turning point:** WP3 calibration had to compare accepted free energy across genuine solved tolerance, time-step and grid probes. Earlier logic mixed a state-distance quantity with an energy quantity and did not establish the final solved grid comparison.
- **Evidence:** [calibration and failure-evidence correction `201fec4`](https://github.com/gracee3/digital-liquid-light-lab/commit/201fec4780f03151406976469d74ea234fc8b436); [Review L accepted closure](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/reviews/review-l-h-wp3-final-correction-closure.md).
- **Status:** demonstrated for the frozen serial dense fixed-gap CPU oracle. Recorded accepted-energy differences are zero for the two tightest tolerances, approximately 0.000472487 for the two time steps and 0.001442001 for the solved coarse/fine grids. These are numerical-solver calibration results, distinct from the repository owner's downstream quantization-calibration interest.
- **Prompt seed:** provide a convincing-looking calibration report with mismatched diagnostics and a nominal grid probe. Ask whether the tolerance floor is supported, then specify the minimum corrected comparison.
- **Checkable outcome:** compare the same observable and physical domain; use consistent cell averages under refinement; actually solve both probes; retain derived inputs and accepted states. Replacing the fine-grid result with the coarse one must invalidate the claimed separation.
- **Broader lesson:** the evaluator can be wrong even when every computation succeeds. Recovering the experimental question is a valuable trajectory.

### DLL-5 — Validate the motion between endpoints and use independent numerical anchors

- **Turning point:** WP4 Review M corrections bound a smooth rest-to-rest temporal law, its interior rate extrema, a specific spatial construction and the certified gap increment. They also normalized work gates and added a three-cell independent energy derivative check that exposes an interpolation-adjoint sign error.
- **Evidence:** [moving-gap correction `a919680`](https://github.com/gracee3/digital-liquid-light-lab/commit/a91968057082732e9a4e57e127629b2f63cb1a79); [Review N](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/reviews/review-n-h-wp4-review-m-correction-closure.md); [tracked correction capsule](https://github.com/gracee3/digital-liquid-light-lab/blob/c5300491883993a66234f15e848ec5506adcdead/docs/research/evidence/h/wp4/moving-gap-v2-review-m-corrections/README.md).
- **Status:** demonstrated for the bounded moving-gap CPU oracle: the review records 17 focused tests, three discriminating mutations, 25 tracked artifact hashes and historical reproduction of 30 generator-owned artifacts. The full ignored raw bundle is not all available in Git.
- **Prompt seed:** give identical valid endpoints with two possible intervening motions, plus endpoint-subtracted and certified increments. Ask which histories are admitted, which values quadrature must consume and whether a supplied test would catch the wrong derivative.
- **Checkable outcome:** reject hidden jumps and unsupported spatial shapes; check analytic interior rate maxima; reuse the once-certified increment rather than reconstructing it by endpoint subtraction; make normalized gates invariant under consistent unit changes; use a nontrivial independent derivative anchor.
- **Broader lesson:** valid endpoints do not prove a valid path, and a symmetric tiny fixture may miss a real coupling error. These can be split into several prompt families later.

### Additional ideas, failed gates and limits

- The review/correction loops themselves are valuable: preserve the original finding, attempted closure, remaining blocker and final discriminating evidence. The initial failure of a gate does not invalidate the successful research trajectory that followed.
- WP3 also contains a strong atomicity scenario: evidence persistence must succeed before an accepted checkpoint is released; envelope, solve and write failures retain the old-state identity. The final review explicitly checks those failures.
- Canonical line endings and hash rebinding appear in `690bf61` and `2741753`; unsigned evidence-bit preservation appears in `eb26252`. These are leads for portable evidence serialization, not independently proven claims in this review.
- [Interactive PR #2](https://github.com/gracee3/digital-liquid-light-lab/pull/2) implements a kinematic plate preview but explicitly leaves RTX presentation, TrackPoint behavior and transport acceptance unverified. Keep “implemented preview” separate from “hardware path accepted.”
- That PR mentions later Candidate H/Review Q history not present in the listed research branch inspected here. Locate the actual later commits or evidence bundle before adding WP5 outcomes. The accessible branch establishes acceptance through WP4 only.
- A useful prompt-only lesson is to classify claims: presenting frames, passing CPU invariants, matching physical behavior and winning a benchmark are different achievements.

## Mirabile

Reviewed main at `a407a82`, the complete returned main history, all 11 listed branch tips, all four PR records, and the additional commit histories for the cockpit, professional wheel and unmerged live-workflow branches. Earlier `astra-*` paths are the same project's pre-rename history.

### MIR-1 — Deliver one asynchronous result to multiple observers without consuming it twice

- **Turning point:** concurrent application observers could both enter a single-consumer Worker inbox. The runtime correction serializes receives and rechecks the application version, pending work and in-flight requests after acquiring the gate.
- **Evidence:** [runtime correctness diff `fdf2612`](https://github.com/gracee3/mirabile/commit/fdf2612a4a3c80e90d79d7afcf4af0c1c762db10). Its regression starts two projection waiters, confirms one receive call, injects one result and checks that both observe the same newer version. Existing stale-success and stale-failure tests are retained.
- **Status:** code/test-backed at the original fix; later [PR #4](https://github.com/gracee3/mirabile/pull/4) reports native stale-result and atomic one-slot failure regressions plus real Worker browser validation.
- **Prompt seed:** give an event trace with two subscribers, overlapping calculation requests and out-of-order completions. Ask why a waiter hangs or an old result replaces a newer view, then specify the correct state transitions.
- **Checkable outcome:** one receiver drives a given inbox; queued observers recheck state before receiving again; stale successes and failures cannot replace current results; an incomplete two-slot calculation does not publish half a new wheel.
- **Broader lesson:** result transport, observer notification and publication authority are separate responsibilities. This is useful as an event-trace diagnosis task.

### MIR-2 — Save a composite edit atomically while preserving the user's conflicted draft

- **Turning point:** saved chart editing spans a factual record and a calculation definition. The repository gained atomic save batches with compare-only dependencies, conflict collection and application-owned draft recovery.
- **Evidence:** [atomic editing `1f6106d`](https://github.com/gracee3/mirabile/commit/1f6106d74d9e87db89d421341ff09a078bac65e1), including memory/IndexedDB implementations and regressions for two-component conflicts, compare-only failure, cancellation and shared records; [cockpit PR #2](https://github.com/gracee3/mirabile/pull/2).
- **Status:** demonstrated in the recorded native and browser conflict/reload gates. This review has not rerun IndexedDB.
- **Prompt seed:** provide two open editors sharing a factual record, current revisions and competing writes. Ask which transaction may commit, which revisions should change and what remains visible after a conflict.
- **Checkable outcome:** compare all dependencies before publishing; commit both changed components or neither; a definition-only edit checks but does not gratuitously revise its factual record; retain local edits and a usable Cancel path; require explicit copy/detach semantics before changing shared facts.
- **Broader lesson:** “save failed” is not a complete interaction contract. Correctness includes preserving user work and the meaning of shared data.

### MIR-3 — Replay structured actions after transient editor identities have changed

- **Turning point:** nested macros stopped depending on runtime draft-item IDs. Version-1 structural selectors resolve resources, list items and query paths against the current read model and report topology mismatches.
- **Evidence:** [structural replay `e6b8182`](https://github.com/gracee3/mirabile/commit/e6b8182e13b01d62382d33639c718437fcc946a1); the diff includes nested replay and topology-failure browser scenarios. [PR #2](https://github.com/gracee3/mirabile/pull/2) records those scenarios passing.
- **Status:** demonstrated at the reported cockpit gate, with explicit backward compatibility for existing version-1 macros.
- **Prompt seed:** supply a recorded nested edit and a reopened document whose transient IDs differ. Ask how to resolve its intended target and when replay must stop because the structure no longer matches.
- **Checkable outcome:** serialize meaningful selectors rather than transient IDs; validate the assumptions of key/ordinal/path selectors; fail visibly instead of editing a plausible wrong row; keep unparsable form text separate from committed typed data.
- **Broader lesson:** replay depends on stable meaning, not on preserving a particular session's incidental identifiers.

### MIR-4 — Remove circular-layout collisions without falsifying the underlying coordinates

- **Turning point:** the professional wheel separated true astronomical anchors from displaced labels and leaders. A later biwheel correction expanded collision handling beyond point labels to angle, zodiac and house landmarks.
- **Evidence:** [professional wheel PR #3](https://github.com/gracee3/mirabile/pull/3); [landmark collision fix `b8893e5`](https://github.com/gracee3/mirabile/commit/b8893e5574eb59234d3a02b2c055664ce5352b95); [final product verification record](https://github.com/gracee3/mirabile/blob/b8893e5574eb59234d3a02b2c055664ce5352b95/docs/goals/charts-wheel-settings-progress.md).
- **Status:** demonstrated for the reported fixtures and viewports. PR #3 records four viewport journeys; PR #4 records three product viewports with no measured point/point or point/landmark overlaps. This is not a proof for every possible dense chart.
- **Prompt seed:** provide a cluster straddling the circular seam, fixed semantic anchors, landmark bounding boxes and viewport bounds. Ask for a deterministic readable placement or a diagnosis of a misleading existing placement.
- **Checkable outcome:** preserve true angular positions and semantic aspect identity; move labels with leaders; account for all relevant label classes; retain stable ordering and accessible descriptions; check actual geometry at compact bounds.
- **Broader lesson:** improving presentation must not silently alter the data it presents. The same problem appears in maps, scientific plots and network diagrams.

### MIR-5 — Capture time and working state so a saved workspace reloads exactly

- **Turning point:** the live-workflow branch introduced captured working charts and explicit time actions. Stepping a saved chart creates an independent variation; workspace persistence captures its exact facts rather than reevaluating “now” on reload.
- **Evidence:** [working-chart foundation `af15f4b`](https://github.com/gracee3/mirabile/commit/af15f4b37f2f0216fa30271af8b9253e1b9a8ca9); [time-step implementation and boundary tests](https://github.com/gracee3/mirabile/blob/b8893e5574eb59234d3a02b2c055664ce5352b95/crates/mirabile-core/src/time_step.rs); [unmerged PR #4](https://github.com/gracee3/mirabile/pull/4).
- **Status:** demonstrated in the reported 215-step product journey at three viewports, using a fixed clock, real application/Worker/XALEN and exact reload comparisons. Merge is still pending.
- **Prompt seed:** supply a saved chart, a captured current-time chart and a sequence of calendar/duration steps, edits, swaps, saves and reloads. Ask for the final per-slot facts, library contents and expected calculation identities.
- **Checkable outcome:** saved originals remain unchanged by working variations; unfinished editor buffers cannot be silently captured; captured time stays fixed on reload; UTC/fixed offsets survive. Under this contract, Jan 31, 2025 plus one month twice becomes Mar 28, not Mar 31: calendar clamping is independent per action, so month steps need not round-trip at boundaries.
- **Broader lesson:** persistence must distinguish facts, live instructions and unfinished edits. A detailed prompt with an exact expected state can exercise this without a UI.

### Additional ideas and the browser rabbit hole

- Screenshot retries in `6470efe` and `854b2b2` were followed by restored standard capture, an explicit blocked-gate record and infrastructure corrections. [The later correction](https://github.com/gracee3/mirabile/commit/e111dc9bc6b8eec3fb2228ab1e8205eec7e3c63e) and final progress record identify full temporary storage, configurable shared-memory use and isolated profiles. A scenario could ask the agent to diagnose the environment from evidence before adding more retries or changing application behavior.
- The final product gate passed; earlier screenshot failures should not be described as a final application failure. Preserve intermediate gate status and later recovery separately.
- The original runtime fix also stopped the deterministic backend from claiming unsupported coordinate systems, corrections and house systems. A capability/provenance audit can be a prompt-only candidate: advertised semantics must match the implementation actually used.
- `c1ac17c` adds time-conversion fingerprinting. Inspect the full cache-key transition before isolating a stale-cache scenario.
- Known-answer checks reach owned live browser output, using independent JPL/Swiss references with circular angle comparisons. This suggests a general evaluator task: an isolated provider test cannot establish that the displayed result came from the intended computation.
- Raw logs and screenshots cited by the PRs remain under ignored local output paths. Public commits and verification records support the historical claims, but extracting a complete visual replay will require those artifacts or a fresh run later.

## Magnolia

Reviewed main at `b42316f`, all 191 main-history commits returned across two pages, all 11 listed branch tips and all nine current PR records. Also inspected the separate native-ASR and old `worktree-1` histories, the rearchitecture audit, and focused implementation/repair diffs. Older history includes several names and architectures; deleted code remains useful evidence.

### MAG-1 — Recover a reproducible headless baseline from accidental workspace coupling

- **Turning point:** a clean checkout could not load without a sibling TensorRT repository, while the nominal core pulled rendering/GPU dependencies. Explicit workspace membership and opt-in visual resources restored a CUDA-free baseline.
- **Evidence:** [baseline repair `92c4280`](https://github.com/gracee3/magnolia/commit/92c4280035f930297d5e232160d83bab5cc81c98); [feature-boundary fix `facb371`](https://github.com/gracee3/magnolia/commit/facb371ca9b250522ec4444201f3f967e4244285); [PR #1](https://github.com/gracee3/magnolia/pull/1) records metadata, headless checks, tests, Clippy and workspace checks.
- **Status:** demonstrated for that historical baseline, despite later replacement of the architecture.
- **Prompt seed:** give a workspace manifest, feature graph and clean-checkout metadata failure. Ask why selecting a headless package still requires an unrelated checkout and how to establish the intended dependency boundary.
- **Checkable outcome:** metadata resolves without sibling repositories; explicit member/default-member sets match the intended build; headless core excludes wgpu, Nannou and the UI crate; opting into rendering restores its required dependencies.
- **Broader lesson:** a useful repair may simplify the build graph instead of fixing each dependency error individually. Deletion of the old implementation does not erase the solved lesson.

### MAG-2 — Make overload and reclamation safe on the real-time callback path

- **Turning point:** native audio hardening replaced callback-reachable publication/recycling panics with bounded fault outcomes and held-block recovery. A second graph activation waits when the retired graph cannot yet be reclaimed off the callback thread. Later work adapts negotiated formats and quanta into preallocated blocks.
- **Evidence:** [callback-boundary repair `f90e768`](https://github.com/gracee3/magnolia/commit/f90e7681c6ee7890eb7e03482253dc645e6002b4); [negotiated-audio adaptation `8de98c9`](https://github.com/gracee3/magnolia/commit/8de98c92c3ca23dd87b34d21d506cfc43d39f075); [Phase 4 live acceptance, PR #8](https://github.com/gracee3/magnolia/pull/8).
- **Status:** foundation regressions are code/test-backed; the later recorded 1,800-second live tier demonstrates 84,463 callbacks with zero observed callback allocations/deallocations, faults or drops in the tested 48 kHz stereo configuration. It is not universal device certification.
- **Prompt seed:** supply a two-queue block-pool trace, a full retirement queue, two pending graph changes and a non-default input quantum. Ask which actions are legal in the callback and how loss/recovery should be represented.
- **Checkable outcome:** defer activation rather than destroy the old graph on the callback; preserve ownership when a queue operation fails; count and report discontinuities; maintain conversion state across buffer boundaries; allocate and reclaim outside the callback.
- **Broader lesson:** bounded storage alone does not make a real-time path safe. Ownership and exceptional paths matter as much as the normal path.

### MAG-3 — Keep authoritative control and final events intact under lossy telemetry overload

- **Turning point:** the replacement shell separates control/projection traffic from binary telemetry and gives each stream a delivery policy. Meter/partial updates can replace older values; waveform/spectrum/diagnostics use bounded dropping; final transcript entries remain ordered application-owned records.
- **Evidence:** [Phase 2 transport and overload record, PR #5](https://github.com/gracee3/magnolia/pull/5); [browser lifecycle/overload tests `a9bcc5f`](https://github.com/gracee3/magnolia/commit/a9bcc5ff41f43cd0e94e79e842c4c26c39d3a825); [earlier audit](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/docs/rearchitecture/audit-2026-08-28.md).
- **Status:** demonstrated structurally with synthetic telemetry, real transport and browser journeys. A 2,000-frame burst still permits a control receipt/projection; twenty reloads retain runtime identity and transcript cursor. Those are not native-ASR or latency-benchmark results.
- **Prompt seed:** present mixed partials, finals, waveform frames and control receipts entering overloaded queues. Ask which may be replaced, which must persist and what a reconnecting client needs to recover.
- **Checkable outcome:** bound telemetry memory; carry sequence/loss/discontinuity metadata; preserve final ordering and cursor access; isolate control progress; release hidden leases without recreating the native runtime on browser reload.
- **Broader lesson:** “drop old data” is a semantic decision. The old audit found that the STT source label did not match its partial-event overflow classifier; retaining only the newest event was not proof that finals survived.

### MAG-4 — Reject a stale lease completion without releasing the currently desired resource

- **Turning point:** a long audio soak passed, but the subsequent browser transitions exposed a lease race. An unrelated reactive rerun invalidated an in-flight subscription; its stale completion could release the analyzer that the current view still wanted.
- **Evidence:** [focused repair `501da3e`](https://github.com/gracee3/magnolia/commit/501da3e887fd2cc3f18eb45902b197998ec5bcea); [Phase 5 rejected-run and repair record](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/docs/rearchitecture/phase-5-observation.md); [final acceptance PR #9](https://github.com/gracee3/magnolia/pull/9).
- **Status:** demonstrated failure-to-fix sequence. The rejected candidate had good core soak counters; the repaired final `20de8cf` passed both complete gates, including ten rapid workspace transitions and close/reopen.
- **Prompt seed:** give subscription generations, visibility changes and out-of-order success/error completions. Ask which completion may update status, which may release the resource and which must do nothing.
- **Checkable outcome:** advance generations at actual lease/visibility boundaries; stale errors cannot clear a current observer; stale success cannot release a still-desired lease; hidden/destroyed views eventually release resources.
- **Broader lesson:** a successful long soak can miss a short lifecycle race. Preserve the good subsystem evidence and the failed overall gate, then design the missing transition test.

### MAG-5 — Publish a recoverable recording and replay more than the raw samples

- **Turning point:** Phase 5 introduced an explicit bounded storage worker, incomplete staging directories, versioned bundles and replay clocks. The recording includes PCM, timeline, semantic controls, analyzer and telemetry evidence rather than only an audio file.
- **Evidence:** [recording/replay implementation `2b2dc92`](https://github.com/gracee3/magnolia/commit/2b2dc92a91c335b809db08ca970a0b4728cf118a); [recording source and tests](https://github.com/gracee3/magnolia/blob/b42316fa3b1f5fd30a387cd98a982ca61bc5ec74/crates/magnolia-observe/src/recording.rs); [PR #9](https://github.com/gracee3/magnolia/pull/9).
- **Status:** demonstrated by the reported seeded recording/replay, atomic-finalization, incomplete-recovery and corruption-rejection gates. Physical microphone samples were analyzed in memory; the recording evidence used generated seeded PCM.
- **Prompt seed:** supply a partially written bundle, manifest, chunk sizes and event timeline. Ask whether it may be recovered, which checks precede publication and what must stay invariant under real-time versus accelerated or deterministic replay.
- **Checkable outcome:** validate JSON, PCM frame boundaries and hashes; flush/sync before atomic publication and sync the parent directory; reject corrupted or still-incomplete bundles; preserve event ordering and compare PCM/timeline/control/analyzer/telemetry identities independently of playback speed.
- **Broader lesson:** replay is an evidence contract. Matching raw input bytes alone does not establish that the same controls, timeline or observations were reproduced.

### Additional ideas, older rabbit holes and partial success

- The old Parakeet history contains timeout, CUDA synchronization, slot-reuse and stop-statistics investigations, and `worktree-1` adds caption throttling. These commits establish attempted interventions, not successful TensorRT execution. Recover retained run artifacts or the owner's explanation before labeling an exact old GPU fault solved.
- The rearchitecture audit identified overlapping module APIs and claims that old lifecycle tests did not prove. A broad architecture-diagnosis scenario could ask which contracts are genuinely established and which require a new boundary; it need not ask for another plugin framework.
- The unmerged native-ASR branch at [`7da122c`](https://github.com/gracee3/magnolia/commit/7da122c2ac9c56557112862f96b2193742f09862) corrects an initial two-artifact provenance blocker: the native-library digest became established, but the model digest remained missing under that project's stated gate. Its [model-free foundation](https://github.com/gracee3/magnolia/commit/95f44b2db0de3b513fbcba16b065112381d189e1) implements partial revisions, immutable finals, gap/reset reduction, cancellation and durable final journaling. Those are extractable code/test leads; live model execution and WER/RTF acceptance were not achieved.
- That provenance stop is a useful evidence-review prompt: distinguish a reproducibility checksum from the specific trusted-origin evidence required by a supplied project contract. Do not silently redefine the contract to make the gate pass.
- Exact-device versus follow-default selection, source disappearance/recovery and cumulative counters have live evidence in PR #8. This could become a separate device-state scenario later.
- The Phase 5 document records the repaired implementation run; PR #9 records the subsequent final documentation-inclusive commit run. Keep their slightly different callback counts and timings attached to their own revisions rather than merging them into one experiment.

## Cross-project ideas for later review

The catalog now contains 40 primary areas, five per project. The next step is to review the turning points with the owner and choose which ones deserve full scenario prompts. A small scenario can retain a difficult judgment even when its inputs are a short log, a graph, a manifest or a few numerical values.

- **Diagnose from a frozen evidence packet:** combine the style of WX-1, QINT-3, QINT-4 and OBS-3. Supply logs from several layers and ask for the root cause, disconfirming evidence and smallest justified next action.
- **Decide whether an experiment established its claim:** use the native-ASR adjudication result, GPT-5, OBS-5 or DLL-4. The correct answer may be acceptance within a limited scope, another discriminating experiment, or a no-go decision.
- **Find a counterexample:** derive small numerical or state examples from GPT-2, DLL-2, DLL-5 and MIR-5. Require an exact answer or independent calculation, not a large implementation.
- **Reduce an event history:** MIR-1, MIR-3, MAG-3 and MAG-4 can become deterministic prompts asking which state, events and resources remain valid after reordered completions, retries or reconnects.
- **Audit a deliverable or recovery plan:** WX-4, ASR-5, QINT-2, QINT-5 and MAG-5 provide manifests, staged files and crash points with objectively checkable publication decisions.
- **Evaluate the evaluator:** DLL-3/DLL-4, Mirabile's browser references and Magnolia's failed post-soak lifecycle gate show how a passing check can miss the actual claim. Ask what the test proves and what additional observation would distinguish the alternatives.
- **Preserve meaning across representation:** transcript consensus, numerical packing, circular label placement and telemetry summaries differ in domain, but all require an explicit account of information lost, retained or transformed.

These are scenario families to consider, not a shared adapter design. Each may become a detailed prompt, a prompt with data, a small extracted repair, or a later hybrid harness.

## Review and extraction TODO

- [ ] For each project, ask the owner which documented moment was actually difficult and what changed their understanding. Add missing constraints or the decisive observation alongside the Git evidence.
- [ ] Mark which primary areas to keep, split or defer. Leave additional leads visible; do not turn every interesting commit into a scenario.
- [ ] For each selected area, preserve the starting revision, solved revision and relevant intermediate failed attempts. Identify any required logs, fixtures or screenshots that currently exist only outside Git.
- [ ] Write the full prompt with explicit supplied evidence, task outcome, permitted assistance and resource budget. Keep solution-bearing diffs and hidden checks out of the agent's starting materials.
- [ ] Define the answer key in terms of observable correctness, including justified alternative solutions and valid “insufficient evidence” or no-go answers.
- [ ] Decide separately whether the task needs execution. Use an answer-only or evidence-analysis task when that captures the turning point faithfully.
- [ ] Run a small pilot only after the prompt and answer key are reviewable. Record unsuccessful attempts, successful continuations, hints and evaluator mistakes rather than collapsing them into one pass/fail label.
- [ ] Compare captured trajectories with the scenario's actual evidence needs before considering any quantization-calibration export.

An attempt that does not solve the task in one run is not automatically a bad trajectory. It may reveal a useful diagnosis, a legitimate blocker, a flawed task or evaluator, or a successful recovery after new evidence. Git can help identify those moments; complete execution logs are needed to judge the actual agent run. Neither this review nor a future replay should infer hidden reasoning.
