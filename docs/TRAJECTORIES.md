# Trajectories

A trajectory is the complete observable record of one agent attempt at one task. The canonical representation should be lossless enough to support future evaluators and exporters without rerunning the model.

## Canonical fields

```yaml
trajectory_id: uuid
task_id: string
split: train|dev|held-out

source:
  dataset: string|null
  repository: string
  base_commit: string

model:
  provider: string
  model_id: string
  checkpoint: string|null
  quantization: string|null

agent:
  runner: string
  version: string|null
  system_prompt_hash: string|null
  config: {}

inference:
  temperature: number|null
  top_p: number|null
  seed: integer|null
  max_tokens: integer|null

execution:
  started_at: timestamp
  finished_at: timestamp|null
  wall_seconds: number|null
  environment_digest: string|null

events: []

artifacts:
  final_diff: string|null
  changed_files: []
  final_response: string|null

metrics:
  input_tokens: integer|null
  output_tokens: integer|null
  tool_calls: integer|null
  commands: integer|null

scores: {}
verdict:
  accepted: boolean|null
  policy_version: string|null
  reasons: []
```

## Event stream

Events should be ordered and timestamped. Suggested event kinds:

- `assistant_message`
- `tool_call`
- `tool_result`
- `command`
- `command_result`
- `file_read`
- `file_write`
- `patch`
- `test_result`
- `system_event`
- `final_response`

Do not assume all runners expose the same internal reasoning. Capture only what the agent/API actually emits. The schema should preserve observable actions and outputs without depending on private chain-of-thought.

## Why raw trajectories are not SFT rows

An SFT exporter may choose a subset or normalized representation of events. The raw trajectory should preserve more information than training needs, including provenance, failed commands, test output, timing, diffs, and evaluator metadata.

This separation lets the same corpus later produce:

- full successful-agent SFT conversations
- compact tool-use SFT examples
- chosen/rejected preference pairs
- verifier training examples
- process-analysis datasets

## Failed steps

A successful trajectory may contain failed intermediate actions. These are often valuable because recovery is part of real software engineering. Acceptance should be based on the whole trajectory and final outcome; exporters may later decide whether to retain or mask selected failed steps for a particular training objective.

## Privacy and licensing

Before publishing raw trajectories, sanitize secrets, credentials, private repository data, personal data, and proprietary prompts. Imported datasets and repositories must retain sufficient license/provenance metadata to determine whether redistribution and derived training artifacts are permitted.
