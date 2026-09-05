# 初始设计与统一展示 Interface

```yaml
status: draft
updated: 2026-09-05
first_consumer: P1-15
```

依据：[PRODUCT](../../PRODUCT.md)、[运行时协作](runtime-collaboration.md)。本契约是设计态行为要求，具体序列化与兼容版本在 P1-15 冻结。

## 初始需求与架构

HumanCollaboration 接受 scoped design request，PlanCompiler 通过正式运行路径返回 clarification 或版本化 proposal。Proposal 包含需求／验收、架构选项与取舍、受影响规范、假设、来源及建议；人可以修改、接受、拒绝或延后，不以沉默表示接受。

InitialDesignDecision 绑定 actor、授权范围、精确 proposal revision、scope 与决定。ControlEngine 接受后产出已接受规范引用与安装意图；规范正文存 ArtifactVault，决定及引用存 Ledger。重复提交返回相同结果，过期来源需重新提案。

初始基线使用显式 install/activation：source baseline 为空且 Project 当前无 active baseline。已有 baseline 时复用演进决定、迁移 Gate 与 CAS；不通过“初始创建”绕过迁移。规范、Baseline 与 Plan 逐步提交的中间状态可见且可恢复；只在所需引用齐备并通过 guards 后派发实施工作。

初始协商可以在 Goal 尚无 active Plan 时运行：以独立 CoordinationJob 绑定已 bootstrap 的 Workspace、调用身份、只读权限和有限预算。它不领取 required implementation Task 的写 lease，不推进 Goal 完成；实施派发仍必须等待有效 Plan 与治理引用。

## 有限协调策略

CoordinationPolicy 由明确 source 安装并版本化，包含适用 scope、允许动作、所依赖规范版本、预算／重试／冷却、升级与撤销条件。Control 校验，不由模型临时解释成更宽的授权。

默认候选仅允许既定目标和验收内的分工、测试补充、返工与安全交接；需求含义、验收和 baseline 变更无显式委托则升级。初版配置给出有限测试预算，不声称任意压缩／token 阈值具有通用有效性。观测事件先去重与聚合，能机械处理则不用模型；需要语义判断才创建有界工作。

## 统一展示

HumanCollaboration 组合一个 scoped presentation：

- 事实视图：Goal／Task／角色／Run 关系、阻塞、Evidence、Decision，包含 observedCursor 与对象 revision；
- 图形：渲染工具使用同一事实视图，默认任务进度／阻塞，可展开架构、来源与生命周期；
- 文本：确定性状态摘要无需模型；语义解释包含 source manifest 与使用的 revision，独立标记 pending／ready／stale／unavailable；
- 待决事项：绑定 proposal revision、影响与允许操作，不直接修改投影状态。

读取过程中版本变化时返回版本清单或重新取一致快照，不能假装所有对象同一时刻更新。语义解释未就绪或过期不阻塞事实展示；报告明确是 report，不替代正式 phase。历史查看保留历史版本标签。

## Test seam

跨项目同名 ID 隔离；决定重复／过期／拒绝；首次安装与已有 baseline 分流；部分提交恢复；策略撤销与越界升级；图文版本错配；解释失败仍可读事实；纯查询不调用模型；任何点击只经 Command 生效。
