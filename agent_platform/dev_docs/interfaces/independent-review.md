# 独立 Reviewer Interface

当前接口包含下方DEF-17恢复增量；其最终证据见独立Ticket。历史验收（VR-02，2026-09-09）：版本化接口已接入源码；全仓249文件/1633项、完整浏览器23项及类型/构建/边界验证通过，A01–A04和最终源码/日志身份已独立确认；已按[本票验收](../verification/2026-09-09-independent-review/acceptance.md)完成限定范围。用户要求本次完成架构和Reviewer后交审，其余功能暂不动工。入口[VR-02](../planning/active/core-verification/VR-02.md)，既有[完成策略](completion-policy.md)、[Module边界](module-boundaries.md)与各对应Module继续适用。

Verification组织一次有明确subject Task/producer Run和已持久工具轮次的独立审阅。全部适用工具PASS且工具义务覆盖完整、仅缺Reviewer时可创建；不能忽略额外无coverage工具FAIL，不能从round总体PASS判断（待审轮次总体为INCONCLUSIVE）。原Task的Plan/TaskLease不改，创建独立ReviewWork/Attempt/Run/Outbox；Reviewer必须独立session和真实readonly工具权限。

Control只依赖StateLedger。受限ReviewLifecycleControlPort仅DI给Verification，受限ReviewDispatchControlPort仅DI给Dispatch，普通HTTP/Control提交不接任意descriptor/verdict/可信标记。Verification校验完整工具材料与原模型报告，Dispatch确认真实公开运行观察与原回答，Control重核canonical身份/权限/协议/版本/CAS并独占原子归约。这是受信模块的组合边界，不以actor字符串冒充认证，不宣称防同进程恶意代码。依赖仍保持12 Module/34边，不反向给Control注入Verification/Context/Vault回调。

正式记录采用TaskReviewProtocol、ReviewWork、ReviewResult；ReviewWork的input、output和resultRef单向固定，原报告位于output.reportRef。首次采用协议与独立工作原子建立；旧未采用Plan保留旧有限行为，旧Plan/Evidence不改写。新普通计划显式协议，采用后拒绝generic verdict或observation覆盖Reviewer，当前资格核对正式Result/Work/Run；未处置的旧FAIL/INCONCLUSIVE不因协议转换或新PASS消失。

Context从canonical Work/intent编制首次有界包，不要求Reviewer已有envelope。原工具材料按精确grant授权给Reviewer，真实current读取核对撤销/来源/版本/摘要/owner；完整索引不能静默漏项，可通过精确有界按需读取取得原文。新包明确类型/版本和容量，旧ReviewPacket不变。SourceSet显式区分Candidate与探索全目录语义，当前HEAD比较不等于Run前态。currentness只豁免受理的精确readonly Reviewer自身prepared/running，其他Writer/unknown仍拒。

ReviewerProfile.current/resolve提供当前Task的真实公开模型配置身份、既选RuntimeBudget和明确readonly角色，ref摘要固定完整快照；密钥不进入profile，累计预算未配置仍null。Runtime在prepare/start核对精确模型revision，不能默切新模型；通过原coding-agent公开内核和observed-model-run执行，不另建模型循环。工具可读范围必须落在source pin内。

Dispatch遵守持久意图→授权/编包→Control start→Runtime→runFact→原正文body-first→canonical output绑定。Verification仅消费该原绑定报告，验证新报告版本、逐Reviewer VR的完整唯一覆盖、结论/定位/依据和当前材料；completed/exit0不构成PASS。合法FAIL/INCONCLUSIVE与格式坏报告分开，逐VR结果按outcome分组最多三条Evidence、Result/Work/Index一次接纳。Task/Goal随后由既有归约入口决定，Task证据不能复制给Gate。

请求同键同载荷重放、异载荷冲突；读状态/回执不启动执行，resume只推进既有授权工作和原结果。未知先查持久事实、不重复模型；确定CAS冲突使用固定新归约请求，未知响应保持原身份。source在使用前后变化/撤销/坏正文阻止新接纳；文件系统与Ledger并非原子快照，已接纳Evidence的一般自动来源失效与正式来源推进留后续工作，不宣称本票已实现。

Reviewer FAIL或未处置旧FAIL仍阻断，不用新请求重抽或普通replacement清除失败。授权返工、新来源完整重验、一般工作Context/记忆/接续/架构迁移等不在本票实施；保留为后继义务，待用户审查架构与Reviewer后再决定动工。

## 已落源码的公开接口

以下路径相对产品根。字段唯一正文为对应 `src/contracts`，本页只记录消费者和权威，避免再维护一份复制的 wire schema。

