# P1-15：需求／架构协商 → 角色协作 → 统一界面回报

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-17
  - P1-06
  - P1-07
  - P1-09
  - P1-14
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/plan-compiler.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/context-compiler.md
interfaces_to_freeze:
  - HumanCollaboration.InitialDesignPort
  - HumanCollaboration.UnifiedStatusPort
  - ControlEngine.CoordinationPolicyPort
contracts_to_create:
  - InitialDesignProposal
  - InitialDesignDecision
  - CoordinationPolicy
  - UnifiedStatusPresentation
input_artifacts:
  - { artifact: architecture-change-notification, source: upstream, producer: P1-14 }
  - { artifact: affected-context-refresh-results, source: upstream, producer: P1-11 }
  - { artifact: reported-interface-conflict-finding, source: upstream, producer: P1-12 }
  - { artifact: completed-work-context-evidence, source: upstream, producer: P1-17 }
  - { artifact: bounded-handoff-packet, source: upstream, producer: P1-06 }
  - { artifact: exclusive-writer-capability, source: upstream, producer: P1-07 }
  - { artifact: provenance-preserving-evidence-join, source: upstream, producer: P1-07 }
  - { artifact: sourced-query-result, source: upstream, producer: P1-09 }
  - { artifact: cas-guarded-baseline-activation, source: upstream, producer: P1-14 }
  - { artifact: new-goal-or-plan-revision, source: upstream, producer: P1-11 }
  - { artifact: real-task-and-human-design-choices, source: external_input, producer: user }
output_artifacts:
  - cross-work-package-decision-feedback
  - sourced-architecture-interface-brief
  - human-confirmed-initial-design
  - versioned-coordination-policy
  - autonomous-role-feedback-trace
  - unified-sourced-status-presentation
verification:
  - cross-package-conflict-before-test-failure
  - decision-feedback-context-refresh-test
  - authorized-change-notification-test
  - completed-work-semantic-continuation-test
  - initial-design-decision-test
  - bounded-role-rework-loop-test
  - policy-authority-and-budget-test
  - unified-view-freshness-test
  - coordinator-rollover-test
```

## Blocked by

- [P1-17](./17-completed-work-context.md)：消费 completed-work-context-evidence。

依赖上述已验收能力，P0-06 未批准前保持 proposed。P1-11 的规划／决定路径通过 P1-14 成为已具备能力；本票不反过来成为基础票前置。

## What it delivers

在同一界面，从一个新目标的需求歧义出发，人和参谋形成方案与明确决定；正式安装规范／初始 Baseline，接受计划；协调者派发有界 Coder 工作、分析验证失败并安排一次授权内返工，最终用图文展示同源状态、证据与决定。测试临时角色和协调者换手，无需共享完整 transcript。

此票是已存在能力的纵向整合与初始协商入口，不重新实现内核、通用流程 DSL 或无限自主规划。

## Module / Interface refs

- [运行时协作](../../../../interfaces/runtime-collaboration.md)；
- [初始设计与统一展示](../../../../interfaces/human-design-status.md)；
- 已冻结的 PlanCompiler、dispatch、Context、query、Decision 与 baseline install/activation 契约；变更时显式升级，不复制私有 schema。

## Acceptance

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 至少两个包工头负责相关工作包，并有指定集成职责；测试尚未失败时发现接口／职责冲突，通过有限交流和调查形成选项。
- 书记保留各方依据与分歧，秘书／参谋向人上报变化与影响；不要求每条消息串行经过全部角色。
- 覆盖拒绝、延后、旧版本决定和已接受变化：仅受影响动作等待，决定经规范／baseline／Plan 路径返回全部受影响工作，材料刷新／重建可追溯。
- 验证已明确授权内的契约变更也主动汇报，可聚合通知且不重复审批；策略本身必须有效，不绕过 P1-14 的 baseline 门禁。
- 消费 P1-17 完成后继承结果；同一真实场景在工作结束后再开相关新任务，识别旧前提不适用。


- 人看到需求歧义、至少两个有实质差别的选项及影响，决定绑定精确 proposal revision；拒绝、延后与旧决定不会激活规范或计划。
- 初始 Baseline 无 source baseline，不伪造 migration；复用 P1-02 的显式 install/activation，并检查 Project 当前无 active ref；已有基线的变更走 P1-14。多产物提交失败时可恢复，不能展示部分完成为整体成功。
- 显式安装有限预算的协调策略；既定义务内测试／返工可自动推进，改变需求、验收或 baseline 的未委托变化必须升级；提示／经验不能扩大策略。
- 按真实任务分工，Coder 失败后协调者依据契约与证据安排一次返工；测试冲突只允许有限澄清，耗尽预算产生可见未解项。
- 自定义短生命周期角色完成局部分析并退出；协调者 rollover 后从事实及 Handoff 接续，独立 Worker 不随之停止。
- 状态图和文本事实引用兼容 revision；解释落后时标 stale，事实可先显示，查询可以绕过参谋。
- G3 的角色协作成功必须包含上述可追溯闭环，P1-07 的并行及唯一 Writer 证据不能单独替代。

## Verification

以可重放协商 fixture 做确定性拒绝／幂等／版本测试，再对用户选定真实任务运行一次模型协作集成，保存本票声明的全部 output_artifacts。记录预算、人工决策次数与返工原因；fixture 不能替代真实任务集成 PASS。

---

## Implementation record（2026-09-07，integrator；Ticket 状态保持 proposed——由开发流程依据 Evidence 更新）

- **验收**：产品根（P1-15 最终 main）；typecheck 0；全量 **174 files / 1193 tests PASS（0 skip / 0 fail）**；双适配器同套件 9+9；集成/restart/evidence 各 1/1；角色回流链 + 真实内核模型 e2e PASS；validate-docs 13/13。
- **证据**：dev_docs/verification/p1-15-implementation-evidence.md；**G3（07+15）PASS**（g3-gate-evidence.md）；**MVP 评议**：dev_docs/verification/mvp-review.md。
- **范围**：初始协商入口（歧义+≥2 选项+精确决定绑定）+ 显式有界协调策略（预算=1 恰一次返工；第二次拒绝+可见未解项）+ unified facts-first 状态视图 + 真实任务模型协作集成（本地 SSE 端点=能力降级证据）。
- **授权与连续性**：有限授权完成；**P1-foundation 全部 17 票（P1-00..P1-17）验收完成，G1..G5 全部 PASS**；GitHub 推送仍需用户授权（本地 main 领先 origin 96 commits）。
