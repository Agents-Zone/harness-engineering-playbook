# 目录

* [关于本书](README.md)
* [引言：为什么需要 Harness Engineering](chapters/00-introduction.md)

### 卷一：可靠的 Agent 编程（1→10x）

* [规约：与 Agent 对齐意图](chapters/02a-intent-alignment.md)
  * [意图对齐：Vibe Coding 为什么失败](chapters/02a-intent-alignment.md)
  * [用结构传达意图：分层与维度](chapters/02b-structured-intent.md)
  * [迭代出一份可执行的规约](chapters/02c-iterative-spec.md)
  * [实践：AILock-Step Feature Workflow](chapters/02d-case-study.md)
* [验证：确保代码忠实于规约](chapters/03-verification.md)
  * [验证的锚点是规约](chapters/03-verification.md)
  * [测试基建前置：把规约变成可执行约束](chapters/03a-test-first.md)
  * [Code Review：补位测试覆盖不到的意图漂移](chapters/03b-code-review.md)
  * [实践：AILock-Step 的验证链路](chapters/03c-practice.md)
* [演进：规约与验证的持续迭代](chapters/evolution-v1.md)
* [卷一回顾：从闭环到演进](chapters/v1-conclusion.md)

### 卷二：规模化 Agent 开发（10→100x）

* [规约：让意图能被并行分发](chapters/04-spec-distributed.md)
  * [spec 从工作台变成载体](chapters/04a-artifact-role.md)
  * [载体的一条原则](chapters/04b-artifact-principle.md)
  * [共同产出与可判定接口](chapters/04c-cocreation.md)
  * [实践：从粗略意图到可用载体](chapters/04d-case-study.md)
* [验证：当人不再能兜底](chapters/05-verification-defense.md)
  * [验证从人转移到机制](chapters/05a-verification-to-mechanism.md)
  * [漂移的两个地点](chapters/05b-drift-locations.md)
  * [三层机制的分工](chapters/05c-three-layers.md)
  * [实践：从载体到通过验证](chapters/05d-case-study.md)
* [演进：harness 必然演化](chapters/evolution-v2.md)

### 卷三：治理百倍速的组织

* [组织重构：当 Agent 改变了协作的前提](chapters/06-hybrid-team.md)
  * [为什么旧结构失效了](chapters/06a-why-old-structure-fails.md)
  * [瓶颈转移：从代码到组织](chapters/06b-bottleneck-shift.md)
  * [让流程匹配 Agent 速度](chapters/06c-process-speed.md)
* [角色重定义：从写代码到设计验证体系](chapters/07-role-redefinition.md)
  * [围绕治理重新设计角色](chapters/07a-new-roles.md)
  * [用机制替代人际协调](chapters/07b-coordination.md)
* [演进：组织资产与新护城河](chapters/evolution-v3.md)

---

* [贡献者](contributors.md)
