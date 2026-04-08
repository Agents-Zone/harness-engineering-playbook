# Practice: The Verification Chain in AILock-Step

The previous two sections established principles and mechanisms. This section continues with the OKR system case from 02d, showing how these mechanisms land in practice within the AILock-Step framework. Section 02d demonstrated the full journey of a spec from a single sentence to an executable document. Here we pick up where that left off: after the spec is complete, how to build verification infrastructure, how to run it, and how to intercept intent drift before code is merged.

Three steps unfold in sequence: define API contracts in the architecture document (the skeleton of verification), generate integration tests from the spec (executable constraints), and use an independent Agent for spec consistency review (covering the blind spots that tests miss).

## Step 1: Define the API Contract

The spec from 02d already included an API contract table listing 6 endpoints with their Method, URL, and Parameters. This table delineated the communication boundary between frontend and backend, but it was not specific enough to build tests from directly.

Tests need to know more than "this endpoint exists." They need to know what it accepts, what it returns, and under what conditions it returns errors. The framework expands contracts into testable interface specifications during the architecture document phase. Using the OKR system's "create objective" endpoint as an example:

```yaml
POST /system/okrObjective:
  request:
    body:
      employeeId: { type: integer, required: true }
      quarter: { type: string, required: true, pattern: "YYYY-QN" }
      title: { type: string, required: true, maxLength: 200 }
  response:
    success: { code: 200, body: { msg: "新增成功", data: { objectiveId: integer } } }
    errors:
      - { condition: "title 为空", code: 500, msg: "目标标题不能为空" }
      - { condition: "quarter 格式不合法", code: 500, msg: "季度格式错误" }
```

