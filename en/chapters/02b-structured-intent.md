# Communicating Intent with Structure: Layers and Dimensions

## Documenting Intent

The previous section diagnosed three Vibe Coding failure modes: early instructions crowded out of the attention window, contradictory instructions with no priority ordering, and compaction actively deleting information. All three problems share a common structural cause: intent lives inside the conversation.

Moving intent into documents alleviates all three problems simultaneously.

When you change your mind, you no longer need to append a new instruction to the conversation and hope the Agent correctly determines which of the old and new instructions takes priority. You edit the relevant line in the document. The current version of the document holds exactly one position, with no contradictions to reconcile.

Documents exist independently of the conversation. They do not compete with chat messages for context space. Your conversation can explore ideas, debug problems, and try experiments freely. The finalized intent settles in the document. When the Agent needs to know your intent, it reads the document. It does not need to extract and infer from dozens of conversation rounds.

Documents persist in the file system. When conversation gets compressed by compaction, the document is unaffected. The Agent can re-read the complete original content at any time. An architectural constraint you spent twenty minutes explaining early in a conversation will not vanish due to compaction, as long as it was written into the document.

Many developers notice an immediate quality improvement in Agent output after switching from pure conversation to maintaining a spec document. The Agent stops dropping constraints, because the constraints live in a document that gets loaded every time. This direction is correct.

But natural language documents introduce a new problem: ambiguity. You write a description and think the meaning is clear. The Agent also thinks the meaning is clear. But you may be understanding different things. You will not discover this gap until you see the code the Agent generates.

The solution to ambiguity is not to write more natural language. A three-page description may contain more ambiguity than a three-line one, because more sentences mean more places that can be interpreted differently. What actually works is getting the structure right. To understand what the right structure looks like, you first need to understand the nature of the information itself.

## Four Layers of Information and On-Demand Loading

Think about how human teams collaborate.

A five-person development team is building a blog system. Programmer A is working on the article editor. Programmer B is working on the search feature. A does not know what field names B is using for the search index. B does not know which rich text library A's editor uses. Yet when their code comes together, search finds articles published through the editor, and content saved by the editor gets correctly indexed by search.

How does this work? A and B share enough high-level context. They both know the product is for non-technical users, so interactions should be simple. They both know the system uses a separated frontend and backend, REST APIs, and a PostgreSQL database. They both know articles are stored in a `posts` table with `title`, `body`, and `published_at` fields. They do not need to know every detail of each other's code, because these shared high-level decisions already constrain their respective implementation directions.

What happens without this high-level alignment? A might define one article data structure and B another, and the two are incompatible. A might store article content as HTML while B's search engine expects plain text. Each person's code runs fine alone, but nothing works when combined.

This observation reveals a fundamental fact about information: it has natural layers, and different layers have entirely different properties.

Using the blog system as an example, you can identify at least four layers from top to bottom.

The top layer is the product vision. "Enable non-technical users to easily publish and manage content." This statement is highly abstract and contains almost no implementation details. But it rarely changes. It may remain constant across the entire product lifecycle. Its scope of applicability is the broadest: every team member needs to know it, and every decision should be consistent with it.

The next layer is architectural decisions. "Separated frontend and backend, REST APIs, PostgreSQL database, React frontend." These decisions are more concrete than the vision but are still system-level. They change occasionally (for example, migrating from REST to GraphQL), but far less frequently than specific features. Their scope of applicability spans the entire development team: everyone must follow them.

The layer below that is the feature spec. "Article search feature: supports keyword full-text search, results sorted by relevance, shows a message when there are no results." This is much more concrete than architectural decisions, and it changes frequently as the product iterates. This month search covers only titles. Next month it might include full-text. Its scope is narrower too. Only the people working on the search feature need to know all its details.

The bottom layer is task details. "In the `buildQuery` function in `search.ts`, call PostgreSQL's tsvector full-text search and return the top 20 results sorted by relevance in descending order." This is the most specific information. It might change based on a single code review comment. Its scope is the narrowest: it is only needed while executing this one task.

The pattern across all four layers is clear: from top to bottom, information becomes more specific, changes more frequently, and is needed by fewer people.

