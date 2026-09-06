# P1-16：连续工作 → 关键理由留痕 → Context 接续

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-06
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/control/dispatch-engine.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - WorkerRuntime.ContextContinuationPort
  - ContextCompiler.WorkContextPort
  - ControlEngine.WorkRecordPort
contracts_to_create:
  - WorkContextBinding
  - ExecutionNote
  - ContextContinuationResult
input_artifacts:
  - { artifact: bounded-handoff-packet, source: upstream, producer: P1-06 }
  - { artifact: fake-agent-run-events, source: upstream, producer: P1-03 }
  - { artifact: context-continuity-fixture, source: local_fixture, producer: P1-16 }
output_artifacts:
  - versioned-context-continuity-contract
  - durable-execution-notes
  - context-continuity-evidence
verification:
  - same-work-multiturn-test
  - incremental-note-crash-test
  - continuation-capability-fallback-test
  - stale-lease-and-context-test
  - real-kernel-continuity-test
```

## Blocked by

- [P1-06](./06-handoff-a-to-b.md)。

用户已授权文档修订；本票仍 proposed，未授权实施。编号不代表排在 P1-15 之后。

## Module / Interface refs

[Context 生命周期](../../../../interfaces/context-lifecycle.md) 是行为正文；复用既有持久化、派发与交接能力。本票只冻结首个实际消费者所需的版本扩展，不创建空管理模块。

## What it delivers

同一任务连续调查、实现和测试，在关键选择时保存简短理由；正常调用继续使用工作 Context。安全点换手或故障后，接续者从当前事实、理由和前沿继续，用户可查询来源与实际恢复方式。复用 P1-06，范围只覆盖同工作连续性，不实现后续新任务检索或 P1-10 用户控制界面。

## Acceptance

- 多次模型／工具反馈不逐次新建工作身份；Run、工作关联与 ContextBundle 区分清楚。
- 理由在关键检查点增量保存、正文先落库引用后登记；重复提交幂等，缺少最终总结时仍能恢复已有记录。
- 原会话继续／恢复能力显式声明；能力不足的新 Run 接续或拒绝可观察，不伪装原进程仍存在。
- Context 组装保留约束、理由、未解项及来源，必要材料缺口显式返回，大小有界。
- 旧 lease／授权、版本变化、迟到反馈与未知副作用沿既有 guards 处理；不改变 CompletionPolicy。
- 明确 P1-03／06 原版本兼容映射或拒绝策略，既有契约回归通过。P1-04 评审隔离语义不被继续执行上下文覆盖。
- 对真实 coding-agent 适配路径完成一次多轮任务与接续验证，并保留能力降级证据；Fake 契约测试不能替代真实内核验证。

## Verification

注入记录正文已保存但引用未登记、崩溃缺最终总结、旧 lease 迟到结果、原会话不可恢复与必要材料缺失。对照接续前后的义务、关键理由、来源和未知结果；通过公开 Interface 与 View 验证，不直接修改存储表。
