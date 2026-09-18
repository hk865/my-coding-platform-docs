# Query 事实、角色表达与高风险声明

依据：2026-09-15 用户的三层联合质量委托。本接口细化既有 Context、Verification、ReadModel 和 Runtime 的协作，不新增状态裁决模块。普通 Query 不默认增加第二次模型审校。

## 结构化事实

VerificationService.queryFacts 按显式 project/workspace/goal 返回当前可观察的工具轮次与独立 Reviewer 阶段。保留历史完成、实际执行终态、正式接纳结果及当前 applicability 的区别；轮次 PASS 不等于当前源码适用，Reviewer 结束不等于正式验收。applicability.ready 的工具轮次携带 material identity/source digest；未取得来源见证不能补造。

ReadModel 的 architectureReviewView 展示三个独立事实：acceptedProposal 是精确候选包的 accept/reject/defer；selectedCandidate 只观察该决定协议内的明确人工选项，提案作者的 optionId 单列为 proposalSelection，不能冒充人工 A/B 选择；activatedBaseline 仅列精确 proposalRef/decisionRef 对应的历史激活记录，不证明当前源码已验收。InitialDesign 的明确 option 决定保留其独立协议，不从 ArchitectureReview 接受记录推导。

ReadModel.queryHumanActions 覆盖域包括 InitialDesign、初始规划问题、执行反馈问题、计划变更、架构审阅、未知外部动作和返工。逐域声明覆盖范围与可用性；读取未闭合时 pendingCount 为 null。只读未决定的 PlanProposal 不能证明需要人工，关闭或被替代的 QueryJob 不因保留旧答案而重现为待处理。反馈核对 source、Workspace revision 与 active Plan。仅匹配到反馈决定 ID 而未核实精确选项时返回 unavailable，不能假装已解决。

返工由组合根捕获既有公开 reworkView，ReadModel 只接收数据及见证，不持有 Dispatch 回调。捕获前取得 Ledger cursor；Verification 版本在捕获前后须稳定且可用；消费完整事件扫描后核对同一 cursor 和 active Plan，最终再核 Ledger tail。空集合也须核这些条件。缺见证或变化为 unavailable/stale；不得以进程内 lastDrive 作为持久人工交接。当前预览不是新任务或正式人工决定。

新观察携带 object、scope、version、observedAt；子记录保留 ref/revision/recordedAt。ready-empty 是声明覆盖域内完整空集合；not_found 是指定对象未找到；unavailable、stale、failed 分开。超容量不得截断后报完整空集。版本摘要排除本次读取时间，保留语义记录和适用性；并发修改不得混合成成功观察。

Context 通过这些公开入口选择事实，保存进实际输入并纳入 freshness。读者失败不能降级为没有记录；子事实的 scope、对象身份、版本与父记录必须一致。现有材料授权、来源和用户记忆复核继续存在。

Verification 的工具轮次和 Reviewer 子记录须与查询的 project/workspace/goal 一致，其他 Goal 的子记录不能随正确父观察进入模型输入。读取完整轮次/Reviewer或其 applicability 子对象时，Context 从已授权的精确路径及捕获父记录派生 applicabilityAuthority，携带观察 family、scope、父观察版本/时间及记录身份/版本/时间；该身份不是模型参数，也不采用材料正文中自称的 family。

## 按角色的短 Skill

秘书、书记、参谋均按用户后续决定使用 V11 可追溯回答；不自动进行第二次模型批准。公共核心保持对象、关系、范围、时间、生命周期阶段与权威来源；禁止跨集合、跨关系、跨阶段或跨时间替换含义。三个角色只加简短职责层。领域词义由 Context 的 contextLabels 给出，不继续把每个业务反例追加到 Skill。contextLabels 是产品定义，不是存在性证据；字段未提供时仍为未观察。开放问题、代码解释和建议保留自由表达。

Context 固化 capturedAt 与 observationScope；对完整已识别的事实容器，read_query_fact 返回 meaning（对象、关系、覆盖范围、权威、scope、inputDigest、observedAt、recordPointer、observationStatus），并随引用持久保存。子记录继续依赖各自明确身份与既有适用性见证；任意子值或用户记忆不能仅凭形状获得该权威标签。读取 envelope 的 ready 不改变正文 unavailable 的观察状态。发布前完整重读包括这些元数据。

