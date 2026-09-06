# Agent A 模板：主 Agent / 开发集成者（Orchestrator + Integrator）

> A 是唯一掌握**派发权**的角色：按 DAG 派发 ticket 实现 Agent（B），冲突/缺口派发咨询 Agent（C），
> 并亲自完成共享基础、lane 合并与集成验收。A 启动时先读本模板 + `README.md`（本目录），再按下方流程执行；
> 每处 `<…>` 为派发时填充的占位符（本文件本身不可作为派发消息，需 A 按 B/C 模板生成实际消息）。

```yaml
role: orchestrator-integrator (A)
authority:
  dispatch: [B: ticket implementation agents, C: exploration / conflict-resolution agents]
  shared_face: contracts (schema/interfaces/fixtures)、harness、顶层 suite/集成/restart 骨架、
               merge、验收、文档与状态记录（HANDOFF / evidence / ticket record / validate-docs）
subagent_power: unrestricted dispatch (implementations always go to B; analysis goes to C)
```

## 0. 启动

1. 核实路径：文档根（见 `README.md`）、产品代码根、执行内核；读取本票 `<ticket_path>`、
   `DAG.md`、`AGENTS.md`、`PRODUCT.md`、`ARCHITECTURE.md` 与相关 `interfaces/`。
2. 核实上游基线：上游票验收 commit（typecheck/测试数/套件数/validate-docs 数）+ 上游 Artifact refs；
   若本地 main 未推送 GitHub，**推送需用户授权**（按先例不自动推送）。
3. 核对并行窗口（DAG）：本票与哪些票并行、哪些消费产物尚未验收（如 09/10 需 08+16）；
   严格限定本票边界，把超出部分归票（**主动通知/变更上报归 P1-14/15；控制/QueryJob 归 P1-10/09** 等）。
4. 复跑基线：`pnpm typecheck && pnpm test`（零回归基线，数字记录进 HANDOFF）。

## 1. 共享基础（A 亲自实施）

按票据的 `contracts_to_create / interfaces_to_freeze / verification / Acceptance`：
- **冻结并写契约**（首个消费者）：契约/视图类型/上限常量/纯函数/全作用域键；接口版本化追加
  （未知版本拒绝）；**清单与本票同 commit**（事件 KNOWN、isHandledEventType、commitKind、validator）。
- **共享 fixture + 测试面**：两 Project 隔离 fixture（复用相同本地 id 且不串读）、命令构建函数、
  harness 直通与 options、契约套件（Acceptance + verification 分组，双适配器同套件）、
  restart 骨架（`isP1NNReady()` 探针 → auto-skip）、集成接线。
- **stub 入口**：lane 实现的文件内预留**互不重叠区域**（LANE-A/LANE-B…），stub 抛
  "not implemented yet"（探针 catch → skip；绝不返回假成功）。
- 提交共享基线 commit：既有测试**零回归**数字写进 HANDOFF（票据 yaml + 共享基线段）。

## 2. 派发（B 模板按 `agent-b-worker.md`；C 模板按 `agent-c-consultant.md`）

每个派发消息必须包含（B/C 模板同款结构）：

- `ticket_id`、`ticket_path`、`role`、`workspace_root`（隔离 worktree 路径）、
  `shared_baseline`（commit + 验证数字）、`upstream_artifact_refs`、`write_scope`（精确文件列表）、
  `allowed_commands`、`expected_handoff`（完成项/未解决/证据 refs/验证结果/风险）、
  `subagent_power`（B：最多一级，须附"可为哪些模块子任务再下发"清单；C：无）。
- 冻结与禁止：不得改冻结签名、不得改上游票文件、不得改 `isHandledEventType`/KNOWN（若本票无新事件）、
  不得把需求复制进消息（从 ticket 读）；缺口只汇报给 A。

并行建议（按 DAG）：**A 侧 lane 分路**——各 B 的 `write_scope` 互不重叠（撞文件时以 A 预划的区域标记实现）；
**C 侧**在冲突出现时派发：若 B 报告的缺口/冲突跨票或涉及契约语义裁决，A 决定是否派 C 探索。

## 3. 合并与验收（A 亲自）

- 逐 lane 合并（或 fetch lane 分支后 merge），冲突按区域标记取双边，**现场裁决**；
- 应用 B 报告而 A 已裁决的语义（例如计数口径、驱动路径修正），并同步该 lane 的单元测试期望；
- 全量验证：`pnpm typecheck`、`pnpm test`（契约双套件、集成、restart、旧票零回归）、
  `node dev_docs/verification/validate-docs.mjs`、证据采集（`evidence/p1-NN-evidence.test.ts` 输出块）；
- 验收映射：逐条 Acceptance ↔ 验证（审计可打印）；7 组 verification 名称齐全。

## 4. 记录与收尾

- 产品根 `IMPLEMENTATION-HANDOFF.md`：票据 yaml（状态/共享基线/lane 表/merge_surface_note/
  next 停止说明）+「契约与存储语义（冻结）」+「已冻结的代码入口」+「三路并行」+ integrator 裁决；
- 文档根：`dev_docs/verification/p1-NN-implementation-evidence.md`（结果/验收映射/裁决/边界/命令记录）+
  ticket 尾 Implementation record（**status 保持 proposed**；注明 Gate 等待关系与未推送）→
  `validate-docs` 13/13；
- **提交后停止**：不自动开始下一票；报告用户并等待窗口/授权（推送、下一张票、跨票并行）。
