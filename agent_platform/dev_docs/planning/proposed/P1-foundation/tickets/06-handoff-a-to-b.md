# P1-06：Handoff A → B

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-04
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/dispatch-engine.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - DispatchEngine.HandoffPort
  - ContextCompiler.HandoffContextPort
  - WorkerRuntime.HandoffControlPort
contracts_to_create:
  - HandoffPacket
  - HandoffContextRequest
  - ReplacementAttempt
input_artifacts:
  - { artifact: fake-agent-run-events, source: upstream, producer: P1-03 }
  - {
      artifact: revision-bound-evidence-bindings,
      source: upstream,
      producer: P1-04,
    }
  - { artifact: accepted-plan-revision, source: upstream, producer: P1-02 }
  - {
      artifact: active-task-attempt-fixture,
      source: local_fixture,
      producer: P1-06,
    }
  - {
      artifact: partial-worker-artifacts-fixture,
      source: local_fixture,
      producer: P1-06,
    }
  - { artifact: runtime-cursor-fixture, source: local_fixture, producer: P1-06 }
output_artifacts:
  - bounded-handoff-packet
  - replacement-worker-attempt
  - handoff-provenance-timeline
verification:
  - forced-context-rollover-test
  - run-crash-recovery-test
  - stale-packet-test
  - late-result-rejection-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-04 Evidence satisfies Task/Gate](./04-evidence-satisfies-task.md)。

## What it delivers

Worker A 在 Context/Run 结束或故障后停止；平台从已提交 State、Evidence、Artifact 和 Runtime cursor 生成 HandoffPacket。Worker B 在新 Run 中接续同一 Task，并继续走原 CompletionPolicy 与 verification path；源 transcript 不整体回放，未知副作用不被隐藏。Goal phase 仍由 P1-05 的 reducer 独立归约。

## Module / Interface refs

- `DispatchEngine.drive/accept`；
- `ContextCompiler.assemble(HandoffContextRequest | TaskContextRequest)`；
- `WorkerRuntime.control/events/start`；
- [ControlEngine](../../../../modules/control/control-engine.md) 处理 lease 与事实；
- [StateLedger](../../../../modules/data/state-ledger.md)、`ArtifactVault` 和 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票首次冻结 DispatchEngine handoff、ContextCompiler handoff-context 与 WorkerRuntime handoff-control 三个最小 Interface；后续恢复场景消费这些版本。本票同时创建 HandoffPacket、Handoff context 与 replacement Attempt contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。Handoff 保留工程轨迹、当前事实版本与语义提示来源，替代 Run 重新校验角色绑定和 Context 缺口；验证旧授权不被交接文本继承。

- HandoffPacket 具有硬大小上限，并包含 objective、constraints、completed、unresolved、Evidence/Artifact refs、Workspace revision 和来源；
- B 不读取 A 的隐藏思维链或完整 transcript；
- source revision 不匹配时 Handoff 明确 stale 并重新投影；
- A 的过期 lease 和迟到结果不能覆盖 B 的新 Attempt；
- outcome_unknown 在交接和 View 中保留，不自动重试不可逆动作；
- B 提交后续结果时，Task verification path 可追溯到 A、B 两个 Run 与同一 Task revision 的合法演进；本票不以 Goal reducer 为验收前置。

---

## Implementation record（P1-06，status 保持 `proposed`，符合阶段守卫）

```yaml
implemented: 2026-09-06
limited_authorization: P1-06 only; 不把本票记成 P1 已验收；不自动推进 P1-15/其他票（G2 需 05+06 双验收）
evidence: dev_docs/verification/p1-06-implementation-evidence.md
git: product main commit a9070e5（+ f3a6a71 handoff final）；NOT pushed to GitHub origin main（需用户授权）
lanes: A 090c3b5 / B ee4405c / C ebe844c（隔离 worktree 从共享基线 ca5c65d 派生，全量合并）
conflict_log: dev_docs/logs/conflict-reports/2026-09-06-p105-p106-merge.md（P1-05 × P1-06，state=closed）
acceptance: 6/6 满足; verification: forced-context-rollover / run-crash-recovery / stale-packet / late-result-rejection 四组通过（双适配器 25/25 × 2、真实 SQLite 集成 1/1、重启证据 1/1、typecheck 0、全量 76 files/641 tests、validate-docs 12/12、P1-05 专属文件 0 行 diff）
```

**2026-09-06 DAG 注记（追加，不改原记录）**：按 [P1 DAG](../DAG.md) 09-06 增量，G2（Continuity）= P1-05 + P1-06 + **P1-16**；本票完成 06 侧证据（05 侧已由 P1-05 交付，16 侧为新增后置边界）。

## 2026-09-06 后续扩展归属

[Context 生命周期](../../../../interfaces/context-lifecycle.md) 新增同工作连续性、关键理由留痕及能力声明，交由 [P1-16](./16-context-continuity.md) 进行兼容性核对和实现；完成后历史继承由 [P1-17](./17-completed-work-context.md) 验证。本票原 Acceptance、冻结版本与 Implementation record 保留，不能以原 PASS 证明新增行为，也不回写原票为新要求未通过。后续契约变化显式版本化。
