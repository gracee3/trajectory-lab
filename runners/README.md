# Runners

Runners adapt a concrete model/agent implementation into the lab's normalized trajectory event stream.

A runner should:

- launch against a clean task environment
- preserve the native transcript when available
- emit observable assistant/tool/command/file events
- record model and inference configuration
- capture timing/token/resource metadata when available
- leave the final workspace intact for evaluators

The normalized interface should not depend on private chain-of-thought. Only observable messages, tool activity, outputs, patches, and final responses belong in the canonical trajectory.
