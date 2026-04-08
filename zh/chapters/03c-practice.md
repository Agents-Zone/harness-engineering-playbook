# 实践：AILock-Step 的验证链路

前面两节建立了原则和机制。这一节用 02d 的 OKR 系统案例继续，展示这些机制在 AILock-Step 框架中怎么落地。02d 展示了 spec 从一句话到可执行文档的全过程。这里接着看：spec 完成之后，验证基建怎么搭建，怎么运行，怎么在代码合并前拦截意图偏离。

三个步骤依次展开：在架构文档中定义 API contract（验证的骨架），从 spec 生成集成测试（可执行的约束），用独立的 Agent 做 spec 一致性审查（补位测试的盲区）。

## 第一步：定义 API Contract

02d 的 spec 中已经包含了一张 API contract 表，列出了 6 个接口的 Method、URL 和 Parameters。这张表界定了前后端的通信边界，但还不够具体，无法直接用来搭建测试。

测试需要知道的不只是"这个接口存在"，还要知道它接收什么、返回什么、什么情况下报错。框架在架构文档阶段会把 contract 扩展为可测试的接口规格。以 OKR 系统的"新增目标"接口为例：

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

这份 contract 做了三件事。它定义了每个字段的类型和约束（title 最大 200 字符，对应 spec 里的 Key Rule），定义了成功响应的结构（和 RuoYi 的 AjaxResult 格式一致，对应 project-context.md 里的约定），定义了错误场景和对应的错误信息。

有了这份 contract，前端开发可以搭建 mock server 模拟后端接口，后端开发可以直接用 contract 里的输入参数测试接口。两端独立开发，各自基于 contract 验证。

## 第二步：从 Spec 生成集成测试

Spec 完成，contract 定义好，下一步是生成测试。框架的 `/generate-tests` skill 从两个来源提取测试用例：spec 中的验收标准和业务规则，以及 API contract 中的接口规格。

从验收标准生成的测试覆盖用户可感知的行为。以 OKR 系统为例，13 条 acceptance criteria 和 5 条 business rules 共 18 条规则，映射为 7 个 E2E 测试用例。映射关系是显式的：

```markdown
| 测试用例 | 覆盖的 Spec 条目 | 验证内容 |
|---------|----------------|---------|
| test-01 | AC-1, BR-004 | 新增目标：必填字段校验 + 季度默认值 |
| test-02 | AC-2 | 目标列表：按季度和员工筛选 |
| test-03 | AC-3 | 编辑目标：修改标题和状态 |
| test-04 | AC-4, BR-001 | 删除目标：级联删除关联 KR |
| test-05 | AC-5 | 目标列表：显示关联 KR 数量 |
| test-06 | BR-002, BR-003 | KR 约束：数量上限 5 + 完成率范围 0-100 |
| test-07 | BR-005 | 季度总览：只读，无编辑按钮 |
```

这张映射表本身就是一份可追溯性文档。任何一条验收标准都能找到对应的测试用例，任何一个测试用例都能追溯到它验证的 spec 条目。如果后续 spec 变更（比如删掉 BR-005），映射表能立即告诉你哪些测试需要同步调整。

从 API contract 生成的测试覆盖接口级行为。以"新增目标"接口为例，contract 定义了 3 个字段的约束和 2 个错误场景，直接转化为接口测试：

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

这些测试在 Agent 编码之前就存在。运行结果全红，因为接口还没实现。Agent 开始编码后，每完成一个接口，跑一轮测试，红变绿就是进度，仍然红就是偏差。

注意：E2E 测试和接口测试验证的层面不同。E2E 测试从用户操作的角度验证完整的行为路径，接口测试从 contract 的角度验证单个接口的输入输出。两者互补，E2E 测试是验收的主力，接口测试提供更细粒度的定位。

## 第三步：Code Review Against Spec

Agent 完成编码后，框架的 `/review-against-spec` skill 启动一个独立的 review session。Review agent 在全新的 context 中运行，没有编码过程的任何信息。它的输入是 Agent 的代码变更和原始 spec，输出是一份 spec 一致性报告。

以 OKR 系统为例，Agent 完成第一轮编码后，review agent 的报告结构如下：

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
| BR-002 | KR数量限制 max 5 | PARTIAL | 前端有 Element UI 提示，后端 ServiceImpl 缺少数量校验。绕过前端直接调 API 可突破限制。 |
| BR-003 | 完成率范围 0-100 | PASS | — |
| BR-004 | 默认季度 | FAIL | handleAdd() 未设置 form.quarter 默认值。新增对话框打开时季度下拉框为空。 |
| BR-005 | 总览只读 | PASS | — |

### Scope Check
- 检测到 login.vue 被修改（+12 行），但 spec 中未涉及登录模块。标记为 OUT_OF_SCOPE 变更。

### Summary
- AC: 13/13 PASS
- BR: 3/5 PASS, 1 PARTIAL, 1 FAIL
- Scope: 1 out-of-scope change detected
- Recommendation: 修复 BR-002 和 BR-004 后重新提交
```

这份报告展示了 code review 补位测试盲区的具体方式。BR-002 的问题（后端缺少数量校验）在测试中可能被前端行为掩盖：E2E 测试通过前端操作，前端的提示会阻止用户添加第 6 个 KR，测试通过了，但后端的漏洞仍然存在。Review agent 通过逐条比对 spec 和代码，发现了实现只覆盖了一半。

Scope check 也是测试覆盖不到的。Agent 修改了 login.vue，测试只覆盖 OKR 相关的页面，不会触发登录模块。但 review agent 扫描 diff 后发现这个变更不在 spec 范围内，标记出来供人类判断。

## 闭环

三步走完：contract 定义了边界，测试在边界上搭建了可执行约束，code review 补位了测试覆盖不到的意图漂移。人类需要看的不是代码，是两份报告：测试结果（行为是否正确）和 spec 一致性报告（意图是否对齐）。两份报告都直接对应 spec 的结构，聚焦在偏离和缺失上。

到这里，规约（第二章）加上验证（本章）构成了单任务的闭环。一个人加上 Agent，可以可靠地完成一个 feature 从 spec 到代码到合并的全流程。

这个闭环是后续所有内容的基础。当你开始同时运行多个 Agent 并行开发不同的 feature（卷二的主题），本章搭建的 contract、测试基建和 review 机制从好的工程实践变成了生存前提。没有 contract，多个 Agent 的产出合在一起就冲突。没有测试基建，集成后的问题无法定位到具体模块。没有独立 review，人类审查能力跟不上多 Agent 的产出速度。

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
