# 0003 返工以计划修订承载、协调角色规格实体化、架构对账分机械与语义两层

```yaml
status: accepted
decided: 2026-09-10
decided_by: user
scope: 返工闭环、角色规格、架构对账与演进门
```

## 2026-09-11 用户确认与现行解释

来源：用户提供的[回复记录](../../human/user-replies-2026-09-11.md)第 3、5、7 节及附录；当前任务 `01a08ec8-cd5f-7a03-9c9b-03aa8d88a0d2` 中，Agent 逐项说明以下方案的合理性与限制后，用户直接回复「如果合理则我接受」。这是本日对下列范围的确认，不倒推证明原始 ADR 每项细节在 2026-09-10 均已获批。

- D1：返工进入新 PlanRevision，旧任务、旧失败与未处置义务保留；来源、范围、显式策略额度、人的拒绝构成自动受理边界。具体额度必须来自显式策略，不自行配置。
- D2：接受基线绑定版本化模块映射、Git 身份与源码摘要的方向。未逐项审阅的捕获工具、摘要算法及激活技术细节不因本次接受获得一揽子选型授权。
- D3：接受机械规则与独立语义审查配合。原 D3-3 的机械检查与当前义务重验是必要条件，不是完整充分条件；不能代替职责/可读性语义审查或适用的人类决定。具体规则和消费者仍依现行规范核对。
- D4：接受由生效角色矩阵签发绑定；规格、版本和权限由 Control 的 canonical 守卫复核，签发不代替授权。
- RW-18 对 D4-1/D4-3 的替代：`requiredOutputs` 是声明性产出期望，取消其独立完成门禁，不取消 Plan 中的正式验收义务。后续产出核对属于现有 12 Module 内的记忆能力，不能据“记忆模块”新增第 13 个 Module。
- 跨工作历史按申请者对精确历史材料的访问授权判断，不能以 Run 是否只读替代；工作区执行权限与历史读取权限分开。

以下原文保留为历史决定正文；与上述后续确认冲突的条款按本节解释。下方“当前源码事实”仅指原 ADR 记录时点，不是今日状态。当前实现与剩余义务仍以[模块状态](../../human/module-status.md)及本次架构核对为准。

记录上述确认时曾按用户要求暂停。随后用户明确「继续执行架构修复」并确认其他执行者已暂停写入，本次审查、修复与验收据此恢复；结果以本次偏差摘要及实际证据为准。本决定不授权下一阶段功能。

## 背景

PRODUCT「证据驱动完成」「连续执行」「角色协作」「架构治理」要求：失败的验收要能变成关联返工并在新版本上重新证明；不同角色的差别要由框架绑定的规格（权限、工具、角色设定、当下与历史 Context）决定；架构漂移要能发现、解释并进入修复或决策路径。

当前源码事实：

- 验证 FAIL 只让 Task=`failed`、Goal=`FAILED` 结束，没有任何返工任务产生者（`rework_required` 只是拒绝码）；
- `applyPlanChange` 硬绑 `tasks: sourcePlan.tasks`，PlanCompiler 自述 out of scope 含 `task set changes`，因此「新增一个承担原义务的任务」没有合法路径；
- 角色只有 `RoleBindingRefV1` 的字符串标识，没有版本化职责正文；
- 产品安装的 ArchitectureBaseline 不含 `sourceBinding`，对账必然 `baseline_unresolved`；MigrationGate 只有守卫、恒返回 `unsupported`。

本 ADR 记录用户已接受的四项决定，供实现与评审共用。

## 2026-09-11 返工材料传递决定

