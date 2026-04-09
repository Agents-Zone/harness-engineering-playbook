# Task Decomposition: Controlling Execution Chunk Granularity

> 🚧 This section is under development.

The core objective of task decomposition is to ensure each execution block completes before the context window collapses. Decomposition operates along two dimensions. The spatial dimension splits tasks along module boundaries and API contracts, keeping each block's scope clear and manageable. The temporal dimension uses frequent session resets to prevent excessive token accumulation within a single session.

Choosing the right granularity is the key tradeoff. If a task block is too large, the context has already degraded by the second half of execution, making the output unreliable. If a task block is too small, the coordination overhead between blocks (handoff, alignment, redundant context establishment) becomes the dominant cost. The optimal granularity depends on the specific project's module coupling and context complexity. There is no universal formula, but there is a judgment framework. This framework builds on the information layering from Chapter 2: the amount of context each task block requires determines its reasonable size.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
