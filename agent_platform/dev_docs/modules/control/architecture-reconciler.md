# ArchitectureReconciler Module

> 当前代码与接线评价见 [模块审计](../../../human/module-status.md) 的对应条目。本页保留职责、Interface 要求及明确标注的首切片约定；旧切片状态不能代替当前实现结论。


```yaml
status: draft
updated: 2026-09-11
plane: Control
```

## Purpose

对照有效 Baseline 与实际源码产生差异、Finding 和可审阅的演进材料。

## Interface

候选操作：`inspect(intent) → assessmentRef；演进操作由 P1-14 冻结`。共享字段、拒绝与状态语义唯一来源为 [运行时协作契约](../../interfaces/runtime-collaboration.md)，本页不复制 wire schema。首个消费者：P1-12；13/14 扩展修复与演进；设计已展开不代表契约已冻结或实现。

## Dependencies

ControlEngine、ContextCompiler、ArtifactVault。长期调用关系以 [ModuleDependencyDAG](../../../ARCHITECTURE.md#moduledependencydag) 为准；运行时事件反馈不是反向源码依赖。

## Hidden Implementation

差异分类、授权内修复提案、候选基线物化与迁移要求；源码变化不自动改写规范。

每一步 Control 登记必须消费回执：只有 `committed` 才继续；拒绝返回 `fail_closed/recording_rejected`，diagnostics 保留拒绝的 commandId、code 和 issues，停止后续登记。此前已提交的记录保留，不能把部分提交说成整个 inspect 成功。此修复不代表实际 delta 到 Finding 的语义路径已完成。

## Test seam

通过上述 Interface 注入依赖 Adapter，验证：索引落后、source baseline 移动、候选 digest 不匹配、无授权激活、迁移失败。 不以内部表或私有方法作为唯一测试入口。

## Context load

实现或扩展本 Module 时读取本页、当前 Ticket 与直接消费的 [运行时协作契约](../../interfaces/runtime-collaboration.md) 小节；初始协商及图文集成另读 [初始设计与统一展示](../../interfaces/human-design-status.md)。原始对话和完整历史按需追溯，不默认装入 Run。

## Context 生命周期与协作扩展

P1-12 支持没有代码变更的版本化接口／架构问题输入；P1-14 保留提案、决定、迁移门禁，不能因模型共识激活 baseline。 行为依据：[Context 生命周期](../../interfaces/context-lifecycle.md)、[运行时协作](../../interfaces/runtime-collaboration.md)、[人类交互](../../interfaces/human-design-status.md)。精确 schema 在对应消费者冻结，文档同步不表示已有实现。

历史记录（不作为当前待办）：2026-09-08 纯图差分新增同 structuralKey 的边端点／类型变化及节点路径移动检测。该算法回归通过不表示 Reconciler 主路径已消费真实 delta；固定 Finding 与 revision 0 基准仍待修。

## 历史缺口：2026-09-08（由下方 2026-09-09 来源绑定实现替代）

当前 ArchitectureBaselineContentV1 仅含 description 与 constraints(name, scope)，没有 pin 到源码 commit／工作树快照／CodeGraph 版本的字段或解析规则；主路径因此仍硬编码 revision 0。还缺源码节点与产品 Module/Interface 的映射，以及 delta 到 material/risk/ambiguous 的可执行分类规则。文档已规定 pin 权威性和回执处理，但不足以凭空决定这两个映射。需要补明确版本化来源绑定及分类约定后，替换固定 Finding／Brief／Proposal；不能将新局部语义索引当作已完成架构对账。


## 2026-09-09 精确来源绑定与分类（兼容扩展）

ArchitectureBaselineContentV1 可保存 sourceBinding，内容本身纳入既有不可变治理摘要。绑定包含 Project/Workspace、真实数字版本、commitHash（可空）、完整许可源码/配置 manifest 摘要、indexVersion、configPath、显式 Module/Interface 路径映射、机械节点/边与未解析关系。它不反向引用自身 baseline pin，避免循环摘要。旧基线保持可读；缺少绑定时机械检查 fail_closed，不回退 revision 0。

真实 SourceWorkspaceReader 通过 Run 读权限、Plan pin 与 Workspace 账本版本核对后，调用 TypeScript 项目 Language Service 的完整 imports 材料，按显式映射折叠图。当前只声明 TS/JS，未映射源码、缺失依赖均保留 unresolved；不将目录名称猜作产品模块，不把静态 imports 称为运行时调用图。源码前后摘要核对与数字版本核对并行生效，仍非文件系统原子快照。

Reconciler 核对实际 PlanRevision pin、治理内容摘要和真实 reader Run。基线/当前图、真实 delta、Brief 正文保存 Vault，保持 reader owner。每条机械变化产生来源可追溯的 Finding：普通结构差异低风险且不判正确性；接口/依赖变化为待审语义问题；新未解析关系为低置信问题。自由文字 constraints 没有被猜测成可执行规则。无变化不产生固定 Finding；超出完整差分/单 Brief 上限明确拒绝，不省略问题。

每步提交消费 committed 回执；部分提交保留，重试使用已存 inspection 时间和相同身份。Brief 选项要求调查或另行提出精确修订，不自动生成已选业务方案；候选物化完整保留 sourceBinding。模型/源码变化均不激活基线。初始基线准备/演进 UI、规则判定与真实角色消费、完整恢复仍待后续闭环。

可选 dependencyRules 只支持 forbid_dependency，规则 ID 唯一且两端必须是 sourceBinding 显式 Module。新增违规依赖的 Finding 为高风险/material，保留具体规则、机械变化和 delta 来源；自由文字 constraints 仍不构成可执行规则。规则正文与 sourceBinding 在候选物化时完整继承，不改变激活协议。
## 当前源码边界（2026-09-11）

当前 inspect 执行取材、差分、Finding/Brief 构造与逐步登记；源码读取及版本供材仍经 Context。本轮同步当前事实，不声称长流程与压缩排版已完成可读性重构。上方 2026-09-09 来源绑定条款是已实现的局部机制，早期 revision 0 与固定 Finding 缺口保留为历史；真实初始基线、产品 inspect 入口、语义架构审阅与 MigrationGate 完整行为仍以唯一模块状态的未完义务为准。

`architecture-reconciler.ts` 通过 ArchitectureContextPort获取精确版本材料，再计算delta、分类Finding、保存正文并逐个检查Control回执。`baseline-evolution-port.ts` 经 BaselineEvolutionContextPort取材后物化候选，Context拒绝不被替换成空图。候选/迁移/激活产品链是否全部接通另看模块状态，不根据这些局部入口推断。

跨 Module 的精确入口与失败/持久兼容规则见 [当前 Module 边界](../../interfaces/module-boundaries.md)。此源码映射不代替整体功能验收。
