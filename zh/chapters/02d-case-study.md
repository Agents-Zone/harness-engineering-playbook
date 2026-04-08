# 实践：从 Principle 到工作流

前面几节建立了规约的方法论框架：信息分层、三个维度、交叉验证、迭代循环。这些概念回答了"为什么"和"是什么"。这一节回答"在真实项目里长什么样"。

先看一个社区成员开发的工作流框架 AILock-Step，展示 principle 怎么变成具体的文档结构和流程。然后用一个在 RuoYi-Vue 项目上跑通的 OKR 系统需求，展示这些结构和流程在实际运行时产出了什么。前者是空壳，后者是填充。

## 框架设计：Principle 怎么变成工具

AILock-Step Feature Workflow 是一个基于 Claude Code 的文档驱动开发框架，用 skill（slash command）编排 feature 的完整生命周期。它在一个 AI agent 平台项目上跑过 40 多个 feature，完整源码在 GitHub 上公开。

这里展示的是一个实现方案。值得关注的是 principle 在具体系统中怎么变成可操作的设计决策，而具体的目录结构和配置格式可以根据项目需要调整。

### 分层原则的落地

前面讲了信息有四个层级（vision、架构、feature、task），应该用不同的文档承载、按不同的策略加载。AILock-Step 用三类文档实现了这个分层。

CLAUDE.md 承载跨层的强制约束。这是 Claude Code 每次启动时自动加载的文件。内容是整个项目生命周期内都不太变的硬性规则。在 RuoYi-Vue 项目中，这份文件包含了项目架构概述（模块依赖关系、启动入口、默认端口）、后端关键约定（Controller 标准写法、权限注解模式、操作日志注解）、前端约定（API 请求封装、路由加载方式、权限指令）和数据库规范（逻辑删除字段、树形结构字段）。这些约束不属于某一个 feature，它们横跨所有任务，每次都需要加载。

project-context.md 承载高层信息，对应 vision 层和架构层。同样以 RuoYi-Vue 项目为例，这份文件的实际内容是：

```
Technology Stack
  | Layer      | Technology    | Version |
  | Runtime    | JDK           | 17      |
  | Backend    | Spring Boot   | 4.0.3   |
  | ORM        | MyBatis       | 4.0.1   |
  | Auth       | Spring Security + JWT | - |
  | Cache      | Redis         | -       |
  | DB         | MySQL         | -       |
  | Frontend   | Vue 2         | 2.6.12  |
  | UI Library | Element UI    | 2.15.14 |

Directory Structure
  ruoyi-admin/          # 启动模块 (Controller 层)
  ruoyi-framework/      # 框架核心 (Security, JWT, 全局配置)
  ruoyi-system/         # 系统业务 (Service + Mapper)
  ruoyi-common/         # 公共模块 (工具类, 基类, 注解)
  ruoyi-ui/             # 前端 Vue 2 项目

Code Patterns
  Controller: XxxController extends BaseController, @RestController
  Service: IXxxService (接口) → XxxServiceImpl (实现)
  Permission: @PreAuthorize("@ss.hasPermi('module:entity:action')")
  Response: AjaxResult {code, msg, data} / TableDataInfo {total, rows}

Critical Rules
  - 新增业务在 ruoyi-system 添加 Service/Mapper，Controller 放 ruoyi-admin
  - 实体类继承 BaseEntity 获得审计字段
  - 所有 Controller 接口需配置权限注解
  - 分页使用 startPage() + getDataTable()
```

这份文档限制在 200 行以内。项目上下文每次都被加载到 Agent 的 context 里，太长会挤压留给 spec 和代码的空间。200 行足够放索引级别的内容：技术栈、目录结构、关键规则、代码模式。实现细节留给具体的 feature 文档。

每次完成一个 feature，如果引入了新的技术栈组件、新的代码模式或新的目录结构，project-context.md 会增量更新。

每个 feature 有自己独立的三份文档：spec.md、task.md、checklist.md。这对应 feature 层和 task 层，只在 Agent 执行该 feature 时才被加载，完成后归档。

