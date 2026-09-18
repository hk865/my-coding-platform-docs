# 本地 GUI：候选验收与文件引用

适用实现：2026-09-08 UIR 修复。此接口细化[统一展示](human-design-status.md)，不替代完整 HumanCollaboration、自动评分器编排或原有完成策略。实现证据见[修复验收记录](../archive/2026-09-08-verification-history/2026-09-08-ui-workflow-repair.md)。

## 独立验收

两个 POST 接口均要求本机 Host、匹配 Origin、有效 `x-platform-token`，JSON 请求上限 2 MB；其他接口仍保持原有 64 KB 上限。

| 接口 | 材料 | 返回与约束 |
| --- | --- | --- |
| `/api/real/verifications/candidates` | projectId、workspaceId、goalId、runId、requestId、Git 文本 patch、patchSha256、origin、benchmark | 候选 ID、补丁摘要、当前文件内容摘要、正式 workspaceRevision；origin 为 model 或 assisted-repair。benchmark 固定 instanceId、datasetRevision、failToPass、passToPass 集合 |
| `/api/real/verifications/import` | 同一作用域、requestId、candidateId、candidateDigest、workspaceRevision、purpose、source、startedAt、completedAt、reportBody | verdict、completeness、通过/失败/缺失计数、具体用例和正式控制回执。purpose 为 original、diagnostic、repair；source 包含 name、reportUri 和报告原始 UTF-8 摘要 reportDigest |

候选登记对当前项目目录执行补丁反向检查，冻结有界内容摘要，通过现有写租约和 `Control.recordPatch` 推进 Workspace revision。后续候选必须标记 assisted-repair，不能替换首次模型候选的指定测试集合。运行必须已结束且结果明确；活动或待对账运行、错误作用域、过期计划/工作区、变化摘要均拒绝。

服务根据冻结的指定集合逐项核对原始 `tests` 报告，不相信客户端的通过声明。所有必测项均 PASSED 才为 PASS；失败、SKIPPED 或缺失均使总分为 FAIL。缺失情况另由 `completeness: missing`、missingTests 和计数解释。重复同名结果冲突时保守采用较差状态。此处理与当前单题官方结果一致；其他评分器接入需核对自身规则。

原始评分、同补丁诊断和辅助修复评分分别持久保存。相同 requestId 和相同材料重放返回同一记录，内容变化拒绝。候选登记留恢复日志，报告先落盘再提交正式命令；完整报告分块保存到 ArtifactVault。通过 `submitEvidence → reduceTask → reduceGoal` 得到正式判定，不直接编辑 ReadModel。策略要求若超出此 benchmark 可覆盖的动态检查，不能仅靠导入成绩宣告完成。

`GET /api/state` 的 liveRuns 按真实运行投影 candidates、verifications 与 commandChecks；UI 保留首次自主成绩，单列诊断与辅助修复。运行状态、正式 Task/Goal 状态、外部评分是不同事实。图的正式验收状态取自 TaskReduction，不能把最近诊断 PASS 当作任务已完成。

## 命令检查投影与正式回执

