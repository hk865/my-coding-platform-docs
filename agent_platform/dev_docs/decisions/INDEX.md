# Agent Platform 决策索引

本目录记录已经接受、难以逆转且需要保留理由的架构决策。ADR 只保存决策及其原因；当前术语、接口和施工状态仍以顶层地图、Module、Interface 与当前 Ticket 为准。

2026-09-11：ADR 0003 顶部已记录本次用户确认的范围及 RW-18 替代关系；MigrationGate 双判据是必要条件，requiredOutputs 不再单独构成完成门禁。原文保留，不以历史条款覆盖后续确认。审查与实现仍暂停。

| ADR                                                     | 状态     | 决策                                                                        | 读取条件                                                  |
| ------------------------------------------------------- | -------- | --------------------------------------------------------------------------- | --------------------------------------------------------- |
| [0001](0001-separate-platform-from-execution-kernel.md) | accepted | Agent Platform 与 Execution Kernel 是两个产品边界                           | 调整产品归属、RuntimePort 或 `coding-agent` 职责时        |
| [0002](0002-module-dag-and-tracer-bullet-tickets.md)    | accepted | 架构、开发与运行时使用彼此分离的图；开发 Ticket 采用 tracer-bullet 纵向切片 | 设计 Module、拆 Ticket、声明 blocking edge 或运行时依赖时 |
| [0003](0003-rework-role-spec-architecture-reconciliation.md) | accepted | 返工以新的 PlanRevision 承载并默认自动受理；角色规格实体化并绑定真实 Run；架构对账分机械规则与语义审阅两层；MigrationGate 双判据 | 实现返工、角色绑定、ContextBundle 取材、架构对账或基线演进时 |
