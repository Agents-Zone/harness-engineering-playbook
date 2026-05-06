# Context Collapse: Why Long Tasks Go Off the Rails

> 🚧 This section is under development.

Each time the Agent completes an iteration in the agentic loop, the iteration's input, tool calls, and return values are all appended to the context window. The token count grows continuously, but the change in output quality is not linear. Within the effective capacity range, the Agent performs reliably. Once it crosses the critical threshold, quality collapses off a cliff: it starts forgetting constraints established early on, repeats work it has already done, and even overwrites its own earlier code.

This wall is a structural characteristic of the LLM attention mechanism, not a bug that can be worked around through prompt optimization. Context compaction can delay when the wall is reached, but compaction itself introduces information loss. Details that get compacted away simply do not exist in subsequent execution. Understanding when this wall appears and why it appears is the prerequisite for every solution in this chapter. Task decomposition, context engineering, and cross-session persistence are all strategies that work under this hard constraint.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
