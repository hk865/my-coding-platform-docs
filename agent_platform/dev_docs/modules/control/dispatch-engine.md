# DispatchEngine Module

> 当前代码与接线评价见 [模块审计](../../../human/module-status.md) 的对应条目。本页保留职责、Interface 要求及明确标注的首切片约定；旧切片状态不能代替当前实现结论。


```yaml
status: draft
updated: 2026-09-11
plane: Control
```

## Purpose

把持久派发意图落实成有权限、有预算、可恢复的 Worker Run，并回交运行事实。

## Interface

候选操作：`drive(trigger) / accept(event) / snapshot(query)`。共享字段、拒绝与状态语义唯一来源为 [运行时协作契约](../../interfaces/runtime-collaboration.md)，本页不复制 wire schema。首个消费者：P1-03；06/07/09/10 分别扩展换手、写 lease、查询和控制；设计已展开不代表契约已冻结或实现。

## Dependencies

ControlEngine、ContextCompiler、WorkerRuntime、ArtifactVault、StateLedger、PlanCompiler。长期调用关系以 [ModuleDependencyDAG](../../../ARCHITECTURE.md#moduledependencydag) 为准；运行时事件反馈不是反向源码依赖。

## Hidden Implementation

领取与启动间的恢复、运行事件关联、正文保存、公开快照适配。启动身份可重放，未知副作用留待对账。

## Test seam

通过上述 Interface 注入依赖 Adapter，验证：竞争领取、启动前崩溃、重复／迟到事件、过期绑定、不可用快照和安全退出。 不以内部表或私有方法作为唯一测试入口。

## Context load

实现或扩展本 Module 时读取本页、当前 Ticket 与直接消费的 [运行时协作契约](../../interfaces/runtime-collaboration.md) 小节；初始协商及图文集成另读 [初始设计与统一展示](../../interfaces/human-design-status.md)。原始对话和完整历史按需追溯，不默认装入 Run。

## Extension records

- **P1-03（首个消费者，冻结）**：DispatchPort = `drive(trigger)`（唯一入口；outbox-before-side-effect：加载 pending → assemble → startRun 提交 → 之后才调用 runtime.start）；实现见 src/control/dispatch-engine/dispatch-engine.ts。
- **P1-06**：换手面 driveHandoff（src/control/dispatch-engine/handoff/handoff-drive.ts）；normal drive 在扫描前跳过带 ReplacementAttempt 的意图（scanned 语义）。
- **P1-07**：WorkspaceDrivePort.driveParallel（版本化新增，不改 P1-03 DispatchPort；切片内全部 intent 并发 assemble→startRun→runtime.start，再并发 consume；run-scoped 幂等键）；公开快照查询面见 [WorkerRuntime](../execution/worker-runtime.md)（HandoffControlPort.snapshot，P1-06 冻结）。

## Context 生命周期与协作扩展

初始计划的已接受 assignments 由 `planned-task-dispatch.ts` 处理；调用 canonical dispatchReadiness 和 claimTask 后沿既有 outbox 驱动执行，不用 UI 矩阵充当派发守卫。`exploration-context-drive.ts` 消费 Context 选择，提交精确 sourcePin 授权并发现报告分块，再编译实际输入。`leased-worker-runtime.ts` 集中管理真实运行的租约与启动前材料失败；可证明尚无模型/工具调用时回交 known failure，无法证明时保持待对账，不隐式重跑。既有 DispatchPort.drive 的顺序和签名保持。

P1-16 落实继续／转交意图及能力降级；P1-15 路由议题和决定，确认受影响工作材料刷新后接续，旧 Run 不能覆盖新状态。 行为依据：[Context 生命周期](../../interfaces/context-lifecycle.md)、[运行时协作](../../interfaces/runtime-collaboration.md)、[人类交互](../../interfaces/human-design-status.md)。精确 schema 在对应消费者冻结，文档同步不表示已有实现。
## 当前源码边界（2026-09-11）

返工驱动接收组合根提供的 `issueMaterials`，不回调 VerificationEngine；缺少问题材料明确返回 unavailable。每个任务组受理后经 Control 的 ReworkDispositionPort 重新解释当前承担者与正式 Evidence，不能把来源 Plan 失效等同于义务已处置。问题事实与原报告保留，结果逐项说明已接手、仍待处理或明确阻塞。工作影响材料同样复用 Control 的工作身份解析。该路径保持既有 ModuleDependencyDAG，不新增 Dispatch→Verification 依赖。

`dispatch-engine.ts` 保留持久outbox协议；RuntimeDispatch隐藏恢复/事件对账，OperatorTaskDispatch隐藏人工运行准备/claim/cancel，PlannedTaskDispatch推进已接受assignments。canonical读取允许用于完整scope和恢复校验，所有正式状态仍通过Control命令；已知/未知副作用必须区别处理，不自动重跑未知执行。

跨 Module 的精确入口与失败/持久兼容规则见 [当前 Module 边界](../../interfaces/module-boundaries.md)。此源码映射不代替整体功能验收。

VR-02 的 `reviewer-dispatch.ts` 通过 `drive/recover` 驱动 canonical 独立审阅工作：真实授权与 Context 编包、原 start/fact、原只读内核、公开持久最终回答及原正文绑定；复用 `dispatch-engine.ts` 的执行事实消费，普通派发不会领取 ReviewWork。`leased-worker-runtime.ts` 使用 Context 的精确 Reviewer 材料，租约仍在本模块管理；未知运行不默重跑。接口与恢复边界见[独立审阅](../../interfaces/independent-review.md)，本批已按[VR-02验收](../../verification/2026-09-09-independent-review/acceptance.md)确认限定能力。

CM-M06-001：alternative-report-preparation.ts在admission前为真实前驱准备精确验读授权，调用Context形成有界完整观察前缀并登记Host一次性能力；coordination-drive.ts负责调用级finally清理。Control不经Dispatch回调Context。any零可读或暂时不可用留既有intent重试原因，不占后继；已合法admission后的模型输入仍走既有材料通道。 精确语义见[运行时协作](../../interfaces/runtime-collaboration.md)，M06 snap-01 已独立通过，范围见当前模块状态。


## CM-1C-001 已接受增量

真实普通首次派发建立确定性参与关系；已有历史不自动换手。消费固定架构决定 intent，逐 Work 投递并驱动既有 Wait/admission，后继以固定 Work/材料进入 Runtime。Host 报告工具只授予 canonical 当前 principal。

涉及本模块文件：`initial-work-assignment.ts`、`coordination-drive.ts`、`coordination-tool-access.ts`。共享值与纯校验位于 Contracts 的 architecture-review.ts / architecture-review-values.ts / initial-work-assignment.ts；组合根 service、composition 和 UI 负责接线，不承接模块权威。Gate C 已针对 CM1C-001-snap-02 独立接受。

C-ACCEPT-01：专用 explore/review 模式不获得协调写入口；普通只读 Work 的协调能力保持独立授予。首次分配检查实际 prepared RunSpec，Runtime 入口再次拒绝非普通模式注入，不能仅靠工具列表隐藏。

## M01 入口收敛

公开 WorkspaceDrivePort.driveParallel 保留作用域接口，WorkspaceDriveEngineImpl 内部适配同一 DispatchEngineImpl.driveOrdinary；普通入口先推进通信，再调用该内部 seam。两种入口共享并发额度、Work/材料准备、start/entry 许可及事实消费。scope 只选择相应 Project/Goal 的普通 outbox，不从旧并行入口推进无关通信。材料缺口沿普通持久退避，不再保持旧入口立即重复尝试。Host 与计划入口已迁移同一有界并行路径，已完成只读并行、写冲突排队和积压视图定向验证；M 系列已在 CMM01-M05-snap-03 独立通过，精确边界见当前模块状态。


## M 阶段已接受迁移

M01：ordinary/planned/operator 共用 outbox owner；计划扫描一次领取所有独立合格 work，保留 canonical readiness。实例内只读重叠、写冲突排队，待写工作前不插入新 reader，其他 Workspace 可前进。旧 WorkspaceDrive 是 scope adapter；RuntimeDispatch 仅 exact Run 对账串行，不串行完整执行。backlog 从持久 pending 计算，Host 的 deadline timer 只触发扫描。

M02：查询将轮次和原请求绑定在 StartQueryJob 事务中；恢复只读取匹配的已持久结果，正常与恢复共用答案校验/提交。未知查询保留待对账，不能从本进程缺失推断终态；需对账项不阻挡 pending 扫描。契约见 runtime-collaboration 的 M02 增量。


## M 系列执行与恢复约束（2026-09-14）

生产装配位于产品 `src/composition`。Dispatch 内部 `execution/` 共享本地容量、执行代际、模型许可和持久退避；`handoff/` 独立维护换手资格和来源。普通、Reviewer 与 Replacement 共享一个本地容量实例，跨进程最终排他仍由 Control/Ledger 和 Workspace lease 确认。

宿主 `DurableWake` 只合并同一作用域的扫描唤醒；扫描期间的新唤醒必须再执行一次。初始规划由持久 Query/Plan 发现，返工消费公开验证事实并产生正式 PlanRevision，由 PlannedTaskDispatch 统一派发。启动恢复扫描已登记作用域的 canonical Goal，不要求先存在运行观察。30 秒周期扫描只补足丢失唤醒，关闭时清理定时器并排空活动工作。

普通取消先提交 desiredState，再调用 Runtime；缺少取消能力返回 unsupported；unknown 保持 unknown。取消身份绑定完整 RunRef，跨 Goal 相同 runId 不共用幂等键。历史库不凭缺少新输入绑定推断成功，也不自动重放未知外部动作。

终态续接由 Host 的 TerminalContinuation 扫描公开持久观察与 canonical Run 终态，所有 DispatchWake、直接启动和宿主启动均触发同一职责。已知结束才进入反馈/重验，未知结果保持待对账；每轮重放依赖原反馈和验证的持久幂等身份，不以本地集合记完成。


Host关闭先停止HTTP受理，排空已进入的完整请求/响应和Runtime，再关服务资源与残留预连接；重复close共用一个Promise。这里不将未完成上传的请求伪装成已完成操作。

