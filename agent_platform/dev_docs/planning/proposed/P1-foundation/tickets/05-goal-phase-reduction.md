# P1-05：Required set → Goal phase

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-04
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interface_refs:
  - ../../../../interfaces/completion-policy.md
contracts_to_create:
  - GoalReductionInput
  - GoalPhase
  - GoalCompletionExplanation
  - SideEffectReconciliation
input_artifacts:
  - { artifact: accepted-plan-revision, source: upstream, producer: P1-02 }
  - {
      artifact: task-or-gate-verification-result,
      source: upstream,
      producer: P1-04,
    }
  - {
      artifact: revision-bound-evidence-bindings,
      source: upstream,
      producer: P1-04,
    }
output_artifacts:
  - deterministic-goal-phase
  - goal-completion-explanation
  - goal-timeline-events
verification:
  - exhaustive-goal-reducer-table
  - non-empty-required-set-tests
  - stale-and-side-effect-guard-tests
  - restart-replay-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-04 Evidence satisfies Task/Gate](./04-evidence-satisfies-task.md)。

## What it delivers

ControlEngine 从当前 PlanRevision 的全部 required executable work Task、required AcceptanceObligation、required GateTask、适用 Evidence 和副作用对账确定性归约 Goal phase。用户能看到 RUNNING、BLOCKED、NEEDS_DECISION、FAILED 或 COMPLETED，以及阻止或支持该结论的准确来源。

## Module / Interface refs

- [ControlEngine](../../../../modules/control/control-engine.md) 的 Goal reducer；
- [StateLedger](../../../../modules/data/state-ledger.md) 提供当前 revision 事实；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 投影 GoalView 与 Timeline；
- [Completion Policy Interface](../../../../interfaces/completion-policy.md) 定义 GoalCompletionGuard 与完整 phase 归约优先级；
- 本票创建 Goal reduction input、phase、completion explanation 和 side-effect reconciliation contracts。

## Acceptance

- 只有所有 required executable work Task、required obligation 与 required GateTask 满足，且无未对账副作用时，Goal 才能 COMPLETED；
- optional work 不阻止完成，但 required work 的 deferred、cancelled、blocked 或 failed 不会伪装成完成；
- `parent_of`、Module、Stage 和完成比例都不改变 required 集合；
- 空 active Plan、空 active required GoalGateTask、空 required obligation 或空 required verification requirement 不能证明完成；
- no-change 只能由带当前 PASS Evidence 的 `AlreadySatisfied GoalGateTask` 表达；
- BLOCKED、NEEDS_DECISION、FAILED、CANCELLED 与 outcome_unknown 有确定性优先级；
- 相同事实重放得到相同 Goal phase 和解释；
- 历史 FAIL 保留，只有当前适用 Evidence 参与本次完成 guard。
