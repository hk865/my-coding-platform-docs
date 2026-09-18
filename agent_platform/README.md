# Agent Platform 文档入口

```yaml
status: draft
updated: 2026-09-11
scope: 文档双入口；实现状态引用 human/module-status.md
```

更新：2026-09-11。这里保存规范与调查，产品代码在 `/mnt/d/1.project/Software/agent_platform`；执行内核已纳入其 `vendor/coding-agent/`。

## 两个入口

- **人类**：[12 Module 当前实现](human/module-status.md)，先了解能做什么、缺什么和已验证范围；更多阅读从 [人类入口](human/README.md) 进入。
- **Agents**：[工作入口](AGENTS.md)，按当前任务加载有关 Module、Interface 与必要证据；不把完整开发历史默认装入 Context。

两个入口共享以下规范正文，避免同一行为维护两份不同版本。

| 查询目的 | 当前正文 |
| --- | --- |
| 产品承诺和范围 | [PRODUCT](PRODUCT.md) |
| 领域词义 | [CONTEXT](CONTEXT.md) |
| Module 职责与设计依赖 | [ARCHITECTURE](ARCHITECTURE.md) |
| 当前实现与接线 | [模块状态](human/module-status.md) |
| 完成/失败/证据规则 | [完成策略](dev_docs/interfaces/completion-policy.md) |
| Context 连续性和历史继承要求 | [生命周期](dev_docs/interfaces/context-lifecycle.md) |
| 角色协作与人类决定 | [运行时协作](dev_docs/interfaces/runtime-collaboration.md)、[人类交互](dev_docs/interfaces/human-design-status.md) |
| 文档维护与归档 | [职责规则](dev_docs/document-ownership.md)、[Agent 流程](dev_docs/agent/README.md) |
| 项目位置 | [项目位置说明](dev_docs/agent/project-location.md) |

## 历史读取

旧 [P1 施工 DAG](dev_docs/planning/proposed/P1-foundation/DAG.md) 和 [验证证据索引](dev_docs/verification/README.md) 用于追溯相应版本，不代表当前产品全量完成，也不是新的实施授权。被替代的说明与长交接见 [归档索引](dev_docs/archive/INDEX.md)。

执行结束后，仍适用的接口与规则保留在当前规范；过程、旧状态与一次性输出进入历史层，按需加载。当前实现状态以本次有来源的模块审计为入口，不从历史测试总数推断。
