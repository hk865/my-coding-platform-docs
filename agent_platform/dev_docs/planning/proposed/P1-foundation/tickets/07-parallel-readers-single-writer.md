# P1-07：Parallel Readers + single Writer

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-05
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/dispatch-engine.md
  - ../../../../modules/control/verification-engine.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - DispatchEngine.WorkspaceLeasePort
  - WorkerRuntime.WorkspaceCapabilityPort
contracts_to_create:
  - WorkspaceReadLease
  - WorkspaceWriteLease
  - ConflictScope
  - IntegrationTaskResult
  - PatchArtifact
input_artifacts:
  - { artifact: deterministic-goal-phase, source: upstream, producer: P1-05 }
  - {
      artifact: two-independent-reader-tasks,
      source: local_fixture,
      producer: P1-07,
    }
  - {
      artifact: target-workspace-snapshot,
      source: external_input,
      producer: environment,
    }
output_artifacts:
  - overlapping-reader-runs
  - provenance-preserving-evidence-join
  - exclusive-writer-patch
  - post-write-workspace-snapshot
  - exclusive-writer-capability
  - goal-gate-evidence
verification:
  - real-run-overlap-measurement
  - read-only-capability-enforcement
  - competing-writer-lease-test
  - evidence-conflict-test
  - goal-gate-full-check
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：双 Reader／单 Writer 暂保留为权限与并发测试候选，不单独代表角色协作。完整规划→执行→集成→返工／验收由 [P1-15](15-human-role-collaboration.md) 及 G3 集成验证，本票保留独立权限与并发证据。

- [P1-05 Goal phase reduction](./05-goal-phase-reduction.md)。

## What it delivers

同一 Goal 中两个无硬依赖的只读 Task 在不同模块或调查方向真实并行。它们的 Evidence 经 IntegrationTask join，冲突被显式保留；随后只有一个 Writer 获得目标 Workspace 写 lease，应用修复并通过 GoalGateTask。

## Module / Interface refs

- [ControlEngine](../../../../modules/control/control-engine.md) 的 RuntimeExecutionDAG、readiness、lease 与 Goal reducer；
- `DispatchEngine.drive/accept` 的并发和预算；
- `ContextCompiler.assemble`；
- `WorkerRuntime` 的两个 Reader Run 与一个 Writer Run；
- `VerificationEngine.verify`；
- [StateLedger](../../../../modules/data/state-ledger.md) 和 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票首次冻结 DispatchEngine workspace-lease 与 WorkerRuntime workspace-capability 两个最小 Interface；P1-12/13 只消费这些版本。本票同时创建读写 lease、conflict scope、Integration result 与 patch contracts。

## Acceptance

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 多个工作包的角色 Context 与权限隔离；同一协调者退出不撤销独立 Worker 的合法 lease。跨包冲突的语义路由由 P1-15 集成，本票验证运行隔离。


- 两个 Reader 的 Run 时间实际重叠，且各自有独立 Attempt、Context、预算和来源；
- 同 Stage 或不同 Stage 不产生隐式先后，只有显式 RuntimeExecutionDAG `depends_on` 阻塞；
- Reader 不能修改目标 Workspace，其输出只作为 Evidence、Artifact 或 Handoff；
- Evidence 冲突不能由后到结果覆盖，IntegrationTask 必须解释或升级；
- 同一 conflict scope 同时最多一个有效 Writer lease；
- Writer 使用已接受 Reader 输出，返回 patch/commit、changed paths、检查结果和 Workspace revision；
- GoalGateTask 的全量检查通过后 Goal 才可完成，失败事实仍可追溯。

## Implementation record (2026-09-06, P1-07)

- **status**: stays `proposed` (per development workflow — ticket status is updated by the process, not by the implementing agent); acceptance documented at `../verification/p1-07-implementation-evidence.md` (product root commit 14886d8; 88 files / 722 tests PASS; 8/8 acceptance + 5/5 verification groups; validate-docs 13/13).
- **two minimal interfaces frozen by the first consumer**: `DispatchEngine.WorkspaceLeasePort` and `WorkerRuntime.WorkspaceCapabilityPort` (src/contracts/workspace-lease.ts + workspace-capability.ts, v1; contract-suite + dual-adapter wiring as the minimal contract test).
- **5 contracts created**: WorkspaceReadLease, WorkspaceWriteLease, ConflictScope, IntegrationTaskResult, PatchArtifact (+ 3 view contracts + parallel drive port).
- **three lanes merged** (lease+capability 26/26, drive+join 9/9, projections+patch+restart 19/19); P1-03/04/05/06 frozen shapes zero-diff-verified; 6 new v1 events + 6 commitKinds in ONE atomic story (KNOWN + isHandledEventType same commit).
- **integrator rulings during merge**: DAG eligibility live-phase overlay (plan snapshots are immutable); taskRevision numeric validators; run-scoped P1-03 start idempotencyKey; release expectedRevision 1; conflict-test plan variant (a FAILed dep correctly blocks, so the conflict join drops that edge); GoalGate full check = full re-verification at the canonical post-write revision (P1-05 reducer untouched); views are event projections.
- **G3 (Role Collaboration) waits P1-07 + P1-15 evidence** — the P1-07 side is now complete; semantic routing / human decision remains P1-15.
- **not pushed**: product root local main (14886d8) is NOT pushed to GitHub origin main (push requires user authorization per P1-04/05/06 precedent).

