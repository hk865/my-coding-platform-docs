# Agent Platform：工作 Agent 入口

```yaml
status: draft
updated: 2026-09-06
scope: 仓库内工作 Agent 的入口与 Context 路由
```

实现子 Agent 必须从一张明确分配的 Development Ticket 开始。没有 Ticket ID 时，先向编排者报告缺少工作入口，不自行选择 backlog 或把整个 Roadmap 当作任务。用户直接委托的文档管理、分析或计划修订以该委托为入口，按 [Agent 维护流程](dev_docs/agent/README.md) 执行。

## 两种 Agent 入口

- **构建本产品的开发子 Agent**：入口是本文件；编排者必须在任务消息中给出 `ticket_id`、`ticket_path`、角色、Workspace 根目录，以及上游 Artifact refs。详细需求、输出和验收只从 Ticket 读取，不在派发消息里复制。
- **产品运行后管理用户工作的 Worker**：入口是版本化 `TaskEnvelope`；P1-03 已有最小派发契约与 Fake Run 实现证据。完整角色编排、Context 连续性与真实内核接续仍须相关后续票验证。

两者不能混用：Development Ticket 用来构建 Agent Platform；Runtime Task 是 Agent Platform 将来调度的用户工作。

最小开发派发信封：

```yaml
ticket_id: P1-xx
ticket_path: dev_docs/planning/.../tickets/xx-....md
role: worker | reviewer | integrator
workspace_root: <explicit path>
upstream_artifact_refs:
  - <artifact + revision>
write_scope:
  - <allowed module/path>
```

## 启动顺序

1. 读取当前 Ticket，并确认 `status`、`blocked_by`、输入来源、预期输出和 Acceptance；
2. 加载 Ticket 指向的 Module、Interface、实际消费的规范条款，以及 DevelopmentTicketDAG 的必要一跳前驱/后继；
3. 解析并校验上游 Artifact ref、revision、Workspace scope 和写入权限；
4. 在 Ticket 的纵向验收路径内实现、汇合并产生 Artifact/Evidence；
5. 返回有界 Handoff：已完成、未解决、Evidence/Artifact refs、验证结果和风险。

完整入口与信息流见 [工作入口与系统结构图](dev_docs/design/agent-entry-and-system-map.md)。

## Context pointers

- 建立项目、派发实现任务或迁移目录时，读取 [项目位置与建项入口](dev_docs/agent/project-location.md)，据此设置代码 Workspace 与文档 Ticket 路径；
- 修改规范、计划、入口或跨文档同步时，读取 [职责划分](dev_docs/document-ownership.md) 和 [Agent 维护流程](dev_docs/agent/README.md)；
- 产品边界发生歧义时，读取 [PRODUCT](PRODUCT.md)；
- 领域词义发生歧义时，读取 [CONTEXT](CONTEXT.md)；
- Module/Interface 归属或依赖方向不清时，读取 [ARCHITECTURE](ARCHITECTURE.md)；
- 只有复核产品意图或处理范围争议时，读取 [用户需求原文](dev_docs/product/用户需求原文.md)；
- 追溯方案推导或核对历史建议时，从 [双方对话来源索引](dev_docs/product/README.md) 选择相关对话，历史回答只作为来源；
- P0-06 未结束时，修订或准备实现 P1 先核对 [当前产品／架构复核](dev_docs/design/human-framework-role-review.md) 对本票的影响；
- 实现 Context 持续、暂停恢复、换手、历史继承时读取 [生命周期契约](dev_docs/interfaces/context-lifecycle.md)；跨包协调与变更上报读取 [运行时协作](dev_docs/interfaces/runtime-collaboration.md) 和 [人类交互](dev_docs/interfaces/human-design-status.md)；
- 历史推导只有当前 Ticket 明确引用时才读取 [Archive](dev_docs/archive/INDEX.md)。

## 完成边界

工作 Agent 提交结果和证据，不直接把 Development Ticket、Runtime Task 或 Goal 标成完成。Ticket 状态由开发流程依据 Evidence 更新；产品内 Task/Goal 状态由 ControlEngine 归约。

提交文档改动前运行：

```text
node dev_docs/verification/validate-docs.mjs
```