加载策略：CLAUDE.md 和 project-context.md 每次都加载，feature 文档只在执行对应 feature 时加载。加载不相关 feature 的文档只会制造噪声。

### Spec 模板、交叉验证和拆分

每个 feature 的 spec.md 模板把三个维度转化成具体字段：意图维度对应"需求描述""用户价值点""用户故事"三个字段，验收维度对应 Gherkin 场景（每个价值点至少一个正常路径和一个异常路径），约束维度对应"上下文分析"字段（参考代码、相关文档、历史需求）。后面的 OKR 案例会展示这些字段被填入什么内容。

三份文档（spec、task list、checklist）构成交叉验证体系。Spec 是第一次理解（需求到方案），task list 是第二次（方案到步骤），checklist 是第三次（步骤反推验收标准）。当需求的用户价值点超过三个，系统建议按用户价值拆分，每个子 feature 有自己的三份文档，进入同样的迭代循环。

这些是空壳。下面看一个真实需求跑过这套体系后产出了什么。

## 一个需求的完整 Walkthrough：OKR 系统

这个案例来自社区成员在 RuoYi-Vue 项目上的实践。输入是一句话的需求，产出是一份经过两轮迭代修正的产品规格书，最终驱动生成了 12 个后端文件、4 个前端文件，通过了 26/28 项合规检查和 7 个 E2E 测试。

一个重要的说明：这个案例用一句话作为输入，是因为它是一个边界清晰、规模可控的 demo 级需求。在正式的产品开发中，输入应该是经过产品流程（vision → user journey → 需求拆分）之后的结构化需求。一句话能跑通，是因为 OKR 系统的业务逻辑足够简单，Agent 能从一句话中推导出大部分决策。复杂的业务需求不具备这个条件，缺少上游的产品工作直接给 Agent 一句话，结果大概率会偏。

### 输入

需求："实现一个OKR系统，支持按季度配置每个员工的O和KR。"

### Agent 展开

Agent 从这句话中识别出三个核心 feature：目标管理（Objective Management）、关键结果管理（Key Result Management）、OKR 季度总览（Quarter Overview）。每个 feature 有独立的 user story 和 acceptance criteria。以下是 Agent 生成的 spec 中"目标管理"这个 feature 的实际内容：

```markdown
### Feature: 目标管理 (Objective Management)

**User Story**: As a 管理员, I want to 为员工创建季度目标
  so that 团队可以明确每个人的工作方向。

**Acceptance Criteria**:
- [ ] 新增目标时，必须选择员工、季度和填写目标标题
- [ ] 目标列表支持按季度和员工姓名筛选
- [ ] 可以编辑目标的标题、状态
- [ ] 删除目标时，同时删除该目标下所有关键结果
- [ ] 目标列表中展示每个目标关联的KR数量

**Key Rules**:
- 同一员工在同一季度下可以创建多个目标，无数量上限
- 目标标题最大长度200字符
- 季度下拉选项范围：从上一年Q1到下一年Q4，默认选中当前季度
- 新建目标的默认状态为"未开始(0)"
```

注意这份 spec 的几个特征。User story 用标准的"As a / I want to / so that"格式，锁定了谁、要什么、为什么。Acceptance criteria 的每一条都是一个可检查的条件，有明确的 pass/fail 判断标准。Key rules 定义了具体的业务约束（数量限制、字符限制、默认值）。这三个部分分别对应前面讲的意图、验收、约束三个维度。

Agent 生成 spec 时做的第一步是锁定术语。它在 spec 开头放了一张术语表：

```markdown
| Term | Definition |
|------|-----------|
| Objective (O/目标) | 一个季度内某员工需要达成的定性目标 |
| Key Result (KR/关键结果) | 衡量目标达成程度的量化指标 |
| 季度 (Quarter) | 格式为 YYYY-QN，例如 2026-Q2 |
| 完成率 | 单个KR的完成百分比，取值范围0~100，手动填写的整数值 |
| 目标状态 | 生命周期状态：未开始(0)、进行中(1)、已完成(2) |
```