GovernanceReadModel.architectureActivation 是项目范围的窄读接口，读取当前基线及提交激活事件，返回激活者与时间，不推测作者。事件扫描不完整为 unavailable，读期间 active revision 改变为 stale，读取异常为 failed；不混入 Goal 的 architectureReviews 或 humanActions。语义版本排除读取时钟，并参与 Query freshness。current_baseline_activation 只在对应权威路径核对并渲染当前捕获的基线/激活者关系；它不证明内容作者、历史所有决定或源码验收。未提供该读能力的旧装配不声称已有激活观察。

旧 V1–V10 指导只由历史持久版本分支加载，保持输入字节。新捕获 queryPresentationVersion=3：三个角色均使用 factReadVersion=4；秘书为 secretary-summary-v11，architecture 为 adviser-cited-v11，handoff 为 scribe-cited-v11。旧 presentationVersion=2 保留秘书 V11/v4、参谋和书记 V10/v3；无该字段的旧观察沿原兼容路径，不升级旧输入摘要。

表达目的不改变权限与正式 RoleSpec。适用用户记忆仅改变表达与选材，不能创建授权、任务或决定；无阻塞或待决定时允许结束。

历史 V1–V9 文本与兼容测试已经独立归档。当前生产默认不导入历史指导；仅旧持久版本分支加载历史兼容模块，保留对应字节。历史阶段描述见 [归档](../archive/query-guidance-pre-v10-2026-09-16/README.md)。

## 高风险声明与独立检查

三个角色均使用下节明确的可追溯发布边界；可选固定事实陈述，自由解释和建议保留。三个角色都不把整个回答变成状态模板。当前最终传输为 schemaVersion=1 的混合片段 JSON，language 为 zh/en，blocks 可按问题组合且无强制栏目：fact 只含 kind/citation；explanation/inference/suggestion/uncertainty 含 kind/text/basis。开放的代码说明、方案比较与探索可以只有自由片段，代码段和自然语言完整保留，不要求无关事实引用。推断、建议标签不是伪造正式状态的免责；其语义仍需独立审阅。

Runtime 根据已核对引用的实际谓词生成 fact 句，拒绝模型在 fact 中添加自由正文。固定句同时检查权威输入位置，用户记忆里形似 GoalPhase 的值不能成为正式 Goal 事实。Task 身份来自 Task ref/taskId 或 Reviewer scope.taskId，不能使用 reviewId 顶替。InitialDesign 决定的明确 option 与 ArchitectureReview 对候选包的接受分开措辞；三态不混同。task_dependencies 从完整 PlanRevision 中按声明 taskId 读取实际依赖边及 requires.kind，不从任务层级、历史描述或两个状态推断阻塞，完整记录超出读界限时不能截断后冒充完整依赖集。

混合答案在发布前仍复核所有已读材料、权限、来源和版本，包括没有引用到的读取。语法错误、未知引用、未经断言的 fact、错误权威位置及超出原答案界限均拒绝发布，不把 JSON 原文或半答作为正式回答。新 Runtime 结果保存 presentation、渲染正文与 sources；Dispatch 将 presentation 一并写入既有 Vault Answer body，正式 QueryJobAnswer 保留渲染正文和引用。UI 显示该持久正文并沿原引用展开输入/摘要/freshness，不根据当前状态重新生成旧答案。旧 v1/v2 和旧持久结果不回写。

该结构只控制已选 fact 句的声明范围；不自动证明事实选择充分、解释因果成立、建议合理或自由文本没有越界。真实质量验收仍须检查开放解释及实际阻塞/失败/待决定的反向用例。普通 Query 不默认增加第二次模型校对。

历史 factReadVersion=3（旧v2仍保留）必须具备 read_query_fact，缺读取器在调用模型前失败。读取完整具体记录并声明 observation_status、goal_phase、task_phase、run_status、run_outcome、decision_outcome、selected_option、baseline_activation、pending_human_action、source_applicability 、verification_outcome 或 task_dependencies 之一；工具核对实际结构化值。run_status 对应携带 Run ref/revision、时间及 Plan 的 starting/running/ended；run_outcome 只对应 ended Run 的 completed/failed/cancelled/budget_exhausted/crashed/outcome_unknown，运行结束或成功不等于 Task 满足、Goal 完成或当前源码通过验收。裸状态或无法检查的总容器应进一步读取具体记录。

v3 已知高风险状态引用发布时必须带已核对断言，包含 decisionFacts 和 Reviewer formal 及其后代。未提供断言时可以返回已授权事实与 supportedAssertions，但不给可发布的 F 标记；不匹配的断言返回拒绝与补全信息。候选仅是该记录可证明的结构化值，不是建议正文或任意语义正确性的证明。无法提供候选的总容器需进一步选择具体记录。正确断言被显式声明并核对后才提供引用标记。

