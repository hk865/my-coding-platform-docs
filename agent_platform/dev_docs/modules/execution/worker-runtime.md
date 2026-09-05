# WorkerRuntime Module

```yaml
status: draft
updated: 2026-09-05
plane: Execution
```

## Purpose

作为内核适配 Interface 执行一次有界 Run，暴露能力、控制回执和运行结果。

## Interface

候选操作：`capabilities / start / control / events；可选 snapshot`。共享字段、拒绝与状态语义唯一来源为 [运行时协作契约](../../interfaces/runtime-collaboration.md)，本页不复制 wire schema。首个消费者：P1-03；06/07/09/10 按消费扩展；设计已展开不代表契约已冻结或实现。

## Dependencies

无反向 Control 依赖；CodingAgentAdapter 与 FakeRuntimeAdapter 实现本 Interface。长期调用关系以 [ModuleDependencyDAG](../../../ARCHITECTURE.md#moduledependencydag) 为准；运行时事件反馈不是反向源码依赖。

## Hidden Implementation

适配模型与工具内核、Run 身份、能力约束、事件 cursor、控制安全点及 outcome_unknown；不重写内核推理循环。

## Test seam

通过上述 Interface 注入依赖 Adapter，验证：启动幂等、只读／写 lease、取消安全点、重复事件、断线恢复、快照 unsupported。 不以内部表或私有方法作为唯一测试入口。

## Context load

实现或扩展本 Module 时读取本页、当前 Ticket 与直接消费的 [运行时协作契约](../../interfaces/runtime-collaboration.md) 小节；初始协商及图文集成另读 [初始设计与统一展示](../../interfaces/human-design-status.md)。原始对话和完整历史按需追溯，不默认装入 Run。