This contract does three things. It defines the type and constraints of each field (title maxLength 200, corresponding to the Key Rule in the spec). It defines the success response structure (consistent with RuoYi's AjaxResult format, as specified in project-context.md). It defines error scenarios and their corresponding error messages.

With this contract in place, frontend developers can build a mock server to simulate backend endpoints, and backend developers can test endpoints directly using the input parameters from the contract. Both sides develop independently, each validating against the contract.

## Step 2: Generate Integration Tests from the Spec

With the spec complete and the contract defined, the next step is generating tests. The framework's `/generate-tests` skill extracts test cases from two sources: acceptance criteria and business rules in the spec, and interface specifications in the API contract.

Tests generated from acceptance criteria cover user-perceivable behavior. In the OKR system example, 13 acceptance criteria and 5 business rules (18 rules total) map to 7 E2E test cases. The mapping is explicit:

```markdown
| Test Case | Covered Spec Items | Verification Content |
|-----------|--------------------|---------------------|
| test-01 | AC-1, BR-004 | Create objective: required field validation + default quarter |
| test-02 | AC-2 | Objective list: filter by quarter and employee |
| test-03 | AC-3 | Edit objective: modify title and status |
| test-04 | AC-4, BR-001 | Delete objective: cascade delete associated KRs |
| test-05 | AC-5 | Objective list: display associated KR count |
| test-06 | BR-002, BR-003 | KR constraints: max count 5 + completion rate range 0-100 |
| test-07 | BR-005 | Quarterly overview: read-only, no edit button |
```

This mapping table is itself a traceability document. Every acceptance criterion can be traced to a corresponding test case, and every test case can be traced back to the spec item it verifies. If the spec changes later (say BR-005 is removed), the mapping table immediately tells you which tests need to be updated.

Tests generated from the API contract cover interface-level behavior. Using the "create objective" endpoint as an example, the contract defines constraints for 3 fields and 2 error scenarios, which translate directly into interface tests:

```typescript
describe('POST /system/okrObjective', () => {
  test('成功创建目标', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: '提升客户满意度'
    });
    expect(res.body.code).toBe(200);
    expect(res.body.data).toHaveProperty('objectiveId');
  });

  test('title 为空时返回错误', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: ''
    });
    expect(res.body.code).toBe(500);
  });

  test('title 超过 200 字符时返回错误', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: 'x'.repeat(201)
    });
    expect(res.body.code).toBe(500);
  });
});
```

These tests exist before the Agent starts coding. All results are red because the endpoints are not yet implemented. Once the Agent begins coding, each completed endpoint triggers a test run. Red turning green means progress. Staying red means deviation.

Note: E2E tests and interface tests verify at different levels. E2E tests validate complete behavior paths from the user's perspective. Interface tests validate single endpoint input/output from the contract's perspective. The two are complementary: E2E tests are the primary acceptance mechanism, while interface tests provide finer-grained fault localization.

## Step 3: Code Review Against Spec

After the Agent completes coding, the framework's `/review-against-spec` skill launches an independent review session. The review Agent runs in a fresh context with no information from the coding process. Its inputs are the Agent's code changes and the original spec. Its output is a spec consistency report.

In the OKR system example, after the Agent completes its first round of coding, the review Agent's report is structured as follows:

```markdown
## Spec Consistency Report

### Acceptance Criteria
| # | Criterion | Status | Evidence |
|---|-----------|--------|----------|
| AC-1 | 新增目标必须选择员工、季度、标题 | PASS | OkrObjectiveController.add() validates required fields |
| AC-4 | 删除目标同时删除关联 KR | PASS | @Transactional + keyResultMapper.deleteByObjectiveId() |
| ... | | | |

### Business Rules
| # | Rule | Status | Finding |
|---|------|--------|---------|
| BR-001 | 级联删除 | PASS | — |
| BR-002 | KR数量限制 max 5 | PARTIAL | Frontend has Element UI prompt, but backend ServiceImpl lacks count validation. Bypassing the frontend and calling the API directly can exceed the limit. |
| BR-003 | 完成率范围 0-100 | PASS | — |
| BR-004 | 默认季度 | FAIL | handleAdd() does not set a default value for form.quarter. The quarter dropdown is empty when the create dialog opens. |
| BR-005 | 总览只读 | PASS | — |

### Scope Check
- Detected modification to login.vue (+12 lines), but the spec does not involve the login module. Flagged as OUT_OF_SCOPE change.

### Summary
- AC: 13/13 PASS
- BR: 3/5 PASS, 1 PARTIAL, 1 FAIL
- Scope: 1 out-of-scope change detected
- Recommendation: Fix BR-002 and BR-004, then resubmit
```

This report demonstrates exactly how code review covers the blind spots that tests miss. The BR-002 issue (missing backend count validation) could be masked by frontend behavior in testing: E2E tests operate through the frontend, and the frontend prompt prevents users from adding a 6th KR, so the test passes, but the backend vulnerability remains. The review Agent, by comparing spec items against code one by one, discovered that the implementation only covered half of the requirement.

Scope checking is another area tests cannot cover. The Agent modified login.vue, but tests only cover OKR-related pages and never touch the login module. The review Agent, scanning the diff, identified this change as outside the spec's scope and flagged it for human judgment.

## Closing the Loop

All three steps complete a chain: the contract defines boundaries, tests build executable constraints on those boundaries, and code review covers the intent drift that tests cannot reach. What humans need to review is not the code itself, but two reports: the test results (is the behavior correct?) and the spec consistency report (is the intent aligned?). Both reports map directly to the spec's structure, focusing on deviations and gaps.

At this point, specification (Chapter 2) plus verification (this chapter) form the closed loop for a single task. One person plus an Agent can reliably complete the full pipeline from spec to code to merge for a single feature.

This closed loop is the foundation for everything that follows. When you start running multiple Agents in parallel to develop different features (the topic of Volume 2), the contracts, test infrastructure, and review mechanisms built in this chapter shift from good engineering practice to a survival prerequisite. Without contracts, the outputs of multiple Agents conflict when combined. Without test infrastructure, post-integration issues cannot be traced to specific modules. Without independent review, human review capacity cannot keep up with the output velocity of multiple Agents.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