This layering is not an organizational preference. It reflects intrinsic properties of the information. Vision is abstract because it describes a goal rather than a path, and many paths can lead to the same goal. Task details are concrete because they describe a single step on a single path, and must be precise enough to execute. These two types of information have inherently different lifecycles, change frequencies, and scopes of applicability. Managing them in a single undifferentiated blob is unreasonable.

This layering connects directly to the Agent's context limitations.

If you stuff all four layers into the Agent's context, most of the information is noise relative to the current task. When the Agent is working on the search feature, it does not need the comment system's spec or the user authentication module's task details. That information takes up context space and draws away attention, but contributes nothing to alignment for the current task.

The sensible approach is to load by layer. High-level information (vision, architectural decisions) is stable and broadly applicable, so load it every time. Low-level information (the current feature's spec, the current task's details) gets loaded only when needed, and can be removed from context once the work is done. This is why you need to split information across different documents rather than maintaining one large file that contains everything.

Ryan's practice is a concrete embodiment of this principle. His project context (`project-context.md`) carries high-level information: the tech stack, architecture, directory structure, module index, and code patterns. This document is capped at 200 lines, and the Agent loads it at the start of every session. Each feature has its own independent spec, task list, and checklist, loaded only when the Agent is working on that feature. He also uses `CLAUDE.md` to store cross-layer mandatory constraints (coding standards, rules against direct database operations, and similar). Three categories of documents correspond to different layers and different loading strategies.

To determine which layer a piece of information belongs to, look at two indicators: change frequency and scope of applicability. If a piece of information stays essentially unchanged throughout the project lifecycle and all tasks need it, it belongs at a high layer and should go in the project context that gets loaded every time. If a piece of information is relevant only to a specific feature and becomes obsolete once that feature is done, it belongs at the task layer and should go in that feature's spec.

Two signals help you tell whether your layering is right. If you find yourself repeating the same information in every feature spec (for example, reminding the Agent "we use REST APIs, do not use GraphQL" every single time), that information should be promoted to the project context layer. Write it once. Conversely, if your project context has ballooned to over a thousand lines and costs the Agent a large chunk of context space every time it loads, there are too many low-level details that should be pushed down into specific feature specs.

## Intent, Acceptance, Constraints

Layering answers "why information needs to be split" and "which layer each piece belongs to." The next question is: what content should each layer contain?

Survey the mainstream spec frameworks in the community and you will find that their template designs vary considerably. BMAD's spec has over a dozen fields. OpenSpec is relatively lean. Spec Kit has its own structure. AILock-Step uses yet another format. But if you look past differences in field names and organizational approach and observe what questions they actually ask you to answer, a consistent pattern emerges: at every layer, you need to describe information along three dimensions.

**Intent: what is this layer's purpose?**

The intent dimension answers "what to do" and "why." Its content differs at each layer, but its function is the same: to establish direction.

At the vision layer, intent is the fundamental problem the product solves. "Enable non-technical users to easily publish and manage content." At the architecture layer, intent is why the system is organized this way. "Separated frontend and backend, because the product needs to support both web and mobile, and sharing one API reduces duplicate development." At the feature layer, intent is typically expressed as a user story. "As a blog author, I want to search published articles by keyword so I can quickly find content I wrote before." This sentence contains three pieces of information: who (blog author), what (keyword search of published articles), and why (quickly find old content). Missing any one of them can send the Agent's understanding off course. Without "who," it cannot distinguish between author-facing search and reader-facing search, and the requirements are completely different. Without "why," it does not know whether search should prioritize precision or speed.

User stories come from the product management discipline, and this is not the place to cover their full methodology. They have a special value in Agent-assisted development: requirements described in terms of user value are inherently verifiable. "Can the user search articles by keyword?" has a clear yes/no answer. This property becomes very important when we discuss hands-on practices later.

**Acceptance: how do you know this layer is correct?**

The acceptance dimension is the most critical of the three. At every layer it checks one core question: does this layer's content faithfully implement the intent of the layer above?

At the vision layer, acceptance checks whether user stories and user journeys reflect the vision. If the vision says "let non-technical users easily publish content" but your user journey includes a step requiring the user to manually configure a Markdown rendering engine, the journey has drifted from the vision. At the architecture layer, acceptance checks whether technical decisions can support the requirements in the PRD. If the PRD requires real-time collaborative editing but the architecture uses a framework that does not support WebSocket, there is a gap between architecture and requirements.