| 提供者 | 公开入口与契约 | 调用与持久边界 |
| --- | --- | --- |
| VerificationEngine | `VerificationService.reviewMaterial/startReview/recoverReview/review/reviewReceipt/resumeReview`；`reviewer-verification.ts`、`verification-service.ts` | 唯一请求 journal 在 `verification/reviewer-record.ts` 与 `reviewer-verification.ts`；只请求正式工作/接纳/归约，读状态和回执不运行模型。已有 `forRun` 返回 `reviews` 用于重开 |
| ControlEngine | `ReviewLifecycleControlPort.createWork/replaceFailedWork/recordValidatedResult`、`ReviewDispatchControlPort.bindOutput`；`reviewer-work.ts` | 宿主分别注入 Verification 与 Dispatch；正式 Work/协议/独立 Run/Attempt/outbox 和 Result/Evidence 原子写入 Ledger；不提供普通 HTTP 提交这些可信材料的入口 |
| DispatchEngine | `ReviewerDispatch.drive/recover`；`reviewer-work.ts` | `control/reviewer-dispatch.ts` 复用既有 `consumeDispatchedRun`，准备/授权/编包/执行/确认公开最终回答/原正文绑定均属 Dispatch；宿主仅启动与排空后台工作 |
| ContextCompiler | `ReviewerProfilePort.current/resolve`、`ReviewerContextPort.inspect/current/recovery/select/assemble/packet/runtime/readMaterial/readSource/openReport/openHistoricalReport`；`reviewer-context.ts` | 保存的公开模型 metadata 与原 Task 持久预算形成固定 profile；canonical 材料、全部授权、公开持久运行观察和来源前后复核归 Context，不回调活 Runtime |
| WorkspaceReader | `VerificationSourceApplicability`、`readReviewerSource`、`reviewerSourcePathAllowed` | 新 `verification_workspace` pin 与 Candidate 来源覆盖一致；探索原 `workspace_paths` 语义不变。路径任意层的忽略目录、越界和链接拒绝，原生源码索引按同一可见性读取 |
| StateLedger | `pendingDispatchIntents(limit, selection?)`；`ledger.ts` | 可选workKind在限流前筛选，普通Dispatch请求ordinary，避免pending Reviewer挡住后置普通任务；省略selection保持原兼容查询。事务版本校验包含首次观察到的协议不存在revision 0，不能让采用协议前的旧归约迟到覆盖 |
| WorkerRuntime | `RunSpec.mode='review'` 与 `review.workRef/profile`；`RuntimeContextAccess.reviewer` | 原 coding-agent 内核/观察/预算复用。仅实际 `read_source`、`read_material` 与受限分析工具；每次模型/工具前后验证当前材料，模型配置变更不静默切换 |
| HumanCollaboration / Host / UI | `/api/real/verifications/reviews/{profile,material,start,recover,read,resume,report}`；`/api/receipts` 的 `independent-review` | 显式选真实轮次与配置，保存请求身份，显示逐要求结论/问题/正式回执；网络丢响应先查同一回执。历史授权撤销按正式 scope 字段构造引用，不携带挂载 metadata |

旧 `ReviewPacket` 保留；新 `independent-review-packet` v1 上限64 KiB，正文按需每页至多32 KiB。原始报告是 `independent-review-result` v1：完整唯一的 Reviewer 义务/要求、PASS/FAIL/INCONCLUSIVE、非空理由、精确材料引用与问题/未知；源引用绑定完整文件摘要和行范围，artifact引用绑定包内材料及实际 JSON Pointer。格式坏报告保存拒绝 assessment；材料临时不可用不伪装永久语义失败。非空理由与定位是可机械验证的最低要求，不证明模型语义审查质量。

首次采用旧 Task 的新协议后，Verification 在模型运行前对 Task/Goal 归约，Control 和 ReadModel 也通过同一纯资格函数拒绝旧 satisfied 缓存冒充新审阅已满足；旧 Plan/Evidence 正文不改写。此步和结果后归约有各自恢复 checkpoint。

ReadModel 的 phase 标签来自最后一次已提交的归约；协议采用与全部phase缓存并非原子刷新。在采用后的checkpoint尚未成功时，先前phase标签可能暂时落后于当前Reviewer资格解释，不能据旧标签放行完成或下游工作。Control的当前资格与事务CAS守卫仍适用。

`openReport` 是结果接纳的当前读取，复核原工具材料与授权；`openHistoricalReport` 只从 canonical Work 取原绑定正文，由原 Reviewer owner 以 `historical_explanation` 读取并校验完整性。UI `/report` 只使用后者且明确标注历史用途。源码过期或原工具 grant 撤销不会删除 Reviewer 自己原报告的历史权利，也不能凭历史读取重新接纳证据。

## 已知启动前失败的产品恢复（DEF-17）

2026-09-10用户明确授权本节范围，实施与最终证据见[DEF-17](../planning/active/external-review-repair/DEF-17-reviewer-product-recovery.md)。此入口仅重新受理可证明未启动的失败，保留原Work/Run/Result/Evidence；不提供真实FAIL、坏报告或副作用未知的重抽，不扩展返工重验。

Context `recovery(workRef)` 从canonical Work/Run/TaskReviewProtocol和ArtifactVault持久公开运行观察核对唯一精确scope/work/profile/session、failed、唯一sequence=1的run_crashed、零trace/usage、canonical ended/crashed与终止事件一致、无output/result及协议尾项，并复核当前材料/配置。任何缺字段、歧义、矛盾或outcome_unknown均拒绝；不能仅凭空trace或错误文本判断未启动。内部proof只跨受信DI端口，不进入HTTP视图。