"完成率"被定义为"手动填写的整数值"。如果不锁定这个定义，后续实现中 Agent 可能在不同地方做不同的假设：完成率是自动计算的还是手动填的？是百分比还是小数？是实时更新的还是定期快照？术语表在 spec 最前面出现，确保后面所有字段和场景用的是同一套语言。

Agent 还为核心操作生成了用户流程（User Flow），把验收维度从"条件列表"扩展为"操作序列"：

```markdown
### 创建OKR流程

**Trigger**: 管理员进入目标管理页面，点击"新增"按钮

**Steps**:
1. 系统弹出新增目标对话框（员工下拉、季度下拉、目标标题输入）
2. 管理员填写信息，点击"确定"
3. 系统创建目标记录，提示"新增成功"
4. 管理员在目标列表中点击操作列的"新增KR"按钮
5. 系统弹出新增KR对话框（KR标题、目标值）
6. 管理员填写KR信息，点击"确定"

**End State**: 目标列表中显示新目标，该目标下有一条KR记录
```

约束维度方面，Agent 生成了 API Contract 表，界定前后端的通信边界：

```markdown
| Endpoint | Method | URL | Parameters |
|----------|--------|-----|-----------|
| 目标列表 | GET | /system/okrObjective/list | quarter, employeeName |
| 新增目标 | POST | /system/okrObjective | Body: domain object |
| 修改目标 | PUT | /system/okrObjective | Body: domain object |
| 删除目标 | DELETE | /system/okrObjective/{ids} | — |
| KR列表 | GET | /system/okrKeyresult/list | objectiveId (required) |
| 季度总览 | GET | /system/okrObjective/overview | quarter (required) |
```

Agent 还生成了 Out of Scope 列表，明确排除了 8 项功能（员工自助录入、OKR 评分考核、目标对齐、历史追踪、导出、通知、多部门视图、权限细分），每项标注了排除理由。以及 5 条编号的业务规则：

| # | Rule | Condition | Outcome |
|---|------|-----------|---------|
| BR-001 | 级联删除 | 删除一个目标时 | 该目标下所有KR同步删除 |
| BR-002 | KR数量限制 | 目标下已有5个KR时 | 新增KR按钮禁用 |
| BR-003 | 完成率范围 | 填写完成率时 | 只接受0~100的整数 |
| BR-004 | 默认季度 | 打开新增目标或总览页面时 | 默认选中当前季度 |
| BR-005 | 总览只读 | 在OKR季度总览页面 | 所有数据仅展示 |

从一句话到这份 spec，信息量增加了大约 50 倍。但新增的信息是从那一句话中推导出来的决策，每个字段都在回答一个如果不回答就会在实现阶段产生歧义的问题。这些问题在一句话的需求里全部是隐含的。Agent 的工作是把它们显式化。

### 迭代：两轮 Review

第一版 spec 有 5 个问题：1 个 Critical、2 个 Major、2 个 Minor。

以下是第一轮 review 发现的 Critical 问题和决策记录的实际内容：

```markdown
#### Decision 1: CRUD API endpoints completely unspecified
- **严重度**: Critical
- **问题**: Spec 描述了三个 feature 的行为，但没有给出任何 API 契约
- **选项**:
  - A) 添加完整的 API 定义（含 request/response schema）
  - B) 只列端点 URL 和 HTTP 方法，实现遵循 RuoYi BaseController 标准
- **决策**: B
- **理由**: 基于最小复杂度原则，RuoYi 的 CRUD 模式高度标准化
```

如果直接交给编码 Agent 而不定义 API 契约，它会自己编造路径和参数格式，前端调用后端的时候对不上。这个项目的决策原则按优先级排列为：最小复杂度 > Demo 可演示性 > 原始意图 > 一致性。大部分决策都落在"最小复杂度"上。

