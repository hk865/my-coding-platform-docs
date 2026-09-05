# P1-12：CodeGraph delta → Finding/DecisionBrief

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-07
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/verification-engine.md
  - ../../../../modules/control/architecture-reconciler.md
  - ../../../../modules/data/context-compiler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/data/workspace-reader.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
interfaces_to_freeze:
  - WorkspaceReader.ReadPort
  - ArchitectureReconciler.InspectionPort
  - VerificationEngine.CodeGraphPort
contracts_to_create:
  - ArchitectureInspectionIntent
  - CodeGraphSnapshot
  - ArchitectureDelta
  - ArchitectureFinding
  - ArchitectureDecisionBrief
  - CandidateBaselineProposal
input_artifacts:
  - { artifact: exclusive-writer-patch, source: upstream, producer: P1-07 }
  - {
      artifact: post-write-workspace-snapshot,
      source: upstream,
      producer: P1-07,
    }
  - {
      artifact: plan-pinned-architecture-baseline-ref,
      source: upstream,
      producer: P1-02,
    }
output_artifacts:
  - revision-bound-codegraph-snapshot
  - raw-architecture-delta
  - classified-architecture-finding
  - architecture-decision-brief
  - candidate-baseline-proposal
verification:
  - deterministic-codegraph-fixtures
  - raw-delta-purity-tests
  - finding-classification-tests
  - baseline-nonmutation-test
  - pinned-baseline-resolution-test
  - candidate-proposal-derivation-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-07 Parallel Readers + single Writer](./07-parallel-readers-single-writer.md)。

## What it delivers

一次已提交 Workspace 变更生成增量 CodeGraphSnapshot。ArchitectureReconciler 的唯一 baseline 输入是 PlanRevision pin：它只沿该 pin 加载 digest/revision 匹配的 immutable ArchitectureBaseline，再执行对账；不得读取 Project active ref 或使用内置 baseline。系统把纯机械 ArchitectureDelta 与带语义解释的 ArchitectureFinding 分开保存；重大或含糊变化形成绑定 source baseline 的 DecisionBrief 与 candidate baseline proposal，本票不自动修复代码，也不物化 candidate baseline 或移动 active ref。

## Module / Interface refs

- `ArchitectureReconciler.inspect`；
- `VerificationEngine` 的 CodeGraph 与目标化 Reviewer seams；
- [ControlEngine](../../../../modules/control/control-engine.md) 记录 observation/finding；
- `ArtifactVault` 保存大体积 CodeGraph 与报告；
- [StateLedger](../../../../modules/data/state-ledger.md) 和 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票首次冻结 ArchitectureReconciler inspection、VerificationEngine code-graph 与 WorkspaceReader read 三个最小 Interface；P1-13/14 只消费这些版本。本票同时创建 inspection、CodeGraph、raw Delta、Finding、DecisionBrief 与 candidate proposal contracts。

## Acceptance

本轮扩展依据：[运行时协作 Interface](../../../../interfaces/runtime-collaboration.md)。ContextCompiler 经 WorkspaceReader 获取版本化源码／Git／图索引，首个真实源码消费者冻结其读契约。验证工作树读取中变化、索引落后和图能力缺失不被误报为当前事实；架构规范不会被当前源码自动覆盖。

- CodeGraphSnapshot、ArchitectureDelta 和 Finding 均绑定 Workspace、Plan 与 baseline revision；
- Plan baseline pin 缺失、悬空或 digest/revision 不匹配时 inspection fail closed，并产生可解释诊断，不生成伪 Delta/Finding；
- ArchitectureDelta 只包含机械结构差异，不携带“正确/错误”裁决；
- Finding 保存分类、风险、置信度、来源和建议，并可来自结构、性能、权限或运行时 Evidence；
- 无 raw Delta 的性能或权限问题仍可形成 Finding；
- material/ambiguous Finding 产生包含原始理由、影响范围、选项、风险和延后后果的 DecisionBrief；
- candidate baseline proposal 从精确 source baseline、选中 Delta/option 与规范化内容确定性派生，记录 source ref、proposal digest 和预期 candidate digest，供 P1-14 物化；
- 相同 snapshot/baseline 输入产生相同 raw Delta；
- Worker/Reviewer 无权修改 baseline 或把自己的 CompletionClaim 作为新 baseline；
- 本票结束时不存在 RemediationTask、migration Gate 或 BaselineActivation side effect。
