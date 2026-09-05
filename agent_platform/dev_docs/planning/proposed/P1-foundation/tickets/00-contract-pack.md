# P1-00：Executable contract pack

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P0-06
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interface_refs:
  - ../../../../interfaces/command-event.md
  - ../../../../interfaces/goal-view.md
  - ../../../../interfaces/state-ledger.md
contracts_to_create:
  - WorkspaceBootstrapCommand
  - WorkspaceBootstrapFixture
  - WorkspaceBootstrapManifest
input_artifacts:
  - {
      artifact: accepted-p0-decision,
      source: external_input,
      producer: P0-06/user,
    }
  - {
      artifact: workspace-bootstrap-source-fixture,
      source: local_fixture,
      producer: P1-00,
    }
  - {
      artifact: multi-scope-create-goal-fixtures,
      source: local_fixture,
      producer: P1-00,
    }
output_artifacts:
  - versioned-workspace-bootstrap-manifest
  - validated-workspace-bootstrap-source-fixture
  - validated-multi-scope-create-goal-fixtures
  - in-memory-bootstrapped-project-workspaces
  - versioned-create-goal-schemas
  - in-memory-create-goal-harness
  - create-goal-contract-evidence
verification:
  - empty-ledger-bootstrap-contract-tests
  - bootstrap-digest-and-revision-tests
  - multi-project-workspace-bootstrap-isolation-tests
  - same-local-id-scope-contract-tests
  - atomic-bootstrap-zero-partial-write-tests
  - runtime-schema-validation
  - in-memory-contract-tests
  - cas-idempotency-cursor-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P0-06 User review](../../../active/P0/tickets/06-user-review.md) — 等待用户接受 P0 与本 DAG。

## Authorization record (2026-09-05)

- 用户明确授权开始 **P1-00** 的并行实施（共享基础 + 三路并行编码 + 集成验收），本次为有限授权：**不**代表 P0/P1 已验收通过，**不**自动推进 P1-01，也**不**改变 P0-06 的 `in_review` 历史状态；
- 授权范围：建立产品工程、编写代码与测试、同步本票涉及的契约/验证记录/状态；执行内核 coding-agent 保持只读；不部署、不推送；
- 产品代码根：`/home/han001/projects/agents/agent_platform`（本票共享契约基线见该目录 `IMPLEMENTATION-HANDOFF.md`）；
- 本票 `blocked_by` 的 P0-06 仍为历史记录；本授权仅对本题生效，作为该前置的显式用户覆盖，不构成对 P0-06 的接受；
- 派发与集成流程记录：开发 DAG 规定 Module Worker 可在单票内围绕已冻结 Interface 并行，并在本票纵向验收路径汇合（见 [决策 0002](../../../../decisions/0002-module-dag-and-tracer-bullet-tickets.md)）。

## What it delivers

受支持的 bootstrap contract 从显式、版本化 source 构造 `WorkspaceBootstrapFixture`，在空 InMemory Ledger 中原子写入至少两个彼此隔离的 Project/Workspace entry。`WorkspaceBootstrapManifest` 记录规范化 source digest、每个 entry 的 identity/revision 与 bootstrap revision。固定 fixture 让两个 Project 故意复用同一个本地 `workspaceId`；随后在两个范围分别发送复用同一个本地 `goalId` 和 idempotency key 的 `CreateGoalCommand`，经 InMemory harness 产生彼此隔离的 `GoalCreatedEvent`、`GoalSnapshot` 和 `GoalView`，形成最小可执行 tracer bullet。

本票不创建 CompletionPolicy、ArchitectureBaseline、ArchitectureEvolutionPolicy revision 或任何 Project active governance ref；它们由各自首个消费者通过显式 install/activation contract 建立。

该票不实现或依赖 SQLite/生产 Adapter；动态创建 Project/Workspace、Plan、Task、Run 和 dispatch intent 均不在范围内。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md)；
- [ControlEngine](../../../../modules/control/control-engine.md)；
- [StateLedger](../../../../modules/data/state-ledger.md)；
- [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- [Command/Event Interface](../../../../interfaces/command-event.md)；
- [Goal View Interface](../../../../interfaces/goal-view.md)；
- [StateLedger Interface](../../../../interfaces/state-ledger.md) 固定 snapshot、原子 commit 与 EventPage 语义；
- 本票创建 Workspace bootstrap command/fixture/manifest contracts。

## Acceptance

- `WorkspaceBootstrapFixture/Manifest` 具有 schema version、非空 Project/Workspace entry 列表、规范化 source digest 和 bootstrap revision；
- MVP fixture 至少包含两个彼此隔离的 Project/Workspace，两个 Project 复用同一个本地 `workspaceId`，且每个 entry 都有完整作用域；
- fixture 与 manifest 不携带 governance revision 或 active ref；
- bootstrap 只接受空 InMemory Ledger，并通过受支持的 bootstrap contract 原子产生 Project/Workspace 与可审计 Event/Manifest；
- entry 缺失、identity 重复或 source digest 不匹配时零写入拒绝；
- 同一 source digest 重放保持幂等；非空 Ledger、不同 digest 重放或未知 schema version 被确定性拒绝；
- `CreateGoalCommand` 只能引用 manifest 中已初始化的 Project/Workspace；同一个本地 `goalId` 和 idempotency key 在不同 Project 中是彼此独立的 command/aggregate identity；
- CreateGoal contract schema 具有版本并可在运行时校验；
- fixture 经 InMemory Adapter 在两个完整作用域得到确定性的 Goal snapshot 与 GoalView；按 `(projectId, workspaceId, goalId)` 查询时不能命中另一范围；
- Goal 初始 `activePlanRevision = null`，且不产生 Plan、Task、Run 或 dispatch outbox；
- expected revision/CAS、idempotency key、事件顺序和 read cursor 有可执行 contract tests；
- 输出明确成为后续 Adapter 必须满足的 Interface，不声称生产 Adapter 已存在或已兼容；
- 只创建当前 bootstrap/CreateGoal fixture 消费的 schema 与 harness。