第一轮另外两个 Major 问题的决策记录：

```markdown
#### Decision 2: KR management UI location is ambiguous
- **严重度**: Major
- **选项**:
  - A) 展开行：每个目标行有可展开的子表格显示 KR
  - B) 详情页：点击目标行跳转到独立页面
- **决策**: A（展开行）
- **理由**: 基于最小复杂度，展开行不需要新增路由页面；
          基于 Demo 可演示性，同一页面内完成操作更紧凑直观

#### Decision 3: Objective status has no transition rules
- **严重度**: Major
- **选项**:
  - A) 自由编辑：状态是简单的下拉选择
  - B) 自动派生：状态根据 KR 完成率自动计算
- **决策**: A（自由编辑）
- **理由**: 基于最小复杂度，自由下拉不需要实现状态机逻辑
```

第二轮 review 发现了 4 个问题（0 个 Critical、2 个 Major、2 个 Minor）。问题的严重度在下降，说明 spec 在收敛。第二轮的 Major 问题是："员工"跟 RuoYi 系统里的 sys_user 是什么关系（决策：直接复用 sys_user，存 user_id 做 FK，join 查询 nick_name），以及"新增KR"按钮应该放在目标行的操作列里始终可见还是放在展开行内部（决策：操作列，始终可见，操作路径更短）。

这 9 个决策中的每一个，如果不在 spec 阶段做出，就会在编码阶段由 Agent 自己做。Agent 的决策可能是合理的，也可能不是。关键问题是：你不知道它做了什么决策，直到你看到代码。在 spec 阶段做一个决策的成本是改一行文字。在代码阶段发现决策不对的成本是改几十行代码加重新测试。

回顾整个链路，人在两轮 review 中对 9 个问题做产品决策，每个决策花几分钟看选项、选一个、确认，总共大概半小时。后面的代码生成、合规检查、E2E 测试，人没有介入。这就是前面讲的分工在实践中的样子：人做意图对齐的判断，Agent 做一致性检查和执行。

### 从 Spec 到实现

Spec 收敛后进入实现。Agent 根据 spec 创建了 2 张数据库表（biz_okr_objective 和 biz_okr_key_result）、12 个后端文件（domain、mapper、service、controller 各两套，加上 XML 映射文件）、4 个前端文件（API 封装和页面组件）和 1 个 SQL 迁移脚本。

实现完成后，合规检查逐项对照 spec 验证。以下是合规报告的摘要：

```markdown
## Acceptance Criteria (13/13 PASS)

| # | Criterion | Status |
|---|-----------|--------|
| AC-1 | 新增目标时，必须选择员工、季度和填写目标标题 | PASS |
| AC-2 | 目标列表支持按季度和员工姓名筛选 | PASS |
| AC-3 | 可以编辑目标的标题、状态 | PASS |
| AC-4 | 删除目标时，同时删除该目标下所有关键结果 | PASS |
| AC-5 | 目标列表中展示每个目标关联的KR数量 | PASS |
| ... | （其余 8 条 AC 全部 PASS） | PASS |

## Business Rules (5/5 PASS)

| # | Rule | Status | Notes |
|---|------|--------|-------|
| BR-001 | 级联删除 | PASS | @Transactional cascade delete |
| BR-002 | KR数量限制(max 5) | PASS | Fixed: backend ServiceException + frontend warning |
| BR-003 | 完成率范围0~100 | PASS | el-input-number enforces |
| BR-004 | 默认季度 | PASS | Fixed: handleAdd() sets form.quarter=currentQuarter |
| BR-005 | 总览只读 | PASS | No edit buttons on overview |
```

注意 BR-002 和 BR-004 的 Notes 列有"Fixed"标记。这意味着第一次合规检查时这两条没有通过，修复后才变成 PASS。BR-002 的问题是 KR 数量限制只有前端的 Element UI 提示，缺少后端的 ServiceException 校验。如果绕过前端直接调 API，就能突破 5 个的限制。BR-004 的问题是新增目标时季度下拉框没有自动选中当前季度。

