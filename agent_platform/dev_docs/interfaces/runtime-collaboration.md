# 运行时协作 Interface：角色、Context、通信与查询

## 2026-09-14 公开运行事实的持久发布（IG11）

普通 Runtime 的公开 all、RunHandle 事件和模型请求证据只读取 journal 已提交的观察；独立 Query 的公开 all 亦如此。执行器内部 records/active 仍负责执行与取消控制。尚在保存中的终态或 usage 不得先交给 UI、Context 或 Control 当已发生的持久事实。

终态事件先原子保存，再进入内部事件列表并 wake；保存失败走未知效果处理，不把未确认的 completed 事件夹带进下一次保存。模型请求证据仍在真实调用边界产生，只有对应 usage/context 已保存后才对外发布。Query 结果入口继续等最终保存；失败不返回未持久答案，恢复按原 unknown 规则处理。该边界不把 Run completed 当 Task/Goal 完成，也不放宽 Verification 的终态/来源核对。

## 2026-09-14 普通只读报告验证（IG10）

任务轮次配置增加显式 `mode: readonly-report`、`kind: static`、`requiredReadPaths` 与原 `appliesTo`，不得携带 command/cwd/timeoutMs。未带 mode 的旧配置仍按命令检查处理。该方式只适用于当前普通只读 Task 的已完成 Run，不适用于 GoalGate，也不为 Run 增加写权限。

Context 从精确 Run 的持久公开观察取得最终报告、成功读取的路径/版本/行区间与工作区副作用事实，并核对最新 Task Run、Plan、Workspace、策略和源码身份。WorkspaceReader 核对每条读取的当前版本及区间覆盖；必读路径需要覆盖当前完整文件，单行或截断读取不足以满足。合法 Host 协调工具的持久协作动作不等同于工作区写入；未知工作区副作用仍不能产生 PASS。

Verification 只判定报告存在、必读来源完整且当前、只读权限及工作区副作用条件，把实际报告正文和观察保存到原 Vault/check journal。它不判断报告内容正确性。检查使用 `observation_complete` 终态，没有命令或写租约；普通命令仍要求 `lease_released`。精确观察未落盘的中断不能重新选择一份改善后的报告刷成通过。已保存观察的恢复沿用原身份和正文，并重新核对当前性。

原始检查材料进入既有独立 Reviewer 的报告读取入口；Reviewer 必须对实际报告及源码判断正式 reviewer 义务。static PASS 不替代 reviewer PASS，Task/Goal 仍由 Control 根据当前有效 Evidence 归约。公开 HTTP、轮次记录及 UI 使用同一配置和持久报告；开发中定向结果与未完成的端到端验收以当前模块状态及测试证据为准。

