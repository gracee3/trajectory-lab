# Project scenario backlog

Status: discovery. This is the working TODO for extracting focused, advanced agent scenarios from existing projects. No project code has been reviewed for this backlog yet, and no scenario or mock is implemented.

We will go through one project at a time: inspect its code and history, record what was difficult in the owner's words, select one issue, and define the smallest faithful environment and independent verifier.

This backlog is the current scenario-selection priority. The parser, migration, and supervisor examples in the [Experiment Plan](EXPERIMENT_PLAN.md) remain optional harness examples rather than prerequisites. Its reproducibility, capture, and independent-evaluation principles still apply.

## How we work through a project

- [ ] Confirm the repository and relevant branch or release.
- [ ] Read the entry points, dependencies, build/run configuration, tests, and relevant history.
- [ ] Capture the original symptom, misleading evidence, unsuccessful approaches, and what finally worked.
- [ ] Link the starting revision and known working solution, including relevant files and commits.
- [ ] Pick one bounded issue and explain what makes it challenging.
- [ ] Define what must remain real and what can be replaced by fixtures or adapters.
- [ ] Define the agent brief, available evidence, resources, and time budget.
- [ ] Define independent behavioral checks that accept alternative valid solutions.
- [ ] Prove that the broken starting state fails, a known valid solution passes, and plausible incomplete fixes fail.
- [ ] Mark the scenario ready for implementation only after the above is captured.

Keep owner recollections, code-confirmed findings, and proposed scenario designs separate. Unknown details stay pending. A project can eventually produce several scenarios; begin with one.

## 1. WhisperX-batch

Status: first project to discuss. Exact repository URL and revisions pending confirmation.

### What we know

The owner reports that dependency compatibility took substantial effort and that a working solution is available. The precise conflict and fix have not yet been captured here.

### What to capture together

- [ ] What was the first reproducible failure: installation, import, initialization, GPU execution, or batch behavior?
- [ ] Which errors or apparently reasonable fixes were misleading?
- [ ] Which package, runtime, driver, or configuration combination finally worked?
- [ ] Which source files and commits explain that fix?
- [ ] What is the smallest audio workload that still reveals the issue?

### Scenario design TODO

- [ ] Choose one dependency or execution failure from the actual history.
- [ ] Preserve real installation and execution paths needed to reproduce it.
- [ ] Investigate short audio fixtures and a small workload to replace the full corpus.
- [ ] Decide whether GPU access is necessary; do not mock away the compatibility problem.
- [ ] Check a clean installation and actual task output, not just successful imports.
- [ ] Record dependency provisioning, reset procedure, and estimated runtime.

Next discussion: describe one failure that took the longest to understand, and identify the repository or fix if known.

## 2. Heterogeneous Tiger Lake work

Status: candidate. Exact repository, hardware path, and difficult issue pending.

### What to capture together

- [ ] Confirm the project and relevant hardware/software components.
- [ ] Explain the intended behavior and the specific behavior that failed.
- [ ] Identify the difficult capability, backend, scheduling, compatibility, or other issue from evidence; these are questions, not established causes.
- [ ] Locate the known solution and relevant tests or measurements.

### Scenario design TODO

- [ ] Select one real issue with a bounded outcome.
- [ ] Identify a possible device/backend boundary in the existing code.
- [ ] Evaluate recorded capability responses or scripted backend failures as fixtures.
- [ ] Specify which real hardware check is still needed to validate the substitute.
- [ ] Define correctness and recovery checks independently of one preferred patch.

## 3. CPU gpt-oss work

Status: candidate. Exact repository, runtime, and issue pending.

### What to capture together

- [ ] Confirm the repository and known working configuration.
- [ ] Identify whether the difficult part involved building, loading, execution, memory, performance, or another behavior.
- [ ] Capture failure evidence, unsuccessful approaches, and the actual fix.
- [ ] Identify which model artifacts and CPU capabilities the issue depends on.

### Scenario design TODO

- [ ] Extract one issue from its real code path.
- [ ] Determine whether a smaller compatible fixture reproduces the same failure.
- [ ] Keep actual format, loader, or execution behavior real when it is the subject of the task.
- [ ] Separate simulated pipeline checks from claims about actual inference or performance.
- [ ] Define required CPU resources, reproducibility limits, and independent checks.

