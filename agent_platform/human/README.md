# 人类管理与审阅入口

本页提供阅读地图，具体规则以链接正文为准。职责规则见 [文档职责划分](../dev_docs/document-ownership.md)。

## 日常管理

| 你要决定什么 | 管理／审阅文档 | Agent 承接什么 |
| --- | --- | --- |
| 产品目标、范围和非目标 | [PRODUCT](../PRODUCT.md) | 同步规格细节、Ticket 和验收 |
| 系统分责、边界与架构取舍 | [ARCHITECTURE](../ARCHITECTURE.md) | 维护 Module、Interface 和检查 |
| 什么才算交付成功 | [MVP 场景](../dev_docs/evaluation/mvp-scenario.md) | 维护测试、验证程序和 Evidence |
| 阶段范围与优先级 | [ROADMAP](../dev_docs/planning/ROADMAP.md) | 拆分 Ticket、维护 DAG、协调并行与集成 |

你可以直接修改正文，或要求 Agent 按决定修订。下游同步由 Agent 按[更新流程](../dev_docs/agent/README.md)执行，无需你手工逐票修改。

## 按需审阅

- 完成、取消、部分接受等行为变化：[Completion Policy](../dev_docs/interfaces/completion-policy.md) 的相关规则。
- 架构取舍与理由：[决策索引](../dev_docs/decisions/INDEX.md)。
- 术语含义：[CONTEXT](../CONTEXT.md) 的相关条目。
- 直观理解系统：[入口与系统图](../dev_docs/design/agent-entry-and-system-map.md)，其依据仍是正式规范。
- 范围争议：[用户需求原文](../dev_docs/product/用户需求原文.md)。

## 当前审阅入口

文档已按当前意见收敛。最后主要查看 [MVP 场景](../dev_docs/evaluation/mvp-scenario.md) 是否代表你要的完整故事，以及 [P1 DAG](../dev_docs/planning/proposed/P1-foundation/DAG.md) 的阶段范围。角色与架构若需改动仍回到 PRODUCT／ARCHITECTURE；字段、Module 细节与票据同步由 Agent 维护。历史讨论见 [角色复核稿](../dev_docs/design/human-framework-role-review.md) 与 [双方对话索引](../dev_docs/product/README.md)。

项目建立位置见 [项目位置与建项入口](../dev_docs/agent/project-location.md)，由 Agent 维护具体路径和派发配置。

[P0-06](../dev_docs/planning/active/P0/tickets/06-user-review.md) 记录完成规则与 [P1 候选施工图](../dev_docs/planning/proposed/P1-foundation/DAG.md) 的审阅状态。状态以该 Ticket 为准；文档职责划分不代表接受产品方案或启动 P1。

Module 实现说明、接口字段、叶子 Ticket、依赖边、测试输出与 Handoff 由 Agent 日常维护。需要细查时进入 [Agent 维护入口](../dev_docs/agent/README.md)。