本次用户在两项具体选项中选择「保持现有依赖图，传递问题材料（推荐）」，[原话及提问上下文](../../human/user-replies-2026-09-11.md#本次架构收口任务中的后续确认)。组合根从 Verification 取得验证问题材料，再交给 Dispatch；Dispatch 每次受理仍由 Control 复核当前承担者、版本和正式 Evidence。不得以注入回调隐藏 Dispatch→Verification 长期依赖，不增该边。此段补充本次接线决定，不撤销旧 FAIL、义务或正式准入。

## 决定

### D1 返工以新的 PlanRevision 承载，并默认自动受理

1. 返工任务落在**新的 PlanRevision** 内，而不是计划外的新聚合：Evidence 的适用性由 `planRef/planRevision/workspaceRevision/policy/baseline` 五项版本元组机械判定（`policies/evidence.ts`），只有当前生效计划里的任务，其新 PASS 才能自动覆盖原要求。
2. `PlanPatchV1` 增加**任务集增量**（新增／取代／取消）；不允许改动义务正文与验收语义——后者属于 `CoordinationPolicyContentV1.scope.changesRequireHumanDecision`，必须人的决定。
3. 被取代的任务标 `disposition = superseded` 并记录 `replacedByTaskId`；旧 revision、旧 FAIL、旧报告**全部保留**。任务不删除、不消失，只是退出默认视图，改由历史／处置视图查看。
4. **默认自动受理**，但必须同时满足四条边界，否则提交给人决定：(a) 触发源是已提交的验证结论；(b) 改动落在 `inScopeRework`（同义务、同验收语义、只换承担者）；(c) 自动化预算（`maxAutonomousReworks`）未耗尽，该常量从安装期形状校验变为运行时真实上限；(d) 人没有显式拒绝过这条问题。
5. 受理后的提案与决定都落账本（`actor = system`），在界面与时间线可见，人可随时暂停／取消；外部通知通道（如邮件）留出接入点，本轮不实现。

### D2 架构基线来自版本化 source 与现场捕获

1. 模块→路径映射与允许依赖规则写成**目标仓库内的版本化文件**，纳入代码评审；
2. WorkspaceReader 现场捕获真实 TS/JS 关系生成 `sourceBinding`（含 commit 与摘要），随正式 install/activate 固化；
3. baseline revision 记录 `commitHash + treeDigest + 映射文件摘要`；候选 baseline 必须在当前 HEAD 上可复现（重放捕获得到同一 digest）才能激活；激活用 CAS，旧 revision 与旧 Plan 的 pin 不动。

### D3 架构对账分两层：机械规则 + 语义审阅

1. **机械层**：`dependencyRules` 从只有 `forbid_dependency` 扩展为包含 `allow_only`（白名单）、`no_cycle`（模块级无环）、`interface_surface`（公开 interface 集合比对），全部是集合／图比较，可确定性判定；
2. **语义层**：可读性、职责稀释、耦合搬家这类判断由**独立只读审阅**完成（复用 Reviewer 链路，不另造），材料 = baseline 内容 + 真实 diff + 相关规范段落 + 相关历史决定，模板按漂移类别逐条要求 verdict 与来源；来源不可用时标注不确定，**不生成确定结论**；
3. **MigrationGate** = 机械规则校验通过 + 受影响计划既有验证义务在当前 revision 重跑通过，两类结论都产出可追溯 Evidence。

### D4 角色规格实体化，持久事实进入真实 Run

1. 新增治理种类 `RoleSpecRevision`（沿用既有 install/activate/CAS 三段式），内容是版本化角色规格：`purpose`、`responsibility`、必读材料、可选材料、权限与工具、必产出、退出条件；
2. ControlEngine 维护角色矩阵与绑定校验（角色不存在、revision 过期、权限不符即拒绝，零写）；
3. ContextCompiler 按规格取材，必读材料缺失返回 `needs_material` 而不是静默通过；VerificationEngine 按规格校验必产出完备性；
4. **不新增 Module**：规格登记属 ControlEngine，取材与组装属 ContextCompiler，均在既有职责内；真实 Run 的工作身份由 DispatchEngine 在 claim 时建立，历史材料在派发时编译进既有 ContextBundle（选材理由与缺口写入 manifest）。

## 影响

- 需要改动的既有语义：`applyPlanChange` 的草稿一致性检查（扩展为可由源任务集与增量确定性推导）、`dependencyRules` 的规则种类、`maxAutonomousReworks` 的消费方式；
- 保持不变：全局不变量 #1/#2/#3/#7/#12/#13，PlanRevision 的不可变性，旧证据与失败记录的保留；
- 返工任务、角色规格与架构审阅三者共用既有治理与审阅链路，不引入平行状态机。

## 不做

- 不新增 Module、不为角色新造存储或每角色常驻进程；
- 不允许返工改动验收语义；
- 不在来源不可用时生成确定的架构结论；
- 不用自动修复替代需要人的决定。
