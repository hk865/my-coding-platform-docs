# P1-08：Read-only status and Evidence console

```yaml
status: proposed
updated: 2026-09-06
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

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 已有 Run／Handoff 的持续、转交与未知结果按来源展示，不把暂时等待显示为工作完成。议题和主动变更通知由 P1-14／15 扩展，本票不依赖它们。


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

## Implementation record (2026-09-06, P1-08)

- **status**: stays `proposed` (per development workflow — ticket status is updated by the process, not by the implementing agent); acceptance documented at `../../verification/p1-08-implementation-evidence.md` (product root commit 7664d91; 97 files / 768 tests PASS; 12/12 acceptance + 7/7 verification groups; validate-docs 13/13).
- **7 contracts frozen by the first consumer (this ticket)**: PortfolioViewQuery / WorkspaceSummaryView / WorkspaceSelectionRoute / PlanMatrixViewQuery / ActiveAgentsViewQuery / TaskEvidenceViewQuery / TimelineViewQuery — all v1, dual-adapter, opaque-cursor freshness (not_ready != not_found); bounded (timeline 200 / active-agents 100 / matrix 512 / evidence summary 4096B / portfolio 64).
- **versioned additions only**: HumanCollaboration console query group (6 methods, read-only face — pure ReadModelIndex delegation; no Module added; ARCHITECTURE §Plane) + ReadModelIndex same 6 queries. P1-00…P1-07 frozen shapes zero-diff-verified; **0 new DomainEvents** (KNOWN + isHandledEventType unchanged; all console projections consume existing v1 events).
- **two parallel lanes merged** (portfolio/summary 10/10, matrix/agents/evidence/timeline 10/10) + integrator rulings: agentRunCount = claims + replacements; replacement run really started via handoffDrive.driveHandoff (P1-06 path) so "ongoing" rows are sourced from real RunStarted facts; maxEntries frozen as positive-cap; report/formal separation via marker (unverified_report / observed_fact) + modelExplanation four-state (P1-08 = unavailable — no model semantic explanation; deterministic reason codes carry sourceCursor).
- **no hidden control/QueryJob/Planner side effect** (console queries create zero ledger events; TrapControl/TrapStateLedger prove the read-only face); console adapter cannot reach ledger tables (console_* tables exist only in the read-model DB file).
- **G4 (Human Control) waits P1-09 + P1-11** — the P1-08 side is complete; 09/10 need both the P1-08 and P1-16 products; active notification / change reporting = P1-14/15 (read-only passive display only).
- **not pushed**: product root local main (7664d91) is NOT pushed to GitHub origin main (push requires user authorization per P1-04/05/06/07 precedent).