2026-09-11通用角色模板的产品方向见[PRODUCT](../../PRODUCT.md#通用角色模板与工具组合)。后续契约设计优先沿用RoleSpec/RoleBinding、任务级授权及现有Context和派发路径；模板名称不赋予Reviewer资格或工具权限，具体运行资格仍由现有协议和政策核对。本引用不表示候选AgentTemplate字段已成为wire schema，也不改变12 Module依赖。

## 2026-09-11 定向执行反馈增量

普通工作Run在派发Context中获得版本化公开反馈协议（execution_feedback_protocol v1），其Artifact绑定实际RunRef并记入manifest；不依赖任务指令或测试替身私有知识才能得知JSON反馈格式。协议本身不授予工具权限或验收资格。

`QueryJobIntent.execution.kind=execution_coordination` 复用独立只读 QueryJob/QueryRun；`feedback` 固定原 Run、Task、Plan、Workspace revision、公开报告及真实 sourcePin。Context 从持久公开观察提取有界 `execution_feedback`，正文按原 Run 保存；PlanCompiler 提交调查意图，Control 核对身份和版本，Dispatch 签发精确报告授权后才启动。来源、权限或必需材料缺失不启动模型。

协调结果为 `feedback_resolution`，区分 continue/supplement/adjust_plan/needs_decision/blocked 及 available/proven_empty/unavailable/forbidden/stale/investigating。模型文字不是调查证明：WorkerRuntime 从成功 read 的公开结果产生路径/版本见证；只有完整文件读取可证明空正文，不能用空行冒充空文件，也不证明未读取路径不存在。

后继 Run 仅接收同一正式 WorkContextBinding 中关联 Run 的定向回答。Context 核对回答身份、来源见证、当前源码与精确 Vault 授权；Dispatch 不改变原工具权限。材料加入现有 RuntimeContext rules/manifest，保存正文摘要、选择原因和来源，不能解除正式义务。读源码期间变更、调查未完或需要人的决定均拒绝依赖该材料。刷新复用 QueryJob，使用 supersedesQueryJobId 关联精确同工作、原 Run、报告和失败集合；合法替代未完成时后继仍阻塞，旧答案保持可读。

工具 FAIL 材料由 Verification.openIssues 经组合根交 PlanCompiler 请求只读调查。协调角色必须实际读取来源后返回 adjust_plan；ReworkPlanCompiler 将具体修复指令、精确 QueryJobAnswerRef 与答案摘要纳入原正式返工提案。Dispatch 提交前经 PlanCompiler→Context 再核对实际来源，Control 只从账本核对答案、失败绑定、指令及已有权限/版本/义务；不反向调用 Context 或 WorkspaceReader。后继组装再次核对来源，并只接受与正式受理计划匹配的调整。多文件来源核对不是外部文件系统原子事务。

必要的目标澄清使用有界 needs_decision 选项，包含目标、影响、推荐理由和可独立继续的工作。用户从现有 UI 提交精确答案引用及选项 ID，PlanCompiler 复用普通目标变更提案，Control 记录 human UserDecision 并应用 PlanRevision；不以模型答案冒充人的授权。这里只支持保持验收义务的目标澄清，验收或架构基线改变仍走各自正式入口。仅有提案而没有精确已授权决定时，必须重新核对当前来源；已记录决定的部分提交可幂等恢复。决定引用随替代调查和后继材料传递，实际 canonical 决定正文进入运行 Context。

宿主启动从持久 Query/决定/终态运行与既有验证 journal 重建后续工作：先补全已记录决定的计划应用及投递，再调查未处置 FAIL、正式返工、重验和必要 Reviewer。已结算身份幂等重放；prepared/running/outcome_unknown 不靠重复启动内核猜结果。异常或来源缺口保留待处理状态。

返工后 `VerificationService.reverifyRework` 通过 Context 读取正式受理的来源，继承原实际适用工具配置并限域到新承担者，复用 round/check journal。`prepareReworkReview` 只对合格工具轮次和必要 Reviewer coverage 准备既有独立审阅工作；组合根负责调用 Dispatch。新计划保留原独立审阅协议，Task/Goal 仍由 Control 按当前 Evidence 归约。普通 Run 返回 not_rework，来源冲突和未知副作用不靠盲目重跑解决。

上述契约是本次增量；全部恢复、连续多次补料与端到端完成范围以[当前模块状态](../../human/module-status.md)和本次证据为准，不把接口存在当作完整产品验收。

```yaml
status: draft
updated: 2026-09-11
scope: 已认可职责方向的扩展契约；非已实现或冻结的 wire schema
```

依据：[ARCHITECTURE](../../ARCHITECTURE.md)。本文件定义跨 Module 的行为与必要信息；现有 [Command/Event](command-event.md)、[StateLedger](state-ledger.md)、[GoalView](goal-view.md) 仍保留 CreateGoal 首切片类型。实现扩展时显式升级对应 schema，未知版本继续拒绝，不能以宽泛 payload 绕过验证。

完整初始设计与图文组合的消费契约见 [初始设计与统一展示](human-design-status.md)，P1-15 负责整合；不再留给无归属的未来工作。

## 共享引用与角色绑定

所有请求携带调用身份、Project／Workspace scope、correlation、schema version；改变状态的请求另带幂等键和预期版本。来源引用携带来源种类、对象或路径、revision／digest、适用范围；时间戳不能替代版本。

角色模板允许自定义职责名称、步骤与输出，实例化要求见 [自由模板](../agent/templates/short-lived-agent.md)。模板本身不授予权限。

RoleBinding 至少关联模板版本、Agent／Task、QueryJob 或 CoordinationJob、职责、授权策略版本、工具／读写范围、预算和终止条件；Run 引用绑定版本。ControlEngine 接受与撤销绑定，StateLedger 保存事实，ContextCompiler 消费绑定，DispatchEngine 在启动与安全检查点核对有效性。模板变更不追溯改写已有绑定；失效绑定不得启动新 Run，运行中变更按安全点控制处理。

## Context 生命周期与执行记忆

连续范围、事件、留痕、恢复与历史适用性的单一正文为 [Context 生命周期](context-lifecycle.md)。P1-16／17 增量实现；P1-03／06 冻结版本不被本文文字追溯修改。角色绑定、Run 与工作关联支持接续，精确扩展字段由首个消费者版本化冻结。

## 语义编排与跨工作包交互

Semantic Coordination 属于 Control Plane；模型负责理解、分工提案、冲突分析与建议，确定性控制负责授权、版本、预算、路由、派发和状态归约。框架不裁决业务选项优劣，模型的共识不直接激活规范或完成任务。

1. 协调者据已接受规范与计划提出工作包、负责人、输入输出和集成安排，框架接受后派发；包工头可在范围内提出执行、调查和交流请求，所有派发校验权限与预算。
2. 执行者或包工头可在调查、协商或对账时提出 CoordinationIssue，无需测试先失败。议题关联工作、参与职责、来源版本、各方观点、期望答复、预算与收敛责任。
3. 跨包议题复用共同集成负责人；没有时由协调角色提案、框架登记负责人。各方经有界消息直接澄清，必要时正式派发调查。书记按需整理来源、分歧和未解项，不成为强制中转站。
4. 契约内澄清、分工和已授权返工可继续；拟改变已接受架构、模块职责或跨模块接口签名／行为时，由秘书／参谋组织选项、影响和上报。通知与决定规则见 [人类交互契约](human-design-status.md)。
5. 等待新决定时，保存议题、待答复人和工作前沿；仅阻塞依赖未定选择的动作，不影响独立工作。预算耗尽或无法收敛形成可见未解项，不无限讨论。
6. 人的决定绑定精确提案版本；框架校验后经规范／baseline／Plan 及迁移验证路径受理。架构激活不自动改写旧 Plan pin；受影响 Plan 必须显式迁移。
7. 决定及理由路由到所有受影响工作，记录投递及材料刷新／重建结果；执行前 guards 复核当前绑定与版本。仅发出通知不证明 Context 已更新，旧结果不能推进新版本。

执行指令在声明的安全点加载；查询仍走独立只读路径。议题与等待状态可由新协调 Run 从持久记录接续，独立 Worker 不因某个包工头退出而终止。交互失败沿消息提交的幂等、重试与未受理 Artifact 规则处理。

## 当前返工材料与只读解释（2026-09-11）

Verification 的原始问题来自其持久检查/审阅记录，当前义务承担者与重验适用性由 Control 的 ReworkDispositionPort 复用正式 Evidence 资格解释。组合根将问题材料传给 DispatchEngine；驱动不反调 VerificationEngine，每次计划换版后向 Control 重新核对剩余问题。缺少问题材料返回 unavailable，缺少当前归属返回 unknown，二者都不能视为已处置。

运行产出的 canonical 读取通过 Context 的 RunOutputMaterialPort 提供给 Verification；该材料不会直接完成任务。角色规格 requiredOutputs 仅为声明性产出期望，Plan 的正式验收义务仍按当前证据归约。精确 TypeScript 形状见产品 contracts/rework/issues.ts、rework/drive.ts、run-output-materials.ts；ModuleDependencyDAG 不变。

## 各 Module 的扩展 Interface

| Module | 候选操作与可观察结果 | 调用依赖／关键约束 |
| --- | --- | --- |
| HumanCollaboration | `query(request)` → fact / report / pending / unavailable / rejected；协调请求交 PlanCompiler | 事实读 ReadModelIndex；公开快照读 DispatchEngine；新 QueryJob 交 ControlEngine。是否需要解释由请求指定，不强制先过参谋 |
| PlanCompiler | 当前 `request(amendment)` → proposal / needs_material / rejected；`requestInitial(intent)` → accepted(submission) / rejected；`accept(trigger)` → processed / rejected | ContextCompiler 提供材料；通过 ControlEngine 持久化初始协调请求并受理结果。已接同工作 FAIL 语义调整和目标澄清决定回流；完整咨询、跨工作冲突协商仍需真实消费者；源码协议见 contracts/planning.ts |
| ControlEngine | `submit(command)` 保留；扩展角色绑定、工作请求、报告登记、消息路由与提案受理的已知 command 类型 | 从 StateLedger 读取 canonical state 做授权、CAS 与幂等校验；原子提交状态、事件和派发 intent。不依赖 PlanCompiler 或 WorkerRuntime，不在 reducer 内调用模型 |
| DispatchEngine | `drive(trigger)`（候选：accept(runtimeEvent)）；公开运行快照经 WorkerRuntime 控制面（P1-06 冻结 HandoffControlPort.snapshot，noHiddenContextRead）→ report / unsupported / stale / rejected | 消费 ControlEngine 的 durable intent，ContextCompiler 编译，ArtifactVault 存取正文，WorkerRuntime 执行或提供快照；所有角色复用该路径 |
| ContextCompiler | `assemble(request)` → ready(bundleRef, manifest) / needs_material(gaps, selectedRefs) / rejected(reason) | 消费 StateLedger、ReadModelIndex、ArtifactVault、WorkspaceReader；不分配角色或启动模型。ready 前保存有界 Bundle；缺口交调用方经 Control 路径补充，不在内部无限重试 |
| WorkerRuntime | 原 capabilities/start/control/events；新增可选 `snapshot(query)` | capabilities 显式声明快照支持；无此能力返回 unsupported。只暴露公开报告，不询问源 Run 的隐藏上下文 |
| WorkspaceReader | `read(query)` → sourced(result, provenance) / unsupported / stale / rejected | 读取显式 Workspace 的源码、Git 差异及可用代码／测试索引；检查路径、读授权与版本。不创建 Task，不写 checkout |
| ArtifactVault | `put(record)` → immutableRef；`open(ref, accessScope)` → record / unavailable / rejected | 保存报告、交接、Context、语义提示与来源。校验来源元数据不意味着验证正文结论；不依赖 ControlEngine |
| ReadModelIndex | 在现有 goal 查询外增加角色／Run／报告引用查询 | 从已提交事件重建，携带 scope、cursor、来源；不把报告文字投影为正式完成状态 |
| StateLedger | load/commit/events 形状保留，增加角色／工作／通信元数据对应的版本化对象与事件类型 | CAS、原子提交和有序事件语义不变；不存重复源码库，不做语义裁决 |

这是目标扩展面；各操作的具体 wire 字段、枚举和兼容策略在对应首个消费者冻结。实现者不得仅凭此表跳过其纵向契约测试。

## Context 与工程来源

WorkspaceCapabilityPort 的当前实现归属为 ControlEngine 的能力受理政策：宿主将运行时明确支持的 read/write/maxWriteScope 作为配置事实传入，Control 求它与 envelope 权限的交集，缺配置返回 unsupported。既有 capabilitiesFor 的 wire 形状保持；source=runtime 仅说明支持声明来源。该政策不调用活 WorkerRuntime，运行时实际启动/沙箱预检与失败报告仍走 Runtime/Dispatch。此归属澄清消除旧文档将纯受理政策标为 WorkerRuntime 而导致的 Control→Runtime 反向依赖；不扩大任何读写权限。

assemble 输入为已接受工作及 RoleBinding 引用、当前 Workspace／规范版本、所需材料、访问范围和硬预算；输出 manifest 列出实际选取来源、版本、截断／缺失与 freshness。规范义务材料不足或超预算时返回缺口，不静默截去后声称 ready。

WorkspaceReader 明确区分提交快照与含未提交变更的工作树快照；后者需可检测变更的 digest／快照标识。读取中内容变化或代码索引不匹配时返回 stale，调用方重新取材。缺少图索引可使用显式允许的源码／文本检索降级，并标明覆盖不足。

当前事实、开发轨迹与语义提示复用原始来源，不统一复制到 MemoryStore。普通笔记可在已有权限内持久化；只有改变规范、授权或验收时才提交相应提案。检索到历史经验不等于当前有效约束。

需要新语义检索时，调用方通过 ControlEngine 请求有界临时工作；完成后以结果引用重新 assemble。相同工作复用 correlation／幂等身份，预算耗尽返回缺口。ContextCompiler 不依赖 PlanCompiler／DispatchEngine，因而不会形成检索→启动 Agent→再次检索的源码循环。

## 消息与记录提交

有界消息区分 assignment、question、report、proposal、handoff、discussion；包含发送者／接收绑定、所属工作、来源版本、正文引用、correlation、响应要求与有限预算。普通消息不是 RuntimeExecutionDAG 的阻塞边。

正文先经 ArtifactVault 保存，ControlEngine 校验并登记引用与路由事实；大正文不进入 reducer。持久提交前不派发，重复提交不重复唤醒，旧绑定／越权接收者被拒绝。保存正文后提交失败只留下未被采纳的 Artifact，不能显示为已接受事实；清理按独立保留策略处理，不依靠跨存储假事务。

Report 证明“某 Agent 报告了某事”，不证明其业务结论。Message、已提交 Event、状态转换分开；CompletionClaim／Reviewer verdict 仍经 [完成策略](completion-policy.md) 处理。提案结果变旧时必须重新校验，不能依赖此前 Context 中的授权。

## 查询与运行反馈

查询明确选择事实、公开执行报告或新语义回答。fact 来自已提交投影；report 附作者、Run、版本、产生时间与未验证标识；pending 返回独立 QueryJob 引用。unsupported／stale 不自动伪装成最新回答，只有请求允许且预算授权覆盖时才转 QueryJob。

QueryJob 不改变源 Worker 的 Context、lease、预算或任务义务。快照也是只读能力，不向源 Run 注入消息。需要交流时使用单独的有界消息路径，不能把查询隐式升级成执行指令。

Host 消费持久 intent／结果事件，调用 DispatchEngine、PlanCompiler、VerificationEngine 或 ArchitectureReconciler。语义 Run 均由 DispatchEngine 启动；验证与对账所需模型工作同样提交正式工作请求。结果经 Control 提交后再驱动后续处理，而不是 ControlEngine 直接反向调用这些 Module。

## P1-04 extension record：verification / review 冻结（首个消费者）

2026-09-05 [P1-04](../planning/proposed/P1-foundation/tickets/04-evidence-satisfies-task.md) 首次冻结（wire 字段见产品代码，本文件不复制）：

- CompletionClaim／Reviewer verdict 只证明“报告了某事”，不能独立满足 Task/Gate；claim 论证不是 Evidence PASS（EvidenceV1 kind=claim 强制 INCONCLUSIVE）；
- VerificationEngine.verify（VerificationPort，P1-04 首个消费者）与 ContextCompiler.ReviewContextPort(P1-04 版本化扩展，assemble(TaskContextRequestV1) 冻结签名不改) 的 wire schema，以及 CompletionClaim/VerificationPlan/Evidence/EvidenceBinding/EffectiveEvidenceSet/ReviewPacket/VerificationResult 七个契约：见产品代码 src/contracts/{verification,review-context,evidence,reduction}.ts 与 IMPLEMENTATION-HANDOFF.md「P1-04 契约与存储语义（冻结）」；
- 语义要点：评审工作走正式 dispatch Run；ReviewPacket 有界（材料数/摘要/字节上限，无完整 transcript，正文 body-first 入 ArtifactVault）；无语义变化只允许版本化策略显式快放；控制（ControlEngine）是唯一写 TaskReduction phase 者，ReadModel 只投影展示。
## P1-18 extension record：material-access 冻结（首个消费者）

2026-09-08 [跨主体材料读取切片](../verification/2026-09-08-p1-18-material-access.md) 首次冻结（wire 字段见产品代码，本文件不复制）：

- `ArtifactVault.open(ref, accessScope)` 保留 P1-03 owner-only 默认；新增可选 `currentBasis`（读者当前来源版本基线）与注入式 `MaterialAccessResolver`。非拥有者读取必须同时满足：记录中的 grant 指名该读者、包含该精确材料、基线与读者声明一致、签发者是材料拥有者或 Control。基线不一致返回 `rejected/stale`（新增决议码）；无解析器时行为与 P1-03 一致。
- `MaterialAccessGrantV1`（grantId、scope、materials≤64、reader、issuedBy、purpose、basis、grantedAt）经 `ControlEngine.grantMaterialAccess` 以不可变 `material-access-grant`（CAS@0）登记；账本为权威，读模型 `materialAccessGrants` 只作展示与宿主解析入口，本身不做权限判断。
- 授权不改变 Task／Goal 相位、不满足 Evidence、不证明正文业务结论；Control 不读 Vault，存在性由读取时 `unavailable` 回答。内容寻址重放仍保留首次拥有者，第二个生产者读取同一正文需要自己的授权。
- 首个真实消费者：探索路径前驱报告经授权从 Vault 重组，并与已验收 reportDigest 核对（产品 src/app/explorations.ts）。审查运行、接续运行与长期继承复用同一契约。
## 冻结时必须覆盖的测试

- 自定义临时角色可运行与交接；过期绑定、越权工具／路径、写 lease 冲突被拒绝；取消和预算耗尽可安全结束。
- Context 版本错配、必要材料缺失与预算不足显式失败；语义补充重试有限且不自行扩大权限。
- 工作树变更及陈旧 CodeGraph 可检测；无图能力明确降级或返回缺口。
- 普通记录无需逐条人批；报告不能完成任务；权限／规范变更仍经正式受理。
- 消息／结果重复、迟到、提交冲突与正文已保存但提交失败均不重复派发或改变未经接受的状态。
- 直接查询无模型调用；可选快照 unsupported／stale 如实返回；独立 QueryJob 不干扰源 Run。

本文件对应 P1-02/03/04/06/08/09/12 等消费者，仍受 P0-06 与 MVP 范围复核约束。候选契约同步不等于授权开工。
## 2026-09-09 材料授权边界修复

材料授权读取增加精确候选查询 materialAccessCandidates({reader, material})；与最多 256 行的展示查询分离。grant 的 Project/Goal/Workspace 不得只检查“分别存在”：Control 核对真实归属，宿主解析器在读取时重核旧记录。Control 的材料签发权限只在本授权作用域内，首次 owner 的边界见 ArtifactVault Module；这不提供跨项目／跨 Goal 历史转授权。

探索报告新增 scoped-json-v1 分块编码并绑定原始生产者，旧 manifest 读取规则保持，不能以内容寻址重放转移 owner。currentBasis 声明比较和通用版本自动作废保持区别；Reviewer、接续、撤销与长期记忆没有因本修复而完成。

## 全模块补齐增量：材料撤销与历史精确读取

新增 RevokeMaterialAccess：可信宿主携完整 grantRef、reason、CommandIdentity 和 expectedRevision=1 提交。MaterialAccessGrant 聚合从 1 到 2，原 grant 不变，另存 revocation；MaterialAccessRevoked 事件与快照同事务提交。两套 Ledger 检查原 grant 未被改写，幂等重放不复活授权。两套投影保留撤销原因/来源；权限解析在读取时重核 canonical grant，因此投影落后也不能沿用已撤销授权。原 owner 的历史读取保留，撤销不删除正文。

MaterialAccessGrantV1 可选 history={owner: 精确 Run/QueryRun, usage: historical_explanation} 是兼容扩展。仅可信 Control 可签发，默认要求来源/目标同 Project、同 Workspace；来源 Run 的 Goal 与真实 Workspace 在提交和读取时均复核。Vault 核对首次 owner 与 history.owner 完全相同，不重新 put 搬运正文。该授权只授予新 Run 对精确材料的历史读取，不继承旧权限、规则或完成状态；新的授权也可以撤销。2026-09-09 用户明确决定允许显式跨 Workspace 授权，禁止自动共享。兼容字段 history.crossWorkspace={sourceWorkspaceId, authorizedBy:{kind:human,id}} 仅用于同 Project 的不同 Workspace；命令 actor 必须与 authorizedBy 完全相同，Ledger 同时核对事件与命令身份。Control 与读取解析器核对来源真实 Workspace/Goal/Run 及目标精确 Run；缺少该字段的旧授权不扩大范围。源历史正文不因源代码变化而消失，但不能凭此当作当前规则或新完成；目标 Plan/Workspace 适用性仍独立复核。

宿主 currentBasisValid 从账本核对 Workspace 当前 revision、Goal 当前非空 activePlanRevision 和读者 Run.planRef；仅声明旧版本相等不能绕过账本已推进的版本。sourceDigest 的当前性仍由具体源码材料消费者核对，尚无通用源码 pin 注册协议。旧无条件授权是历史读取权限，不证明材料适用于当前任务。



## 2026-09-09 来源图与检查日志兼容扩展

架构基线内容可携 sourceBinding（src/contracts/architecture-source.ts），绑定精确许可源码 manifest/commit/indexVersion 与显式 Module/Interface 映射。CodeGraphSnapshot 可携 sourceSnapshot；真实读取要求 requesterRunRef，省略时不伪造正文 owner。旧无绑定基线不再用 revision 0 机械比较。ArchitectureInspection 的 maxTokens 可为 null，纯机械路径不生成累计模型预算。Finding/Brief 的来源从实际 delta 或 reportInput 生成，基线激活仍走正式决定。

命令检查应用日志保留执行前意图与报告落盘 checkpoint；effects 与结果分类独立。原报告经 check-evidence 进入 Ledger Evidence，精确正文摘要与当前源码/计划匹配是接纳条件；reconcile-check 只对账已证实结果与租约，不能重放命令。阶段证据与未覆盖项见 verification/2026-09-09-module-completion.md。

## 独立只读语义运行增量

探索当前材料读取已接入 MaterialBasisV1.sourcePin：Context 从宿主可信 SourceApplicabilityPort 采集，Dispatch 以原报告时间和完整 basis 生成精确新授权，Vault 每次读取重新核对实际源码与 canonical 授权/版本/撤销。旧 digest-only 授权不自动补 pin、不放宽；当前树必须先与已接受探索摘要一致。前驱报告与操作者审阅保持原 owner，历史授权仍只提供历史解释权限。证据与限制见 [模块职责修复](../verification/2026-09-09-architecture-repair.md)。

QueryJobIntentV1.execution 可选携 kind（semantic_query / initial_coordination）、RoleBinding 和 RuntimeBudget。旧记录保持原查询约束；新记录允许无实施计划的 Goal/Workspace 只读 Context，仍由持久 Job/Run 意图及启动 CAS 驱动，不造 Task/Plan、不推进 Goal。初始协调只在无 activePlan 时组装材料；真实任务分工与提案受理正在后续切片接线。

真实只读 Adapter 与执行者使用同一个 observed-model-run：模型配置、最终输入容量检查、公开事件、工具结果和实际用量同源；只读运行仅开放读取和源码工具，不拥有 Coder 写租约、编辑或 shell。记录保留输入摘要、角色、源版本前后值；运行中来源变化返回 gap，展示时来源/Goal/Workspace 变化标历史。恢复中的 running 记录保持 outcome_unknown，不自动重跑。真实模型验收与完整中断恢复尚未执行。



## 2026-09-12 可扩展角色模板与协作选材增量

沿用 RoleSpecRevision、ProjectRoleSpecActive 和协调策略角色矩阵。InstallRoleSpecRevision.payload 新增可选 revision（正整数）；省略仍是 1，保留旧命令的摘要与幂等口径。每个 (projectId, roleId, revision) 独立 CAS@0，安装新版本不自动激活、不改旧版本及矩阵 pin。角色视图列出所有已观测安装版本；当前可选 pin 优先指向生效版本，没有生效版本时指向最新安装版本。扫描不完整仍显示缺口。

初始规划可以输出自定义模板标识；Context 提供当前矩阵中已安装、摘要匹配且已激活的模板正文与精确来源。计划派发按模板上界与本任务授权取交集，Control 仍执行正式受理。无矩阵时仅沿用 executor/integrator 兼容入口，自定义角色不能以名称绕过规格。Runtime 以信封权限选择 read/edit/shell 及 read 所属的索引工具；探索与独立 Reviewer 保持原只读条件和资格核对。

协调查询的 collaborationWork 提供当前计划各工作指派、依赖及持久 TaskReduction 的引用、版本与依据；reductionMatchesPlan 明示是否匹配计划，缺记录为 null，超过 64 项的遗漏单独计数。它是跨工作重规划的材料基础，不将旧归约、运行结束或模型摘要当作当前业务完成；不声称已具备任意多工作包的持续联合协商。

产品设置支持创建有名称、简介、职责视角和现有工具集合的模板；安装、版本激活与矩阵选择使用原治理路径。尚未提供外部工具适配器注册、任意 Prompt 文件编辑或完整 Agent 实例管理。模板产出期望仍不替代任务验收义务。


## CM-1A-001：普通 Task 与协作后继的实际执行协议

本票复用现有 Control/Ledger、唯一 Dispatch drive、ContextCompiler、Vault 与 Runtime。Host 协调工具由 exact Run/当前参与关系授予；请求/回应正文先入 Vault，再提交正式命令，只有已提交回执可报告受理。Work 的当前参与关系由 Control 原子维护；每 AgentInstance 在同 Project/Workspace 的 active participation 由账本事务唯一约束。换手保留旧参与历史，后继使用 admission 固定的 Work、参与者、RoleBinding 与权限，不能按 goal/task 再换 Work。

普通 Task/本票后继在 start CAS 中固定 executionAuthorization，当前消费者 fresh-commit execution_entered 后才调用任何 Runtime adapter。start replay 或旧代际拒绝不调用执行端口。恢复只可原子撤销尚未 entered 的授权并给同一 Run/TaskAttempt 退避；已 entered 的 Run 不因空日志、租约到期或进程退出而重跑。

真实模型请求逐个绑定：Runtime 保存实际输入/manifest 后经 Control 写 RuntimeInputBound；每个最终 ModelClient.stream 请求前重核固定材料，签发并一次消费精确许可，成功才进入 provider。取消/暂停/结束、RolePolicy 或 grant 撤销与过期材料阻止新许可/attempt。Run、角色策略、Delivery/Grant 版本与许可消费同事务 CAS。Control 只读引用和 canonical 状态；正文/currentness 由 Host 实际读取，源码不是跨文件原子快照。

取消先提交 canonical desired state，再调用 Runtime。unknown 通过 ReconcileRun 检查可信持久 journal；实际终态证据可对账为 done/cancelled，无证据则显式 quarantine。普通 runFact 仍禁止终态覆盖。通信 intent 的 unknown/过期 sideEffectStarted 通过独立 reconcile 命令隔离；不允许重领以重做外部动作。Runtime 终态不是 provider 可验证 ack，也不是 Task/Goal 完成。

阅读顺序：产品 src/contracts/{coordination,dispatch,runtime-preparation,runtime-dispatch}.ts → control/control-engine/{coordination,run-facts,run-reconciliation,communication-reconciliation}.ts → control/dispatch-engine/{coordination-drive,dispatch-engine,model-call-access,runtime-dispatch}.ts → execution/worker-runtime/observed-model-run.ts。字段的写者/读者/固定/重放/历史缺失语义见 [协议五问](../planning/active/collaboration-memory/CM-1A-001-PROTOCOL-CONSTRAINTS.md#411-调用前绑定与逐请求许可continuation-04-当前口径)。旧 Query/Reviewer/Handoff 路径继续使用各自协议，M/I 迁移未实施；新增可选字段不为旧记录补造授权或无副作用证明。

## CM-M06-001：可替代报告等待与验读准入

当前实现中的加法契约（独立冻结验收尚未完成）：RegisterWait.mode可省略，省略仍为all且旧指纹不变；any仅用于本Work的可替代报告，request_closed、取消/过期及必需Verification义务不构成替代报告。候选来自真实DirectedRequestResponded对应的已投递Delivery，按DeliveryRecorded游标排序；扫描不完整或超过64候选明确unavailable。

Dispatch在唯一drive入口为真实前驱Run准备确定性的精确MaterialAccessGrant，再调用Context验读正文、摘要、当前basis/source。已撤销的精确grant不重签，也不能被另一个宽grant恢复；来源检查不可用（包括Vault无法区分的stale）整轮重试，不冒充较早报告无效。正文缺失的确定观察可以保留在前缀中，继续检查后面的报告。

Host保存本次完成的观察，给出按调用隔离的一次性token；Control只读取这个已完成观察，不反向触发Context/Vault I/O。token不进入稳定命令指纹，所有返回/异常由Dispatch finally清理，重启必须重新准备。模型/命令不能传readable或指定权威赢家。Control沿用当前参与/角色权限复核，Ledger在同一admission事务重算canonical前缀、精确grant与basis，CAS检查Wait/Work/参与/角色政策/前驱/范围及材料版本。

首个合格候选的conditionIndex、DeliveryRef、DeliveryRecorded游标固定在Wait，完整已检查前缀随CommunicationAdmission保存；零可读不创建后继、不满足Wait，既有CommunicationIntent记录重试及原因。后继仍需自己的精确授权和Context/provider边界检查，前驱验读不继承成后继许可。外部文件系统不声称与SQLite原子；合法admission后来源变化阻断执行，不重开已满足Wait。迟到报告留存，其他Work不默认取消。

订阅startCursor是显式有限历史起点，每页最多512事件prefix、整体固定horizon；不是累计512条上限。显式起点必须不晚于创建事务之前已提交的最后事件，两个Ledger在实际append位置拒绝未来cursor；null仍固定为本次SubscriptionCreated位置。已存在的旧future订阅不自动改写，需取消后重新订阅。取消只停止未来路由，不删除已投递材料或代替明确撤权。

CommunicationViewIndex按持久事件重建等待、时间线、积压和失败，暴露sourceCursor/not_ready；“已有回应”“报告已投递”“后继已受理”“实际进入模型”分开表达。主界面消费真实state接口；未知/缺口/不完整历史不得伪装成空邮箱或完成。


## CM-1C-001：架构决定回流与首次参与分配（实施中）

真实普通派发在 Work 身份建立后、DispatchStartRun 前，经显式启用的首次分配入口登记确定性 AgentInstance/WorkParticipation。它只适用于 active 且从未参与、仅 link 发起 Run 的 Work，以及对应 pending 普通 outbox、starting 且尚无信封的 Run。命令 principal、Work/Run/intent 的角色、计划及作用域必须完全一致。initial-participation-start 的最终账本事务重核这些事实，并只允许原 Work 增加 currentParticipationRef/revision；不为已有历史换手，不用于查询、review 或 admission 后继。grant 仍为只读判定，缺授权不会反向创建身份。

普通协作消息及自然语言最终报告不能代替正式架构上报。具备能力的调查发现需新取舍的跨Work接口冲突时，调用report_architecture_conflict并依据回执说明成功或拒绝；不得仅发送消息后声称已正式上报。

report_architecture_conflict 从当前 Run 的正式 principal 进入检查、Brief、Proposal/Candidate 与 ArchitectureReview。修改另建提案；最终接受/拒绝/延后复用 ArchitectureChangeDecision。Review 的固定目标集覆盖当前工作区全部已存在 Work，逐项标记 resume/notify，并用目录指纹和 Work 版本在提交事务校验完整性。决定正文包含精确提案内容及摘要；Vault 引用、字节长度和摘要必须对应同一规范正文。

每个目标的 Delivery 与 intent 完成、需要时的真实 predecessor Wait 同事务提交。架构 Wait 的 admission 必须携带唯一且完整的 canonical Delivery，不能删去或代换；当前材料及调用许可仍复用既有协议。没有真实参与者/前驱或调用失败时明确显示失败/不可用。接受提案不激活基线，不替代 MigrationGate、角色与材料权限。新增 Work 不追溯加入历史目标集。

本节描述当前实现契约；Gate C 仍待冻结回归与独立判断。


C-ACCEPT-01 收口：首次分配还必须匹配宿主持久准备记录中的 exact Run/Task/scope，且 status=prepared、mode 未设；专用 explore/review 不自动建立参与关系。LeasedWorkerRuntime 不为这两种模式请求协调 grant，CodingAgentRuntime.start 也在执行开始前拒绝注入协调上下文。普通只读角色和正式协作后继不以 readOnly 被一概禁止，仍按既有 grant/admission 工作。


## M02 查询执行绑定与恢复增量（实施中）

新 QueryRun 可选携 `execution: QueryExecutionBindingV1`，包含 schemaVersion、roundIndex、原始 readonly 请求和 selectedSources，随 StartQueryJob 在 Job/Run 同一事务提交。Control 受理与 Ledger 提交都核对轮次、精确身份、问题、预算和有界来源；这是材料与重放依据，不授予写权限。历史记录没有该字段时保持原字节，不猜原请求。

ReadOnlyQueryPort.inspectQuery 只查询 exact 请求指纹对应的已持久结果或当前活跃观察；不得执行模型或重新选择原始来源。驱动恢复已保存答案，仍核对身份、来源、权限、大小及当前性。新鲜度用当前 Context 与原始 selectedSources 比较，不能拿新来源替换旧来源。没有可确认结果的查询保留 canonical 状态，工作台显示“需对账”；该观察不是失败终态或重跑授权，也不占据 pending 查询扫描额度。

取消入口先以 CloseQueryJob(cancelled) 记录用户意图，再向 exact QueryRun 发取消信号，重复请求可补发信号；晚到答案不能覆盖 closed。新取消的 QueryRun.outcome 为 cancelled，修正旧实现把取消映射成 answered 的错误；历史已写结果不重写。Runtime 在准备阶段也记住取消，之后不进入模型，关闭 Host 时等待准备/执行退出。查询始终不改变源 Worker 的租约、权限、预算或任务义务。


### 换手与队列迁移补充（2026-09-14）

Host `/api/real/handoff` 接受完整工作区/目标范围、sourceRunId、requestId 和有界 reason。仅 canonical ended 的普通来源可申请，Control 重新核对当前 TaskLease、Plan、角色和权限。相同请求及内容重放相同 ReplacementAttempt；改变内容拒绝。新 Run 不承诺同会话热恢复。

宿主构建的 HandoffPacket Artifact 正文是去除自身 bodyRef 的 canonical JSON，避免摘要自引用；登记 snapshot 保留返回的 bodyRef。ContextBundle 固定新 Run/Attempt、范围、版本、权限、来源和直接依赖，并只选有界 packet 字段。Runtime 实际输入包含这些交接材料及其 manifest 来源，不继承完整对话，不把历史完成声明当验收证据。

## 显式单次响应容量（2026-09-14 用户决定）

用户选择统一普通与只读工作的显式单次响应容量。公开校验以产品 src/contracts/runtime-budget.ts 为准：perResponseTokens 接受用户显式值，普通工作不再额外限8192；2026-09-15核对当前DeepSeek官方文档后上限对齐393216（384K）。通用未识别模型默认4096；已知DeepSeek型号的生产Host默认见下文。上下文容量和累计输入/输出、请求、工具、时长是不同维度，未配置累计预算仍为null，不随单次容量新增默认累计限制。实际Provider不接受某容量时显式失败，不能静默截小或声称模型效果通过。旧配置读取与旧调用参数兼容，UI使用同一容量常量。

IG07（2026-09-14）：Query 的架构决定材料保留最小 conflict={reportSummary,impact,briefRef,planRef,baselinePin,matchesCurrentPlan,authority}，用于回答原始冲突理由及正式列出的影响集。来源复用已公开 ArchitectureReviewView 的原始上报摘要与 DecisionBrief.impact，不加载所有 brief 选项或报告全文。Context 核对同 Project/Workspace、brief/proposal 引用及来源计划关系；其他 Goal 不选入，跨scope/伪造来源返回材料缺口。旧 Plan 的已记录决定保留为历史说明并标 matchesCurrentPlan=false，不一律拒绝，也不得当作当前接受事实。原始上报理由和声明影响集不证明实现、迁移或验证完成；新增材料参与 dynamicFactSet，理由、影响集和当前适用性改变会使旧解释失效。
IG08（2026-09-14）：Query 的 goalPhase 独立读取同 Goal 的 StateLedger GoalPhase，携原 ref/revision/planRef/phase/reasonCodes/reducedAt 与 matchesCurrentPlan/authority；不从 Task 合取重新归约。无记录为 null，旧 Plan 记录只可作历史解释，跨 Goal/Project 或伪造引用返回材料缺口。来源摘要与 dynamicFactVersions/dynamicFactSet 同时覆盖该记录及其缺失；GoalPhase 单独变更也使旧 Query 失效。只读 Query 禁止自行改变或推导正式完成状态，但可准确引用计划相符且来源当前的已持久化 COMPLETED；这条输入契约不证明真实模型的解释质量。
IG08 来源边界补充：goalPhase 表示 reducedAt 时最后已记录的正式状态；matchesCurrentPlan 仅比较计划身份。Query 的原生源码 freshness 只证明本次查询观察未变化，不将旧验收与当前源码建立对应关系。材料没有提供验收到当前源码的适用性见证时，可以引用“账面记录 COMPLETED”，但不得声称新源码或外部改动已经验收；这不触发新的归约或扫描全部 Evidence。

IG12 回应组织（2026-09-14）：semantic_query 的 Context输入和持久Runtime观察记录可选responseGuide（当前semantic-query-response-v4，v1/v2/v3原文保留）。该版本只选择产品内固定的只读回答组织规则，最终ModelRequest在容量计量/授权摘要前包含同一静态系统指令；每个工具后继请求均适用。原始偏好、来源正文和用户问题仍在数据输入，不可插值为系统指令，也不增加工具、写权限或验收权威。规则区分“核对全部事实/前置”与“向用户逐项列举”，简洁偏好允许归组表达，但不能省略改变结论的失败、未知或门禁前置。v2为简洁输出给出三句/80词或160汉字的呈现约束，当前请求或适用偏好要求详细时不适用；这不是Runtime累计预算或截断机制。已接受工作等待执行/验证不产生额外审批；缺少决定记录也不等于已有待决请求。v3进一步要求保留责任主体：项目习惯不把调查/实现/验证转交用户；仅真实产品取舍或授权工作无法取得的信息才请求用户。v4要求输出前按明确集合核对数量和全部/部分等量词，区分已满足/未知/失败/过期及尝试采用/仅通知；同时明确latestRun只选TaskLease普通执行，GoalGate无普通Worker Run不等于未进行独立检查。结构化规划/执行协调不采用该普通回答规则，旧观察无该字段仍可读取和重放原结果。实际措辞/密度须用真实模型按原量表判断，规则或版本字段存在不等于质量通过。

2026-09-15 结构化协调接线：Context 对 initial_coordination、execution_coordination 选择并留存 coordination-json-response-v1；WorkerRuntime 按该固定版本为每轮模型请求附加 JSON object 格式和静态格式指令，进入既有容量计量后再提交。偏好只能影响契约字段内的表达，不能替代响应结构。普通 semantic_query 仍是自然语言回应；旧记录无此字段时只重放其既有结果，不重新调用或追溯改写。JSON 模式不替代 PlanCompiler/反馈消费者的严格解析、来源/权限/版本和正式受理；带前置散文、围栏或截断的结果仍拒绝，不能截取一个子串冒充有效提案。

初始规划响应契约须说明现有 static、dynamic、reviewer 三类要求。要求独立语义审阅时，在相应工作及目标门禁映射明确 reviewer 义务；“独立验收”的标题不产生审阅权限。可执行检查与语义审阅分别表达，不能把需要语义判断的正确性写成只有工具才能满足的 static 要求，再借此阻断 Reviewer。实际 Reviewer 仍由既有 Verification/Dispatch 工作流创建，规划不增加实现 assignment 或绕过既有受理。历史已接纳计划不追溯改写。

2026-09-15 新增项目查询取消：Host 构造 QueryJobRef 只取 projectId、workspaceId、queryJobId；项目挂载的 root/name 不进入规范引用。先验证目标归属，再持久关闭该 QueryJob，之后取消 exact QueryRun；重复取消可重放，重启不重做模型调用。挂载元数据仍保留原请求兼容语义，不通过全局改变scope破坏旧身份。

IG13 引用位置便利索引（2026-09-14）：read_material只有在offset=0、complete=true、nextOffset/totalBytes与实际UTF-8原文长度一致时，才可附加citationLocations及citationLocationsTruncated。位置相对原始Artifact根，RFC6901转义；完整非JSON仅根空指针，部分页不生成完整索引。按层广度优先，至多256项、8层、8192索引字节、单路径2048字符并保留现有工具输出预算；不为索引截去原文。索引不是阅读或正确性证明，也不授予更多材料读取权。Reviewer仍须读原文并选择相关依据，最终Assessment从持久原文核对digest与路径；无效旧报告不修写为有效、不因索引新增自动重抽。

IG14 协作事实佐证（2026-09-18修订）：只读检查报告可包含collaborationFacts，Context仅从生产者精确输入绑定、不可变ArchitectureReviewRecorded(open)事件及当前canonical Delivery/Review/Work记录派生；不复制私有模型上下文，也不扫描未受理邮箱。inputDeliveries完整列出该Run输入绑定的有界Delivery集；decisions只列其中实际投递给该Run的架构决定，并复核精确授予、目标、版本、source pin与输入身份；reportedReviews列该Run自己正式上报时的Review状态和后来current状态。后来接受或拒绝reported Review不等于该Run收到决定；空decisions只能在完整inputDeliveries范围内证明没有输入决定。该事实随原检查Artifact受Reviewer精确授权及引用检查；Reviewer currentness重新核对原来源、授予与事实一致性，失效不得继续作为当前佐证。它可证明精确输入Delivery、已投递决定、上报Review及Work→Task关系，不证明源码消费者、基线激活或语义验收。历史无该字段保持未知而非补造。新增说明以ReviewerPacket.guidanceVersion=collaboration-corroboration-v1选择，未标版本的原输入格式保持；旧正式FAIL不得因补材重新抽取同一Review。

初始规划的 taskHierarchy.parentOf 明确使用 parentTaskId/childTaskId，二者只引用计划内 Task；Stage 归属由 task.scope 表达，不使用层级边代替。PlanCompiler 在提交前复用公共 ApplyPlanRevision schema 校验，记录可定位字段的结构错误；Control 在实际变更边界仍独立执行同一结构校验及当前来源/图/权限受理。不得通过字段别名猜测、删边或自动改图来接受模型错误计划。

2026-09-15 Q01 复验增量：新 semantic_query 使用 semantic-query-response-v5；V1–V4 静态原文及持久版本映射保留（V4 SHA256 48602041d082217a6a3f0d44607bbe5191b03f31aad6d2bf58ca0b251b20d3a6）。前置分类只帮助模型区分“尚未全部满足”与“全部未满足”，不能制造完成、当前源码就绪或人的额外审批。真实质量需新快照独立验收，指导存在或替身通过不等于质量 PASS。

2026-09-15 真实17回答事实边界增量：新semantic_query使用semantic-query-response-v6，V1–V5原文与持久映射保留（V5 SHA256 6f0118e1788686f5fb7008cbcf34994079649b3b8eb5e506df306deea1b773ee）。指导要求逐句区分读取不可用与已观察缺失、接受多候选提案与明确选定候选、当前计划前置与未来变更条件；未知current-source适用性不自动新增实施或重验义务，已完成时不为凑下一步句式编造待办。真实17机械闭环通过但Q03–Q06回答质量失败保留；V6指导及定向契约验证不能代替新快照真实质量验收。

## 2026-09-15 Query 事实引用候选（I 阶段，尚未独立接纳）

用户选择先验证事实工具、引用、简短 Skill 与适用记忆，不将普通 Query 默认改为双模型阶段。语义 Query 使用版本化 V7 表达指导；V1–V6 原字节及已持久回应保留，协调 JSON 协议不改变。DeepSeek 配置可显式保存 low/high/max；省略字段的旧设置保留服务商默认，旧入口保存时保留已有显式值，null 恢复默认。当前用户配置为 max；配置版本与实际运行绑定，配置与生成强度不同于单次输出容量。

QueryExecutionMaterialPort 增加可选 readFact(request,{inputDigest,pointer})。ContextCompiler 复用现有 running claim、精确 intent、Vault 和当前适用记忆选材复核，仅返回 /material 或 /maintainedPreferences 内的 RFC6901 位置；model 不能另传项目、Run、owner 或 grant。ready 包含原值、inputDigest、sourceBundle 与 captured_query_input 观察性质；原值的 unavailable 不归约为无记录。not_found 仅表示该位置不在完整捕获输入中，不代表业务对象不存在；超 16KiB 返回 too_large、不裁剪。输入变化返回 stale，授权/读取失效返回 unavailable。该工具不新增 Ledger 权威，也不承诺读取后的实时状态；最终既有版本/来源 freshness 判断继续执行。

WorkerRuntime 仅对显式 factReadVersion=1 的 semantic_query 装配 read_query_fact，只读且身份由 Host 绑定；正常模型调用/工具结果/用量沿原内核循环。工具建立短 F 引用标记，最终回答引用未知标记或被引用记录在再次读取时失效，结果为 gap；取消和源码前后检查仍保留。每次最多登记24个事实引用，既有合并来源清单64项上限保留，这是结果容量而非新增累计模型预算。不存在算法确认任意自然语言语义正确的承诺。

引用沿既有 Query answer.sources、Vault bodyRef 与持久运行输入保存。UI 展开时使用当前项目/Workspace/Goal查询，按精确 QueryRun 和 input SHA256核对引用原值，展示捕获记录及版本、历史标记；不从模型文案推断进度。引用位置、权限及版本检查只能证明取材/定位边界；对象、时间、范围、确定程度与责任归属是否支持陈述，继续由真实样例与独立审阅评估。

偏好仍是 preference_only：只按明确来源、版本、目的和项目/用户范围选用。主动偏好不能创造授权/任务；简洁不能将未知写为不存在；候选/推测画像及被排除内容不作为当前规则。现有查看、纠正、删除和跨项目传播/撤权机制不改变，不新增自动画像或自动 Skill 演化。

补充：发布前重读所有实际成功读取过的事实，不依赖模型是否正确写出引用；输出来源只列最终答案引用的条目。漏标记不能绕过撤权，错误组合标记不冒充已核实引用。该边界有故障反例和最终复核时取消 barrier 定向证据，见产品 fact-skill-max。


### 2026-09-15 DeepSeek 默认容量与重复受理

生产 Host 在新请求受理时读取当前模型设置：deepseek-flash及官方兼容V4别名，显式max采用单次131072 Token，其余当前支持的思考配置采用65536 Token，声明上下文1000000。依据官方 https://api-docs.deepseek.com/api/create-chat-completion/ 与模型页，属于公开容量声明而非实测provider窗口。用户显式budget字段优先；累计限制仍为null。只读Query、初始规划、普通执行和探索共用配置解析；Reviewer及后继继承其正式工作预算。

重复请求补默认时以原已受理/已准备预算为基础，不能因模型设置改动改写旧预算；显式变更仍交原身份与冲突守卫拒绝。未知型号或其他provider保持通用默认。短连接探测256仍只是有意的小探测。模型设置不更改用户记忆，不新增累计限制。

Context独立审计确认既有24样本33次ModelRequest均含完整原始input；不据此宣称API内部无变更或全部业务事实完备。当前普通Run观察不覆盖独立验证执行全集，架构reader不覆盖所有渠道的人类请求，历史完成不包含当前源码适用性见证。read_query_fact返回的是已授权捕获事实，不能补造上述未观察内容；缺口和旧回答语义失败保留在产品context-output-18证据。

2026-09-15 CONTEXT-019 修正：配置了架构读取器且存在明确 Goal 时，Context 必须读取该范围的记录，不以当前 accepted Plan 的存在为前提；没有当前 Plan 的历史记录 matchesCurrentPlan=false。成功的范围内空结果为 ready/rows=[]，无 Goal 不推断项目级不存在；缺少读取器、读取失败、跨范围结果继续分别处理。该集合参与 freshness 重算。real17 的 unavailable 原因曾错误声称读取器未配置，实际为选择器跳读；旧语义错误 unavailable→无记录仍保留。

Workbench与可达legacy开发任务入口：未填写容量时不发送容量覆盖值，由Host在新请求受理时选择已配置模型默认；累计限额未填写仍为null。明确填写的预算继续生效；清空恢复上述未配置语义。Workbench旧pending v1的身份和预算顺序保留，默认改变不能静默重用或替换未决请求ID。该浏览器接线不代表真实模型语义质量已通过。


### 取消意图与正式运行终态归约（2026-09-17）

取消请求先持久记录 desiredState，cancel_requested 只表示信号已发送。Control 的 ReconcileControlIntent 只消费同scope、同Run的正式事实：cancelled归约applied，其他确定终态归约rejected，outcome_unknown保持明确未知；被较新意图替代时记录superseded。记录ControlIntentReconciled及精确Run revision、outcome、时间，不能伪造SafePointAcknowledged或增加ackCount。

Control先检查系统身份、对象和版本，Ledger提交时再次按当前Run/Intent验证归约并对两者CAS。无变化不追加事件。普通drive完成、取消入口以及新进程恢复均可触发归约；恢复扫描已有ControlIntentRecorded，覆盖旧数据库中的已结束Run/queued意图，不重放Runtime。扫描不完整明确失败，不宣称无待处理事项。

这不增加pause/steer能力，不发明未持久的deadline或自动timed_out策略。recordSafePointAck旧revision宽松行为为已确认的延期缺口（B5，无已发现生产消费者），不将本次取消归约验证误写为全部安全点协议通过。
