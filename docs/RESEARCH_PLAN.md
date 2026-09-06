# Research Plan: Codex CLI observable lifecycle

Status (2026-09-06): the [Research Report](RESEARCH_REPORT.md) delivers the source/documentation investigation, with a pinned upstream commit and explicit runtime-access limitations. No implementation or machine-readable schema changes were made. The research-only handoff below remains the scope and acceptance reference.

## Handoff objective

Research the observable execution lifecycle of Codex CLI and produce an evidence-backed report. **This handoff requests research and documentation, not implementation code.**

Read [Vision](VISION.md) and the [README](../README.md) first. Keep the investigation focused on one agent and events already exposed to the user. Do not build a recorder, change the schemas, implement an exporter or replay tool, patch Codex internals, or introduce a multi-agent framework.

## Evidence and version baseline

Record the examined Codex CLI version, upstream repository commit, relevant execution mode, platform, and relevant configuration with secrets omitted. Inspect available local source/help first and use official OpenAI documentation and upstream source to resolve gaps.

Cite exact source paths and symbols at an immutable commit where possible. Distinguish documented behavior, source findings, direct observations, and open questions. Do not claim a runtime experiment was performed if only source was inspected. If the local runtime or a needed surface is unavailable, document that limitation and continue the source-based report.

Small, benign observation experiments using existing commands are appropriate in a disposable workspace. Preserve sanitized excerpts and exact reproduction steps. Creating a research report does not require building software.

## Research questions

### 1. Where does a session start?

- What initiates a session in each relevant Codex CLI mode?
- How are session, conversation, and turn boundaries represented, if exposed?
- Which identifiers are available, and what do they actually identify?
- How do completion, cancellation, errors, exit, and resumption appear?
- Which distinctions are visible to the user, and which are only implementation details?

### 2. How are tool calls emitted?

- Where does a tool request originate in the inspected implementation, and how does it reach a user-accessible surface?
- What tool names, arguments, identifiers, and lifecycle states are exposed?
- Are records streamed, buffered, summarized, or truncated?
- How do approval requests or blocked actions appear, if exposed?
- What ordering guarantees exist when activity overlaps?

### 3. Where do tool results flow?

- How does an execution result return through the agent to the user-visible surface?
- Which result fields, output fragments, exit statuses, and errors survive that path?
- What identifiers connect calls and results?
- Can partial output be distinguished from a completed result?
- What is omitted, transformed, redacted, or unavailable?

### 4. What are the observable boundaries?

- Which existing outputs, structured streams, persisted session records, or documented interfaces are available for the examined version?
- For each candidate surface, is it supported, experimental, or an implementation detail? What access is needed?
- What differs between interactive and noninteractive operation, where applicable?
- Are interruption, resumption, context compaction, and usage statistics exposed? If so, at what level of detail?
- What can a record establish, and what must remain unknown?
- Can the surface support observation without changing Codex behavior?

These are investigation questions, not assertions that any named event or interface exists.

## Required deliverable

Create `docs/RESEARCH_REPORT.md` containing:

1. **Version and evidence baseline:** examined versions, modes, configuration, sources, and runtime-access limitations.
2. **Lifecycle map:** a concise diagram and explanation of verified boundaries and transitions. Mark unknowns explicitly.
3. **Observable event inventory:** a table with surface, native event name/type, emission boundary, exposed fields, identifiers, ordering/timing behavior, stability, evidence, and limitations.
4. **Tool-call/result walkthrough:** one sanitized trace if runtime access permits; otherwise a clearly labeled source walkthrough and unperformed reproduction procedure.
5. **Coverage and gaps:** a distinction between emitted information, internal implementation details, and unavailable information.
6. **Capture recommendation:** the smallest existing surface sufficient for a first recorder, with tradeoffs and evidence. Recommend only; do not implement.
7. **Schema considerations:** propose a minimal envelope based on confirmed fields, keeping native payloads and unavailable values explicit. Do not finalize or edit the machine-readable schemas.
8. **JSONL and replay feasibility:** explain what can be exported and replayed faithfully, plus what cannot be reconstructed.
9. **Open questions and next step:** list unresolved findings and propose one bounded implementation milestone for later review.

## Completion criteria

- Claims about Codex behavior have versioned evidence.
- Internal symbols are not presented as exposed interfaces without tracing their exposure.
- Observed traces are distinguished from examples and proposals.
- No hidden reasoning, chain-of-thought reconstruction, or unsupported claims of complete model-context capture.
- The report is understandable to a reader learning the agent lifecycle.
- The handoff ends with the report and a recommendation; implementation remains a later milestone.
