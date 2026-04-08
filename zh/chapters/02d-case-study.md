# 实践：AILock-Step Feature Workflow

前面几节建立了规约的方法论框架：信息分层、三个维度、交叉验证、迭代循环。这一节用一个具体的工作流系统来展示这些概念在实践中怎么落地。

AILock-Step Feature Workflow 是我们社区成员开发的一个基于 Claude Code 的文档驱动开发框架。它用 skill（slash command）编排整个 feature 的生命周期，从需求进入到代码交付。这个系统已经在一个 AI agent 平台项目上跑过 40 多个 feature。完整源码在 GitHub 上公开。

这里展示的是一个实现，不是唯一的实现。重要的不是照搬它的目录结构或配置格式，而是观察它怎么把前面讲的 principle 变成可执行的工作流。

## 信息分层的落地

前面讲了信息有四个层级（vision、架构、feature、task），不同层级的演进速度和适用范围不同，应该用不同的文档承载、按不同的策略加载。AILock-Step 用三类文档实现了这个分层。

**CLAUDE.md 承载跨层的强制约束。** 这是 Claude Code 每次启动时自动加载的文件。它的内容是整个项目生命周期内都不太变的硬性规则：必须用 TypeScript 严格模式、禁止直接操作数据库、commit message 的格式要求、禁止在代码中硬编码配置。这些约束不属于某一个层级，它们横跨所有层级，所有任务都必须遵守。因为内容稳定且每次都需要，放在自动加载的位置。

**project-context.md 承载高层信息。** 这对应前面讲的 vision 层和架构层。它的内容包括：

```
Technology Stack（技术栈表）
  | Category | Technology | Version | Notes |
  | Frontend | React      | 18.x    | ...   |
  | Backend  | Node.js    | 20.x    | ...   |
  | Database | PostgreSQL | 15      | ...   |

Directory Structure（目录结构）
  src/
  ├── components/     # 共享组件
  ├── pages/          # 页面路由
  ├── services/       # API 服务
  ├── hooks/          # 自定义 hooks
  └── utils/          # 工具函数

Critical Rules（关键规则）
  Must Follow:
  - 所有 API 返回值使用统一的 ResponseWrapper 格式
  - 数据库操作必须通过 ORM 层，禁止裸 SQL
  Must Avoid:
  - 不要在组件中直接调用数据库
  - 不要在 utils/ 中放业务逻辑

Code Patterns（代码模式）
  命名规范、import 风格、错误处理模式

Testing Patterns（测试模式）
  单元测试位置、命名规则、E2E 框架

Recent Changes（最近变更）
  最近几个 feature 的改动和影响
```

这份文档有一个硬性限制：200 行以内。这个限制直接来自前面讲的 context 约束。项目上下文每次都会被加载到 Agent 的 context 里，如果它太长，会挤压留给 spec 和代码的空间。200 行足够容纳一个中等规模项目的关键信息，但要求你只放索引级别的内容，而不是把实现细节全写进去。

这份文档不是写一次就不变的。每次完成一个 feature，如果引入了新的技术栈组件、新的代码模式或新的目录结构，project-context.md 会增量更新。文档底部有一个 Update Log 记录每次更新的内容和日期。这对应前面讲的"高层信息偶尔变"。

**每个 feature 有自己独立的三份文档：spec.md、task.md、checklist.md。** 这对应 feature 层和 task 层。这些文档只在 Agent 执行该 feature 时才被加载，完成后归档。

这个三层结构的加载策略很清晰：CLAUDE.md 和 project-context.md 每次都加载（它们是所有任务都需要的背景信息），feature 文档只在执行对应 feature 时加载（它们是任务级的信息，加载不相关 feature 的文档只会制造噪声）。

## 三个维度在 Spec 模板中的体现

AILock-Step 的 spec.md 模板把前面讲的三个维度（意图、验收、约束）转化成了具体的字段。

**意图维度**体现在三个地方。"需求描述"字段用自然语言描述要解决什么问题。"用户价值点"字段列出这个 feature 包含的独立用户价值，由 Agent 分析生成。"用户故事"字段用标准格式表达：作为 [角色]，我希望 [目标]，以便 [价值]。

**验收维度**体现在 Gherkin 场景中。模板要求为每个用户价值点生成至少一个正常路径场景和一个异常场景。比如用户注册功能会有两个场景：

```gherkin
# 场景 1: 注册成功
Given 用户在注册页面
When 用户输入有效的用户名 "testuser" 和密码 "Test123!"
And 点击注册按钮
Then 账户创建成功
And 显示欢迎信息

# 场景 2: 用户名重复
Given 用户在注册页面
And 用户名 "testuser" 已存在
When 用户输入用户名 "testuser" 和密码 "Test123!"
And 点击注册按钮
Then 显示错误提示 "用户名已存在"
```

这些场景就是前面讲的验收维度：每个场景在检查 feature 的行为是否跟意图一致。正常路径验证"做了该做的事"，异常路径验证"遇到问题时行为合理"。

**约束维度**体现在"上下文分析"字段中，包含三部分内容：需要参考的现有代码（项目里已有的可以复用的模块）、相关文档（设计文档、API 文档）、相关历史需求（之前做过的类似功能）。这个字段告诉 Agent 改动的背景和边界：什么可以用、什么已经做过了、什么不该碰。