2026-09-14 增量：轮次编辑器可显式选择只读报告检查，配置必读的项目内路径。此方式固定 static，不发送命令/目录/超时；详情依据持久 definition 和 readonlyReport 展示原始报告、来源读取版本及工作区副作用，不把没有 shell 执行显示成丢失沙箱结果。完整资格及独立 Reviewer 边界见[普通只读报告验证](runtime-collaboration.md#2026-09-14-普通只读报告验证ig10)。

命令检查记录（liveRuns[].commandChecks）额外投影 `command`、`kind`、`timeoutMs`、`startedAt`、`finishedAt`，与 `status`（running／finished／interrupted 记录生命周期）和 `result.observations[].result`（PASS／FAIL／INCONCLUSIVE 检查结论）分开呈现。旧记录没有这些字段时返回 null，界面显示“未记录”，不补造值。

`POST /api/real/verifications/check-report` 返回 `{requestId, status, command, kind, timeoutMs, startedAt, finishedAt, observations, reports}`。`reports` 是 ArtifactVault 中持久报告正文的数组，命令、分类、退出码／超时、stdout／stderr、来源摘要、工作区版本与计划引用都在正文里；响应根只做记录投影。观测项存在但没有正文时，界面明确显示“报告未保存”。

`POST /api/receipts`（作用域＋requestId＋kind: goal／real-task／command-check，可选 runId）返回服务端对同一逻辑请求的正式事实：Goal 读模型、Run 聚合与执行器记录，或持久命令检查记录；`found: false` 表示请求未到达服务端。前端在网络中断、超时或刷新后用它区分“同一未决请求重试”和“用户再次发起相同操作”：未决重试复用原 requestId 与原载荷，正式回执（受理或拒绝）后该逻辑请求结束，下一次相同内容生成新标识。浏览器只持久保存 requestId 与载荷摘要，不保存指令、命令、引用正文或凭据。

这是经过本机授权的材料导入路径。来源由提交材料的人负责，服务核对材料摘要与绑定；尚未自动启动官方评分器，也不声称摘要能证明外部报告未经有权限提交者伪造。

## 文件引用与阅读状态

`GET /api/files/preview` 对可读文本增加 SHA-256，沿用项目范围、私有目录、符号链接和 256 KB 文本预览边界。文件在右侧独立标签打开；同项目同路径复用，项目切换移除旧文件标签。阅读位置按项目与路径保存于当前页面。

“添加到对话”形成 project/workspace/goal 范围内的任务草稿，当前支持整个文件引用，尚无行选区引用。`POST /api/real/tasks` 可带 `references: [{path,sha256}]`，最多 8 项。服务通过同一个文件读取边界重新核对当前字节摘要；变化或不可读则在启动前拒绝。核对后的路径与摘要附入真实内核输入，并明确内容尚未加载、Agent 按需读取，不能标成已读。删除草稿引用不修改项目文件。

新任务文字与引用草稿按目标隔离；切换目标开始时立即停用旧范围，异步旧响应不能把引用写入原目标。提交材料在首次点击时固定，不允许等待鉴权期间被另一份材料替换。

运行记录按节点追加，保持自动同步期间的滚动位置、详情展开和文本选区；原本在底部才随新内容滚动。整页重新加载不会恢复文字选区。右侧视图标签横向滚动，中栏最小 280px；设置只开放已连接能力，未接入的记忆、规则和角色卡片说明真实状态。

## 操作者配置的只读探索

2026-09-08 EXP 切片在空 Goal 接受 2–8 个工作节点和整体核验节点，planOrigin 为 operator。接口继承本机 Host、Origin、token 与 64 KB 请求边界：

| 接口 | 材料与行为 |
| --- | --- |
| POST /api/real/explorations/plan | 作用域、requestId、tasks（taskId/title/instruction/dependsOn）；校验重复、缺失依赖、环和已有计划；冻结来源摘要 |
| POST /api/real/explorations/run | 作用域、requestId、taskId、可选 budget；正式领取 Task，仅 read、空 writeScope，每个节点独立 Run 与上下文 |
| POST /api/real/explorations/review | 作用域、requestId、taskId、工作节点 runId、reviewVerdict、reviewText、reviewOrigin: operator；通过 Evidence 和 Task/Goal 归约形成结论 |

报告必须来自已完成且正式终结的真实 Run，含非空最终助手内容和成功 read 来源，绑定任务、计划、工作区版本及来源摘要。正式终态暂未同步时继续等待。审阅不覆盖既有结果；整体核验要求全部工作节点正式满足且报告齐全。后继只继承直接前驱已审阅报告及同作用域已 applied PASS 的 reviewText/reviewId；审阅备注标明 operator 来源，独立于模型原文，仍须核对，不继承间接祖先或其他失败目标。平台不因报告存在自动判断语义通过。

state.exploration 返回计划、报告、来源、审阅和报告错误；图显示实际依赖，报告在右侧打开。队列串行消费，独立节点可提交但不宣称并行执行。目标用量汇总所有 Run 调用，上下文窗口独立显示。累计输入／输出／调用次数／时长默认不设置；任何项目只有在用户显式提交这些字段后才受累计约束，服务端不补默认上限。服务端 explorationContextOnlyRoots 只影响单次响应输出容量（scoped 任务可用更大值），不改变累计限额语义。

当前 read 只能读文件，本次实验使用带机械索引与来源清单的快照。该切片不证明自动规划、自动语义审阅或通用项目准备已完成。[验收记录](../archive/2026-09-08-verification-history/2026-09-08-project-exploration/README.md)区分本地内核替身验证与两项目真实模型探索；实际探索因运行链报告语义错误未完成闭环。

运行列表按创建时间排序，旧记录回退至首运行事件/trace时间，无时间的旧等待记录稳定置尾。同任务失败重试尚未接入GUI；不得以普通claim绕过已有租约或正式失败状态。
