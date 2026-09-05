# P1-01：Goal persisted and visible

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-00
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
input_artifacts:
  - {
      artifact: versioned-workspace-bootstrap-manifest,
      source: upstream,
      producer: P1-00,
    }
  - {
      artifact: validated-workspace-bootstrap-source-fixture,
      source: upstream,
      producer: P1-00,
    }
  - {
      artifact: validated-multi-scope-create-goal-fixtures,
      source: upstream,
      producer: P1-00,
    }
  - {
      artifact: versioned-create-goal-schemas,
      source: upstream,
      producer: P1-00,
    }
  - {
      artifact: in-memory-create-goal-harness,
      source: upstream,
      producer: P1-00,
    }
  - {
      artifact: create-goal-contract-evidence,
      source: upstream,
      producer: P1-00,
    }
output_artifacts:
  - sqlite-workspace-bootstrap-record
  - sqlite-ledger-adapter
  - sqlite-read-model-adapter
  - persisted-goal-event-stream
  - restart-snapshot-load-and-view-rebuild-evidence
verification:
  - sqlite-empty-ledger-bootstrap-test
  - bootstrap-digest-and-revision-replay-test
  - sqlite-multi-project-workspace-isolation-test
  - sqlite-same-local-id-scope-test
  - shared-adapter-contract-tests
  - sqlite-transaction-tests
  - snapshot-load-and-event-page-view-rebuild-test
  - idempotency-and-cas-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-00 Executable contract pack](./00-contract-pack.md)。

## What it delivers

平台只在空 SQLite Ledger 上通过 P1-00 的 bootstrap contract 写入版本化 `WorkspaceBootstrapFixture`，原子持久化至少两个隔离 Project/Workspace entry 与 manifest。用户随后通过最小命令入口在两个范围分别创建复用同一本地 `goalId` 的 Goal；系统按完整作用域原子保存 Event 与当前 snapshot 并投影 GoalView。进程重启后，ControlEngine 从各自持久 `GoalSnapshot` load canonical Goal；ReadModelIndex 独立消费 `EventPage` rebuild GoalView，不从 Event fold canonical Goal。

本票首次证明 SQLite Ledger/View Adapter 满足 P1-00 的同一 Interface；动态创建 Project/Workspace 不在 P1 范围内。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 的最小 CLI 或 HTTP Adapter；
- [ControlEngine](../../../../modules/control/control-engine.md)；
- [StateLedger](../../../../modules/data/state-ledger.md) 的 SQLite bootstrap/Ledger Adapter；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 的 SQLite Adapter；
- [Command/Event Interface](../../../../interfaces/command-event.md)；
- [Goal View Interface](../../../../interfaces/goal-view.md)；
- [StateLedger Interface](../../../../interfaces/state-ledger.md) 固定 canonical snapshot load 与 EventPage projection rebuild 的分工；
- P1-00 的 Workspace bootstrap command、fixture 与 manifest contracts。

## Acceptance

- bootstrap 只允许空 SQLite Ledger，并且只能通过受支持的 bootstrap contract 写入，禁止直接插入 SQLite 表；
- SQLite 持久化至少两个 Project/Workspace entry 的完整 identity、fixture schema version、source digest 和 bootstrap revision；
- 两个 Project 可复用相同本地 `workspaceId/goalId` 和 idempotency key；aggregate、幂等、snapshot、Event、View 与锁键都以完整 Project 作用域隔离；
- 同一 manifest 重放保持幂等；非空 Ledger、不同 digest 重放或旧 expected revision 被拒绝且无部分写入；
- 重启后 canonical Goal 只从持久 `GoalSnapshot` load；不从 Event Log fold 或修复 canonical snapshot；
- GoalView 由 ReadModelIndex 消费持久 `EventPage` rebuild，结果与重启前一致；
- 重启后可以验证相同 WorkspaceBootstrapManifest；
- `CreateGoalCommand` 引用 manifest 中的 Project/Workspace，不接受未初始化 identity；
- Event 与 current state 在一个 SQLite 事务内提交，expected revision/CAS 生效；
- SQLite Ledger/View Adapter 通过 P1-00 的共享 contract fixtures，调用者不改变 Interface；
- bootstrap、CreateGoal 与 restart 路径不创建或推断 governance revision/active ref；
- 同一 Project 作用域内重复 idempotency key 不创建第二个 Goal；跨 Project 使用相同 key 不会互相判重；
- 进程重启后，两个作用域的 loaded Goal revision 与 rebuilt GoalView 分别和重启前一致，任何查询都不会返回另一范围；
- 不产生隐式 Plan、Task、Run、Todo 或 dispatch side effect；
- 命令与展示 Adapter 不直接访问 SQLite 表；
- 没有运行时 Project/Workspace 创建路径。
