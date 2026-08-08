# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。

## T-045: TEXT_CONFIG 位置参数性质枚举化

**Kind:** refactor
**Status:** draft
**Origin:** 延后项 1 检查确认——programFirst+positionalsNotFiles 实为同一概念（位置参数性质）却用布尔建模，与 D-027 的 valueKind 枚举不一致；无效组合理论可表示。

**Goal:** `positional: "file"|"program-first"|"set"` 枚举替代两个性质布尔；`inPlace` 行为布尔保留；无效组合结构性不可能；行为零变化。

### Architecture

- text-transform.ts：`PositionalNature` 枚举 + `TextConfigEntry.positional?`（默认 file）；删 `programFirst`/`positionalsNotFiles`
- 条目：sed/awk → `positional: "program-first"`，tr → `positional: "set"`，sort/uniq → 默认
- parseOptions：`programPending = positional === "program-first"`；intent 分支 `if (positional !== "set")`
- PositionalConfig Pick 更新

### Out of Scope

- inPlace 行为布尔保留（独立概念，不并入性质枚举）

### Requirements

1. 三性质（file/program-first/set）映射与原布尔组合行为完全等价。
2. `npm test` 全绿（629，行为零变化）。

### Verification

- `npm test` 全绿（tx 48 用例锁定行为等价）。
- 收尾：T-045 清空，T-046 占位保留。

## T-046: 待创建
