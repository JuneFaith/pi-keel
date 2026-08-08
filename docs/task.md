# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。

## T-042: 注册 tr 为字符转换只读工具（text-transform adapter）

**Kind:** feature
**Status:** draft
**Origin:** 用户批准 unknown command（`tr "\n" " "`）——按 T-040 od 先例注册；D-031 “POSIX 只读检查工具”封闭范畴已覆盖，无需修订决策。

**Goal:** tr 注册为 inspect 类；positionals（SET1/SET2）是字符集非文件路径，不产生路径 intent（D-027 值性质）。

### Architecture

- text-transform.ts：TR_OPTS（全 flag：`-c/-d/-s/-t` + 长选项）+ TEXT_CONFIG.tr + 新标记 `positionalsNotFiles`（positional 消费但不产生 read intent）
- 测试：command-semantics-tx.test.ts 追加 tr 用例

### Out of Scope

- tr 的文件参数（GNU/POSIX 均无——输入仅 stdin，文件经重定向由编译器处理）

### Requirements

1. `tr '\n' ' '` → inspect，intents []（字符集非文件）。
2. `tr -d/-ds/--delete/a-z A-Z` → inspect 不 opaque，intents []。
3. TDD RED→GREEN；`npm test` 全绿。

### Verification

- 新增 5 用例全绿；`npm test` 全绿无回归。
- 收尾：T-042 清空，T-043 占位保留。

## T-043: 待创建
