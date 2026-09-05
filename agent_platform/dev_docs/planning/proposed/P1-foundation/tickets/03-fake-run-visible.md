# P1-03：Eligible Task → Fake Run View

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-02
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
  - ArtifactVault.ArtifactPort
  - DispatchEngine.DispatchPort
  - ContextCompiler.TaskContextPort
  - WorkerRuntime.RunPort
contracts_to_create:
  - DispatchIntent
  - TaskLease
  - TaskAttempt
  - TaskEnvelope
  - RuntimeEvent
  - ActiveAgentView
input_artifacts:
  - { artifact: accepted-plan-revision, source: upstream, producer: P1-02 }
  - { artifact: runtime-execution-dag, source: upstream, producer: P1-02 }
  - { artifact: eligible-task-fixture, source: local_fixture, producer: P1-03 }
output_artifacts:
  - durable-dispatch-outbox
  - leased-task-attempt
  - bounded-task-envelope
  - fake-agent-run-events
  - active-agent-and-run-view
verification:
  - readiness-table-tests
  - competing-claim-test
  - outbox-before-side-effect-test
  - duplicate-late-runtime-event-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-02 PlanRevision accepted and visible](./02-plan-revision-visible.md)。

## What it delivers

一个 RuntimeExecutionDAG 前置条件已满足的 Task 被 DispatchEngine 唯一领取。ContextCompiler 生成有界 TaskEnvelope，FakeRuntimeAdapter 发出可重放 Run events，用户随后在 ActiveAgents 与 TaskDetail 中看到 lease、Attempt、Run、预算和当前状态；Run 结束不会在本票中自动满足 Task。

## Module / Interface refs

- [ControlEngine](../../../../modules/control/control-engine.md) 的 readiness、claim 与 Runtime fact 入口；
- `DispatchEngine.drive/accept`；
- `ContextCompiler.assemble(TaskContextRequest)`；
- `WorkerRuntime.capabilities/start/events` 与 `FakeRuntimeAdapter`；
- [StateLedger](../../../../modules/data/state-ledger.md) 和 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票首次冻结 DispatchEngine dispatch、ContextCompiler task-context 、WorkerRuntime run 与 ArtifactVault artifact 四个最小 Interface；P1-06/07/09/10 只能消费或显式扩展这些版本。本票同时创建 dispatch、lease、Attempt、TaskEnvelope、RuntimeEvent 与 ActiveAgent View contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。TaskEnvelope 引用已接受的角色模板／绑定版本；自定义短生命周期角色复用 dispatch，验证越权、旧绑定、预算耗尽与安全退出。Context 不自行启动 Agent；报告正文保存后，只有 Control 登记成功才成为可查询的已接受引用。

- 只有显式 RuntimeExecutionDAG 硬依赖已满足、desired state 为 active、无 Blocker 且资源可用的 Task 才 eligible；
- 两个 Dispatcher 竞争同一 Task 时最多一个 lease/Attempt 成功；
- dispatch intent 先持久化到 outbox，之后才调用 Fake Runtime；
- TaskEnvelope 绑定 WorkspaceSnapshot revision、权限、预算和来源，具有硬大小上限且不含完整 transcript；
- 重复、乱序和迟到 RuntimeEvent 不会回退 Task 或 Run revision；
- crash 与 outcome_unknown 分开投影，不能猜成成功；
- ActiveAgents 与 TaskDetail 可从事件重建；
- Runtime exit 或 `exit=0` 不会直接写 `Task.phase = satisfied`。
