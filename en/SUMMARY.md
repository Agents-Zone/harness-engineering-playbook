# Table of Contents

* [About This Book](README.md)
* [Introduction: Why You Need Harness Engineering](chapters/00-introduction.md)

### Part I: Reliable Agent Programming (1→10x)

* [Specification: Aligning Intent with the Agent](chapters/02a-intent-alignment.md)
  * [Intent Alignment: Why Vibe Coding Fails](chapters/02a-intent-alignment.md)
  * [Conveying Intent Through Structure: Layers and Dimensions](chapters/02b-structured-intent.md)
  * [Iterating Toward an Executable Spec](chapters/02c-iterative-spec.md)
  * [In Practice: AILock-Step Feature Workflow](chapters/02d-case-study.md)
* [Verification: Ensuring Code Stays Faithful to the Spec](chapters/03-verification.md)
  * [The Anchor of Verification Is the Spec](chapters/03-verification.md)
  * [Test Infrastructure First: Turning Specs into Executable Constraints](chapters/03a-test-first.md)
  * [Code Review: Catching Intent Drift That Tests Miss](chapters/03b-code-review.md)
  * [In Practice: AILock-Step's Verification Pipeline](chapters/03c-practice.md)
* [Evolution: Continuous Iteration of Specs and Verification](chapters/evolution-v1.md)
* [Part I Recap: From Closed Loop to Evolution](chapters/v1-conclusion.md)

### Part II: Scaling Agent Development (10→100x)

* [Letting Agents Run: Decomposition, Context, and Memory](chapters/04-long-running.md)
  * [Context Collapse: Why Long Tasks Go Off the Rails](chapters/04a-context-wall.md)
  * [Task Decomposition: Controlling Execution Chunk Granularity](chapters/04b-task-decomposition.md)
  * [Context Engineering: Deciding What the Agent Sees](chapters/04c-context-engineering.md)
  * [Cross-Session Persistence: Memory and Handoff](chapters/04d-memory.md)
* [Multi-Agent Parallelism: Isolation and Integration](chapters/05-multi-agent.md)
  * [Isolation: Preventing Concurrency Conflicts Between Agents](chapters/05a-isolation.md)
  * [Integration: Ensuring Consistency Across Independent Outputs](chapters/05b-integration.md)
  * [Platform Engineering: Building Multi-Layer Feedback Infrastructure](chapters/05c-platform.md)
* [Evolution: From Manual Inspection to Automated Drift Detection](chapters/evolution-v2.md)

### Part III: Governing the 100x Organization

* [Organizational Restructuring: When Agents Change the Premise of Collaboration](chapters/06-hybrid-team.md)
  * [Why the Old Structure Fails](chapters/06a-why-old-structure-fails.md)
  * [Bottleneck Shift: From Code to Organization](chapters/06b-bottleneck-shift.md)
  * [Making Processes Match Agent Speed](chapters/06c-process-speed.md)
* [Role Redefinition: From Writing Code to Designing Verification Systems](chapters/07-role-redefinition.md)
  * [Redesigning Roles Around Governance](chapters/07a-new-roles.md)
  * [Replacing Informal Coordination with Explicit Mechanisms](chapters/07b-coordination.md)
* [Evolution: Organizational Assets and the New Moat](chapters/evolution-v3.md)

---

* [Contributors](contributors.md)
