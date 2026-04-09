# Table of Contents

* [Introduction: From Vibe Coding to Harness Engineering](chapters/00-introduction.md)

### Part I: Reliable Agent Programming (1.5x → 5-10x)

* [Specification: Aligning Intent with the Agent](chapters/02-specification.md)
  * [Intent Alignment: Why Vibe Coding Fails](chapters/02a-intent-alignment.md)
  * [Conveying Intent Through Structure: Layers and Dimensions](chapters/02b-structured-intent.md)
  * [Iterating Toward an Executable Spec](chapters/02c-iterative-spec.md)
  * [In Practice: AILock-Step Feature Workflow](chapters/02d-case-study.md)
* [Verification: Ensuring Code Stays Faithful to the Spec](chapters/03-verification.md)
  * [Test Infrastructure First: Turning Specs into Executable Constraints](chapters/03a-test-first.md)
  * [Code Review: Catching Intent Drift That Tests Miss](chapters/03b-code-review.md)
  * [In Practice: AILock-Step's Verification Pipeline](chapters/03c-practice.md)
* [Systems Need Iteration Too: Evolving Specs and Verification](chapters/evolution-v1.md)
* [Part I Recap: From Closed Loop to Evolution](chapters/v1-conclusion.md)

### Part II: Scaling Agent Development (5-10x → 100x)

* [Letting Agents Run: Decomposition, Context, and Memory](chapters/04-long-running.md)
  * [What Happens After You Let Go: The Context Wall](chapters/04a-context-wall.md)
  * [Cutting Big Tasks into Agent-Sized Chunks: Task Decomposition](chapters/04b-task-decomposition.md)
  * [More Context Is Not Always Better: Context Engineering](chapters/04c-context-engineering.md)
  * [Session Over, Where Did the Knowledge Go: Cross-Session Memory Engineering](chapters/04d-memory.md)
  * [When a Task Doesn't Finish: Task-Level Handoff Documents](chapters/task-handoff/04d1-task-handoff.md)
  * [A Session Is Not a Chat Log: Engineering the Execution Unit](chapters/04e-session.md)
  * [From Hand-Holding to Letting Go: The Shift in Execution Mode](chapters/04f-letting-go.md)
* [Multi-Agent Parallelism: Isolation and Integration](chapters/05-multi-agent.md)
  * [Two Agents at Once: Conflicts and Isolation](chapters/05a-isolation.md)
  * [Each One Correct, Together They Explode: Contracts and Integration](chapters/05b-integration.md)
  * [Build the Runway Before Launching the Planes: Platform Engineering First](chapters/05c-platform.md)
    * [Millisecond Feedback: Static Analysis and Code Standards](chapters/05c1-linters.md)
    * [Seconds-to-Minutes Feedback: CI/CD as a Feedback Channel](chapters/05c2-cicd.md)
    * [Minutes-to-Days Feedback: Observability](chapters/05c3-observability.md)
    * [Measuring Business Quality: Benchmark-Driven Feedback Loops](chapters/05c4-benchmark.md)
    * [Environment as Code: Reproducibility as the Foundation of Reliable Feedback](chapters/05c5-iac.md)
    * [When the Signal Turns Red: Agent Troubleshooting Capability](chapters/05c6-troubleshooting.md)
  * [How Many Can You Manage: Span of Control](chapters/05d-span.md)
* [When Nobody's Watching: Automated Drift Detection](chapters/evolution-v2.md)

### Part III: Governing the 100x Organization

* [Multi-Person Collaboration: Reshaping Team Structure and Processes](chapters/06-hybrid-team.md)
  * [Why Your Team Structure No Longer Works](chapters/06a-why-old-structure-fails.md)
  * [Bottleneck Shift: From Code to Organization](chapters/06b-bottleneck-shift.md)
  * [Making Processes Match Agent Speed](chapters/06d-process-speed.md)
  * [Define Boundaries Before Deploying Agents: Conway's Law Still Applies](chapters/06e-conway.md)
  * [No Silver Bullet, but There Are Principles](chapters/06f-principles.md)
* [After You Stop Writing Code: Redefining the Engineer's Role](chapters/role-redefinition.md)
  * [Redesigning Roles Around Governance, Not Execution](chapters/06c-new-roles.md)
* [Organizational Assets for the New Era](chapters/evolution-v3.md)
  * [Organizational Assets for the New Era](chapters/07-beyond.md)

---

* [Contributors](contributors.md)
