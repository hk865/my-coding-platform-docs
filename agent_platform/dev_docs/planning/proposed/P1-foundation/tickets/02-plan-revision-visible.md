# P1-02：PlanRevision accepted → Plan/Task View

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-01
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interface_refs:
  - ../../../../interfaces/completion-policy.md
contracts_to_create:
  - VersionedCompletionPolicyFixture
  - VersionedArchitectureBaselineFixture
  - CompletionPolicyRevision
  - ArchitectureBaselineRevision
  - ProjectCompletionPolicyActiveRef
  - ProjectArchitectureBaselineActiveRef
  - InstallCompletionPolicyRevisionCommand
  - ActivateProjectCompletionPolicyCommand
  - InstallArchitectureBaselineRevisionCommand
  - ActivateProjectArchitectureBaselineCommand
  - ApplyPlanRevisionCommand
  - PlanRevisionAcceptedEvent
  - PlanRevisionSnapshot
  - PlanGraphView
  - TaskDetailView
  - PlanValidationError
input_artifacts:
  - {
      artifact: sqlite-workspace-bootstrap-record,
      source: upstream,
      producer: P1-01,
    }
  - { artifact: persisted-goal-event-stream, source: upstream, producer: P1-01 }
  - { artifact: sqlite-ledger-adapter, source: upstream, producer: P1-01 }
  - { artifact: sqlite-read-model-adapter, source: upstream, producer: P1-01 }
  - {
      artifact: versioned-completion-policy-local-fixture,
      source: local_fixture,
      producer: P1-02,
    }
  - {
      artifact: versioned-architecture-baseline-local-fixture,
      source: local_fixture,
      producer: P1-02,
    }
  - {
      artifact: hand-authored-plan-revision-fixture,
      source: local_fixture,
      producer: P1-02,
    }
output_artifacts:
  - persisted-immutable-completion-policy-revision
  - persisted-immutable-architecture-baseline-revision
  - project-active-completion-policy-ref
  - project-active-architecture-baseline-ref
  - accepted-plan-revision
  - plan-pinned-completion-policy-ref
  - plan-pinned-architecture-baseline-ref
  - task-hierarchy
  - runtime-execution-dag
  - plan-and-task-view
verification:
  - governance-local-fixture-schema-and-digest-tests
  - governance-install-roundtrip-tests
  - governance-activation-cas-tests
  - plan-schema-and-cycle-tests
  - required-set-and-goal-gate-guard-tests
  - effective-governance-ref-resolution-test
  - immutable-plan-pin-and-default-movement-test
  - cas-and-idempotency-tests
  - projection-replay-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-01 Goal persisted and visible](./01-goal-persisted-and-visible.md)。

## What it delivers

本票首次消费 CompletionPolicy 与 ArchitectureBaseline。ControlEngine 只通过显式、版本化 local fixture 和 install contract 持久化两类 immutable revision；install 不自动激活。独立 activation contract 再以 CAS 建立 typed Project active refs，fixture 不能直接写 canonical state，缺少输入时也不生成或选取内置默认值。

用户随后把一份手写、结构有效的 PlanRevision 应用到已持久化 Goal。ControlEngine 从 canonical Project active refs 解析精确 CompletionPolicy 与 ArchitectureBaseline revision，并把两者固定到 PlanRevision；随后原子接受 Stage、Runtime Task、AcceptanceObligation、GateTask、TaskHierarchy 和 RuntimeExecutionDAG。ReadModel 显示 Plan Graph、Task Detail 与两个 pin；本票不派发 Task，也不创建 Run。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 提交 `ApplyPlanRevisionCommand`；
- [ControlEngine](../../../../modules/control/control-engine.md) 校验 install/activation command 并接受 Plan revision；
- [StateLedger](../../../../modules/data/state-ledger.md) 保存 immutable governance revision，并原子提交 active-ref/PlanRevision 事件与状态；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 投影 Plan/Task View；
- [Completion Policy Interface](../../../../interfaces/completion-policy.md) 定义 Plan 激活前的 active required GoalGateTask、required Task、obligation 与 verification requirement 非空 guard；
- 不产生 dispatch outbox、TaskAttempt 或 AgentRun。

