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
- [x] qwen38-int8-lab
- [x] gpt-oss-rs, including heterogeneous/Tiger Lake work
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

## qwen38-int8-lab

Reviewed main at `4494971`, the returned complete 61-commit history, all eight listed branches, all 16 PR records, and targeted implementation diffs plus architecture, candidate, evaluation, and recovery reports. The current recovery report is newer than the PR #14 description and adds RTX 3090 evidence; the narrower older description must not overwrite that result.

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
