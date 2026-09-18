# WorkspaceReader Module

> 当前代码与接线评价见 [模块审计](../../../human/module-status.md) 的对应条目。本页保留职责、Interface 要求及明确标注的首切片约定；旧切片状态不能代替当前实现结论。


```yaml
status: draft
updated: 2026-09-11
plane: Data
```

## Purpose

在显式读权限内访问源码、Git 差异及可用代码／测试索引，隐藏工作区读取差异。

## Interface

候选操作：`read(query) → sourced / unsupported / stale / rejected`。共享字段、拒绝与状态语义唯一来源为 [运行时协作契约](../../interfaces/runtime-collaboration.md)，本页不复制 wire schema。首个消费者：P1-12 首个真实源码读取切片；设计已展开不代表契约已冻结或实现。

注：运行访问控制面（WorkspaceReadLease / WorkspaceCapabilityPort，P1-07 冻结）与本模块分工不同——前者只管借用与能力声明，不读取工作区内容；本模块 read() 首个消费者仍为 P1-12。

## Dependencies

无其他产品 Module 调用依赖；文件／Git／索引能力注入。长期调用关系以 [ModuleDependencyDAG](../../../ARCHITECTURE.md#moduledependencydag) 为准；运行时事件反馈不是反向源码依赖。

## Hidden Implementation

完整探索目录的来源读取归 `src/data/workspace-reader/exploration-source.ts`，原应用路径只保留兼容导出。`ExplorationSourceApplicability` 是宿主绑定真实 Project/Workspace 根的 SourceApplicabilityPort Adapter；只接受完整 `workspace_paths:['.']`，重读两次相等后产生 pin，读取失败或期间变化明确拒绝。保留原探索摘要中普通文件/二进制、目录、链接条目和 dist 等完整范围，不改用忽略目录的代码变更摘要；链接只记身份、不跟随。该 Adapter 的 manifestDigest 保持 exploration-source-v1 语义，identity.workspace 标明算法和真实根，commit=null，不冒充 Git commit。

I阶段完整协作验证暴露大型二进制反复读取成本后，文件读取缓冲按已打开文件大小分配，上限1 MiB（空文件保留最小1字节缓冲），大文件由原64 KiB块提升；内存仍有界，摘要算法、完整读取范围、容量拒绝和前后身份检查不变，不缓存来源有效性。跨块完整摘要与末字节变化已有定向验证；完整场景耗时及最终快照结果以本批集成记录为准，不由局部吞吐改善推断产品全流程通过。

路径解析与逃逸防护、commit 与工作树快照区别、索引版本和能力检测；只读，不初始化代码目录。

## Test seam

通过上述 Interface 注入依赖 Adapter，验证：读取中变化、越界路径及符号链接、索引落后、缺少图能力和有限检索结果。 不以内部表或私有方法作为唯一测试入口。

## Context load

实现或扩展本 Module 时读取本页、当前 Ticket 与直接消费的 [运行时协作契约](../../interfaces/runtime-collaboration.md) 小节；初始协商及图文集成另读 [初始设计与统一展示](../../interfaces/human-design-status.md)。原始对话和完整历史按需追溯，不默认装入 Run。

## Context 生命周期与协作扩展

P1-17 复用当前来源读取与版本检查；既有文本／快照可降级，不能把旧执行理由当作当前代码事实。 行为依据：[Context 生命周期](../../interfaces/context-lifecycle.md)、[运行时协作](../../interfaces/runtime-collaboration.md)、[人类交互](../../interfaces/human-design-status.md)。精确 schema 在对应消费者冻结，文档同步不表示已有实现。

## 2026-09-08 实际源码查询切片

产品 src/data/workspace-reader/source-index.ts 提供 SourceIndex.query 与 excerpt，运行宿主通过同一个工作区沙箱读能力注入。探索和开发模式均注册 code_index / source_excerpt；这两个工具是本模块的实际消费者，不依赖架构图夹具。

复用已安装 TypeScript Language Service，限定显式选择的最多 64 个 TS/JS 文件，支持声明、给定位置的定义、语义引用（包括跨文件别名导入）。坐标为从 1 开始的行／UTF-16 列。读取只经过沙箱和读权限，不加载项目插件、tsconfig、package 配置或外部依赖；未选择的导入可以无法解析。coverage 始终声明局部覆盖，不是完整调用图。语法及缺失模块诊断单列，不把缺少库声明带来的类型错误冒充项目检查失败。Python 仍通过既有 symbols 工具做语法 AST，Python 跨文件语义、C++ 与其他语言未实现。

每文件上限 256 KiB，单查询源码上限 4 MiB，结果最多 200 条；这些是单次材料容量，不是累计模型预算。snapshot 是排序后的路径／内容 SHA-256 与解析器版本的摘要，表示选定工作树文件集，不表示 Git commit 或 Control workspaceRevision。每次查询重建并再次读取核对来源，变更返回 stale，删除／拒绝／超容量返回 rejected；不静默复用旧缓存。没有持久全仓增量索引，重开可从相同源码重建同一 snapshot。该核对检测可观察变化，不代替多文件原子快照或正式工作区租约。

source_excerpt 要求精确内容摘要、行范围（最多 200 行／32 KiB），变更返回 stale。工具结果包含来源与覆盖，进入内核后续模型请求及运行记录；这不是已完成上层角色补料通信协议。

依据：[TypeScript Language Service 官方接口](https://github.com/microsoft/TypeScript/wiki/Using-the-Language-Service-API)。实现沿用上述只读／时效／降级约束；既有 P1-12 CodeGraphReadQuery 不改成这个局部查询协议，正式架构图入口仍待接通。

## 当前需要补足的策略

- 可信 tsconfig／依赖索引的加载范围、Python 与 C++ 语义提供者、全仓覆盖／增量缓存策略尚未形成完整开发约定。当前保守实现显式文件集；不自动把整仓及依赖放入材料。
- 来源 snapshot 与账本 workspaceRevision、已 pin 架构 baseline 的正式映射尚缺。不能把文件集合摘要强行转成数字 revision，也不能沿用固定 revision 0 充当真实基准。
- AST 关系到产品 Module／Interface，以及哪些变化构成违规／需要人工决定的分类规则尚不具体。语义引用是真实代码事实；架构裁决需独立规则，不能根据同名符号或一条依赖边直接产生固定 Finding。

这些缺项影响正式架构对账和跨角色交接，未阻止独立源码查询实现。当前状态与证据仍回到 [模块状态](../../../human/module-status.md) 与 [基础修复证据](../../archive/2026-09-08-verification-history/2026-09-08-framework-acceptance/foundation-repair.md)。

## 全模块补齐：项目源码服务

新增 ProjectSourceIndex 与实际运行工具 project_index。可读文件发现不再要求操作者指定最多 64 个文件；解析 tsconfig/jsconfig 的 include/exclude、extends、paths 与可读工作区内依赖，配置只作数据，不执行插件。一个 Run 内复用 TypeScript Language Service，按内容摘要更新脚本版本，配置与根文件变化重建服务。新增/修改/删除/重命名及分支变化进入来源身份；重命名明确记录为删除+新增。

查询符号、定义、引用、导入与静态调用候选，携文件/位置/内容摘要、符号标识、来源 manifest 摘要及宿主读取的 HEAD commit（非 Git 仓库为 null）。全量相关输入进入摘要，输出来源清单有独立截断标记，分页不能冒充全量。读取前后重新核对清单和内容，检测变更返回 stale；不是多文件原子快照。单文件 2 MiB、快照 128 MiB、内核发现最多 60000 文件；发现不完整会拒绝。TS solution project references 当前要求分别查询引用配置，未声称完整 solution 分析。

新增 PythonSourceIndex / python_index，通过固定 Jedi 程序分析权限过滤后的临时源码副本。来源副本不含项目环境、二进制扩展、答案目录或隐藏文件，Python 项目代码不执行。每次查询新进程重建，跨文件推断可不完整，动态关系保留 unknown/static_candidate，未声称 Python 增量服务或外部依赖已完成。工具依赖通过 scripts/setup-source-analyzers.py 安装到产品 .local（Jedi 0.19.2 / Parso 0.8.4，wheel 摘要有记录）。[Jedi API](https://jedi.readthedocs.io/en/latest/docs/api.html) 是 API 来源。

C/C++ 已有受限 libclang 语义提供者，TS/JS 图已接显式源码映射与 baseline pin；这些局部能力不代表完整语言/依赖/动态调用图。全语言项目配置和跨角色补料仍待完成。验证范围与失败追踪见 [实施记录](../../verification/2026-09-09-module-completion.md)。
## 当前源码边界（2026-09-11）

`role-source-reader.ts` 隐藏角色 code 材料的原生列举、读取、根目录与路径边界；`denied-prefixes.ts` 是对应拒绝前缀的共同来源。ContextCompiler 通过窄端口索取有界索引/源码，不自行创建底层工作区读取适配。来源 pin、实际版本和不可用/越界结果保留原义；这一归位不扩展语言、持久增量或多文件原子快照能力。

`src/data/workspace-reader/source-workspace-reader.ts` 的 ProjectArchitectureSourceReader 只捕获原生图；正式 Run/Plan/Workspace 绑定与正文保存由 Context 的 SourceGraphContextCompiler 负责。QueryWorkspaceSourceReader 提供独立查询来源版本，CandidateWorkspaceReader 保留候选摘要原有路径/模式位/文件摘要算法。探索来源 pin 与各语言工具复用对应路径、排序和能力规则；不同 ignore/字节语义不可强行合并而破坏历史记录。WorkspaceReader 不调用 Ledger 或 Vault。

跨 Module 的精确入口与失败/持久兼容规则见 [当前 Module 边界](../../interfaces/module-boundaries.md)。此源码映射不代替整体功能验收。

VR-01 的 `VerificationWorkspaceReader` 复用 Candidate 原摘要与同次 inventory，独立 Git 根只读 ls-tree 原始 blob 比较，不运行项目 clean filter。非 Git、外层 Git 子目录或比较不可靠时保留当前快照并明确变化范围未知；HEAD 比较不能冒充 Run 前态。源码/HEAD 读取不稳定拒绝，范围不完整不截断冒充成功。Context 负责后续 canonical 身份绑定，详见 [来源交接](../../verification/2026-09-09-core-verification/context-handoff.md)。

VR-02 的 `verification-source-applicability.ts` 提供显式 `verification_workspace` 来源集合，`reviewer-source-reader.ts` 提供有界原文和同一可见路径策略。Candidate排除目录在任意路径层均不可读，不改变旧探索全目录pin。Runtime复用既有索引/工具并注入该策略，WorkspaceReader不承担材料授权或Evidence接纳；详见[独立审阅](../../interfaces/independent-review.md)，本批已按[VR-02验收](../../verification/2026-09-09-independent-review/acceptance.md)确认限定能力。
