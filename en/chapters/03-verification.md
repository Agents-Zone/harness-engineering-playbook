# Verification: Ensuring Code Is Faithful to the Spec

In the OKR case study from the previous chapter, the spec went through two rounds of iterative refinement, with all 13 acceptance criteria and 5 business rules clearly defined. The Agent generated 12 backend files and 4 frontend files based on this spec. The compliance check result was 26/28 PASS.

The two misses were not accidental. BR-002 required "a maximum of 5 KRs per objective." The Agent added an Element UI prompt on the frontend but failed to add a ServiceException validation on the backend. Calling the API directly, bypassing the frontend, could break the limit. BR-004 required "the current quarter to be selected by default when opening the creation page." The Agent did not implement this default value logic.

These two issues share a common trait: the code itself was well-written, with reasonable API design, proper naming conventions, and passing tests. They were not code quality problems. They were intent loss. The spec stated it; the code did not do it.

Specification solves "telling the Agent what to do." Verification solves "whether the Agent actually did it."

## Intent Drift: The Blind Spot of Existing Tools

We are not the only ones who noticed this problem. Anthropic explicitly stated in a series of articles on harness design that separating generation from evaluation is the most effective lever for quality control. Industry consensus has formed: verification is one of the most critical components of harness engineering.

Various frameworks have made extensive attempts in this direction. BMAD designed a three-tier information-asymmetric code review system, where different reviewers see different scopes of information to reduce confirmation bias. Spec Kit uses constitution governance and 10 types of ambiguity detection to intercept quality issues at the spec stage. Both OpenAI and Anthropic have released multi-Agent parallel code review systems and eval frameworks. Community discussions around TDD, Trophy testing, and adversarial review have never stopped.

If you have used these verification tools at scale in real projects, you will have noticed something: code can pass all linting, all unit tests, all adversarial reviews, yet the functionality it implements is not what the spec described.

Missing null checks, inconsistent naming, security vulnerabilities: AI can find and fix these code quality issues on its own. What is truly hard to detect is intent drift, where the spec says A but the code implements B. BR-002 above is exactly this type of problem: the code was well-written, it simply missed one business rule from the spec. No review that ignores the spec would catch it.

The TDAD paper published in March 2026 provided quantitative evidence. Researchers found that giving the Agent abstract process instructions like "please follow the TDD workflow" actually increased the regression rate from 6.08% to 9.94%. But when a tool analyzed the spec and told the Agent specifically which tests to check, the regression rate dropped to 1.82%. Process instructions are not the answer; spec context is.

These tools fail for the same reason: their frame of reference is the code itself, not the spec. They can answer "does this code have bugs" but cannot answer "is this code what the spec required." The missing backend validation for BR-002 would not be caught by any code-based check, because the code itself has no bugs. It simply lacks a feature the spec required.

## The Anchor for Verification Is the Spec

To answer "is this code what the spec required," you need a frame of reference outside the code. That frame of reference is the specification. The single source of truth for verification is the spec, not the code.

Every verification action answers one question: does the code match what the document says? Tests verify "whether the code implements the behavior the spec describes," not "whether the code is correct." Code review asks "is the code consistent with the spec," not "is the code good." Doing review without the spec only produces feedback like "the code structure is clean, consider adding exception handling," feedback that is correct but useless because it does not answer whether the code is faithful to the spec.

This is also why the specification chapter comes before the verification chapter. Without a good spec, verification has no anchor.

## Verification Must Be Upfront, Continuous, and Modular

Verification infrastructure must be in place before coding begins, run continuously during execution, and be split by module. This conclusion follows from two structural characteristics of Agents.

Agents drift during execution. The specification chapter already established this concept: Agent output shifts as context accumulates, and small deviations compound across subsequent steps. In the code implementation phase, this means the code at step ten may have already drifted from the spec referenced at step one. If you wait until everything is done before verifying, you face dozens of failing tests, each pointing to a different issue, with fix costs far exceeding those of mid-process correction. Verification must happen during the process, not after it.

Agents have limited context capacity. This constraint applies not only to coding but also to verification. If an Agent writes 2,000 lines of code in one go and another Agent is asked to review it, the reviewing Agent must simultaneously understand the spec content and the 2,000-line implementation, then judge whether the two are consistent. This exceeds its effective processing range. The result is either missed issues or a stream of generic feedback like "consider improving naming." Verification granularity must match execution granularity: if execution is split into small chunks, verification must follow suit.

These two constraints point to the same conclusion: convert the spec into an executable test framework first, so the Agent has a feedback signal from step one. The DORA 2025 report corroborated this with large-scale data: AI is an amplifier of existing engineering practices. For teams with verification infrastructure, Agent speed amplifies output. For teams without it, Agent speed amplifies chaos.

So what does upfront verification infrastructure specifically include?

## Tests and Code Review Are Both Indispensable

Verifying spec consistency requires two mechanisms, not one. A recurring pattern in the community is that teams use only one, have a bad experience, and conclude that "AI coding is unreliable."

Teams that do code review but not testing find that review can detect deviations between code and spec but cannot verify runtime correctness. Review saying "this logic looks right" does not mean the code actually runs correctly.

Teams that do testing but not code review encounter a different problem: tests can verify behavior, but coverage rarely reaches the spec's full intent. The Agent may use hardcoded values to game the tests, for example, directly returning expected values to make tests pass. Tests are all green, but the code harbors issues that only surface under specific conditions.

Both mechanisms share the same goal: turning the spec into executable constraints. But they cover different dimensions. Tests verify behavioral correctness: did the code deliver the functionality the spec described? Code review verifies semantic completeness: has the code drifted from the spec's framework, or done things the spec did not say?

Their cost structures also differ. Tests are low-cost, fast to run, and provide immediate feedback, making them the workhorse for continuous mid-process verification. Code review is more expensive and better suited for milestone checkpoints. The two complement each other in cost and coverage.

This chapter covers both mechanisms. The next section discusses test infrastructure: how to convert the spec into an executable test framework before the first line of business code is written, so the Agent has feedback at every step of execution. After that comes code review: how to use an independent Agent to audit consistency between code and spec, covering the intent drift that tests cannot reach. Finally, the AILock-Step framework's practice demonstrates the complete pipeline of both mechanisms.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