模板里还有一个"技术方案"字段，标注为"开发过程中填写"。这个字段在 spec 生成阶段是空的，在 Agent 实际开发时才填入具体的实现方案。这对应前面分层原则里"低层信息在需要时才生成"的思想。

## 交叉验证的落地

Spec、task list、checklist 三份文档的分工直接对应前面讲的交叉验证。

Spec 是第一次理解：从需求出发，生成意图描述、验收场景和约束分析。

Task list 是第二次理解：从 spec 出发，拆解成具体的执行步骤。模板按模块/组件、API 接口、前端页面、其他四个类别组织任务项，每个任务项是一个 checkbox。底部有进度记录表，Agent 在执行过程中更新。

Checklist 是第三次理解：从"怎么做"反推"怎么算做对了"。模板包含五个检查类别。开发完成（所有任务是否完成、边界情况是否处理）。代码质量（风格是否规范、有没有代码坏味道）。测试（单元测试是否编写、是否通过）。文档（spec 的技术方案是否已填写、相关文档是否更新）。提交准备（变更是否暂存、commit message 是否就绪）。

如果 checklist 里出现了 spec 没有提到的检查项，说明 Agent 在 task 阶段引入了 spec 没有覆盖的内容。如果 checklist 某一项跟 spec 的描述矛盾，说明意图在传递过程中发生了偏移。三份文档之间的一致性就是对齐的证据。

## 拆分的实际过程

当 /new-feature 这个 skill 分析出一个需求包含三个以上的用户价值点时，系统会建议拆分。这个过程在测试中有完整的记录。

输入一个需求："用户认证系统，支持注册、登录和权限管理。"Agent 分析出三个独立的用户价值点：用户注册（创建新账户）、用户登录（访问系统）、权限管理（控制访问权限）。因为用户价值点等于 3，触发拆分建议。

系统展示拆分方案：原始需求拆成三个子 feature。feat-auth-register（用户注册，无依赖）、feat-auth-login（用户登录，依赖 feat-auth-register）、feat-auth-permission（权限管理，依赖 feat-auth-login）。依赖关系由系统自动设置。

确认拆分后，系统为每个子 feature 创建独立的目录，每个目录包含自己的 spec.md、task.md 和 checklist.md。同时在 queue.yaml 里记录三个子 feature 的状态、优先级和依赖关系，以及一个父需求条目追踪整体进度。

执行引擎在启动一个 feature 之前会检查它的依赖是否已完成。如果你试图启动 feat-auth-login 但 feat-auth-register 还没完成，系统会阻止并提示"依赖未满足"。

每个子 feature 进入同样的迭代循环：Agent 展开 spec，交叉验证，人 review 用户故事，收敛后执行。

## 一个需求的完整 Walkthrough

> **TODO**: 这里需要一个真实的、已完成的 feature 作为案例。需要展示：填好的 spec（包含实际的用户故事和 Gherkin 场景），对应的 task list（包含实际的任务项），对应的 checklist（包含实际的检查项），以及迭代过程中发现和修正问题的记录。待从实际项目中获取素材后补充。

## 人的实际时间投入

在整个工作流中，开发者实际花时间的地方集中在一个点：spec 生成后 review 用户故事和 Gherkin 场景。确认 Agent 理解的"这个功能为谁解决什么问题"跟自己的意图一致。如果一致，放手让 Agent 继续后面的 task 拆分、checklist 生成和代码实现。

这验证了前面讲的人在迭代中的角色：意图对齐只能由人来做，一致性检查可以交给 Agent。开发者的注意力集中在最高 ROI 的检查点上（上层方向），下层的一致性由 Agent 的交叉验证覆盖。

完整实现的源码在 GitHub 上公开（[AILock-Step Feature Workflow](https://github.com/auenger/AILock-Step/tree/feature/dev-agent-subagent-optimization/feature-workflow)）。这里展示的只是一个实现方案。读者不需要使用同样的工具或同样的目录结构。关键是理解它背后的 principle：信息分层管理、每层用三个维度表达清楚、用交叉验证检测层间漂移、迭代直到收敛。用什么工具实现这些 principle，取决于你的项目和团队。

## 本章小结

规约的本质是意图对齐。Vibe coding 的失败源于意图活在对话里，而对话是一个会膨胀、会矛盾、会丢失内容的载体。Agent 有限的 context 和不均匀的注意力，让意图对齐从一个沟通问题变成了一个工程问题。

这个工程问题的解法建立在两个基础上。信息天然有层级，不同层级的演进速度和适用范围不同，你需要用结构化的方式把它们拆分到不同的文档里，高层持久加载，任务层按需加载。在每一层，你用三个维度（意图、验收、约束）表达清楚，其中验收维度的核心作用是检测当前层跟上层的对齐有没有丢失。

Spec 是迭代出来的。你写意图，Agent 展开，交叉验证暴露问题，你修正或拆分，再验证，直到收敛。在这个循环里，只有人能判断意图是否对齐，因为意图只存在于人脑子里。循环该跑多重取决于做错了返工的代价有多大。

规约解决的是"做什么"的问题。但做对了 spec 不等于做对了代码。Agent 产出的代码到底对不对，怎么验证，下一章展开。