## P1-02 implementation record（2026-09-05，有限授权；status 保持 proposed）

```yaml
status: proposed           # 阶段守卫：P0-06 in_review、P1 DAG proposed；本记录不改变状态
authorized_by: user        # 有限授权：仅 P1-02（票据级），不代表 P0/P1 验收通过；不自动推进 P1-03
implementation: verified (limited authorization — acceptance evidence appended 2026-09-05)
product_root: /home/han001/projects/agents/agent_platform
handoff: <product root> IMPLEMENTATION-HANDOFF.md（P1-02 契约与存储语义=冻结语义清单）
evidence: dev_docs/verification/p1-02-implementation-evidence.md（P1-02 only；14 项 Acceptance 逐项对照 + 283 tests PASS + 重启证据块）
```

### P0-06 复核影响核对（按 AGENTS.md）

- human-framework-role-review.md：初始 Baseline fixture 只验证安装机制、不证明人参与架构生成；角色完成权不与名称绑定；View/解释配对契约未冻结。本票仅安装/激活/接受/投影机制，不派发、不判定完成、不创建 ArchitectureEvolutionPolicy——与复核结论一致，无需修订本票角色/输入输出/验收。
- completion-policy.md 与票据 Acceptance 无冲突（四正交维度、parent_of/depends_on 分离、非空 guard 顺序、无内置 fallback 均一致）。实施以票据 Acceptance 为准；本轮同时以 P1-02 扩展记录同步接口文档（state-ledger / command-event / goal-view / 三个 module）。

### 共享基线（integrator 第一阶段）

- 契约：src/contracts/governance.ts（fixture/revision/pin/install/activate/digest/fingerprint/resolution）、src/contracts/plan.ts（Stage/Runtime Task 四维度/AcceptanceObligation(requirementLevel)/VerificationRequirement/GateTask/TaskHierarchy/RuntimeExecutionDAG/PlanRevisionSnapshot/PlanValidationError）、src/contracts/plan-view.ts（PlanGraphView/TaskDetailView + freshness）；
- 版本化扩展：LedgerCommit += governance-install|governance-activate|plan-revision（v1 goal-create/bootstrap 语义不变）、AggregateRef/Snapshot += governance/Plan 各 aggregate、GoalSnapshot.activePlanRevision 可非空（v1 创建仍 null/1）、DomainEvent += 5、GoalView.activePlanRevision 可非空、ProjectionStallReason += unsupported_event_type、ControlEngine += install/activate/applyPlan（委托三个独立入口文件）、ReadModelIndex += planGraph/taskDetail；
- 共享夹具：src/contracts/fixtures/governance-fixtures.ts、plan-fixtures.ts（版本化 fixture + 确定性 build/fold 辅助）；双 Adapter 经 src/contracts/ledger-validation.ts 跑同一套 commit 校验（InMemory + SQLite）；
- 契约套件：tests/contract-suite/{governance,plan}.contract.suite.ts（参数化，按 Adapter 接线）、tests/contract-suite/p1-02-harness.ts；重启骨架：tests/restart/p1-02-*（skipIf 探针，实现落地后自动启用）。

### 验证记录（2026-09-05，integrator）

- `pnpm typecheck` 0 errors；`pnpm vitest run` **29 files / 283 tests PASS**（既有 158 零回归）；双 Adapter 契约套件（InMemory+SQLite）29×2 PASS；真实 SQLite 集成 4 PASS；重启证据 1 PASS；`validate-docs.mjs` 12/12；
- 静态证据：Adapter 之外 0 原始 SQL / 0 node:sqlite；既有 state-ledger/goal-view 契约套件零修改；package/lock/tsconfig/vitest 零差异（零新增依赖）；
- 详见 evidence 文件；票据 status 保持 proposed（阶段守卫），本票完成不等同 P1 完成，不自动推进 P1-03。

