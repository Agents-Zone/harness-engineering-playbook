# Iterating Toward an Executable Spec

The previous two sections established a methodological framework: information should be layered, each layer has three dimensions, and cross-validation between layers detects drift. This section covers what you actually do when you face a concrete requirement with this framework in hand.

A key mental model: a spec is not written in one sitting. It is iterated into existence. You do not need to sit down and produce a perfect spec in a single pass. You write a starting point, the Agent expands it, cross-validation exposes problems, you fix them, validate again, and repeat until convergence. The whole process is more like a structured dialogue between you and the Agent than a one-sided document-writing exercise.

## The Starting Point: A User Story

The earlier discussion covered four information layers: vision, architecture, feature, and task. The spec iteration described here operates at the feature and task layers. It assumes the upstream work is already done: the product vision has been set, the system architecture has been chosen, and this high-level information has been captured in the project context. The derivation process from vision to user journey to concrete requirements belongs to the domain of product methodology, which this book does not cover. But that process is equally important. It determines whether the requirement you receive at the feature layer is itself sound.

Your starting point is a requirement that can be expressed as a user story. "As a blog author, I want to search published articles by keyword so I can quickly find content I wrote before."

Writing this sentence requires three decisions: who it is for (blog authors, not readers), what it does (keyword search of published articles), and why (quickly find old content). If you cannot write even this one sentence, your understanding of the requirement is not yet clear enough. Go back upstream to clarify, rather than letting the Agent start executing.

This sentence is not the final spec. It is the seed for iteration. You do not need to figure out where the search box goes, how results are sorted, or what happens with empty results at this stage. Those details will surface during subsequent iterations.

## Agent Expands the Spec Draft

You hand the user story and a rough scope to the Agent and ask it to generate a spec draft.

The Agent does more in this process than you might expect. It does not merely expand your one sentence into a longer description. It reads the project context to find existing modules related to the search feature. It analyzes the code areas this requirement might affect. It generates acceptance scenarios, attempting to cover the happy path, boundary cases, and error cases. It may think of situations you had not considered, such as how to handle special characters in the search term. It may also suggest a technical approach, such as recommending PostgreSQL full-text search over application-layer fuzzy matching.

Ryan's `/new-feature` skill is one implementation of this process. The Agent loads the project context and `CLAUDE.md`, analyzes user value based on the described requirement, generates Gherkin acceptance scenarios, and estimates the feature's scope. Within a few minutes it produces a spec draft covering all three dimensions.

This draft will almost certainly have issues. Maybe it missed a boundary case you care about. Maybe its understanding of the search scope differs from yours (you intended title-only search, it assumed full-text). Maybe the impact analysis overlooked a module that should have been mentioned. These issues are normal at this stage. The next step will surface them.

## Cross-Validation Detects Drift

You ask the Agent to derive a task list from the spec: what specific work needs to be done to complete this feature, which files change, and in what order. Then derive a checklist from the task list: what to check after completion and what counts as passing.

The principle of cross-validation was covered earlier. In practice, the purpose of this step is to test spec quality through three successive interpretations by the Agent. The first interpretation produced the spec (from requirement to plan). The second interpretation produced the task list (from plan to steps). The third interpretation produced the checklist (from steps to acceptance criteria).

You compare the three documents and look for contradictions.

The checklist includes an item: "Verify search supports Chinese word segmentation." But your spec never mentioned Chinese word segmentation. Where did this come from? Perhaps the Agent inferred from the project context that the product has Chinese-speaking users and added this requirement on its own. This might be a reasonable addition, or it might be outside the scope of your current release. Either way, you need to know about it and then decide whether to add it to the spec or remove it from the checklist.

The task list includes a step: "Modify the article listing page's sort logic to reuse search sorting." But your spec's constraint dimension states "only modify search-related code, do not touch the listing page's sort logic." Contradiction. The Agent made a decision that exceeded the constraint boundary while breaking work into steps. You need to correct this step in the task list.

When the three documents are consistent, with no contradictions and no unexpected new content, it means the Agent saw the same thing from three different angles.

## Document-Level Correction

Problems found through cross-validation point directly to what needs changing in the spec. You fill in missing acceptance scenarios, clarify ambiguous descriptions, and add overlooked constraints. Then ask the Agent to regenerate the task list and checklist from the corrected spec, and review once more.

This loop usually converges within one or two rounds. Each round costs little. It is all document-level work: edit a few lines of the spec, have the Agent regenerate two documents, spend a few minutes comparing. But the problems these rounds catch, if left to the coding phase, could take hours or even days to fix.

## Splitting by User Value

Sometimes you will find that iteration does not converge. You run two or three rounds of cross-validation, fix one contradiction only to see a new one appear. The spec keeps getting longer, acceptance scenarios keep multiplying, and dependencies between scenarios grow increasingly complex.

This usually means the spec contains multiple independent user values whose interactions prevent the Agent from handling them well within a single context.

A few concrete signals: more than three user value points, more than seven or eight acceptance scenarios with complex interdependencies, and more than four or five affected modules. These are not hard rules, but they serve as empirically effective warnings.

At this point you need to split. Splitting is not a separate phase outside iteration. It is how you handle the "will not converge" situation within the iteration process.

LLMs have strong natural language comprehension, which makes them highly effective at decomposing tasks. You only need to give them the splitting principle: split by user value, where each sub-unit is a function that a user can independently verify.

AILock-Step's splitting provides a good example. The requirement "user authentication system," if left unsplit, might involve registration, login, password reset, OAuth, and permission management (five or more user value points), making it hard to handle well in a single context. Splitting by user value produces three independent features: feat-auth-register (users can register, independently deliverable), feat-auth-login (users can log in, depends on registration being complete), and feat-auth-permission (users can manage permissions, depends on login being complete). Each feature has its own user story and acceptance scenarios and can be iterated independently.

