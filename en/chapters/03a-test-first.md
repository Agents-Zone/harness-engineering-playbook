# Test Infrastructure First: Turning the Spec into Executable Constraints

Tests are the executable form of the spec. Acceptance criteria are written in natural language; tests translate them into machine-runnable assertions. This translation must be completed before the first line of business code is written, so the Agent has a feedback signal from its very first step: does this step faithfully follow the spec?

But a feature is not a monolith. It spans frontend pages, backend endpoints, and database operations across multiple modules. Without defined boundaries between modules, tests can only run end-to-end, from frontend interactions all the way to database writes. End-to-end tests have value, but their granularity is too coarse: when a test fails, you cannot tell whether the problem lies in frontend rendering, backend logic, or a database query. Diagnosis is expensive and fix cycles are long. To give the Agent precise feedback during execution, tests must be runnable at the module level. That requires defining testable module boundaries first.

## API Contract: Testable Boundaries

An API contract defines the inputs and outputs of each module: what parameters a backend endpoint accepts, what structure it returns, under what conditions it reports an error, and what that error looks like. With this definition in place, three things become possible.

First, modules can be tested independently. Before the backend is written, the frontend can set up a mock server based on the contract and verify that its own calling logic is correct. Before the frontend is written, the backend can test its endpoints directly using the input parameters defined in the contract. Each module verifies "did I implement the behavior the contract specifies?" within its own boundary, without waiting for other modules to be completed.

Second, test assertions gain a clear reference point. If the contract says an endpoint returns a JSON object containing `status` and `data` fields, the test asserts that the return value contains those two fields. The reference point is the contract document, not whatever implementation the Agent happened to produce.

Third, problem diagnosis becomes precise. When a module's tests fail, the problem is within that module's boundary. No need to trace through the entire chain layer by layer.

This is also a prerequisite for multi-Agent parallel development (covered in Chapter 5): contracts are what allow multiple Agents to develop different modules independently without conflicting with each other.

With boundaries defined, the next step is to build tests on those boundaries.

## Generating Tests Before Coding

Test inputs come from two sources: acceptance criteria in the spec, and API contracts in the architecture document. At the time tests are written, business code does not yet exist. This sequencing guarantees that tests are independent of the implementation, serving as a direct translation of the spec's intent rather than being contaminated by implementation details.

Tests generated from acceptance criteria cover user-perceivable behavior. If the spec says "a record appears on the billing page after successful payment," the test is: call the payment endpoint with valid parameters, verify that a new record was added to the billing table, and verify that the endpoint returned a success status code. If the spec says "each objective has a maximum of 5 KRs," the test is: create an objective, add 5 KRs, attempt to add a 6th, and verify that the endpoint returns an error. Each acceptance criterion maps to one or more test cases, and the Given/When/Then structure naturally maps to the test's setup/action/assertion.

Tests generated from API contracts cover endpoint-level behavior. The contract defines each endpoint's input parameters, return structure, and error codes; the tests verify one by one whether these agreements are honored. These tests are finer-grained than acceptance-criteria tests and cover contract compliance at the module boundary.

Dependencies that do not yet exist are replaced with mocks. Frontend tests use a mock server to simulate backend endpoints; backend tests use a mock database or mock external services. Mock behavior is based on the API contract definitions, not fabricated from thin air. This lets the test framework be assembled and run before coding begins (all red, since the implementation does not yet exist).

Why are integration tests the workhorse for acceptance, rather than unit tests? Integration tests verify whether a user story works end to end, directly corresponding to the intent in the spec. Unit tests verify whether a function returns the correct value, which is an implementation detail too far removed from spec intent. More critically, an Agent can adjust the internal structure of its implementation to make unit tests pass, but fabricating a complete user journey (from API call to database write to return-value verification) is far more difficult. Unit tests are not useless: the Agent is free to write unit tests to support its own development process. But as the basis for acceptance, integration tests are the more reliable signal.

## Continuous In-Process Verification

Once the test framework is in place, verification becomes part of the execution process, not a post-hoc checking step.

The Agent's work is broken into small chunks: implement an endpoint, complete a user story, handle an edge case. After each chunk, the relevant tests run immediately. If the tests pass, move to the next chunk. If a test fails, fix it on the spot instead of carrying the deviation forward.

The value of this rhythm lies in compressing the survival time of deviations to the minimum. As analyzed in the introduction, Agents drift during long chains of execution: code at step ten may have already deviated from the spec referenced at step one. If a deviation arises at step five but is not discovered until step fifty, the output of the intervening forty-five steps may all rest on a flawed foundation. If the deviation is caught and corrected by tests at step five, the blast radius is confined to that single step.

This closed-loop structure is a projection of the same pattern as the iteration loop from the spec chapter, applied at a different phase. The spec-phase loop is: human writes intent, Agent expands it, cross-validation exposes issues, corrections are made, re-validation follows. The execution-phase loop is: spec defines behavior, tests encode behavior, Agent implements, tests provide feedback, corrections are made, move to the next step. The common trait of both loops is that every step has a feedback signal independent of the Agent's own output. In the spec phase, that signal is cross-validation. In the execution phase, it is tests.

## Validating the Spec Through Tests

Putting test infrastructure first is the earliest possible check on spec quality.

If an acceptance criterion cannot be turned into a test, the problem is not with the test but with the spec. "The system should have a good user experience" cannot be tested because it defines no observable behavior. "The user sees a confirmation page within 2 seconds of submitting the form" can be tested because it defines a specific input (form submission), output (confirmation page), and constraint (within 2 seconds).

This check happens before coding begins. If the spec has ambiguities, the test-writing phase will expose them, and you can return to the spec phase to make corrections at low cost. If you wait until coding is complete to discover that the spec is vague, the cost of fixing it includes the code and tests already written, far higher than correcting it at the spec stage.

Test infrastructure solves verification of behavioral correctness: did the code accomplish what the spec said? But as noted in the introduction, tests cannot cover every form of intent drift. Hardcoded values, out-of-scope changes, architectural deviations: these are not behavioral issues, and tests will not catch them. The next section covers how code review fills these blind spots that tests miss.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
