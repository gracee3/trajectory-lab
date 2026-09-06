# Autonomous Codex execution and capture

Future phase-2 runners create a fresh task workspace, launch one Codex CLI agent under the declared contract, capture existing exposed events, and freeze the final artifact for independent evaluation.

Validate the actual installed CLI and capture surface first. Keep stdout events, stderr, initial prompt/configuration, process termination and collector metadata distinct. Record timeouts, intervention, resume and capture gaps. Do not silently expand permissions or supply corrective prompts to preserve an apparent autonomous success.

The initial recommendation is exec JSON, subject to the [Research Report](../docs/RESEARCH_REPORT.md). App-server requires a documented choice if richer events are necessary. No runner is implemented yet; see [Experiment Plan](../docs/EXPERIMENT_PLAN.md).
