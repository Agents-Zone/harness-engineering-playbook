# Code Review: Covering the Intent Drift That Tests Miss

Tests verify behavior: did the code accomplish what the spec said? But there is another class of issues in code where the behavior is entirely correct, tests are all green, yet intent is quietly drifting. Code review's job is to catch these blind spots that tests cannot see.

## What Tests Cannot See

The spec requires database connection strings to be managed via a configuration file. The Agent hardcodes the connection string directly in the code. Everything works perfectly, all tests pass, but deployment to a different environment will fail.

The spec requires implementing user registration. The Agent implements registration and, while at it, refactors the error-handling logic in the login module. The registration tests pass, but the login module's behavior has changed without any review.

The spec defines an entity called Order. The Agent names it Transaction in the code. There is no functional issue whatsoever, but when the next Agent takes over this module, it looks for the Order implementation based on the spec and finds a class called Transaction with no explicit link between the two.

These problems share a common trait: they violate no behavioral assertions, so tests will not catch them. Hardcoded values, out-of-scope changes, architectural deviations, over-implementation, terminology drift: all fall into this category. They are not bugs, but they are signals that the code is slowly diverging from the framework defined by the spec. Left undiscovered, these divergences accumulate over subsequent iterations, gradually blurring the mapping between code and spec until the spec loses its role as the single source of truth.

## The Review Standard Is the Spec

The checklist for traditional code review includes questions like: Are variable names consistent with conventions? Are there potential performance issues? Is exception handling thorough? The frame of reference for these questions is the code itself and general engineering best practices.

Code review in Agent development asks only three things. Is this code doing the same thing the spec describes? Is this code doing anything the spec did not ask for? Do the concepts in the code correspond to the terminology in the spec?

The frame of reference shifts from the code to the spec. The core inputs for the review agent are the changed code and the spec. It does not need to see the Agent's coding process or understand its implementation reasoning. The job is to compare code changes against the spec item by item, marking matches, deviations, and omissions.

This means the review output is not a vague list of "suggested improvements" but a spec-consistency report: which acceptance criteria have been met, which show deviations, and which are missing entirely. This report maps directly to the spec's structure, so human reviewers can focus on the deviated and missing items without reading code line by line.

## The Coding Agent and the Reviewing Agent Must Be Independent

If the same Agent first writes the code and then reviews its own output, the context accumulated during coding will influence its judgment. It decided to rename Order to Transaction while writing code, and the reasoning behind that decision remains in its context. During review, it sees Transaction, the reasoning in its context tells it this naming is justified, and it will not re-evaluate against the spec. The backend missed a validation, but the Agent remembers it already handled it on the frontend, that memory is in its context, and it will not question whether the backend also needs it. The review becomes a confirmation of the reasoning already present from the coding process, not an independent verification against the spec.

This is why the coding Agent and the review Agent must run in independent sessions. The review agent has no context from the coding process. It does not know why the code was written this way. It only knows what the spec requires, what the code does, and whether the two are consistent.

Using different models for cross-review further reduces the risk of shared blind spots. Different model families have different training data and reasoning biases; an issue the coding Agent systematically overlooks may be exactly what another model is sensitive to. This is the same principle as the cross-validation discussed in the spec chapter: only an independent perspective can detect drift.

Anthropic lists writer/reviewer separation as one of the highest-priority recommendations in their Claude Code best-practices documentation, explicitly noting that fresh context significantly improves review quality. Their internal code review system runs multiple agents in parallel on nearly every PR, each targeting a different problem category (logic errors, edge cases, API misuse, permission vulnerabilities, project conventions). After launch, the proportion of PRs with substantive findings rose from 16% to 54%. For large PRs over 1,000 lines, 84% received findings, averaging 7.5 flagged issues per PR. The rate at which engineers disagreed with review conclusions was below 1%.

OpenAI's practice provides another scale reference: their automated code review system processes over 100,000 PRs per day, with a positive feedback rate exceeding 80%.

These data points lead to a practical conclusion: independent review based on explicit standards is viable at scale and more consistent than human review. When Agent output velocity far exceeds human review capacity, using an independent Agent for spec-consistency review is the only approach that can keep pace with the volume of output.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
