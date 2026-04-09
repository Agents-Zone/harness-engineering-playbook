# Walkthrough Outline: OKR 项目的第二个 Feature

## 目的

用 OKR 项目的第二个 feature（跨部门 OKR 对齐）走一遍文档演进的完整流程。读者看完应该知道"具体怎么做"。

## 前置状态

第一个 feature（OKR 基础 CRUD）已完成并归档。当前 project-context 记录了：
- 已有模块：OKR 目标管理、KR 管理
- 已有业务规则：每个目标最多 5 个 KR（BR-002）、新增页面默认选中当前季度（BR-004）
- 已有 API：目标 CRUD、KR CRUD

---

## Step 1: 新需求到达，创建 change 包

**Takeaway**: 每个变更是一个自包含的 feature 包，从用户故事到验收条件完整覆盖所有层级。不是代码级 diff，是 spec 级的完整描述。

**内容要点**:
- 产品说：支持跨部门的 OKR 对齐，部门负责人可以把自己的目标关联到上级部门的目标
- 创建 change 包目录：`changes/cross-dept-alignment/`
- 包内文件结构（展示目录树）：
  ```
  changes/cross-dept-alignment/
  ├── story.md          # 用户故事：谁，做什么，为什么
  ├── design.md         # 架构影响：新增 alignment 模块，修改目标查询接口
  ├── delta-spec.md     # 对主 spec 的增量修改（ADDED/MODIFIED/REMOVED）
  ├── tasks.md          # 任务分解
  └── checklist.md      # 验收检查项
  ```
- 重点展示 delta-spec.md 的内容格式：
  - ADDED：对齐关系的 CRUD、对齐视图的查询接口、对齐权限（只有部门负责人能操作）
  - MODIFIED：目标查询接口需要返回对齐状态字段
  - REMOVED：无
  - 每条 requirement 带 acceptance scenario（GIVEN/WHEN/THEN）
- 关键原则：delta-spec 不只写"加了什么"，还要显式声明"没改什么"。BR-002（最多 5 个 KR）在这次变更中不受影响，但 delta 中需要注明这条规则仍然有效。这是防止引言场景（规则被静默覆盖）的关键

---

## Step 2: 创建时验证

**Takeaway**: change 包在编码之前就跑验证，这是成本最低的拦截点。发现问题只需要改文档，不需要改代码。

**内容要点**:
- 两层验证：
  - **自动化验证**：delta-spec 中 MODIFIED 的每条 requirement 在主 spec 中是否存在且 base 匹配（防止基于过时 base 修改）。新增的 requirement 是否和已有规则冲突（比如新加的对齐功能是否隐含地修改了 KR 数量限制）。design.md 声明的受影响模块是否覆盖了 delta-spec 中所有变更涉及的模块
  - **人工审查**：用户故事是否准确反映产品意图。架构影响是否完整。delta-spec 的变更范围是否正确（改对了没有，改漏了没有）
- 人是意图的唯一裁判，但自动化验证替人过滤掉了一致性层面的问题，让人聚焦在意图层面
- 审查通过后，change 包成为 Agent 执行的输入

---

## Step 3: 执行闭环（略写）

**Takeaway**: 执行阶段和 02/03 章完全一致，不需要新的方法。

**内容要点**:
- Agent 根据 delta-spec 和 tasks 执行实现
- 验证对照 checklist，测试对照 acceptance scenario
- 这一步不展开，引用 02/03 章

---

## Step 4: 合并 delta 到主 spec + 合并时验证

**Takeaway**: 合并后跑一次文档级的验证，确认新 delta 合入主 spec 后整体仍然一致。这和验证章的测试基建是同一个思路：代码合并后跑测试，spec 合并后也要跑验证。

**内容要点**:
- **合并逻辑**：ADDED 追加到主 spec 对应模块，MODIFIED 替换原有 requirement 的完整内容，REMOVED 删除
- **合并后验证**（callback 到验证章的测试基建思路）：
  - 合并后的主 spec 是否存在内部矛盾（比如新增的对齐功能的权限模型和已有的 KR 编辑权限是否冲突）
  - 新增的 acceptance scenario 和已有 scenario 之间是否有矛盾（比如新接口的返回值格式和已有接口是否一致）
  - 已有的业务规则（如 BR-002）在合并后是否仍然完整存在
- **更新 project-context**：新增"对齐"模块到已有模块列表，更新 API 列表
- 合并后，主 spec 反映系统当前的完整状态

---

## Step 5: 归档 change 包

**Takeaway**: 归档保留完整上下文，让未来的问题可追溯。

**内容要点**:
- change 包移动到 `archive/YYYY-MM-DD-cross-dept-alignment/`
- 归档内容包括：原始的用户故事、架构分析、delta-spec、task list、checklist、执行结果
- 为什么要全部保留：三个月后如果对齐功能出了问题，你能回到这个归档，看到当时的完整决策链路（为什么做、怎么设计、验证了哪些条件）

---

## Step 6: 下一个 feature 开始前的同步检测

**Takeaway**: 同步检测是下一次迭代的安全门。即使前面的验证都通过了，开发过程中仍然可能产生 spec 和代码的不一致，同步检测是最后的兜底。

**内容要点**:
- 第三个 feature 来了（比如：OKR 评分功能）
- Agent 先读 project-context，了解当前系统状态
- 对照主 spec 检查：有没有代码实现了但 spec 没有记录的行为？有没有 spec 声明了但代码没有实现的功能？
- 即使严格走 change 包流程，仍然可能产生不一致：实现过程中 Agent 的残留漂移（验证章讨论过，测试和 review 拦截大部分但不是全部），或者 delta 合并时遗漏了关联模块的更新（比如改了接口返回值，但依赖这个接口的另一个模块的 spec 没有同步）
- 同步检测的结果需要人判断：差异是有意的演进还是无意的遗漏
- 同步检测和 Step 2 的创建时验证形成闭环：Step 2 验证 change 包和主 spec 的一致性，Step 6 验证主 spec 和代码的一致性。两道门覆盖了 spec 生命周期的两端

---

## 篇幅与位置

- 在 Section 3 中，替换目前的三个分散的机制描述
- 总论段保留（代码管理的类比），然后直接进入 walkthrough
- 预估 1000-1500 字
- Step 3 和 Step 5 可以简写（各 2-3 句），重点在 Step 1（change 包结构）、Step 2（创建时验证）、Step 4（合并时验证）、Step 6（同步检测）