Splitting by technical layer (feat-auth-db, feat-auth-api, feat-auth-ui) does not achieve this. Is the database schema correct? You will not know until the API and frontend are also done. Each sub-task lacks independent acceptance criteria, so you cannot perform cross-validation at that granularity.

After splitting, each sub-feature's spec enters the same iteration loop: Agent expansion, cross-validation, correction, until convergence. There is one additional check: alignment between the sub-spec and the original requirement. If the original requirement mentioned OAuth login but the split feat-auth-login spec only covers username/password login, cross-validation will expose this omission at the checklist stage.

## Criteria for Spec Readiness

Cross-validation no longer produces contradictions. The three documents describe the same thing from three different angles. The spec is ready for execution.

This does not mean the spec is perfect. Edge cases you never thought of will not appear in any document. But it does mean the spec is internally consistent and the Agent's understanding of the requirement is aligned across multiple perspectives. This is a reasonable level of confidence for starting to write code.

If a dimension genuinely has no content (for example, this feature does not affect any existing modules), write "None" explicitly. Leaving a field blank and writing "None" mean different things. A blank field implies the question was never considered, and the Agent may treat it as an oversight. "None" implies the question was considered, and the conclusion is that there is nothing.

## Intent Alignment: A Judgment Only Humans Can Make

In the iteration loop described above, the Agent does most of the work: expanding the spec, generating the task list, generating the checklist, running consistency checks. But there is one step only a human can do.

Cross-validation can check consistency between documents. The spec says A, the task list says A, the checklist says A. All three documents are perfectly consistent. But cross-validation cannot answer a more fundamental question: is A actually what you want?

Your user story says "users can search articles." The Agent interprets this as full-text search and expands the acceptance scenarios and tasks accordingly. Cross-validation passes. All three documents are perfectly consistent. But in your head you meant title-only search. This discrepancy will not be caught by any automated check, because all three of the Agent's thinking rounds are based on the same interpretation. It is just that this interpretation differs from yours.

Intent exists only in your head. You are the software's end user, or the end user's proxy. Only you can make the judgment "is this what I want," because the information "what I want" is something the Agent cannot access. It can only infer from what you have written.

So there is one step in the iteration loop that must be done by a human: after the Agent expands the spec, look at the user story and the key acceptance scenarios. "Look at" is meant literally. You do not need to review the task list line by line or check every item in the checklist. You only need to confirm two things: the user story's "who," "what," and "why" match what is in your head, and the key acceptance scenarios cover the situations you care about most.

When the upper-level direction is right, the lower levels rarely go badly wrong. The Agent's cross-validation handles lower-level consistency. When the upper-level direction is wrong, everything below it is wasted effort. So your time should be spent on upper-level intent checking.

Ryan's experience confirms this. The only place he actually spends time during the entire spec iteration process is reviewing the user story and scenario descriptions after the spec is generated. He says, "A user story plus a few scenario descriptions is already enough to tell whether the AI's understanding matches mine." For the subsequent task splitting, checklist generation, and code implementation, he rarely intervenes step by step.

The Agent excels at a different kind of checking: cross-document consistency. The spec says search results should be sorted by relevance. The checklist says to verify sorting by time. Contradiction. The spec mentions five acceptance scenarios but the task list only covers three. Omission. The spec uses the word "article" but the task list sometimes writes "post." Terminology inconsistency. These checks do not require knowing the intent. They only require comparing structural alignment between documents. The Agent can process the complete contents of all documents simultaneously and is far more reliable than a human at this type of checking.

The division of labor in the iteration loop works like this: the Agent handles expansion, generation, and consistency checking (structural, automatable work). The human handles intent alignment judgment (semantic work that only the intent owner can do).

## Match Iteration Depth to Rework Cost

The iteration loop described above has a full-fledged form: write intent, Agent expansion, cross-validation, correction or splitting, human review of user stories, re-validation until convergence. But not every task deserves the full loop.

Human attention is the scarcest resource in the entire process. The Agent performing cross-validation, consistency checks, and generating the task list and checklist costs you almost no time. But reviewing user stories, judging whether intent aligns, and deciding whether to modify the spec all require your focused attention. The Agent's part can run automatically. The human's part cannot.

What tasks deserve your focused attention? Look at the cost of rework.

Changing a button label, if done wrong, takes one minute to redo. Writing a full spec and reviewing user stories for this task is not a good trade. Just tell the Agent what to change, add one constraint (do not touch anything else), and let it run its own cross-validation as a self-check. You can even skip cross-validation, because even if it gets it wrong, the rework cost is trivial.

A new feature touching three modules, if done wrong, might waste a full day of work. It is worth spending ten minutes reviewing the user story and key acceptance scenarios to make sure the direction is right before letting the Agent start coding.

A large requirement involving five or more modules, if done wrong, might waste a week. It is worth splitting first, reviewing each sub-feature's spec carefully, and running the full cross-validation loop for each one.

Note that the criterion is not the amount of code. It is how much it hurts to get it wrong. A two-line change that affects the payment flow may have rework costs an order of magnitude higher than a hundred-line change to admin panel styling. The former deserves a careful spec review. The latter probably just needs a single sentence.

The same iteration loop can run very light or very heavy. At its lightest, you write one sentence of intent plus one constraint, the Agent expands and self-checks, you skip the review, and execution begins. At its heaviest, you write intent, the Agent expands, you review every acceptance scenario in depth, split into multiple sub-features, iterate each one separately, and review each one. These are not two different processes. They are the same loop running at different levels under different risk profiles. The loop itself stays the same. What changes is where the human intervenes and how deeply.