## 4. Prometheus / Supermicro observability project

Status: candidate. Exact repository URL and issue pending confirmation.

### What to capture together

- [ ] Confirm the repository and relevant exporter, collector, query, rule, or deployment component.
- [ ] Describe one difficult failure and the symptom visible to an operator.
- [ ] Identify the response samples, logs, configuration, and fix that explain it.
- [ ] Record expected behavior under missing, stale, malformed, or changing data where relevant.

### Scenario design TODO

- [ ] Select one fault spanning enough of the pipeline to preserve the engineering challenge.
- [ ] Consider recorded hardware responses or local fixture endpoints.
- [ ] Keep the relevant parser, metrics exposition, queries, or alert evaluation real.
- [ ] Define exact expected values and label behavior from controlled inputs.
- [ ] Determine whether hardware access adds necessary evidence or only cost.

## 5. Qwen INT8 quantization project

Status: candidate. Known project reference: [qwen38-int8-lab](https://github.com/gracee3/qwen38-int8-lab). Relevant branch, issue, and starting/fixed commits pending review.

### What to capture together

- [ ] Choose one difficult issue already solved in the project.
- [ ] Record the failure, configuration, diagnostic evidence, and verified fix.
- [ ] Identify whether it concerns recipe construction, dependencies, artifact integrity, runtime loading, kernel selection, or another boundary.
- [ ] Identify which evidence requires a real model and GPUs.

### Scenario design TODO

- [ ] Separate orchestration/validation behavior from numerical quantization behavior.
- [ ] Evaluate a small compatible model or artifact fixture for the selected issue.
- [ ] Use simulated expensive jobs only when their real numerical behavior is irrelevant to the task.
- [ ] Specify a real integration check for any hardware/kernel/numerical claim.
- [ ] Protect existing checkpoints by running scenarios on disposable inputs and outputs.
- [ ] Define resources and runtime before choosing this as an early implementation.

## Shared adapter and bootstrap investigation

This is a design backlog, not a commitment to build a generic framework first. Learn the interface from the first concrete scenario.

- [ ] Identify the existing subsystem boundary before introducing a mock.
- [ ] For Rust, assess an extracted workspace/crate and a trait with real and fixture implementations.
- [ ] For other languages, use native test fixtures or the same local service protocol where appropriate.
- [ ] Define shared prepare, start, reset, run, freeze-artifact, evaluate, and collect operations through commands and files.
- [ ] Give fixture responses explicit versions and provenance.
- [ ] Support controlled errors, partial responses, and capability changes only where the scenario needs them.
- [ ] Validate that the substitute reproduces the selected failure and does not make a superficial fix pass.
- [ ] Record which behavior the mock cannot establish.

## Attempts and continuations

A failed attempt is not automatically an unusable trajectory. Preserve separate observations about task outcome, capture quality, assistance, and eventual calibration suitability.

- [ ] Record fresh attempts with clean starting state.
- [ ] Link resumed attempts to the original session and preserve their checkpoints.
- [ ] Record hints and other human intervention explicitly as assisted continuations.
- [ ] Preserve unresolved, partially solved, timed-out, and infrastructure-failed attempts.
- [ ] Keep independent evaluator results separate from the agent's own claims.
- [ ] Avoid making independent success depend on a single run when the experiment is explicitly studying continuation.

## Calibration objective

The intended downstream use discussed here is quantization calibration, not fine-tuning. These scenarios should expose realistic technical inputs and tool exchanges across domains. Solving a task and being useful calibration material are different properties.

- [ ] Capture representative messages, commands, outputs, code, configuration, and recovery behavior.
- [ ] Record the target model/agent conversation format before creating calibration exports.
- [ ] Decide how to sample useful segments without letting repeated failure output dominate.
- [ ] Keep calibration scenarios and variants separated from held-out evaluation families.
- [ ] Compare calibration corpora under a controlled quantization recipe and token budget.
- [ ] Treat calibration export and quantization experiments as later work; this TODO implements neither.

## Next action

Start with WhisperX-batch. Confirm its repository, then pair a code/history review with the owner's description of one hard dependency problem. Update this section with evidence before designing the first fixture.
