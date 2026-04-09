# 集成：确保独立产出的一致性

> 🚧 本节正在开发中。

隔离解决了物理冲突，逻辑冲突是下一个问题。两个 Agent 各自在独立的分支上工作，各自的测试全绿，合并后系统崩溃。前端 Agent 假设 API 返回 `{data: [...]}` 格式，后端 Agent 实际返回 `{items: [...]}` 格式，两边的单元测试分别验证了各自的假设，没有人验证这两个假设是否一致。

解决逻辑冲突的核心机制是契约。API 契约（OpenAPI spec、GraphQL schema、接口类型定义）在开发开始前定义，所有 Agent 基于同一份契约工作。集成测试在合并前验证多个模块的协作是否符合契约预期。CI 频率需要跟上 Agent 的产出速度：人类开发时一天跑两三次 CI 就够了，多 Agent 并行时可能需要每次提交都触发。

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
