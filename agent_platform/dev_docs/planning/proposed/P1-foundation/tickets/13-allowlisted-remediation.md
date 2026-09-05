# P1-13：Allowlisted remediation → Writer → verify

```yaml
status: proposed
updated: 2026-09-05
kind: tracer-bullet-vertical-slice
blocked_by:
  - P1-12
architecture_ref: ../../../../../ARCHITECTURE.md
module_refs:
  - ../../../../modules/control/dispatch-engine.md
  - ../../../../modules/control/verification-engine.md
  - ../../../../modules/control/architecture-reconciler.md
  - ../../../../modules/data/artifact-vault.md
  - ../../../../modules/execution/worker-runtime.md
  - ../../../../modules/control/control-engine.md
  - ../../../../modules/data/state-ledger.md
  - ../../../../modules/data/read-model-index.md
contracts_to_create:
  - VersionedArchitectureEvolutionPolicyFixture
  - ArchitectureEvolutionPolicyRevision
  - ProjectArchitectureEvolutionPolicyActiveRef
  - InstallArchitectureEvolutionPolicyRevisionCommand
  - ActivateProjectArchitectureEvolutionPolicyCommand
  - RemediationPlanPatch
  - RemediationTask
  - RemediationDeduplicationKey
input_artifacts:
  - {
      artifact: classified-architecture-finding,
      source: upstream,
      producer: P1-12,
    }
  - {
      artifact: sqlite-workspace-bootstrap-record,
      source: upstream,
      producer: P1-01,
    }
  - {
      artifact: plan-pinned-completion-policy-ref,
      source: upstream,
      producer: P1-02,
    }
  - {
      artifact: plan-pinned-architecture-baseline-ref,
      source: upstream,
      producer: P1-02,
    }
  - { artifact: exclusive-writer-capability, source: upstream, producer: P1-07 }
  - {
      artifact: versioned-architecture-evolution-policy-local-fixture,
      source: local_fixture,
      producer: P1-13,
    }
output_artifacts:
  - persisted-immutable-architecture-evolution-policy-revision
  - project-active-architecture-evolution-policy-ref
  - accepted-remediation-plan-patch
  - deduplicated-remediation-task
  - remediation-writer-patch
  - remediation-verification-evidence
verification:
  - evolution-policy-local-fixture-schema-and-digest-tests
  - evolution-policy-install-roundtrip-test
  - evolution-policy-activation-cas-test
  - allowlist-policy-guard-tests
  - active-evolution-policy-resolution-test
  - remediation-governance-revision-binding-test
  - remediation-deduplication-test
  - exclusive-writer-lease-test
  - current-revision-verification-test
```

## Blocked by

2026-09-05 [设计复核](../../../../design/human-framework-role-review.md)：本票是待复核候选。开工或冻结契约前，按复核表确认本票的角色、输入输出与验收是否需要修订；P0-06 仍未关闭。

- [P1-12 CodeGraph delta to Finding/DecisionBrief](./12-codegraph-finding-decision-brief.md)。

## What it delivers

本轮协调策略复核补充：本票仅覆盖架构 Finding 的 allowlist 修复，尚不覆盖压缩次数、Token 开销、生命周期等全部诊断信号。事件分类、聚合／冷却、恢复预算和升级路径按 [复核稿](../../../../design/human-framework-role-review.md) 重新分配到相应切片；不把协调记忆或 Skill 当作当前授权。

本票首次消费 ArchitectureEvolutionPolicy。ControlEngine 只通过显式、版本化 local fixture 和 install contract 持久化 immutable revision；install 不自动激活。独立 activation contract 再以 CAS 建立 Project active ref，只有完成该步骤后，系统才从 canonical ref 加载 digest/revision 匹配的 policy。只有局部、确定、可逆且命中该版本化 allowlist 的 ArchitectureFinding 才生成绑定 evolution policy、Plan baseline pin 与 CompletionPolicy pin 的 RemediationPlanPatch。该 patch 经普通 Control/Scheduler/唯一 Writer/Verification 链形成并完成 RemediationTask；未命中策略、ref 无效或验证失败的 Finding 保持未解决，本票不修改 ArchitectureBaseline。

## Module / Interface refs

- `ArchitectureReconciler.inspect` 消费本票显式安装并激活的版本化 `ArchitectureEvolutionPolicy`；
- [ControlEngine](../../../../modules/control/control-engine.md) 接受 PlanPatch、去重 Task 并归约状态；
- `DispatchEngine`、`WorkerRuntime` 与 `VerificationEngine` 复用 P1-07 的唯一 Writer 路径；
- `ArtifactVault`、[StateLedger](../../../../modules/data/state-ledger.md) 与 [ReadModelIndex](../../../../modules/data/read-model-index.md)；
- 本票创建 ArchitectureEvolutionPolicy fixture/install/activation，以及 remediation patch/task 与 deduplication contracts。

## Acceptance

- local fixture 具有显式 schema version、revision、source identity 与 content digest；缺失或无效时拒绝安装，不使用内置 allowlist；
- install contract 持久化 digest/revision 精确匹配的 immutable ArchitectureEvolutionPolicy；同一 identity/revision 不可覆写，重启后仍可解析；
- activation contract 只接受已安装的精确 target ref，并以 expected Project revision 做 CAS；失败时不创建或移动 active ref，也不进入 remediation；
- 只有同时满足 allowlist kind、scope、risk、reversibility、policy revision 和 drift budget 的 Finding 才能自动进入 remediation；
- ArchitectureEvolutionPolicy 只沿 canonical Project active ref 解析；missing/dangling/digest mismatch 时不创建 RemediationTask，也不使用内置 allowlist；
- RemediationPlanPatch 固定 Finding、Workspace、evolution policy、Plan baseline 与 CompletionPolicy revisions；任一 ref 在接受前变化都由 CAS 拒绝或重新评估；
- RemediationTask 通过普通 PlanPatch 与 Control guard 创建，ArchitectureReconciler 不能直接写 Task state；
- 同一 finding/policy/workspace revision 只产生一个有效 RemediationTask；
- Writer 使用 P1-07 的唯一 conflict-scope lease，并返回 patch、changed paths 与新 Workspace revision；
- Verification 使用修复后的当前 Workspace revision 与 Plan-pinned CompletionPolicy；过期 PASS 不解决 Finding；
- remediation FAIL、BLOCKED 或 outcome_unknown 保留原 Finding 和所有历史 Evidence；
- 验证通过后 Finding 以新 Evidence 标记 resolved，不删除原始 Delta；
- ArchitectureBaseline active ref 在整个本票中保持不变。