这类问题的模式很有代表性：spec 写对了，但代码没跟上。Agent 在实现时遗漏了 spec 中某条规则的全部含义。合规检查的价值在于它用 spec 作为标准，逐条比对实现，把这类遗漏在交付前暴露出来。

7 个 E2E 测试全部通过，覆盖了从新增目标到删除目标的完整用户流程。以下是其中两个测试的代码片段：

```typescript
test('01: 新增目标', async ({ page }) => {
  await navigateTo(page, 'OKR管理', '目标管理');
  await page.locator('button:has-text("新增")').click();
  await waitForDialog(page);
  // 选择员工
  await page.locator('.el-dialog__body input[placeholder="请选择员工"]').click();
  await page.locator('.el-select-dropdown:visible .el-select-dropdown__item')
    .first().click();
  // 验证季度默认值（对应 BR-004）
  const quarterInput = page.locator('.el-dialog__body input[placeholder="请选择季度"]');
  await expect(quarterInput).toHaveValue(getCurrentQuarter());
  // 填写标题并提交
  await page.locator('.el-dialog__body input[placeholder="请输入目标标题"]')
    .fill('提升客户满意度');
  await page.locator('.el-dialog__footer:visible button:has-text("确 定")').click();
  await expect(page.locator('.el-message--success')).toBeVisible({ timeout: 5000 });
  await expect(page.locator('.el-table__body-wrapper'))
    .toContainText('提升客户满意度');
});

test('07: 删除目标(级联删除)', async ({ page }) => {
  // 对应 BR-001：删除目标时同步删除所有 KR
  await navigateTo(page, 'OKR管理', '目标管理');
  await page.locator('.el-table__body-wrapper tbody tr').first()
    .locator('button:has-text("删除")').click();
  await page.locator('.el-message-box__btns button:has-text("确定")').click();
  await expect(page.locator('.el-message--success')).toBeVisible({ timeout: 5000 });
  const rows = page.locator('.el-table__body-wrapper tbody tr');
  await expect(rows).toHaveCount(0);
});
```

这些测试直接对应 spec 的 acceptance criteria 和 business rules。测试 01 验证了 AC-1（新增目标的必填字段）和 BR-004（默认季度）。测试 07 验证了 BR-001（级联删除）。测试怎么设计、怎么防止 Agent 自己出题自己答，这些属于验证的话题，下一章展开。

## 本章小结

规约的本质是意图对齐。Vibe coding 的失败源于意图活在对话里，而对话是一个会膨胀、会矛盾、会丢失内容的载体。Agent 有限的 context 和不均匀的注意力，让意图对齐从一个沟通问题变成了一个工程问题。

这个工程问题的解法建立在两个基础上。信息天然有层级，不同层级的演进速度和适用范围不同，你需要用结构化的方式把它们拆分到不同的文档里，高层持久加载，任务层按需加载。在每一层，你用三个维度（意图、验收、约束）表达清楚，其中验收维度的核心作用是检测当前层跟上层的对齐有没有丢失。

Spec 是迭代出来的。你写意图，Agent 展开，交叉验证暴露问题，你修正或拆分，再验证，直到收敛。在这个循环里，只有人能判断意图是否对齐，因为意图只存在于人脑子里。循环该跑多重取决于做错了返工的代价有多大。

OKR 系统的案例展示了这条链路的实际运行：一句话需求经过结构化展开和两轮迭代修正，变成了一份覆盖三个维度的可执行 spec。在这个过程中，人的投入集中在 spec review 阶段的产品决策，后面的实现和验证由 Agent 完成。

规约解决的是"做什么"的问题。但这个案例里的合规检查暴露了一个事实：做对了 spec 不等于做对了代码。Spec 写了"默认选中当前季度"，代码里没实现。Spec 写了"最多 5 个 KR"，后端校验漏掉了。怎么系统化地验证 Agent 的产出跟 spec 一致，下一章展开。
