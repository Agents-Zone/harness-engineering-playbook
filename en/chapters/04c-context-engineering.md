# Context Engineering: Deciding What the Agent Sees

> 🚧 This section is under development.

Task decomposition answers "how large should each block be." Context engineering answers "what goes into each block's context window." Intuitively, more information seems better: let the Agent see the full picture so it can make correct decisions. In practice, the opposite is true. When the context is filled with irrelevant documentation, example code, and historical discussions, critical constraints get drowned out by noise. The Agent starts making changes in unrelated places, treats documentation examples as real code, and reopens discussions on decisions that were already settled. Information overload is more dangerous than insufficient information.

The principle of context engineering is layered design, flat execution. During the design phase, organize information hierarchically (project-level, module-level, task-level). During the execution phase, inject only the layer the current task needs into the context window. Specific forms of context pollution that need to be managed include: outdated design documents, module information unrelated to the current task, redundant examples and explanations, uncleaned historical conversations, repeated constraint declarations, and the Agent's own intermediate reasoning output.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
