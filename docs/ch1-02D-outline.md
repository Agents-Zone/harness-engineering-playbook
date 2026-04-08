# 02D Outline：Case Study

> 状态：outline
> 日期：2026-04-07

## Section Takeaway

用一个真实项目的完整 walkthrough 展示 02A-02C 的所有内容在实践中怎么串起来。读者看完知道：分层的项目上下文长什么样，一份 spec 从生成到交叉验证到收敛的实际过程，拆分在真实场景里怎么发生，人在哪个环节花了多少时间。

---

## 结构

### 背景介绍
- Ryan Yang，独立开发者，在用 Agent 开发 AnyClaw（AI agent 平台）
- 他的工作流系统 AILock-Step Feature Workflow，经过 40+ feature 实战验证
- 他的演进路线：vibe coding → BMAD（太重）→ 纯 spec（太轻）→ 混合方案
- 简短说明，不展开历史，重点是让读者知道这是真实的、经过验证的

### 信息分层的落地（对应 02B）
- 三类文档对应不同层级：
  - CLAUDE.md：跨层强制约束（行为规则）
  - project-context.md：高层信息（技术栈、架构、模块索引），200 行限制
  - feature 目录下的 spec/task/checklist：任务层信息，按需加载
- 展示 project-context.md 的实际内容结构（技术栈表、目录结构、关键规则、代码模式、测试模式、最近变更）
- 说明加载策略：CLAUDE.md 和 project-context.md 每次加载，feature 文档只在执行该 feature 时加载

### Walkthrough：一个小需求
- 需求："项目设置页面的 Project Context 展示区域，从硬编码文本改成读取真实文件内容"
- 这是一个不需要拆分的中等任务，展示完整的迭代循环
- 步骤：
  1. Ryan 写一句意图描述
  2. Agent 加载 project context，发现已有 IPC 模块可以复用，生成 spec（用户价值点、Gherkin 场景、影响分析标注"不需要新增后端接口"）
  3. Ryan review 用户故事：方向对，继续
  4. Agent 从 spec 生成 task list，从 task list 生成 checklist
  5. Checklist 里明确标注"不需要新增后端接口"，跟 spec 一致，没有矛盾
  6. 收敛，进入执行。Agent 在独立的 Git worktree 里开发

### Walkthrough：一个需要拆分的大需求（简短）
- 需求：如果是"完整的用户认证系统"
- Agent 分析出多个用户价值点（注册、登录、权限管理），超过阈值
- 按用户价值拆成三个子 feature，每个有独立的 spec
- 每个子 spec 进入同样的迭代循环
- 不需要长篇展开，跟小需求的区别只是多了拆分这一步

### Ryan 的实际时间投入
- 强调：整个过程中 Ryan 花时间的地方只有一个：spec 生成后看用户故事
- 后面的 task 拆分、checklist 生成、代码实现，他基本不逐一介入
- 这就是 02C 讲的"人在循环中的角色"的实际体现

### 指向源码
- AILock-Step Feature Workflow 的 repo 链接
- 说明这只是一个实现，读者不需要照搬
- 重要的是理解背后的 principle，用什么工具实现取决于项目和团队

---

## 本章小结
- 回扣全章 takeaway
- 承上启下到验证章节
