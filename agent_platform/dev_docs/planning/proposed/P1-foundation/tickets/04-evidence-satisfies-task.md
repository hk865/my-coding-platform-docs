# P1-04：CompletionClaim → Evidence → Task/Gate SATISFIED

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-03
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/verification-engine.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interface_refs:
  - ../../../../interfaces/completion-policy.md
interfaces_to_freeze:
  - VerificationEngine.VerificationPort
  - ContextCompiler.ReviewContextPort
contracts_to_create:
  - CompletionClaim
  - VerificationPlan
  - Evidence
  - EvidenceBinding
  - EffectiveEvidenceSet
  - ReviewPacket
  - VerificationResult
input_artifacts:
  - { artifact: fake-agent-run-events, source: upstream, producer: P1-03 }
  - { artifact: accepted-plan-revision, source: upstream, producer: P1-02 }
  - {
      artifact: completion-claim-fixture,
      source: local_fixture,
      producer: P1-04,
    }
output_artifacts:
  - immutable-evidence-records
  - revision-bound-evidence-bindings
  - task-or-gate-verification-result
  - task-detail-verification-view
verification:
  - deterministic-check-provider-tests
  - reviewer-port-tests
  - evidence-applicability-tests
  - task-gate-reducer-table-tests
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-03 Eligible Task to Fake Run](./03-fake-run-visible.md)。

## What it delivers

Fake Worker 提交 CompletionClaim 后，VerificationEngine 为精确 Task、Workspace、Baseline 与 Policy revision 编译 VerificationPlan并收集适用 Evidence。ControlEngine 只把当前证据齐全的单个 work Task 或 GateTask 归约为 SATISFIED；失败、缺失或过期证据产生可解释返工或阻塞，本票不归约整个 Goal。

## Module / Interface refs

- `VerificationEngine.verify` 与内部 `CheckPort`、`ReviewerPort`；
- `ContextCompiler.assemble(ReviewContextRequest)`；
- [ControlEngine](../../../../modules/control/control-engine.md) 记录 Evidence 并归约 Task/Gate；
- [StateLedger](../../../../modules/data/state-ledger.md) 保存不可变 Evidence 与 binding；
- `ArtifactVault.put/open`；
- [ReadModelIndex](../../../../modules/data/read-model-index.md) 投影验证详情；
- [Completion Policy Interface](../../../../interfaces/completion-policy.md) 定义 EffectiveEvidenceSet、AcceptanceObligation 与 Task/Gate 满足规则；
- 本票首次冻结 VerificationEngine verification 与 ContextCompiler review-context 两个最小 Interface；ArtifactVault artifact 消费 P1-03 已冻结版本；P1-05/06/07/09/12/13/14 只通过这些版本消费验证与 Artifact。本票同时创建 claim、verification、evidence、binding 和 ReviewPacket contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。Reviewer 模型工作也走正式 dispatch；普通报告／语义提示不等于 Evidence 满足。验证迟到 verdict、旧来源版本和重复结果不能错误满足义务。

- Worker 与 Reviewer 只能提交 claim、observation 或 verdict，不能直接写 `Task.phase`；
- 每个 required verification requirement 都有当前适用 PASS Evidence 时，目标 Task/Gate 才能 SATISFIED；
- 静态或动态 FAIL 进入返工，缺失输入进入 BLOCKED，当前 binding 过期时显示 STALE；
- revision 改变只重新计算 EvidenceBinding applicability，不修改历史 Evidence；
- 新 PASS 可以进入当前 EffectiveEvidenceSet，旧 FAIL 仍可追溯；
- 普通语义变化使用有界 ReviewPacket；无语义变化只有命中版本化策略才快放；
- 单个 `exit=0`、Worker 自报或 Reviewer verdict 都不能独立满足 Task/Gate；
- Goal phase 在本票结束时仍由后续 Goal reducer 决定。