At the feature layer, acceptance is typically expressed as Given/When/Then scenarios. For example: Given the database contains an article with the title "Performance Optimization," When the user enters "Performance" in the search box and clicks the search button, Then the search results display this article and the matching keyword is highlighted.

A single feature usually needs multiple acceptance scenarios to cover different cases. The happy path is one scenario. Boundary cases are another: what happens when the search term is empty? Error cases are yet another: what is displayed when there are no matching results? Each scenario tests whether the feature's behavior aligns with its intent.

The value of acceptance scenarios is that they turn "done correctly" from a subjective impression into a set of checkable conditions. Without acceptance scenarios, you and the Agent may have completely different definitions of "done." The Agent considers the search feature complete: it searches, it returns results. You consider it incomplete: results are not highlighted, there is no pagination, and an empty search term triggers an error. Both definitions of "done" are reasonable on their own. They were simply never aligned. Acceptance scenarios are the tool for surfacing these hidden expectation gaps early.

In practice, Ryan found that acceptance scenarios are also his most efficient review tool when checking specs. He only needs to glance at the user story and the corresponding Gherkin scenarios to tell whether the Agent's understanding of the requirement matches his own. In his words: "A user story plus a few scenario descriptions is already enough to tell whether the AI's understanding matches mine."

**Constraints: where are this layer's boundaries?**

The constraint dimension answers "what should not be done" and "what already exists that can be reused." At every layer it prevents the same thing: changes outside the scope of intent that produce unexpected side effects.

At the vision layer, constraints might be market limitations, compliance requirements, or business model boundaries. At the architecture layer, constraints are the boundaries of technical capabilities, performance requirements, and compatibility with existing systems. At the feature layer, constraints answer: what is the boundary of this change? Which modules must not be touched? What existing components can be reused?

Feature-layer constraints are particularly easy to overlook. You ask the Agent to add a search feature and it may also refactor the article listing page's sort logic, reasoning that search result sorting and list sorting should be unified. From a technical standpoint this might be reasonable. But the change is outside your expectations. It might break existing user habits. It might affect other features that depend on the listing sort logic. The constraint dimension explicitly tells the Agent the boundaries of the change: build the search feature only, do not touch the listing page's sort logic.

Ryan's spec template includes a Context Analysis field with three parts: reference code (existing modules in the project that can be reused), related documentation (design documents and API docs), and related historical features (similar features built previously and their change records). This field helps the Agent understand what already exists, effectively preventing it from reinventing the wheel.

The cost of empty fields is worth emphasizing. Ryan observed a consistent pattern: when fields related to the technical approach are left empty in the spec, the Agent responsible for coding starts making up its own approach. The invented approach may be incompatible with the project's existing architecture, may introduce unnecessary dependencies, or may re-implement functionality that already exists. An empty field sends a clear signal: intent alignment on that dimension has not happened. The Agent received no guidance on that dimension and can only guess.

## Drift During Decomposition

The three dimensions describe what content each layer should contain. But in practice you do not write all layers at once. You write the feature spec first, then have the Agent generate a task list from the spec, then derive a checklist from the task list. Each step generates new, lower-level information.

This generation process itself introduces drift.

Drift has two sources. The first is the Agent injecting its own interpretation during the "translation." Your spec says "users can search articles." The Agent translates this into concrete execution steps: "Add a `buildQuery` function in `search.ts` that calls PostgreSQL's full-text search." During this translation the Agent makes a series of decisions: what to name the function, which database API to call, what format to return results in. These decisions may or may not align with your intent. You did not specify in the spec whether to use tsvector or a LIKE query. The Agent chose one on its own. If its choice does not match your performance expectations, that is drift.

The second source is more subtle. When the Agent generates the task list, a large amount of information is already stacked in context: project context, feature spec, conversation history, previously generated code snippets. A constraint in the spec may get overlooked due to insufficient attention allocation. This is the same attention decay mechanism described earlier, except this time it occurs during layer generation rather than during conversation. The Agent did not deliberately ignore your constraint. There was simply too much information in context, and that constraint lost the competition for attention.

