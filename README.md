# my-coding-platform-docs

> README v1 — 本仓库的概览入口。完整的人类入口与文档路由见
> [`agent_platform/README.md`](agent_platform/README.md)。

## 这是什么

本仓库只保存 **Agent Platform** 产品的设计与规划文档，不含产品代码。

Agent Platform 是建立在 `coding-agent` 执行内核之上的独立产品：它保存长期
Goal、Task、Evidence 与 Agent 状态，负责规划、调度、验证、交接与人机协作；
`coding-agent` 只负责一次 Run 内的模型、工具、安全和 Context 循环。
两者是两个产品边界（见 [ADR-0001](agent_platform/dev_docs/decisions/0001-separate-platform-from-execution-kernel.md)）。

当前阶段：**设计期（P0 进行中，P1 为候选）**，产品代码尚未开始。

## 快速导航

| 我想…… | 读这里 |
| --- | --- |
| 了解产品是什么 | [PRODUCT](agent_platform/PRODUCT.md) |
| 理解领域术语 | [CONTEXT](agent_platform/CONTEXT.md) |
| 看 Plane / Module / 依赖 | [ARCHITECTURE](agent_platform/ARCHITECTURE.md) |
| 人类管理与审阅入口 | [human/README](agent_platform/human/README.md) |
| 工作 Agent 执行入口 | [AGENTS.md](agent_platform/AGENTS.md) |
| 当前规划与施工图 | [ROADMAP](agent_platform/dev_docs/planning/ROADMAP.md) |
| MVP 评价场景 | [mvp-scenario](agent_platform/dev_docs/evaluation/mvp-scenario.md) |
| 任务完成规则 | [Completion Policy](agent_platform/dev_docs/interfaces/completion-policy.md) |
| 文档责任划分 | [document-ownership](agent_platform/dev_docs/document-ownership.md) |

## 目录结构

```text
agent_platform/
├── README.md                  人类入口与文档路由
├── AGENTS.md                  工作 Agent 入口与 Context 路由
├── PRODUCT.md                 产品承诺与 MVP 边界
├── CONTEXT.md                 唯一领域词典
├── ARCHITECTURE.md            Plane、Module registry、长期依赖
├── human/                     人类管理与审阅入口
└── dev_docs/
    ├── document-ownership.md  文档职责与权威关系
    ├── agent/                 Agent 执行与变更同步流程
    ├── modules/               施工前沿的 Module 说明
    ├── interfaces/            跨 Module 的版本化契约
    ├── planning/              ROADMAP、阶段 DAG 与叶子 Ticket
    ├── decisions/             已接受的架构决策（ADR）
    ├── evaluation/            端到端发布门禁
    ├── product/               用户表达、双方对话与推导索引（按需读取）
    ├── design/                导航图与候选设计复核
    ├── verification/          文档结构检查与 P0 Evidence
    └── archive/               已替代内容的历史快照（不进入默认 Context）
```

## 现状摘要（2026-09-05）

- **P0 产品定义与开发地图**：`in_review`。已归档旧版来源、重写产品/领域/架构地图、
  切分 Plane→Module→Interface、形成首个纵向切片契约与 P1 候选施工图；正等待用户
  审阅 Completion Policy 与施工地图（P0-06）。
- **P1-foundation**：`proposed`。以 Agent-sized 纵向切片交付“可持久化、可观察、
  可换手、Evidence 归约”的最小闭环；激活需 P0-06 用户批准。
- 旧版 v0.3 快照保存在 [archive](agent_platform/dev_docs/archive/INDEX.md)，
  仅用于历史追溯。

## 文档治理

- 三个入口（人类、开发 Agent、未来的 Runtime Worker）各走各的入口文档；
- 三个 DAG（Module 依赖 / 开发 Ticket / 运行时执行）用途与生命周期不同，不得合并；
- 同一事实只保存在一个层级，顶层地图只做路由，不复制细节；
- 归档与用户需求原文不进入默认 Agent Context，按需加载。

## 文档检查

提交文档改动前运行（在仓库根目录）：

```bash
node agent_platform/dev_docs/verification/validate-docs.mjs
```

脚本校验本地链接、状态守卫、DAG 无环、Ticket 元数据与归档摘要等治理规则。
