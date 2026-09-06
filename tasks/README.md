# Tasks

Task definitions describe what an agent must solve and how the task environment is reconstructed.

Planned task families:

- `swe/` — imported SWE-style repository tasks
- `rust/` — Rust compiler, test, refactor, and feature tasks
- `linux/` — Linux/system administration tasks
- `agentic/` — long-horizon and tool-use behavior tasks

Each task should eventually declare an immutable repository/base commit, user instruction, environment image or build recipe, validation commands, provenance/license metadata, and dataset split.

Held-out tasks must never be reused for training trajectory generation.
