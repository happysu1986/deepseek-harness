# AGENTS.md — 文档标准

本文件规定 document structure、Markdown tier、写作规则和 `verify-doc-budgets` ceiling。使用 [dsh-doc](../.agents/skills/dsh-doc/SKILL.md) 判断位置并执行验证，使用 [dsh-prose-standard](../.agents/skills/dsh-prose-standard/SKILL.md) 判断必需覆盖内容并进行编辑；相关 rationale 由 [doc-tiers Agent Note](../.agents/notes/implemented/process/2026-07-04-doc-tiers-and-budgets.md) 负责。

## 文档结构

这些规则适用于面向人的文档；[Agent Notes](../.agents/notes/README.md) 不在其范围内。[postmortem](postmortem/README.md) 是针对 incident 的 reference；chronology 用于记录 evidence，而不是教学顺序。文档主题及其在目录树中的位置决定 scope：以适当细节描述自身主题；对直接子级只描述 purpose、responsibility 和 high-level behavior；更低层细节应链接到负责它的后代文档。Document type 不会扩大该 scope。Reference 只能对自身主题做到穷尽。Testing mechanism、fixture 和 harness 应位于负责它们的最低层级；更高层文档只链接到该位置。

将 scope 内的每份文档归类为 tutorial 或 reference。Tutorial 按顺序引导读者达成结果，并且每一步只引入当下需要的内容。Reference 定义查询 scope 和当前行为，不使用教学顺序。大量 tutorial 和 reference 内容应分开；任一部分较小时应标明 section 类型。

编写 tutorial 前，先在内部判断读者的起始知识，并将每个 concept 分为 beginner、intermediate 或 advanced。在依赖 concept 前建立 prerequisite，逐步提高难度，并将非必需的 advanced material 移到后续 tutorial 或 reference。

按以下顺序编写：确定文档在目录树中的位置；确定允许的细节层级；选择 tutorial 或 reference；对于 tutorial，按 prerequisite 和难度排列 concept；迁移由后代文档负责的细节；用指向 owner 的 link 替换低层解释。

## Tier 分类：每个事实只有一个归属位置

每个事实只有一个归属位置，即负责它的 tier；其他位置应链接到这里。

