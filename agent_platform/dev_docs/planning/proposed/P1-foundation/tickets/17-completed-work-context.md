# P1-17：完成工作 → 重启 → 相关新任务继承

```yaml
status: proposed
updated: 2026-09-06
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-16
  - P1-05
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
  - ContextCompiler.CompletedWorkContextPort
contracts_to_create:
  - CompletedWorkContextRequest
  - ExecutionMemorySelection
input_artifacts:
  - { artifact: deterministic-goal-phase, source: upstream, producer: P1-05 }
  - { artifact: durable-execution-notes, source: upstream, producer: P1-16 }
  - { artifact: versioned-context-continuity-contract, source: upstream, producer: P1-16 }
  - { artifact: completed-history-fixture, source: local_fixture, producer: P1-17 }
output_artifacts:
  - completed-work-context-evidence
verification:
  - completed-work-restart-test
  - stale-rationale-selection-test
  - history-scope-isolation-test
  - missing-history-test
```

## Blocked by

- [P1-16](./16-context-continuity.md)。
- [P1-05](./05-goal-phase-reduction.md)。

用户已授权文档修订；本票仍 proposed，未授权实施。编号不代表排在 P1-15 之后。

## Module / Interface refs

[Context 生命周期](../../../../interfaces/context-lifecycle.md) 是行为正文；复用既有持久化、派发与交接能力。本票只冻结首个实际消费者所需的版本扩展，不创建空管理模块。

## What it delivers

A 完成工作并归档，框架按 P1-05 归约 Goal；平台重启后，B 在相关新任务中取得当前规范／代码引用、此前关键取舍及适用性清单。工作间 Context 默认重新装配，无需恢复原作者会话。

## Acceptance

- 通过正常执行与完成路径产生历史，重启后按 Project／Workspace、工作、受影响模块／接口和版本选材；作者身份不限制合法接手。
- 复用 P1-16 的工作记录与 Context 编译，支持文本／精确引用，无需等待 P1-12 图索引或引入独立 MemoryStore。
- 用正常版本化来源提供当前代码／规范快照；需读 Workspace 时复用既有读能力或明确材料缺口，不预先冻结 P1-12 的图接口。
- 改变一个历史前提，选材清单标明适用、历史解释或缺口，输出给 B 的来源清单可定位已改变的前提；B 对该变化的实际语义理解由 P1-15 验证；不得继承旧授权、旧 Task phase 或旧 Evidence 的自动有效性。
- 缺记录、超预算、跨 scope 和权限不足显式处理；必要材料缺失不能声称 ready。保留原始来源，工作完成不自动删除引用。
- P1-05 的 required 集合与 Evidence 完成规则原样成立；历史检索不直接写完成状态。

## Verification

构造完成任务与新的相关任务，覆盖重启、同模块多次工作、跨模块来源、陈旧前提、缺失正文及越权。精确选材与版本用确定性测试；实际语义继承效果在 P1-15 真实模型场景另验，不能仅靠模型声称记得。
