# Isolation: Preventing Concurrency Conflicts Between Agents

> 🚧 This section is under development.

Two Agents working in parallel on the same project is essentially concurrent modification of shared state. Shared files, database schemas, configuration files, and dependency declarations are all potential conflict points. One Agent modifies the schema and adds new fields while another is still writing queries using the old field names. The later commit overwrites the earlier one, or the two sides' modifications conflict at the file level.

Isolation strategies range from lightweight to heavyweight across multiple levels: git worktree gives each Agent an independent working directory, branching strategies manage merge cadence, and module-level boundaries limit each Agent's scope of modification. Trust boundaries defined in Skill definitions (which files can be read, which can be modified) provide the finest-grained isolation control. Which level to choose depends on the degree of coupling between Agents: completely independent modules only need branching, while modules that share core data models may require stricter coordination.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
