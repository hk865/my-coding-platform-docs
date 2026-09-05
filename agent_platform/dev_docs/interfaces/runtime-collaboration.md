# 运行时协作 Interface：角色、Context、通信与查询

```yaml
status: draft
updated: 2026-09-05
scope: 已认可职责方向的扩展契约；非已实现或冻结的 wire schema
```

依据：[ARCHITECTURE](../../ARCHITECTURE.md)。本文件定义跨 Module 的行为与必要信息；现有 [Command/Event](command-event.md)、[StateLedger](state-ledger.md)、[GoalView](goal-view.md) 仍保留 CreateGoal 首切片类型。实现扩展时显式升级对应 schema，未知版本继续拒绝，不能以宽泛 payload 绕过验证。

完整初始设计与图文组合的消费契约见 [初始设计与统一展示](human-design-status.md)，P1-15 负责整合；不再留给无归属的未来工作。

## 共享引用与角色绑定

所有请求携带调用身份、Project／Workspace scope、correlation、schema version；改变状态的请求另带幂等键和预期版本。来源引用携带来源种类、对象或路径、revision／digest、适用范围；时间戳不能替代版本。

角色模板允许自定义职责名称、步骤与输出，实例化要求见 [自由模板](../agent/templates/short-lived-agent.md)。模板本身不授予权限。

RoleBinding 至少关联模板版本、Agent／Task、QueryJob 或 CoordinationJob、职责、授权策略版本、工具／读写范围、预算和终止条件；Run 引用绑定版本。ControlEngine 接受与撤销绑定，StateLedger 保存事实，ContextCompiler 消费绑定，DispatchEngine 在启动与安全检查点核对有效性。模板变更不追溯改写已有绑定；失效绑定不得启动新 Run，运行中变更按安全点控制处理。

## 各 Module 的扩展 Interface

| Module | 候选操作与可观察结果 | 调用依赖／关键约束 |
| --- | --- | --- |
| HumanCollaboration | `query(request)` → fact / report / pending / unavailable / rejected；协调请求交 PlanCompiler | 事实读 ReadModelIndex；公开快照读 DispatchEngine；新 QueryJob 交 ControlEngine。是否需要解释由请求指定，不强制先过参谋 |
| PlanCompiler | `request(intent)` → accepted(jobRef) / clarification / rejected；`accept(resultRef)` → proposalRef / explanationRef / unresolved / rejected | ContextCompiler 提供材料、ArtifactVault 提供结果；通过 ControlEngine 持久化协调工作及正式提案。原 `compile` 名称不再代表完整协调接口 |
| ControlEngine | `submit(command)` 保留；扩展角色绑定、工作请求、报告登记、消息路由与提案受理的已知 command 类型 | 从 StateLedger 读取 canonical state 做授权、CAS 与幂等校验；原子提交状态、事件和派发 intent。不依赖 PlanCompiler 或 WorkerRuntime，不在 reducer 内调用模型 |
| DispatchEngine | `drive(trigger)` / `accept(runtimeEvent)`；`snapshot(query)` → report / unsupported / stale / rejected | 消费 ControlEngine 的 durable intent，ContextCompiler 编译，ArtifactVault 存取正文，WorkerRuntime 执行或提供快照；所有角色复用该路径 |
| ContextCompiler | `assemble(request)` → ready(bundleRef, manifest) / needs_material(gaps, selectedRefs) / rejected(reason) | 消费 StateLedger、ReadModelIndex、ArtifactVault、WorkspaceReader；不分配角色或启动模型。ready 前保存有界 Bundle；缺口交调用方经 Control 路径补充，不在内部无限重试 |
| WorkerRuntime | 原 capabilities/start/control/events；新增可选 `snapshot(query)` | capabilities 显式声明快照支持；无此能力返回 unsupported。只暴露公开报告，不询问源 Run 的隐藏上下文 |
| WorkspaceReader | `read(query)` → sourced(result, provenance) / unsupported / stale / rejected | 读取显式 Workspace 的源码、Git 差异及可用代码／测试索引；检查路径、读授权与版本。不创建 Task，不写 checkout |
| ArtifactVault | `put(record)` → immutableRef；`open(ref, accessScope)` → record / unavailable / rejected | 保存报告、交接、Context、语义提示与来源。校验来源元数据不意味着验证正文结论；不依赖 ControlEngine |
| ReadModelIndex | 在现有 goal 查询外增加角色／Run／报告引用查询 | 从已提交事件重建，携带 scope、cursor、来源；不把报告文字投影为正式完成状态 |
| StateLedger | load/commit/events 形状保留，增加角色／工作／通信元数据对应的版本化对象与事件类型 | CAS、原子提交和有序事件语义不变；不存重复源码库，不做语义裁决 |

这是目标扩展面；各操作的具体 wire 字段、枚举和兼容策略在对应首个消费者冻结。实现者不得仅凭此表跳过其纵向契约测试。

## Context 与工程来源

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

## 冻结时必须覆盖的测试

- 自定义临时角色可运行与交接；过期绑定、越权工具／路径、写 lease 冲突被拒绝；取消和预算耗尽可安全结束。
- Context 版本错配、必要材料缺失与预算不足显式失败；语义补充重试有限且不自行扩大权限。
- 工作树变更及陈旧 CodeGraph 可检测；无图能力明确降级或返回缺口。
- 普通记录无需逐条人批；报告不能完成任务；权限／规范变更仍经正式受理。
- 消息／结果重复、迟到、提交冲突与正文已保存但提交失败均不重复派发或改变未经接受的状态。
- 直接查询无模型调用；可选快照 unsupported／stale 如实返回；独立 QueryJob 不干扰源 Run。

本文件对应 P1-02/03/04/06/08/09/12 等消费者，仍受 P0-06 与 MVP 范围复核约束。候选契约同步不等于授权开工。
