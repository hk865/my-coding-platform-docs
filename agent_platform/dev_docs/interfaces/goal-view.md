# Goal View Interface：Goal 创建切片

```yaml
status: draft
updated: 2026-09-05
slice: 创建 Goal → 持久化 → 投影显示
participants: HumanCollaboration, ReadModelIndex
```

## Purpose

2026-09-05 [设计复核](../design/human-framework-role-review.md)：本契约仅覆盖 Goal 创建查询，尚不覆盖统一状态图、语义解释、需求／架构协作或其一致性契约。

本 Interface 固定 Goal 查询结果和 read-after-write freshness 语义。调用者可以用写入返回的
`commitCursor` 判断投影是否追上，而不读取 canonical store、猜测等待时间或回显提交请求。

本契约返回事实投影及其 freshness，不执行代码静态分析或 Agent 语义验证。ContextCompiler 可在查询授权范围内消费
这种视图，并结合其它材料编译任务 Context；正式状态推进仍由 Control 对 canonical state 校验。

## Interface

完整协作的扩展行为见 [运行时协作 Interface](runtime-collaboration.md)。以下类型仍限定于 CreateGoal 首切片；新增角色／消息／运行或报告查询必须显式扩展 schema，不把本切片类型直接用作完整协作类型。

```ts
type GoalViewQuery = {
  projectId: string;
  workspaceId: string;
  goalId: string;
  atLeastCursor?: CommitCursor;
};

type GoalView = {
  goalId: string;
  projectId: string;
  workspaceId: string;
  objective: string;
  desiredState: "active";
  activePlanRevision: null;
  aggregateRevision: 1;
  sourceCursor: CommitCursor;
};

type GoalViewResult =
  | { status: "ready"; goal: GoalView; observedCursor: CommitCursor }
  | {
      status: "not_ready";
      requiredCursor: CommitCursor;
      observedCursor: CommitCursor | null;
    }
  | { status: "not_found"; observedCursor: CommitCursor | null };

type ProjectionReceipt = {
  throughCursor: CommitCursor | null;
  appliedEventIds: string[];
};
```

`not_found` 只表示投影已经达到 `atLeastCursor`（若提供）但没有匹配 Goal；否则必须返回
`not_ready`。`observedCursor` 是 ReadModelIndex 已连续处理到的位置；`sourceCursor` 只记录当前 Goal 行
最后一次被哪个 Event 改变，不能单独证明整个投影的 freshness。

## Hidden Implementation

本文件没有 Implementation。等待、轮询或订阅策略属于 `HumanCollaboration`；索引、checkpoint、重建与
projection storage 属于 `ReadModelIndex`。

## Dependencies

- `CommitCursor` 与 `commitCursor` 使用 [Command/Event Interface](command-event.md) 的同一 opaque type；
- ReadModelIndex 的输入页使用 [StateLedger Interface](state-ledger.md) 中唯一的 `EventPage`；
- 不依赖数据库 schema、transport、UI framework 或 canonical snapshot 类型。

## Invariants

### Global

- Goal View 是 Event 的可重建投影，不拥有业务状态；
- freshness 由 cursor 证明，不由时间延迟或进程内调用顺序推断；
- View 字段只能来自已提交 Event。

### Local

- `ready.observedCursor` 必须已经连续覆盖查询的 `atLeastCursor`；
- `ready.goal.sourceCursor` 不得晚于 `ready.observedCursor`，但可以早于 `atLeastCursor`；
- `ready.goal` 的 projectId、workspaceId 与 goalId 必须逐项等于 query；
- `(projectId, workspaceId, goalId)` 是查询键；不得仅按 `workspaceId` 或 `goalId` 命中其他 Project 的行；
- 达不到 `atLeastCursor` 时，即使索引中暂时没有 Goal，也只能返回 `not_ready`；
- `GoalCreated@1` 投影出的 `activePlanRevision` 必须为 `null`；
- 同一 Event 重放不改变 Goal View，也不重复报告 applied Event。

## Test seam

以固定 GoalCreated/EventPage fixture 驱动任意 ReadModelIndex Adapter，再只通过 `goal` 查询。契约测试覆盖
cursor 落后、刚好追上、追上后不存在、ready、重复 Event、从空索引重建，以及两个 Project 复用相同
`workspaceId/goalId` 时仍精确隔离。

## Explicitly not responsible

- 不定义 Goal 创建 Command、业务 guard 或 canonical state；
- 不定义 Plan、Task、Run、Todo 或完整 Dashboard；
- 不承诺同步投影或特定等待时长。

## Context load

实现 Goal 查询或展示时默认只装载：本文件、当前 Ticket，以及
[`HumanCollaboration.Interface`](../modules/interaction/human-collaboration.md#interface) 或
[`ReadModelIndex.Interface`](../modules/data/read-model-index.md#interface) 中与当前角色直接相关的一节。
只有实现 cursor/event 适配时再读取 [Command/Event Interface](command-event.md)与
[StateLedger Interface](state-ledger.md)。
