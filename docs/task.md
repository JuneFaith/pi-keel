# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。

## T-043: parseOptions 窄接口重构（消除参数簇）

**Kind:** refactor
**Status:** draft
**Origin:** T-042 审计观察 D（Data Clump）——`parseOptions` 6 参、3 布尔同源于 config 且总是一起传递；用户确认现在实施。

**Goal:** 签名 6→3 参：parseOptions 直接消费 config 条目（窄契约 Pick），消除 `=== true` 拆包；行为零变化。

### Architecture

- text-transform.ts：提取 `TextConfigEntry` 命名类型；`PositionalConfig = Pick<TextConfigEntry, schemas/inPlace/programFirst/positionalsNotFiles>`（窄契约，避免接受未用字段的宽耦合）
- parseOptions 签名 → `(args, config: PositionalConfig, index)`，函数内解构 + 默认值
- 调用点 → `parseOptions([...node.args], config, 0)`，去 3 处 `=== true`
- 行为保持：既有 629 测试为安全网，无新测试（纯重构）

### Out of Scope

- TEXT_CONFIG 可选 flag 蔓延模式（根源）：值不得为 5 命令重构数据模型，延后

### Requirements

1. parseOptions 签名 6→3 参，窄契约 Pick 保依赖显式。
2. 行为零变化：`npm test` 全绿（629）。
3. 调用点无 `=== true` 拆包。

### Verification

- `npm test` 全绿（含 tsc 类型检查）。
- 收尾：T-043 清空，T-044 占位保留。

## T-044: 待创建
