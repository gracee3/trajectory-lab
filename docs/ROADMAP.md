# Roadmap

## Milestone 0 — scaffold

- [x] Define project scope and repository layout.
- [x] Document trajectory and evaluation philosophy.
- [ ] Add machine-readable trajectory schema.
- [ ] Add machine-readable task manifest schema.
- [ ] Add one checked-in example trajectory.

## Milestone 1 — first vertical slice

Target: one reproducible Rust task end to end.

- task manifest
- clean containerized environment
- one runner adapter
- normalized event capture
- final git diff capture
- deterministic evaluator
- score/verdict artifact
- accepted/rejected split

Success criterion: the same task can be rerun from a clean state and produces an auditable trajectory plus evaluator output.

## Milestone 2 — candidate generation

- run N attempts per task
- support multiple model/agent adapters
- rank passing solutions
- rejection-sampling acceptance policy
- token, timing, and tool-use metrics

## Milestone 3 — dataset export

- Qwen-compatible SFT exporter
- preference-pair exporter
- verifier-example exporter
- dataset manifest with provenance and hashes

## Milestone 4 — broader systems task packs

- Rust repository repair
- shell / CLI tasks
- Linux service debugging
- Docker/container tasks
- networking and observability tasks
- multi-file feature work
- failure-recovery tasks

## Milestone 5 — training loop

- baseline model evaluation
- small LoRA/SFT experiment
- post-training held-out evaluation
- quantize tuned checkpoint
- post-quantization regression evaluation

## Research questions

- How many high-quality trajectories are needed before a 27B model shows measurable gains?
- Is one best trajectory per task better than several diverse passing trajectories?
- Which failed intermediate steps are useful to retain during SFT?
- How strongly should process efficiency affect selection after correctness gates pass?
- Do model-specific tool formats transfer cleanly through a normalized trajectory representation?
- Which gains survive W8A8 quantization?
- Can trajectory-derived verifiers improve best-of-N inference before additional training?
