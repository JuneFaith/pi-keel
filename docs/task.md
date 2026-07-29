# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。

## T-004: 收紧 plan sealing boundary 与 verifier budget proof

**Kind:** refactor
**Status:** in-progress

### Requirements

- 只有官方 compiler 入口能够产生可提交给 Policy Kernel 的 `CompleteAccessPlan`。
- 不暴露通用 `issueAccessPlan()` 或 raw plan constructor；调用方不能用手工操作列表绕过 Shell/Direct compiler。
- `isCompleteAccessPlan()` 必须独立验证完整 resource budget，包括 `maxCommands`、`maxOperations`、`maxCwdCandidates` 和 `maxInputLength`。
- 保留 `CompleteAccessRequest`、`RequestCoverage` 等 type-only compatibility aliases；不保留可制造已认证 plan 的旧 runtime factory。
- Direct/Shell 的语义分析和现有 Policy Kernel 行为不变。
- forged、copied、mutated、over-budget plan 必须在 Kernel seam 被拒绝。

### Out of Scope

- 不提供 OS-level isolation、TOCTOU 消除、其他 Extension/`user_bash` 入口 enforcement。
- 不改变 Shell glob、动态 token、opaque control flow 的现有 fail-closed 规则。
- 不改变 Profile、PathPolicy、Guidance 文案或审批语义。

### Design

#### Rejected alternatives

1. **只删除 `issueAccessPlan` 的 barrel export**：不够。源模块仍可被同进程代码直接 import，且 raw `createAccessPlan` 仍可能制造绕过 parser 的 plan。
2. **保留公开 factory，但在 factory 中增加更多校验**：能改善结构完整性，但不能证明 plan 来自 Shell/Direct compiler；factory 调用方仍可直接构造语义对象。

#### Chosen architecture

采用单一 compiler sealing boundary：

```text
Shell/Direct draft compiler
        ↓ 仅产生不可提交的 PlanDraft 或 typed reject
compiler-entry sealing boundary
        ↓ private WeakSet + private brand + defensive copy + deep freeze
CompleteAccessPlan
        ↓ isCompleteAccessPlan() 再验证
Policy Kernel
```

- `access-request-types.ts`：保留 domain types、typed compiler outcomes 和 type-only aliases；新增内部 `AccessPlanDraft`/draft result 类型。
- `shell-compiler.ts`、`direct-tool-compiler.ts`：只生成 `PlanDraft`，不发行 plan；它们不再暴露可直接获得 `CompleteAccessPlan` 的 raw factory。
- 新的 `compiler-entry.ts`（或等价 boundary module）：集中官方 `compileShellCall`、`compileDirectToolCall`、`compileToolCall` 入口，持有模块私有 `ISSUED_PLANS`、brand 和 sealing function；只有该 boundary 能把 draft seal 成 plan。
- `access-plan-verifier.ts`：保留无副作用的完整性验证逻辑，由 boundary 使用私有 issuance state 调用；不导出 issuer。
- `access-request.ts`：保留 reject/evidence/path/effect helpers；删除 `createAccessPlan`/`createRequest` 的 runtime issuance 责任。旧 runtime factory 不作为兼容 API 保留，因为它与 compiler-only authenticity 不相容。
- `gate/index.ts`：只导出官方 compiler 入口和 `isCompleteAccessPlan` predicate；不导出 draft、issuer 或 raw constructor。

### Verifier contract

`isCompleteAccessPlan()` 的 budget proof 必须集中在 verifier 内，至少检查：

- `coverage.commandCount <= ANALYSIS_LIMITS.maxCommands`
- `operations.length <= ANALYSIS_LIMITS.maxOperations`
- `cwdCandidates.length <= ANALYSIS_LIMITS.maxCwdCandidates`
- `resourceUsage.inputLength <= ANALYSIS_LIMITS.maxInputLength`
- coverage counts 与 operation partitions 一一对应
- resource usage 与 coverage/operation 实际数量完全相等
- 所有 count 是 non-negative safe integer

Factory 在 sealing 前执行同一验证；verifier 不依赖 factory 已经做过检查。

### Migration

1. 新增 `PlanDraft` 和 draft compiler result 类型。
2. 将 Shell/Direct compiler 改为 draft producer。
3. 建立 compiler sealing boundary，迁移 private WeakSet、brand、copy/freeze 和 complete-plan creation。
4. 从所有公共入口移除 `issueAccessPlan`、`createAccessPlan`、`createRequest` runtime export；保留 type aliases。
5. 更新 Kernel、tests 和 imports，全部通过官方 compiler seam 获取 plan。
6. 增加 over-budget command plan、issuer absence、raw factory absence、draft-to-plan sealing 和 existing forged-copy 回归测试。
7. 更新 D-022/R-13 与 `docs/security-boundaries.md`，明确 JS 同进程任意代码不属于此 boundary 的强威胁模型；其他 Extension/非 gate 入口仍由 R-09 覆盖。

### Current Implementation

- `compiler-entry.ts` owns private plan sealing, WeakSet issuance, official compile entrypoints, and public plan predicates.
- Shell/Direct compiler modules now emit `CompilerDraftResult`; they cannot produce Kernel-acceptable plans directly.
- `access-plan-verifier.ts` no longer exports an issuer and independently checks `maxCommands` in addition to the other analysis budgets.
- Runtime `createAccessPlan`, `createRequest`, and `issueAccessPlan` exports were removed; type aliases remain.

### Current Evidence

- TDD RED confirmed the raw issuer export and missing command-budget check before implementation.
- `npm test`: 24 skill validations and all Profile/Path/Gate/Shell parser/command semantics/Index/Footer suites pass.
- Access request suite: 37/37 pass, including raw API absence and 129-command verifier rejection.
- `npx tsc --noEmit`: pass.
- `git diff --check`: pass.

### Remaining Verification

- 新 Pi session 运行时确认所有受管工具仍经官方 compiler entry。
