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
- 本票创建 CompletionPolicy/ArchitectureBaseline fixture、install/activation，以及 Plan command、event、snapshot、validation error 与 View contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。本票仍使用版本化 Plan fixture，不在 P1-03 前引入模型运行依赖；正式协调请求／结果由后续消费者展开。测试重复提交与过期提案不会重复或错误激活 Plan。

- 两个 local fixture 都具有显式 schema version、revision、source identity 与 content digest；fixture 缺失或无效时拒绝安装，不使用内置内容；
- install contract 持久化 digest/revision 精确匹配的 immutable CompletionPolicy 与 ArchitectureBaseline；同一 identity/revision 不可覆写，重启后内容与 ref 仍可解析；
- activation contract 只接受已安装的精确 target ref，并以 expected Project revision 做 CAS；悬空、digest mismatch 或竞争更新失败时不移动 active ref；
- CompletionPolicy 与 ArchitectureBaseline active ref 按 kind 独立存在；P1-02 不要求或创建 ArchitectureEvolutionPolicy active ref；
- 两个 Project active refs 建立后，一个满足全部非空 guard 的 PlanRevision 被原子接受并成为 Goal 的 active revision；
- PlanRevision 只接受能从 canonical Project active refs 解析到 identity/revision/digest 完整匹配的 effective CompletionPolicy 与 ArchitectureBaseline，并把精确 refs 固定为不可变 pin；
- Project 默认 ref 后续移动不改变既有 Plan pin；改变 pin 只能创建新 PlanRevision/PlanRebase；
- 缺失、悬空、内容不匹配或在 CAS 窗口变化的 active ref 使整个 PlanRevision 零写入拒绝，不使用内置 fallback；
- 每个可激活 PlanRevision 至少包含一个 required executable Task、一个 active required GoalGateTask 和一个 required AcceptanceObligation；每个 required executable Task 映射非空 required obligation，每个 required obligation 编译出非空 required verification requirements；
- `parent_of` 只进入 TaskHierarchy，`depends_on` 只进入 RuntimeExecutionDAG，Stage 不自动生成依赖；
- 空 required 集合、缺少 active required GoalGateTask、悬空边和环被拒绝且无部分写入；
- 重复 command 保持幂等，旧 expected revision 被 CAS 拒绝；
- 重启重放后 Plan Graph、Task Detail 与 active revision 一致；
- 不产生 dispatch outbox、TaskAttempt 或 AgentRun。
