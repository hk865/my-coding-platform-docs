# P1-06：Handoff A → B

```yaml
status: proposed
updated: 2026-09-05
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