Both types of drift produce the same outcome: lower-level information that is inconsistent with upper-level intent. And drift compounds across layers. If the spec-to-task translation drifted slightly, the task-to-code translation may drift further on top of that. If you only check alignment at the final output, you may face a result that has diverged significantly from the original intent, with no easy way to pinpoint where the divergence began.

This is why acceptance checking is not something you do once at the end. Every time new lower-level information is generated, you need to verify its consistency with the layer above.

### Detecting Drift with Cross-Validation

One effective method is to have the Agent examine the same intent from different angles, then compare whether the multiple outputs are consistent.

Ryan's approach uses three rounds of thinking. In the first round, starting from the requirement, the Agent generates an implementation plan and impact analysis, producing the spec document. This is the Agent's first interpretation of the requirement. In the second round, starting from the spec, it breaks work down into concrete execution steps, producing the task list. This is the second interpretation. The act of breaking things into steps is itself a form of validation: if a step cannot be clearly defined, it is often because the spec is not specific enough at that point, revealing a gap in the plan. In the third round, starting from the task list, it derives acceptance criteria, producing the checklist. This is the third interpretation, and it runs in the opposite direction from the first two: instead of reasoning from "what to do" to "how to do it," it reasons from "how to do it" back to "how to know it was done correctly."

Consistency among the three documents is evidence of alignment. Contradictions are signals of drift.

If the checklist contains a check item that has no corresponding functionality in the spec, it means the Agent added something during task generation that the spec did not cover. This might be a reasonable addition (you genuinely missed that case) or it might be the Agent overstepping (it misunderstood the scope of the requirement). Either way, the contradiction itself is worth a look.

If a checklist item directly contradicts the spec (for example, the spec says "search results sorted by relevance" but the checklist's acceptance criterion says "sorted by time"), it means intent shifted during the spec-to-task-to-checklist pipeline. Without this cross-check, that shift would carry all the way into the final code.

Ryan calls the third round "adversarial testing." The name fits: it re-examines the same requirement from a completely different angle (acceptance rather than implementation). If three different angles all see the same thing, you have good reason to believe alignment held. If three angles see different things, you caught the problem before any code was written.

The essence of this method does not depend on the specific form of "three documents." The core idea is: when you generate lower-level information from a higher level, use a different angle to check whether the two are consistent. The specific implementation could be Ryan's spec/task/checklist three-document system, having a separate independent Agent review the first Agent's output, or an automated consistency-checking tool. Spec Kit's `analyze` command does exactly this: it scans all generated documents in read-only mode, checking for duplication, contradiction, omission, or terminology inconsistency across documents.

Forms can differ. The principle is the same: every time new lower-level information is generated there is a risk of drift, and you need a mechanism to detect it at that boundary.

## Eliminating Ambiguity with Structure

The preceding content might give the impression that eliminating ambiguity requires writing more content and filling more fields. This misconception is worth clarifying explicitly.

The root cause of ambiguity is not insufficient information volume. It is key questions in key dimensions going unanswered. You could write a three-page natural language description of a search requirement, specifying in detail where the search box should go, what color it should be, and what font size to use. But if you have not answered "what happens when search results are empty," "does the search scope include article body text," and "is it search-as-you-type or click-to-search," your three-page description has the same level of ambiguity as a one-sentence prompt. The only difference is that the ambiguity is buried in more text and harder to find.

The value of the structured dimensions (intent, acceptance, constraints) is not that they make you write more. It is that they put the questions you must answer in front of you, making them impossible to skip. A five-line spec that answers "for whom," "what," "how to know it is correct," and "what should not change" is far more effective than a ten-page document filled entirely with implementation details.

Spec Kit's `clarify` workflow offers an interesting design. It uses 10 categories of structured questions to detect ambiguity in specs: functional scope, domain model, interaction design, non-functional requirements, integration points, edge cases, constraints, terminology definitions, completion signals, and placeholder markers. Each round asks at most 5 targeted questions. These questions are directed at the person writing the spec, helping you discover places where your own thinking is not yet clear. After a single round of clarify, the spec might only be a few lines longer, but ambiguity drops significantly because the decisions that needed to be made have been made.
