# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。

## T-044: docs 校验脚本（空位不变量结构校验）

**Kind:** feature
**Status:** draft
**Origin:** 延后项 2 评估确认——空位机制是会话核心设计，机械保护与投入相称；仅做结构性校验，不做编号 vs 历史比对（编号可合法重编号，历史比对会误报）。

**Goal:** `scripts/validate-docs.ts` 断言每容器（candidates/task/decisions）恰一个 `## X-0NN: 待创建` 槽位、为末行、前缀匹配容器；挂入 `npm test`。

### Architecture

- scripts/validate-docs.ts：镜像 validate-skills 模式（node:fs 读取 + 内联自检负向验证）
- 检查项：恰一个槽位行、槽位为最后非空行、前缀与容器匹配（C/T/D）；编号不比对历史
- package.json test 前置：`tsx scripts/validate-docs.ts && …`

### Out of Scope

- 编号 vs Git 历史比对：语义错误（合法重编号会误报），不做

### Requirements

1. 三容器各恰一个槽位、为末行、前缀匹配；违规样例被自检拒绝。
2. `npm test` 全绿（新增 validate-docs 前置）。

### Verification

- `npm test` 全绿（含 validate-docs 自检 + 三容器实检）。
- 收尾：T-044 清空，T-045 占位保留。

## T-045: 待创建