最终发布前重读全部已经提供过的事实材料，包括没有获得 marker、没有在正文引用、或曾因断言不匹配返回的材料；失效、撤权、身份/版本不符、错误 marker 和取消不得发布答案。读取清单独立于可发布引用清单，不能因未发 marker 绕过授权复核。发布拒绝持久保存受控原因类别，区分事实失效/不可读、标记格式/未知标记和断言问题，不回显底层异常或秘密。旧协议保持兼容，不宣称历史记录已经经过新检查。

引用持久保存 pointer、input digest、source bundle 和已检查断言，check 明确为 structured-state-only。UI 展开的是对应持久输入，需核对摘要并标明历史 freshness。

source_applicability 仅接受 Context 派生的 Verification 身份，不能仅因人工事项为 unavailable 或激活集合为 not_found 就核准为源码适用性。该身份随引用持久保存，并纳入最终完整事实复核；裸子对象仍绑定其 Verification 父记录。合法轮次/Reviewer 的 stale、unavailable 及轮次 failed 不得被误写成记录不存在。ready 的源码适用性断言仍要求 source digest 见证；仅有 Reviewer ready 状态而没有该见证时，可以引用就绪观察，不能据此扩成当前源码通过验收。旧持久答案不被追溯改写。

该检查不能证明任意自然语言正确，不能把弱断言当作整段文字的背书。独立质量审阅仍须逐项检查完成、阻塞、用户选择、正式待办、不存在和源码适用性，核对对象、时间、范围及确定程度。尤其历史 PASS 不等于当前适用，accepted 不等于选择 A，unavailable 不等于没有记录。无引用声明与语义范围过推同样属于失败。

## 验收边界

当前定向证据位于产品 evidence/collaboration-memory/batch/integration/semantic-reliability-19；模型替身测试只证明契约、真实模块/HTTP/内核消费者和拒绝行为。新真实统一场景、正确 Context 下的回答质量、集中回归与独立 I04 尚未通过。保存旧 Q03–Q06、V7 回放失败及 snap-13 real18 失败；若完整组合仍出现同类语义错误，应拒绝 I01/I04，评估高风险专用复核或更强结构化输出，不通过重抽挑答案接纳。

固定Goal/Run句在已知matchesCurrentPlan为false时、Task在实际归约记录存在且reductionMatchesPlan为false时前置标为历史记录，不将旧计划完成冒充当前完成。InitialDesign决定的真实引用位置为humanActions的域/记录/value/decisions数组成员，接受、拒绝、延后三种结果均保留；只有accept能支持selected_option。该路径由真实Control→Ledger→ReadModel→Context→fact消费者核对，不以手写端口对象代替接线证明。

缺少Task归约时保留unknown，不将reductionMatchesPlan=false解释为历史记录。v3的Task正式事实须引用含适用性信息的完整collaborationWork父对象或完整Reviewer记录；裸reduction拒绝建立固定事实引用，并在工具调用时提供纠正路径。三个角色选择固定 fact 块时均适用，开放解释、推断和建议保持自由表达；类型及引用检查不证明自由文字语义。

v3 read_query_fact 成功建立引用时同时返回 presentationUse：fact=true 仅表示该引用附带已校验、可由平台渲染的 assertion；basis=true 表示可用作自由解释等段落的依据引用，不表示任意自然语言得到证明。没有 assertion 的目标文字、汇总等引用可作为 basis，但不得用于 fact 块。工具说明该区别；发布时无 assertion 的 fact 块仍拒绝，并报告 assertion_required 与对应 marker，不自动降级成自由解释或自动丢弃该块。此输出只应用于现行混合发布协议，旧 v1/v2 引用响应保持兼容。所有最终来源重验、撤权和 freshness 规则不变。

## 三角色可追溯回答（用户后续决定，2026-09-16）

秘书负责自然总结，参谋负责分析，书记负责整理追溯；三个角色都允许用户点击引用查看回答时的原材料。发布不以第二次模型语义批准为门槛；引用可追溯通过不等于整段解释已获语义证明。该取舍适用于三个角色，不默认增加模型复核。普通开放问题仍可自由解释与建议。

v4 继续使用混合片段传输，自由片段的 basis 引用绑定精确输入摘要和记录路径。read_query_fact 对已授权高风险记录也可返回无 assertion 的可引用 marker；固定 fact 块仍必须提供可核对 assertion。读取材料后完全省略引用会拒绝发布；这一确定性检查只保证存在可追溯引用，不识别每句话的语义或证明每句都有充分依据。未读取事实材料的开放问题无需无关引用。所有已经读取的材料，包括未引用材料，继续在发布前重验授权、来源和 freshness；未知 marker、损坏版本、撤权和不可用不能靠摘要模式绕过。

