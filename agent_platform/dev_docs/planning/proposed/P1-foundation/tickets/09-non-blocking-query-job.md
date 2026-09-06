# P1-09：Non-blocking QueryJob

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
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - DispatchEngine.SnapshotPort
  - WorkerRuntime.PublicSnapshotPort
  - HumanCollaboration.QueryJobPort
  - ContextCompiler.QueryContextPort
  - WorkerRuntime.ReadOnlyQueryPort
contracts_to_create:
  - SubmitQueryJobCommand
  - QueryContextRequest
  - QueryJob
  - QueryJobResult
  - QueryJobView
input_artifacts:
  - { artifact: versioned-context-continuity-contract, source: upstream, producer: P1-16 }
  - { artifact: status-and-evidence-view, source: upstream, producer: P1-08 }
  - { artifact: immutable-evidence-records, source: upstream, producer: P1-04 }
  - {
      artifact: revision-bound-evidence-bindings,
      source: upstream,
      producer: P1-04,
    }
  - {
      artifact: selected-source-task-refs,
      source: local_fixture,
      producer: P1-09,
    }
output_artifacts:
  - query-followup-provenance
  - bounded-query-context
  - isolated-query-agent-run
  - sourced-query-result
  - query-job-view
verification:
  - bounded-query-followup-test
  - source-run-noninterruption-test
  - read-only-capability-test
  - context-size-and-provenance-test
  - query-budget-timeout-test
```

## Blocked by

- [P1-16](./16-context-continuity.md)：消费 versioned-context-continuity-contract。

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：原 QueryJob 聚焦只读问答；秘书／参谋辅助需求确立和架构协商的范围、角色及产物需在复核中明确，不能直接等同本票。

- [P1-08 Read-only status and Evidence console](./08-status-evidence-console.md)。

## What it delivers

本轮 [查询与交流复核](../../../../design/human-framework-role-review.md) 明确三条路径：事实查询直接返回；运行内核已暴露的公开快照可由工具读取；需要新语义回答才创建独立 QueryJob。用户可跳过参谋，参谋可只做路由，无须再做一次语义审核。执行者报告不等于正式完成状态；快照不支持或过期时明确返回，不假设可读取隐藏思维或向源执行 Context 插话。公开快照在本票冻结；有限角色讨论由 P1-15 在运行时协作契约基础上集成，不将其冒认为已有实现。

用户从状态控制台对正在执行的工作提出需要工程判断的问题。平台从已提交 State、Evidence、Artifact 和 Handoff refs 编译有界只读 Context，启动独立 QueryJob 并显示带来源的回答；源 Worker 不暂停，源 lease 和 TaskAttempt 不被 QueryJob 改变。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 提交和跟随 QueryJob；
- `ContextCompiler.assemble(QueryContextRequest)`；
- `DispatchEngine` 与 `WorkerRuntime` 启动独立只读 Run；
- [ControlEngine](../../../../modules/control/control-engine.md) 校验预算、权限和归属；
- [StateLedger](../../../../modules/data/state-ledger.md)、`ArtifactVault` 和 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票首次冻结 HumanCollaboration query-job、ContextCompiler query-context、WorkerRuntime read-only-query，以及 DispatchEngine snapshot／WorkerRuntime public-snapshot 扩展；后续消费者只能使用该版本或提交显式版本升级。本票同时创建 QueryJob command、context、result 与 View contracts。

## Acceptance

2026-09-06 扩展依据：[Context 生命周期](../../../../interfaces/context-lifecycle.md)、[运行时协作](../../../../interfaces/runtime-collaboration.md) 与 [人类交互](../../../../interfaces/human-design-status.md)。新增条款尚待本票实施验证。

- 消费 P1-16 的连续性契约：QueryJob 可有多轮有界澄清；相关追问引用此前适用结果，仍不改源 Worker Context／lease／预算。结束、超时、缺口与过期回答可观察。


本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。本票补充冻结 DispatchEngine.SnapshotPort 与 WorkerRuntime 可选公开快照能力；测试 unsupported／stale 显式返回、仅在请求与授权允许时转 QueryJob。查询角色可由自由模板实例化，仍只读且不影响源 Run。

- 机械问题仍由 P1-08 ReadModel 直接回答，不创建 QueryJob；
- QueryJob 具有独立 ID、Run、Context、预算、超时和只读权限；
- Query Context 有硬大小上限，只含选择任务的当前状态、相关 refs 与来源，不含隐藏思维链或完整 transcript；
- QueryJob 启动、运行和完成期间，源 Worker 的 current phase、lease 和预算不变；
- QueryJob 无权写目标 Workspace、提交 Runtime Task CompletionClaim 或刷新源 lease；
- 回答引用使用的 revision/Evidence/Artifact，并在输入变 stale 时明确标记；
- QueryJob 失败或超时不会改变源 Task/Goal phase。
