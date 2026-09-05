# P1-14：User Decision → migration Gate → BaselineActivation

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-12
  - P1-11
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/verification-engine.md
  - ../../../../modules/control/architecture-reconciler.md
  - ../../../../modules/interaction/human-collaboration.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - HumanCollaboration.ArchitectureDecisionPort
  - ArchitectureReconciler.BaselineEvolutionPort
  - VerificationEngine.MigrationGatePort
contracts_to_create:
  - ArchitectureChangeDecision
  - CandidateArchitectureBaseline
  - MigrationPlan
  - MigrationGateTask
  - BaselineActivation
input_artifacts:
  - { artifact: architecture-decision-brief, source: upstream, producer: P1-12 }
  - { artifact: candidate-baseline-proposal, source: upstream, producer: P1-12 }
  - { artifact: authorized-user-decision, source: upstream, producer: P1-11 }
  - {
      artifact: persisted-immutable-architecture-baseline-revision,
      source: upstream,
      producer: P1-02,
    }
  - {
      artifact: project-active-architecture-baseline-ref,
      source: upstream,
      producer: P1-02,
    }
  - {
      artifact: plan-pinned-architecture-baseline-ref,
      source: upstream,
      producer: P1-02,
    }
output_artifacts:
  - immutable-candidate-baseline
  - migration-plan-and-gate-evidence
  - cas-guarded-baseline-activation
  - baseline-change-view
verification:
  - candidate-materialization-digest-test
  - stale-source-baseline-rebase-required-test
  - decision-target-matching-tests
  - migration-gate-tests
  - activation-cas-race-test
  - existing-plan-pinning-test
  - versioned-baseline-evolution-interface-contract-tests
  - migration-gate-port-contract-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：现有迁移后 baseline activation 只覆盖架构演进的一段，需求／架构共同建立和变更协商的上游契约仍待明确。

- [P1-12 CodeGraph delta to Finding/DecisionBrief](./12-codegraph-finding-decision-brief.md)；
- [P1-11 Goal/Plan change to new revision](./11-goal-plan-change-revision.md)。

## What it delivers

ArchitectureReconciler 先确认 P1-12 candidate proposal 的 source baseline 仍等于 Project 当前 active baseline，再从 proposal 与该精确 source 确定性物化、保存 immutable candidate baseline；其 content-addressed ref 必须等于 proposal 的预期 digest。P1-11 的 `authorized-user-decision` 只证明授权校验路径已经存在，不是对该 candidate 的预授权。用户随后从 DecisionBrief 接受、拒绝或延后该精确 candidate。接受时平台保存绑定 source/candidate refs、受影响 Plan 与 Workspace revision 的 migration plan；只有本票新建的 matching ArchitectureChangeDecision、migration Gate PASS 与 Project CAS 同时成立，ControlEngine 才推进 Project 默认 baseline，并展示既有 Plan 仍固定在哪个 revision。若 Project active baseline 在 proposal、Decision、Gate 或 activation 之间移动，整组演进工件变为 STALE，必须基于新 active baseline 重新派生 proposal、candidate、Decision 与 migration Evidence。

## Module / Interface refs

- [HumanCollaboration](../../../../modules/interaction/human-collaboration.md) 提交 ArchitectureChangeDecision；
- `ArchitectureReconciler` 产生 candidate baseline 与 migration plan；
- [ControlEngine](../../../../modules/control/control-engine.md) 校验 Decision、Gate 与 CAS 并执行 BaselineActivation；
- `VerificationEngine` 生成 migration Gate Evidence；
- [StateLedger](../../../../modules/data/state-ledger.md) 保存 Decision、candidate、Gate 和 activation；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 展示默认 baseline 与既有 Plan pin；
- 本票首次冻结 HumanCollaboration architecture-decision、ArchitectureReconciler baseline-evolution 与 VerificationEngine migration-gate 三个最小 Interface；它们不扩张 P1-12 的 inspection/code-graph 端口，后续消费者只能使用这些版本或显式升级。本票同时创建 architecture decision、candidate baseline、migration 和 activation contracts。

## Acceptance

- candidate baseline 必须从 P1-12 proposal 指定的 source baseline 与规范化内容确定性物化；物化时 Project active baseline 必须等于 proposal source ref，且 source/candidate digest 必须匹配，否则不能请求 Decision 或运行 migration；
- candidate 先以 immutable revision 持久化，本票再经 P1-11 已验收的 authority path 创建新的 ArchitectureChangeDecision；该 Decision 记录 subject、outcome、actor、authority、authorized target、当前 from ref 与精确 candidate ref；
- reject、defer、未授权或 target 不匹配的 Decision 不产生 BaselineActivation；
- accepted Decision 只授权其精确 candidate；proposal source、candidate parent/source、Decision from、MigrationPlan from 与 activation 时的 Project active baseline 必须是同一个精确 ref；migration Gate 必须在该 candidate 和当前 Workspace revision 上 PASS；
- MigrationPlan/Gate 明确列出当前 Project default、candidate、受影响的 pinned Plan 与迁移 Evidence；Gate 不能通过隐式 rebase 消除差异；
- BaselineActivation 以 expected Project revision 和 proposal source/current active baseline ref 做 CAS，竞争激活最多一个成功；
- source ref 在任一阶段不相等或 CAS 失败时，candidate/Decision/Gate 标为 STALE 且不激活；不得把旧分支直接覆盖到新默认 baseline，必须从新的 active ref 重新提案并重新裁决、验证；
- 每个 ArchitectureBaseline revision 不可变，activation 只移动 Project 默认 ref；
- 既有 Plan 保持其 pinned effective baseline，只有显式 PlanRebase/新 PlanRevision 才重新计算 Evidence applicability；
- activation 不重写旧 FAIL、Finding、Decision 或 Evidence；
- 控制台可以解释“谁授权、迁移证据是什么、哪些 Plan 尚未 rebase”。