角色回答不写正式 Task/Goal 状态、决定、任务或权限。正式状态始终由既有 Control/Ledger 归约。记忆只影响适用范围内的表达；推断、建议保留标签，摘要不获得额外执行权。

UI 正文的 [F#] 可点击定位该引用；按当前用户范围读取对应 Run 的持久输入，核对 inputDigest 后展示原记录、输入版本、来源对象、范围与捕获时间。已过期回答标为历史记录，不用当前数据重建旧引用。无法取得对应记录或版本不符显示明确错误，不以模型正文冒充原材料。历史记录缺时间时明确显示未提供。

该改动不把 snap20 real23 或 snap21 real24 改判通过；引用支持关系、简洁偏好及真实回答质量仍单独记录，不能由引用存在替代。先验证可追溯发布和原材料查看，再继续统一真实场景及独立验收。

## 用户发起的独立复核（候选34）

参谋与书记的已发布正式 Answer 提供“复核这段／复核整答”。用户点击前不调用第二模型；秘书及普通 Query 不自动复核。复核是独立持久操作，不是 Task 的 Reviewer 验收，不覆盖原答案、不增删正式任务、决定或权限。失败、冲突或无法确认不撤销原回答，只附加结果。当前不提供自动修订或再次复核循环。

公开 Host 入口 `/api/real/queries/review/view|start|cancel` 使用既有身份与项目授权，并校验精确 project/workspace/goal/queryJob/answer。仅当前正式 answered 的参谋／书记答案可以启动；绑定原 Run、输入摘要、段落及原引用。新操作使用用户当时配置的模型，另存其配置及计量，继承原回答配置的预算数值；不冒充原模型版本，不自动增加用户未配置的累计限制。

开始、调用前和结果发布前重新检查原来源与权限、适用记忆和输入摘要；取消在异步检查返回后也再次检查。来源变化后不向模型发送旧材料，查看旧复核时遮蔽评估并明确过期。取消不要求旧源材料仍可读。已完成操作的取消请求不删除结果；返回不包含评估，正式读取仍须重新核当前来源。复核原输入来自精确已持久执行绑定，不能用其他材料替换。

复核只收到所选段落、各段声明的引用，以及同一捕获输入中相应集合的 object/relation/coverage/authority 四项定义。定义绑定 marker/inputDigest/collectionPointer，仅解释范围，不证明存在性；不转发全部 Context、不补入未引用记录、不自动修补 basis。代码效果、常识和建议未必能从这些平台记录判断，应允许无法确认。

结果按段落与原文片段记录 supported（依据充分）、citation_insufficient（引用不足）、conflict（存在冲突）、unverifiable（无法确认）。模型判断可错；确定性解析只核段落覆盖、原文片段、引用归属和结果结构，不证明任意语义。界面明确是模型评估，原材料仍可展开。

同 Host 的同 requestId 重放不新增调用，内容冲突拒绝，同答案并行请求排斥。进程中断的 running 操作恢复为 outcome_unknown，保留用量预留，不自动重做。跨 Host 同目录唯一启动尚未证明，不以此候选宣称该部署模式通过。取消与关闭不修改原答案；后续显式新操作有新身份。

历史 v3 的 high-risk-v1 发布前复核保留持久兼容，但不应用于新的 v4 回答。此前 real23–real25 失败和候选33实验原样保留，不能按新取舍反写 PASS。当前候选的契约测试、浏览器、真实模型和最终独立验收分别记录。


## 手动修订提问与格式拒绝（候选35）

复核发现引用不足、冲突或无法确认时，用户可将原问题、精确原文片段及复核意见追加到现有提问草稿。意见标为待核材料，原引用编号只属于原回答；新回答须重新读取当前事实。保留用户未提交草稿、原Answer及原复核，回填本身零模型调用。用户可编辑，点击既有“提问”才创建新只读Query；不自动修订或再复核，不创建Task、决定或授权。保留原architecture/handoff用途；UI的记录整理对应handoff。超过4096字符明确要求编辑，不能静默截断后提交。

最终传输JSON解析失败与结构校验失败分别报告invalid_json、invalid_structure；未知引用等既有拒绝仍保留各自错误码。无效回答不发布、不部分修补、不自动重试。格式拒绝不能报告为已经确认的权限或来源失效；正确传输也不证明自然语言结论正确。
