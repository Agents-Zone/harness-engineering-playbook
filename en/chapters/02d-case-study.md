# In Practice: AILock-Step Feature Workflow

The previous sections established the methodological framework for specification: information layering, three dimensions, cross-validation, and the iteration loop. This section uses a concrete workflow system to show how these concepts land in practice.

AILock-Step Feature Workflow is a document-driven development framework built on Claude Code, developed by a member of our community. It uses skills (slash commands) to orchestrate the entire lifecycle of a feature, from requirement intake to code delivery. The system has been used to ship over 40 features on an AI agent platform project. The full source code is publicly available on GitHub.

What follows is one implementation, not the only one. The point is not to copy its directory structure or configuration format. The point is to observe how it turns the principles described earlier into an executable workflow.

## Information Layering in Practice

The earlier discussion covered four information layers (vision, architecture, feature, task), noting that different layers evolve at different speeds, have different scopes of applicability, and should be carried by different documents with different loading strategies. AILock-Step implements this layering with three categories of documents.

**CLAUDE.md carries cross-layer mandatory constraints.** This is the file that Claude Code loads automatically on every startup. Its content consists of hard rules that rarely change across the entire project lifecycle: TypeScript strict mode is required, direct database operations are prohibited, commit message format is specified, hardcoded configuration in code is prohibited. These constraints do not belong to any single layer. They cut across all layers, and every task must follow them. Because the content is stable and always needed, it goes in the automatically loaded position.

**project-context.md carries high-level information.** This corresponds to the vision and architecture layers described earlier. Its content includes:

```
Technology Stack
  | Category | Technology | Version | Notes |
  | Frontend | React      | 18.x    | ...   |
  | Backend  | Node.js    | 20.x    | ...   |
  | Database | PostgreSQL | 15      | ...   |

Directory Structure
  src/
  ├── components/     # Shared components
  ├── pages/          # Page routes
  ├── services/       # API services
  ├── hooks/          # Custom hooks
  └── utils/          # Utility functions

Critical Rules
  Must Follow:
  - All API responses use the unified ResponseWrapper format
  - Database operations must go through the ORM layer; raw SQL is prohibited
  Must Avoid:
  - Do not call the database directly from components
  - Do not put business logic in utils/

Code Patterns
  Naming conventions, import style, error handling patterns

Testing Patterns
  Unit test location, naming rules, E2E framework

Recent Changes
  Changes and impact from the most recent features
```

This document has a hard limit: 200 lines maximum. This limit comes directly from the context constraint discussed earlier. The project context gets loaded into the Agent's context every time. If it is too long, it squeezes the space available for specs and code. 200 lines is enough to hold the key information for a medium-sized project, but it requires you to include only index-level content rather than writing full implementation details.

This document is not written once and left unchanged. Every time a feature is completed, if it introduced new tech stack components, new code patterns, or new directory structures, project-context.md gets an incremental update. A changelog at the bottom of the document records each update's content and date. This corresponds to the earlier observation that "high-level information changes occasionally."

**Each feature has its own independent set of three documents: spec.md, task.md, and checklist.md.** These correspond to the feature and task layers. These documents are loaded only when the Agent is working on that feature, and archived upon completion.

The loading strategy for this three-tier structure is clear: CLAUDE.md and project-context.md are loaded every time (they are background information all tasks need), while feature documents are loaded only when executing the corresponding feature (loading documents for unrelated features only creates noise).

## Three Dimensions in the Spec Template

AILock-Step's spec.md template translates the three dimensions (intent, acceptance, constraints) described earlier into concrete fields.

The **intent dimension** is expressed in three places. The "Requirement Description" field describes in natural language what problem is being solved. The "User Value Points" field lists the independent user values this feature contains, generated through Agent analysis. The "User Story" field uses the standard format: As a [role], I want [goal], so that [value].

The **acceptance dimension** is expressed in Gherkin scenarios. The template requires generating at least one happy-path scenario and one error scenario for each user value point. For example, a user registration feature would have two scenarios:

```gherkin
# Scenario 1: Successful registration
Given the user is on the registration page
When the user enters a valid username "testuser" and password "Test123!"
And clicks the register button
Then the account is created successfully
And a welcome message is displayed

# Scenario 2: Duplicate username
Given the user is on the registration page
And the username "testuser" already exists
When the user enters the username "testuser" and password "Test123!"
And clicks the register button
Then the error message "Username already exists" is displayed
```

These scenarios are the acceptance dimension in action: each scenario checks whether the feature's behavior aligns with its intent. The happy path verifies "it does what it should." The error path verifies "it behaves reasonably when problems occur."

The **constraint dimension** is expressed in the "Context Analysis" field, which contains three parts: reference code (existing modules in the project that can be reused), related documentation (design documents, API docs), and related historical requirements (similar features built previously). This field tells the Agent the background and boundaries of the change: what is available, what has been done before, and what should not be touched.