`POST /api/real/verifications/reviews/recover` 仅接scope、requestId、previousRequestId、allowExecute=true、reason，沿本机Host/Origin/token认证；额外proof/actor/verdict拒绝，human actor由服务端指定。Verification `recoverReview` 先保存独立授权请求，再冻结替代命令。当前视图提供recovery的allowed/code/failureReason/issues，回执保留前驱与授权原因；原请求仍能查询。

Control `replaceFailedWork` 先重放相同身份指纹的已提交Work，再判断首次替代的canonical版本/协议尾项/CAS；首次建立替代Work、Run、Attempt、Outbox和协议追加记录为单次原子提交。未提交的冻结命令再次执行前重核可信证明，提交丢响应保持原命令，不能生成新的身份掩盖未知。Verification按替代指纹验证新Work，随后沿原派发、原报告绑定、Result/Evidence接纳和Task/Goal归约链继续。两套ReadModel都投影替代Work的Agent行。

读`/reviews/read`或`/api/receipts`不会执行模型；`resume`只推进已持久授权请求。UI保存未决授权身份，丢响应先查同一回执，不能因刷新生成第二个Work。不同授权同时竞争原Work时只允许协议尾项被替代一次。

历史记录无需迁移：满足上述完整持久证据且来源/配置仍当前的failed/crashed可以人工恢复；旧outcome_unknown、只有人工日志称零调用、缺少或矛盾的持久观察均保持未知和拒绝。旧FAIL、旧Run和旧Evidence原文不改写。

持久观察Journal核对文件名与内容身份，重复身份保留所有歧义行并报告`integrityIssues`，不交回Runtime作为可恢复运行，也禁止覆盖歧义文件。Context恢复资格存在任何持久身份问题时失败关闭；磁盘事实保持原样，不以加载顺序选一个failed覆盖unknown。正常旧文件格式与身份继续兼容。


## 普通 GoalGate 独立取证增量（CM-I01-GATE-001）

VerificationRoundScope 可显式携带 gateSubject: 'goal'。taskId 标识当前 Plan 的 active GoalGate，runId 标识同 Goal、同当前 Plan 已成功结束的普通 Producer Run。省略标志仍严格要求被验 Task 与 Run 的 Task 一致；标志不能用于普通 work、其他作用域 Gate、Reviewer Producer 或跨 Goal/Plan 借用。Gate 不创建虚构的 Worker Run，Producer Run/Attempt 只承担真实来源、工具租约及报告所有权；Gate 的 VerificationPlan、工具检查、聚合 Evidence、独立 Reviewer Work 和覆盖都以自身正式义务为准。

首次 Gate 轮次经 Context 读取全部 active required work 与显式前置，交 Control 刷新其当前 Task 归约。未满足、已取消、来源不可用或有未对账运行时，不启动检查。前置 Evidence 索引、Review 协议和最新 Run 身份固定为 prerequisiteDigest；刷新和来源捕获前后比较，冻结后前置变化使轮次过期。Reviewer 的 Control 受理另核前置当前 Evidence 与归约，版本加入 CAS 读取集，不能用历史 satisfied 掩盖新 FAIL。

startRound、reviewMaterial、ReviewerProfile、startReview、read/resume/recover 使用同一个显式 scope；roundReceipt 保留 gateSubject，旧 scope/旧 journal 无该字段仍可读。Gate 按显式注册配置重新执行全部适用工具；工具未覆盖或正式 Reviewer 缺项保持 incomplete/INCONCLUSIVE，工具结果不得转抄 Producer 的 Task Evidence。正式要求包含 Reviewer 时，独立只读 Run 使用 Gate 的 ReviewPacket、真实 Producer session 与单独 Reviewer session；Task/Goal 完成始终由 Control 归约。既有未知副作用不重跑和精确请求幂等继续适用。

这次 Module 集成测试使用真实命令、正式 Control/Vault 和确定性 Reviewer 报告；服务 reopen 只证明 journal 重开，不代表新进程强杀、真实模型质量或 I01–I04 已验收。集成/最终证据由本批测试矩阵管理。

2026-09-15 引用预检增量（真实16拒绝后，尚待新快照验收）：新包携带citationCheckVersion=original-pointers-v1，新增指导仅对该标记生效；无标记历史包的运行输入原文不变。现有read_material可请求citationPointers（1–32个，每个至多2048字符），仍只读取包内唯一材料并执行前后权限/来源复核。完整offset0原文按RFC6901返回逐位置存在性；部分/矛盾页和响应超限明确unavailable，不判不存在，不裁剪原文。检查不证明相关性、语义、源码新鲜度或完成，也不接纳Evidence；最终Assessment仍重新读取授权原文并独立复核。共用纯路径规则排除JSON数组的JS length等元数据。原始坏报告和rejected assessment保留，不自动替换成PASS。
