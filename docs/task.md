# Tasks

> 活跃任务。验证完成后，提炼长期信息到 `docs/decisions.md`、`docs/security-boundaries.md` 或 `CONTEXT.md`，然后清空对应 Task Record 章节。编号取末尾最大 `T-xxx` + 1，不复用历史 ID。

## T-030: 提示词体系重构（Prompt Surface 边界）

**Kind:** refactor
**Status:** in-progress
**Goal:** 在不引发 LLM 理解问题（无歧义、语义完整）的前提下，精简与 LLM 交互的提示词：principles 恒定注入成本下降、skills 全部瘦身、消除格式重复漂移源。

### Architecture

三层提示词面：`src/bootstrap/principles.md`（恒定注入）、`skills/`（按需加载）、access-gate guidance（失败路径，不动）。重构核心是两条约束：① skill 单一职责、调用时全量消费；② 格式/规则单一来源 = principles.md Quick Reference，技能只引用不内嵌。

### Out of Scope

- **guidance 文本精简**：失败路径负 ROI，保持原样。Revisit when guidance 文本总量显著增长。
- **合并 skill（draft-spec/brainstorm、grill-docs/grill-plan 等）**：触发场景互斥，合并违反全量消费约束。Revisit when 两 skill 触发场景实际重合。
- **新建 project-records skill**：指针引用依赖模型主动 read，不可靠且可能多注入。Revisit when 格式内容超过 principles 承载的合理上限。
- **token 基线测量与提示词行为测试**：用户明确不做额外验证。Revisit when 出现可观察的遵守度问题。

### Requirements

1. 每个 skill 单一职责；调用时其内容全部被使用。
2. 格式/规则只在 principles.md 定义一次，skill 只引用不内嵌。
3. 修改后语义零变更：不删唯一语义、不引入歧义、不改变触发条件。
4. principles.md Quick Reference 压缩（112 → ~80 行），只删同义重复。
5. `plan-writing` 的 `## Global Constraints` 空节删除。
6. 修改后 `npm test`（validate-skills + tsc）通过。

### Plan

- [ ] P1: principles.md Quick Reference 压缩（去重不删义）
- [ ] P2: 6 个 skill 删内嵌格式副本，改为引用 principles（survey-context、domain-modeling、plan-writing、draft-spec、implement-work、security-review）
- [ ] P3: 簇 A bug 三兄弟职责重划（diagnosis=回路 / investigation=调查+记录 / systematic=根因）
- [ ] P4: 簇 B brainstorm Step 5 自身体系完整；plan-writing 空节删除
- [ ] P5: 簇 D code-audit 内部去重 + Supply Chain 引用 security-review
- [ ] P6: 簇 E fix-validation 引用 evidence-first；evidence-first 内部去重
- [ ] P7: 簇 C grill-plan/grill-docs 冗余检查（保持独立）
- [ ] P8: `npm test` 验证 + doc-sync

### Evidence

- `npm test`：24 skills validated，0 error / 0 warning；tsc --noEmit 干净；508 access-gate tests pass。
- principles.md：318 → 274 行；Quick Reference 仅语法级精炼（删连接词/重复导航句），全部唯一语义保留——`imperative wording`/`priority`/`roadmap commitment`/`user approval`/去向列举/`current work or session`/`external ownership boundaries`/`durable content`/`two authority levels`/`only during`/`promote to Task/Decision/other authority`/`Git retains history`/`full conclusion` 均已确认或恢复。
- 6 个 skill 内嵌格式副本改为引用 principles（survey-context、domain-modeling、draft-spec、implement-work、security-review；plan-writing 原已引用）。
- code-audit：Agent Readability 节与 Code Style/Types 逐字重复，整节删除合并（4 项规则全部并入）。
- 第二轮语义审计修复 13 处：principles 10 处恢复（roadmap commitment、user approval、去向列举、or session、external、durable content、two authority levels、only during、promote 去向、Git retains history、full conclusion、priority）；bug-investigation 恢复锚定句+example（文件自足）；security-review 恢复 `in docs/task.md`；draft-spec 恢复 "never sufficient approval"；domain-modeling 修复 pre-existing 引用断裂（User-Project CONTEXT.md Structure）。
- 第三轮反向→正向优化 7 处（§1 格言、§1 pick、§4a literal form、Authority keep on course、survey-context only-when×2、domain-modeling retired-not-superseded）；保留必要反向 ~61 处（安全门禁/否定误解/排除边界/铁律/防循环），每处保留均有理由。§4a 微调 "but only in literal form" 消除双 Use Shell 重复，并与 guidance 措辞（"every argument must be fixed text"）一致。

### Durable Updates

- [ ] CONTEXT.md：新增 Prompt Surface / Skill Single Responsibility 术语
- [ ] docs/decisions.md：新增 D-030 决策
- [ ] 完成后清空本 Task Record 章节