The template also includes a "Technical Approach" field, marked as "to be filled during development." This field is empty during spec generation and gets filled in with the concrete implementation plan when the Agent starts actual development. This corresponds to the layering principle of "lower-level information is generated only when needed."

## Cross-Validation in Practice

The three documents (spec, task list, checklist) and their respective roles directly correspond to the cross-validation described earlier.

The spec is the first interpretation: starting from the requirement, it generates intent descriptions, acceptance scenarios, and constraint analysis.

The task list is the second interpretation: starting from the spec, it breaks work down into concrete execution steps. The template organizes task items into four categories (modules/components, API endpoints, frontend pages, other), each as a checkbox. A progress tracking table at the bottom gets updated by the Agent during execution.

The checklist is the third interpretation: from "how to do it," it reasons backward to "how to know it was done correctly." The template includes five check categories. Development completion (are all tasks done, are edge cases handled). Code quality (is the style consistent, are there code smells). Testing (are unit tests written, do they pass). Documentation (is the spec's technical approach filled in, are related docs updated). Commit readiness (are changes staged, is the commit message ready).

If the checklist contains a check item the spec never mentioned, it means the Agent introduced content during the task phase that the spec did not cover. If a checklist item contradicts the spec's description, it means intent shifted during transmission. Consistency among the three documents is evidence of alignment.

## The Splitting Process in Practice

When the `/new-feature` skill determines that a requirement contains more than three user value points, the system recommends splitting. This process is fully documented in tests.

Input a requirement: "User authentication system supporting registration, login, and permission management." The Agent identifies three independent user value points: user registration (creating a new account), user login (accessing the system), and permission management (controlling access privileges). Because the user value count equals 3, a split recommendation is triggered.

The system presents the splitting plan: the original requirement is divided into three sub-features. feat-auth-register (user registration, no dependencies), feat-auth-login (user login, depends on feat-auth-register), and feat-auth-permission (permission management, depends on feat-auth-login). Dependency relationships are set automatically by the system.

After confirming the split, the system creates an independent directory for each sub-feature, with each directory containing its own spec.md, task.md, and checklist.md. It also records the three sub-features' status, priority, and dependency relationships in queue.yaml, along with a parent requirement entry tracking overall progress.

The execution engine checks whether a feature's dependencies are satisfied before starting it. If you try to start feat-auth-login while feat-auth-register is not yet complete, the system blocks and displays "dependencies not satisfied."

Each sub-feature enters the same iteration loop: Agent expansion of the spec, cross-validation, human review of the user story, and execution after convergence.

## A Full Walkthrough of One Requirement

> **TODO**: A real, completed feature is needed here as a case study. It should show: a filled-in spec (with actual user stories and Gherkin scenarios), the corresponding task list (with actual task items), the corresponding checklist (with actual check items), and a record of problems discovered and corrected during iteration. To be added once material is obtained from an actual project.

## The Human's Actual Time Investment

Throughout the entire workflow, the developer's actual time is concentrated at a single point: reviewing the user story and Gherkin scenarios after the spec is generated. Confirming that the Agent's understanding of "who this feature helps and what problem it solves" matches the developer's own intent. If it matches, the developer hands off and lets the Agent continue with task splitting, checklist generation, and code implementation.

This validates the human role described earlier: intent alignment can only be done by a human, while consistency checking can be delegated to the Agent. The developer's attention is focused on the highest-ROI checkpoint (upper-level direction), and the Agent's cross-validation covers lower-level consistency.

The full implementation source code is publicly available on GitHub ([AILock-Step Feature Workflow](https://github.com/auenger/AILock-Step/tree/feature/dev-agent-subagent-optimization/feature-workflow)). What is shown here is just one implementation approach. Readers do not need to use the same tool or the same directory structure. The key is to understand the principles behind it: layered information management, clear expression at each layer through three dimensions, cross-validation to detect inter-layer drift, and iteration until convergence. Which tools you use to implement these principles depends on your project and team.

## Chapter Summary

The essence of specification is intent alignment. Vibe Coding fails because intent lives inside conversation, and conversation is a medium that expands, contradicts itself, and discards content. The Agent's limited context and uneven attention transform intent alignment from a communication problem into an engineering problem.

The solution to this engineering problem rests on two foundations. Information has natural layers with different rates of change and scopes of applicability. You need to split them into different documents in a structured way: high-level information loaded persistently, task-level information loaded on demand. At each layer, you express the content clearly along three dimensions (intent, acceptance, constraints), where the core role of the acceptance dimension is to detect whether alignment with the layer above has been lost.

A spec is iterated into existence. You write intent, the Agent expands, cross-validation exposes problems, you correct or split, validate again, and repeat until convergence. In this loop, only a human can judge whether intent is aligned, because intent exists only in the human's head. How heavy the loop should run depends on how costly it would be to get things wrong.

Specification solves the "what to build" problem. But getting the spec right does not guarantee getting the code right. How to verify whether the code the Agent produces is actually correct is the subject of the next chapter.
