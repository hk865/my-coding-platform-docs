# P1-10：Pause/resume/cancel and safe steer

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-16
  - P1-08
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/dispatch-engine.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - HumanCollaboration.ControlCommandPort
  - DispatchEngine.ControlIntentPort
  - WorkerRuntime.LifecycleControlPort
contracts_to_create:
  - PauseCommand
  - ResumeCommand
  - CancelCommand
  - SteerCommand
  - RuntimeControlIntent
  - SafePointAcknowledgement
input_artifacts:
  - { artifact: versioned-context-continuity-contract, source: upstream, producer: P1-16 }
  - { artifact: status-and-evidence-view, source: upstream, producer: P1-08 }
  - { artifact: fake-agent-run-events, source: upstream, producer: P1-03 }
  - {
      artifact: active-task-attempt-fixture,
      source: local_fixture,
      producer: P1-10,
    }
  - {
      artifact: fake-runtime-safe-point-fixture,
      source: local_fixture,
      producer: P1-10,
    }
output_artifacts:
  - context-resume-outcome
  - desired-state-events
  - durable-runtime-control-intent
  - safe-point-acknowledgement
  - control-timeline-view
verification:
  - resume-original-or-replacement-test
  - lifecycle-transition-table-tests
  - idempotent-control-command-tests
  - safe-point-delivery-test
  - late-ack-and-outcome-unknown-tests
```

## Blocked by

- [P1-16](./16-context-continuity.md)：消费 versioned-context-continuity-contract。

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-08 Read-only status and Evidence console](./08-status-evidence-console.md)。

## What it delivers

用户从控制台对当前 Goal、Task 或 Run 发出 pause、resume、cancel 或不改变目标/验收的 steer。ControlEngine 先提交 desired state 与 durable control intent，Runtime 只在声明的安全点应用 steer 并回交 acknowledgement；用户随后看到 current state 是否已经收敛。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 提交控制命令；
- [ControlEngine](../../../../modules/control/control-engine.md) 校验 expected revision 与状态转换；
- `DispatchEngine.drive/accept` 与 `WorkerRuntime.control/events`；
- [StateLedger](../../../../modules/data/state-ledger.md) 持久化 desired state、intent 和 acknowledgement；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 投影 current/desired state 与 Timeline；
- 本票首次冻结 HumanCollaboration control-command、DispatchEngine control-intent 与 WorkerRuntime lifecycle-control 三个最小 Interface；P1-11 只消费已确认的控制结果。本票同时创建 lifecycle commands、Runtime control intent 和 safe-point acknowledgement contracts。

## Acceptance

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 暂停保留工作前沿与待决动作；恢复按内核能力继续原运行或明确用新 Run 接续。不能以命令送达证明原 Context 已恢复。
- 重复恢复与旧回执保持幂等／版本拒绝；取消正常留痕、故障缺失记录与未知副作用显式保留，消费 P1-16 接续能力。


- pause、resume、cancel 和 steer 均要求 expected revision 与 idempotency key；
- 命令先持久化 desired state/control intent，再触发 Runtime side effect；
- pause/cancel 请求与 current state 分开显示，只有 Runtime fact 才证明已经停止；
- steer 只能在 Runtime 声明的安全点投递，并保留 payload digest、delivery cursor 和 acknowledgement；
- steer 不改变 Goal objective 或 AcceptanceObligation；这类变化进入 P1-11；
- 重复命令不重复 side effect，过期 acknowledgement 不覆盖新 Run；
- Runtime 无法确认结果时保持 outcome_unknown，不猜成暂停、取消或成功；
- UI 不能直接杀进程或修改 Task/Goal phase。

## Implementation record（有限授权，2026-09-06 连续窗口；status 保持 proposed）

- 证据：dev_docs/verification/p1-10-implementation-evidence.md。
- 实测（产品根 ee53869）：typecheck 0；全量 130 files / 947 tests PASS / 0 skip；双适配器 7+7；restart 1/1；集成 2/2；validate-docs 13/13（上轮记录）。
- 冻结：HumanCollaboration.ControlCommandPort、DispatchEngine.ControlIntentPort、WorkerRuntime.LifecycleControlPort（src/contracts/control-intent.ts，v1）。
- 本票无 Task/Goal phase 写入；P1-11（←10）已解锁。
- 旧验收记录未改动；仅追加本条。
