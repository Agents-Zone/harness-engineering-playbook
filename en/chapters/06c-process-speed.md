# Making Processes Match Agent Speed

> 🚧 This section is under development.

Code review, deployment pipelines, testing processes, release approvals: these processes served as quality safeguard mechanisms at human development speed. Agent output speed has turned them into throughput bottlenecks. But they cannot be eliminated wholesale just because they have become slow. Each process originally existed for a quality reason.

The correct approach is to analyze each one individually: which processes can be accelerated through automation (for example, replacing part of manual review with lint and type checking), which processes need their trigger frequency adjusted (for example, switching from scheduled deployments to event-driven continuous deployment), and which processes have design premises that no longer hold and need to be fundamentally redesigned (for example, when the granularity of Agent-produced PRs is completely different from human-written ones, both the unit and the criteria of review need to be redefined). The goal is not to make every process faster, but to make the cost of quality assurance match Agent output speed.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