| Tier | 职责 | 不应包含 |
|---|---|---|
| 根目录 `AGENTS.md` | Standing order：agent 每个 session 都需要置于 context 中的规则，每条一至三行，并链接到其归属位置 | Story、worked example、情景 procedure，以及 linked home 已有内容的重复表述 |
| 子目录 `AGENTS.md`（`packages/`、`docs/`、`.agents/notes/`） | 该子目录特有的 order | 根文件已经包含的 repo-wide rule |
| [architecture.md](architecture.md) | 有序 map：composition、core package、loop、seam 和 extension point；修改 `packages/` 前必须阅读 | Type definition（→ subsystem）、逐 package 细节（→ package README）、decision rationale（→ Agent Note）、implementation-status annotation |
| [subsystems/](subsystems/README.md) | 每个 subsystem 一个 reference page：type definition、semantic 和生成的 Cordis API | Behavior narration（→ architecture.md） |
| [Agent Notes](../.agents/notes/README.md) | Active decision record：原因、放弃的方案和必需验证；`implemented/` note 使用现在时描述已经发布的 reality | Decision 发布后的 migration plan、acceptance-task checklist、fixture walkthrough 和 spec-speak（“should…”）；archived note 是冻结历史，绝不是当前 authority |
| [postmortem/](postmortem/README.md) | Incident story——唯一允许 war-story narrative 的 tier | — |
| [Persistence history](persistence-changes/README.md) 和 [format reference](persistence-changes/historical-formats/README.md) | Type-change acknowledgement、release comparison 和完整 historical format schema | 仅涉及 behavior 的变更；当前 runtime contract |
| [cookbook/](cookbook/adding-a-package.md) | 包含编号 verify step 的分步 how-to | Design rationale（→ 每份 guide 所链接的 Agent Note） |
| [user/](user/index.md) | 由 documentation website 发布的面向 product 的 guide | 生成的 reference table、contributor procedure、decision history |
| Package README | 每个 package 的 contract：config、semantic、limitation、extension point 和 [Model Experience](cookbook/adding-a-package.md#4-write-the-package-readme) | JSDoc 重述、generated catalog 重述（event/tool table）、其他 package 的 concern |
| [development.md](development.md) | Contributor setup、日常 workflow 和 CI summary；属于 [i18n contract](i18n/README.md) 下的 bilingual pair | Runtime/version rationale（→ Agent Note）、会与 `package.json` script 漂移的逐项 check list |
| Generated reference：[subsystems/](subsystems/README.md) 中每页的 `cordis-surface` region、[Cordis core API + inherited tier](cordis-api/context.md)、[tool-catalog](tool-catalog.md)、[config-catalog](config-catalog.md)、[persistence-catalog](persistence-catalog.md)、[module-graph.md](module-graph.md) | 从 source 重新生成并由 freshness gate 检查的完整 English source；经过 review 的 Chinese counterpart 遵循 [pairing workflow](i18n/README.md#scope-and-exclusions) | 手动编辑生成的 English source 或 region；Chinese counterpart 只能通过 pairing 更新 |
| Skill（`.agents/skills/`） | 可复用 workflow 和专门的 decision standard | Product 与 runtime contract（→ docs 或 source） |

位置选择：bug → postmortem；rationale → Agent Note；procedure → cookbook；type definition → subsystem；package contract → README；standing order → 根目录 `AGENTS.md` 并附 rationale link。

## 写作规则

- **记录当前状态。** 将 history 保留在 commit、PR、Agent Note、postmortem 或 scoped persistence record 中。其他 prose 应命名当前 mechanism，而不是 change 或 stack position。一般 Session-format prose 链接到[版本/状态 authority](session-format-status.md)；只有 version-specific contract、example 或 evidence 才保留数字。
- **应用 Agent Note 创建标准。** Mechanical/local edit 可以豁免，包括局部 UI change；保持已有 owner note 准确（[scope](../.agents/notes/README.md#when-to-write-one)）。
- **每个 paragraph 只占一个物理行**（`verify-md-wrap`）：使用 editor soft-wrap。Code block、table 和 list structure 保持原有格式；code comment 不超过 linter column limit。
- **Fenced `ts` block 必须能够编译**（`doc-typecheck`）；粘贴的 type declaration 及其原始 JSDoc 使用 ` ```ts type-equiv `，移除 body 的 public class declaration 使用 ` ```ts public-api `；两者都要在 manifest 中注册，避免发生 drift（[mechanics](development.md#documenting-types-verbatim-ts-type-equiv)）。
- **重塑已记录 type 的同一次变更必须更新负责它的 [subsystems page](subsystems/README.md)。** `verify-type-equiv` 能捕获已经 drift 的 paste，不能捕获从未记录的新 type；type 记录在声明它的 package group page 上（[page scoping](../.agents/notes/implemented/process/2026-08-03-package-anchored-subsystem-pages.md)）。
- **Pair 必须同步更新**：[terminology-guided](i18n/terminology.md) 的 single-pass active-agent 工作会重新放置 first-use annotation、保留未修改 prose 并重新记录；`dsh-translate-docs` 仍只能由用户调用（[contract](i18n/README.md)）。
- **Comment 和 JSDoc 陈述完整 contract，而不是 reasoning transcript。** 保留 behavior、failure、timing、ownership、modality、exception、consequence 和非显然 orientation；删除 narration、test walkthrough、review analysis 和 code restatement。保留局部 contract 并链接其 rationale。详细规则见 [dsh-prose-standard](../.agents/skills/dsh-prose-standard/SKILL.md)。
- 直接写作：明确 actor 和 fact（[decision](../.agents/notes/implemented/process/2026-08-09-concrete-prose-names-actors-and-recorded-facts.md)）。`seam` 只用于已定义的 capability。明确命名具体 check、type、API、operation 或 behavior，不要使用比喻性的 “gate”、“vocabulary” 或 “surface”。

## Wordcount budget

[scripts/doc-budgets.manifest.json](../scripts/doc-budgets.manifest.json) 设置 standing-doc ceiling；`pnpm run verify-doc-budgets` 会拒绝超限或缺失的文件。

Gate 失败时：

1. **迁移：** 将属于其他 tier 的内容移到正确位置；必要时留下一行 link。
2. **压缩：** 缩短应保留在这里但可以更简洁的内容。
3. **提高：** 只有内容确实需要这些篇幅时才提高 ceiling；在 PR 中说明 manifest diff 的理由。过低的 ceiling 属于 budget bug。

Ceiling 是 guardrail，不是缩减目标。处于或低于 target 时，至少保留 5% headroom；高于 target 时冻结 ceiling，直到迁移或压缩使文档回到 target 以下。只有文档仍有空间时才降低 ceiling。Target：根目录 `AGENTS.md` ≤ 1,950；`architecture.md` ≤ 2,400；子目录 `AGENTS.md` ≤ 600，但 `packages/AGENTS.md` ≤ 750、本文件 ≤ 1,320；`packages/README.md` ≤ 994；另外，`cordis-primer.md` 600、`defensive-patterns.md` 550、`testing.md` 1,300、`examples/AGENTS.md` 310。未设置 budget 的 tier 由 review 约束。

## Slop checklist

在任何文档中查找以下问题；[dsh-doc](../.agents/skills/dsh-doc/SKILL.md) 会将此列表作为 audit 执行：

- 重复规则：搜索有辨识度的 phrase；只保留一个归属位置，其他地方链接到它。
- History 出现在不允许的 tier：陈述当前事实，并链接 historical owner。
- Prose 或 diagram 中出现 implementation-status annotation（“implemented!”、“future: …”）。Status 会腐化；repo layout 和 package manifest 承载它。
- 在 source 或 generator 已是 authority 时，手动重述 catalog、JSDoc，或 test、package 和 status inventory。
- Reasoning transcript：逐步 implementation narration、对显然 branch 的证明、test walkthrough 或被拒绝的局部 alternative。保留最终 contract 或 durable rationale；删除推导过程。
- 在 sibling method 旁重复 rationale，而不是在负责它的 capability 或 helper 中集中陈述一次。
- Paragraph wall：一个 paragraph 承载多个规则和括号 aside。拆分它，或将细节降到其归属位置。
- Emphasis inflation：随处使用 bold、CAPS 或 “critically” 会使所有强调失去意义。只强调会改变行为的 clause。
- `implemented/` Agent Note 中出现 spec-speak：“should”、migration plan、acceptance checklist。根据 [implemented-note instruction](../.agents/notes/implemented/AGENTS.md)，implemented Agent Note 描述已经存在的状态。

## 仓库引用

当前文件使用相对 Markdown link，历史引用使用 tag 或 PR number。`verify-md-links` 检查 local target。[Reference validation](../scripts/verify-repository-references.ts) 会拒绝 maintained file 中真实的 commit identifier 和不允许的 organization URL。
