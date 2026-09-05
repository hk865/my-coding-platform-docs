# PlanCompiler Module

```yaml
status: draft
updated: 2026-09-05
plane: Control
```

## Purpose

把人的意图与运行反馈组织成有界协调工作及可受理提案；保留名称但不限于生成计划。

## Interface

候选操作：`request(intent) / accept(resultRef)`。共享字段、拒绝与状态语义唯一来源为 [运行时协作契约](../../interfaces/runtime-collaboration.md)，本页不复制 wire schema。首个消费者：P1-11；P1-15 扩展初始协商；设计已展开不代表契约已冻结或实现。

## Dependencies

ControlEngine、ContextCompiler、ArtifactVault。长期调用关系以 [ModuleDependencyDAG](../../../ARCHITECTURE.md#moduledependencydag) 为准；运行时事件反馈不是反向源码依赖。

## Hidden Implementation

请求去重、来源关联、缺口补充和结果分类。模型运行经持久 intent 交 Dispatch，不在本 Module 持有永久对话。

## Test seam

通过上述 Interface 注入依赖 Adapter，验证：重复结果、过期提案、缺少材料、预算耗尽；已知事实不必调用模型。 不以内部表或私有方法作为唯一测试入口。

## Context load

实现或扩展本 Module 时读取本页、当前 Ticket 与直接消费的 [运行时协作契约](../../interfaces/runtime-collaboration.md) 小节；初始协商及图文集成另读 [初始设计与统一展示](../../interfaces/human-design-status.md)。原始对话和完整历史按需追溯，不默认装入 Run。
