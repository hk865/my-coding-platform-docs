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

---

## 有限授权与 Implementation record（P1-05）

- 授权：2026-09-05 用户有限授权实施本票（P1-05 仅限；共享基线为 P1-04 commit `5a278cb` 之上的 `3256167`）。本票首次真实消费并冻结 GoalPhase 聚合面（含纯函数 reduceGoalPhase + GoalCompletionGuard + 确定性 explanation 模板）、GoalStatus/Timeline 视图面与 ControlEngine.reduceGoal 三个最小面；创建 4 个契约（GoalReductionInput/GoalPhase/GoalCompletionExplanation/SideEffectReconciliation）与 goal-reduction commitKind；共享契约套件 + InMemory/SQLite 双 Adapter、in-memory/persistent 双 harness、重启证据模式、隔离 worktree→main 合并模式复用 P1-00…P1-04，未重写。
- 实施记录：共享基线 → 三路并行（A goal reducer / B 视图+事件 / C 重启证据+集成骨架）→ 产品根集成（三路子 Agent 因基础设施故障中断，按 P1-04 先例由 integrator 接管完成 A/B，C 校验无需改动）→ 验收证据见 `dev_docs/verification/p1-05-implementation-evidence.md`；冻结语义清单见产品根 `IMPLEMENTATION-HANDOFF.md` “P1-05 契约与存储语义（冻结）”。
- 命令结果：typecheck 0 errors；全量 64 files / 537 tests PASS（P1-00…04 基线 486 零回归 + P1-05 新增 51）；P1-05 双契约套件 8/8 × InMemory + 8/8 × SQLite（同一套件定义、无调参）；真实 SQLite 集成 1/1；重启证据 1/1；validate-docs 12/12。
- 状态守卫：本票 status 保持 `proposed`，与阶段守卫一致；不把本票记成整个 P1 已验收；不自动推进 P1-07/08（DAG：05 验收后才出现 07/08 并行窗口；如需进度可单独申请授权）。P1-06 并行 session 由用户暂停，其 worktree 合并面已记录（events/ledger/validation.ts）。
- 完成边界（AGENTS.md）：工作 Agent 只提交结果与证据；Ticket 状态由开发流程依据 Evidence 更新。


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
