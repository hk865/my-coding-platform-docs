# P1-08：Read-only status and Evidence console

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-05
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/data/read-model-index.md
interface_refs:
  - ../../../../interfaces/goal-view.md
contracts_to_create:
  - PortfolioViewQuery
  - WorkspaceSummaryView
  - WorkspaceSelectionRoute
  - PlanMatrixViewQuery
  - ActiveAgentsViewQuery
  - TaskEvidenceViewQuery
  - TimelineViewQuery
input_artifacts:
  - {
      artifact: versioned-workspace-bootstrap-manifest,
      source: upstream,
      producer: P1-00,
    }
  - { artifact: deterministic-goal-phase, source: upstream, producer: P1-05 }
  - {
      artifact: task-detail-verification-view,
      source: upstream,
      producer: P1-04,
    }
  - { artifact: goal-timeline-events, source: upstream, producer: P1-05 }
output_artifacts:
  - read-only-console-adapter
  - portfolio-and-workspace-view
  - status-and-evidence-view
  - cursor-bound-view-snapshots
verification:
  - portfolio-list-and-switch-test
  - multi-project-workspace-isolation-test
  - same-local-id-scope-test
  - no-model-status-query-test
  - read-only-adapter-test
  - cursor-freshness-test
  - restart-view-rebuild-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：原只读控制台不足以覆盖统一图文界面。需明确状态图、事实来源与文本解释的配对，以及与 QueryJob／参谋协作的分工，再修订交付和验收。

- [P1-05 Goal phase reduction](./05-goal-phase-reduction.md)。

## What it delivers

用户在一个最小控制台中列出至少两个已初始化且彼此隔离的 Project/Workspace，切换当前查看范围，并只读查看该范围内的 Goal、PlanMatrix、Task、ActiveAgents、Evidence 和 Timeline。用户能从 displayed phase 追到当前 revision 与来源。范围切换只是 ReadModel 查询路由，不修改 canonical state；机械状态查询不启动模型，也不提供 pause、steer、QueryJob 或目标修改入口。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 的只读展示 Adapter；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 的 query Interface；
- [Goal View Interface](../../../../interfaces/goal-view.md) 的 cursor/freshness 语义；
- 本票创建 Portfolio、WorkspaceSummary/Selection、PlanMatrix、ActiveAgents、TaskEvidence 与 Timeline query contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。扩展角色／Run／报告引用投影，正式状态与未验证报告分开标记；验证纯事实查询不调用模型，跨 scope 或过期解释不能混入当前事实。

- Portfolio 至少列出 bootstrap manifest 中两个彼此隔离的 Project/Workspace，用户可以选择和切换当前查看范围；
- 两个 Project 可以复用相同本地 `workspaceId/goalId`；列表、路由、查询结果与缓存键始终使用完整作用域，不能串读 Goal、Evidence、AgentRun 或计数；
- 用户可以从所选 Workspace 进入一个 Goal，并查看当前 Plan、Task、AgentRun、Evidence 与 Timeline；
- 列出与切换 Workspace 只改变展示查询范围，不写 canonical state、不创建隐式“当前 Workspace”领域事实；
- 每个 displayed phase 带 source revision/cursor，完成结论可追到当前 EffectiveEvidenceSet；
- 机械状态查询只访问 ReadModel，不启动模型、不刷新 Worker lease；
- 控制台 Adapter 不能直接读取 SQLite 表、写 Todo 或修改 canonical state；
- projection 落后时显示 `not_ready` 或等价 freshness 状态，不回显请求伪装成功；
- 平台重启后相同事件重建出等价 View；
- 本票没有隐藏的控制、QueryJob 或 Planner side effect。
