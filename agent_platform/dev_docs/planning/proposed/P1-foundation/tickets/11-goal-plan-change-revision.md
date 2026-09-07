# P1-11：Goal/Plan change → Proposal/Decision/new revision

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-10
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/plan-compiler.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - HumanCollaboration.GoalChangePort
  - PlanCompiler.PlanProposalPort
  - ContextCompiler.PlanningContextPort
contracts_to_create:
  - AmendGoalRequest
  - PlanProposal
  - PlanPatch
  - ChangeImpactAnalysis
  - UserDecision
  - GoalRevision
  - PlanRevisionSupersededEvent
input_artifacts:
  - { artifact: safe-point-acknowledgement, source: upstream, producer: P1-10 }
  - { artifact: persisted-goal-event-stream, source: upstream, producer: P1-01 }
  - { artifact: accepted-plan-revision, source: upstream, producer: P1-02 }
  - {
      artifact: revision-bound-evidence-bindings,
      source: upstream,
      producer: P1-04,
    }
  - {
      artifact: goal-or-plan-change-request,
      source: external_input,
      producer: user,
    }
output_artifacts:
  - affected-context-refresh-results
  - bounded-change-proposal
  - affected-subgraph-analysis
  - authorized-user-decision
  - new-goal-or-plan-revision
verification:
  - affected-context-refresh-test
  - affected-subgraph-selection-tests
  - decision-authority-tests
  - revision-cas-tests
  - evidence-applicability-recompute-test
  - versioned-planning-interface-contract-tests
  - bounded-planning-context-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-10 Pause/resume/cancel and safe steer](./10-lifecycle-controls-safe-steer.md)。

## What it delivers

本轮复核区分“已有目标内的分工／测试／返工调整”与“改变目标含义或验收”。前者可由协调者在已有授权内驱动框架，不能全部强制走用户决定；后者继续核对当前授权。本文 UserDecision 路径保留为未委托变更的候选，新增委托策略路径需单独明确契约后同步。

用户提出会改变 Goal objective、范围或 AcceptanceObligation 的请求。平台在安全点暂停受影响子图，PlanCompiler 产生受限 Proposal/Patch 与影响分析；只有有权限 Decision 被 ControlEngine 接受后才创建新 Goal/Plan revision，并在控制台展示保留与失效的工作。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 提交 amend request 与 UserDecision；
- `PlanCompiler.request/accept`，语义工作经 Control durable intent 与 Dispatch 执行；
- `ContextCompiler.assemble(PlanningContextRequest)`；
- [ControlEngine](../../../../modules/control/control-engine.md) 执行 authority、CAS 和 revision guard；
- [StateLedger](../../../../modules/data/state-ledger.md) 保存 Proposal、Decision 与新 revision；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 投影影响与版本变化；
- 本票首次冻结 HumanCollaboration goal-change、PlanCompiler plan-proposal 与 ContextCompiler planning-context 三个最小 Interface；后续规划或变更票只能消费这些版本或提交显式版本升级。本票同时创建 amend、proposal/patch、impact、decision 和 revision contracts。

## Acceptance

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 变更影响分析列出受影响工作 Context、旧假设及待刷新材料；框架登记刷新／重建结果后才继续相关动作，独立工作可继续。
- 新任务／版本显式引用相关前沿和理由；拒绝／延后不实施提案。架构／接口变更必须上报，用户决定与通知义务分开；完整跨包语义场景由 P1-15 验证。


扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。本票冻结 PlanCompiler 有界协调的请求／结果契约及 planning-context；测试重复结果、过期来源和无授权提案不改变正式 revision，Context 缺口通过有预算的工作补充，不在编译器内部启动模型。

- 只有真正受影响的 RuntimeExecutionDAG 子图暂停；不受影响 Task 可以继续；
- Proposal/Patch 绑定 source Goal/Plan/Workspace/Baseline/Policy revision，并具有明确 in-scope/out-of-scope；
- Planner 和 HumanCollaboration 都不能直接激活新 revision；
- reject 或未授权 Decision 不改变 active revision；
- accepted Decision 的 subject、outcome、actor、authority 与 authorized target 完整匹配后，ControlEngine 才以 CAS 创建并激活新 revision；
- 被替代 revision 和历史 FAIL 保留；Evidence 只通过新 binding 重新计算 applicability；
- 新 revision 的 required 集合重新通过 P1-02/P1-05 guards；
- 控制台显示哪些 Task 保留、取消、替代、重新验证或恢复执行。

---

## Implementation record（2026-09-07，integrator；Ticket 状态保持 proposed——由开发流程依据 Evidence 更新）

- **验收**：产品根 main **6d70494**；typecheck 0；全量 145 文件 / **1025 tests PASS（0 skip）**；
  双适配器同套件 10/10 + 10/10；真实 SQLite 集成 2/2；restart 等价 1/1（close→reopen 同 DB：view/observedCursor/goal revision 一致）；
  P1-11-EVIDENCE 块 1/1；doc validate-docs 13/13。
- **证据**：dev_docs/verification/p1-11-implementation-evidence.md（验收映射 + 裁决记录）。
- **范围**：三个最小 Interface 首次冻结（HumanCollaboration.GoalChangePort / PlanCompiler.PlanProposalPort /
  ContextCompiler.PlanningContextPort）；apply fold = [PlanRevisionAccepted, PlanRevisionSuperseded, GoalRevisionRecorded]；
  任务集合在本票不可变；委托策略路径不在本票（P0-06 复核：需单独明确契约后同步）。
- **Gate**：G4（= P1-09 + P1-11）在本票验收后**成立**（G4 证据：dev_docs/verification/g4-gate-evidence.md）。
- **授权与连续性**：有限授权（2026-09-06 连续窗口）；本运行继续推进下一票（P1-13 或按 DAG 顺序）；GitHub 推送仍需用户授权。
