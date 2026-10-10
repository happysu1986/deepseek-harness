---
name: dsh-translate-docs
description: 手动运行扩展 DeepSeek Harness 双语文档 workflow，包括 generated briefing、委派 prose translation、whole-document translation 和 scoped pairing verification。
disable-model-invocation: true
user-invocable: true
---

# 翻译 DeepSeek-Harness 文档

## 调用 boundary

只有用户明确点名调用 `dsh-translate-docs` 时才运行此扩展 workflow。普通文档工作、其他 skill 或推断出的翻译需求绝不能选择或加载它；常规翻译遵循 [docs/AGENTS.md](../../../docs/AGENTS.md) 的 one-shot、one-pass 规则。

## 本 skill 的用途

**本 skill 是指导原则，不是 translation memory。** 它是保持 `foo.md ↔ foo.zh.md` pair 在两种语言中一致自然的 workflow map。两种语言 authority 相同——一次变更可以在任一侧 authored，该侧即为此次更新 source。你是 translator：下列规则规定必须满足什么，不规定具体 sentence 如何措辞——phrasing 由你判断，terminology 不可自行决定。

## 按 change type 分类——它决定后续全部流程

- **Update**（pair 已存在，一侧被编辑）：遵循 [update path](#the-update-path-briefing-driven)。它以 briefing 驱动且刻意低成本：不读取 guidance corpus，不做 git archaeology，只编辑最小 counterpart。绝不能为应用 update 而重新翻译整份 document——minimal update 会保留未变内容已经 reviewed 的 phrasing，重新翻译则会丢弃这些 review。
- **New pair**（尚无 counterpart）：遵循 [whole-document path](#the-whole-document-path-new-pairs)。
- **Deleted 或 renamed doc：** 同步删除或 rename counterpart 与 `.i18n.yaml`，否则 gate 会报告 incomplete pair。

`.agents/notes/archived/` 下的 frozen Agent Note 不属于 translation work。其完整 triplet 已由 archive verifier seal；归档后绝不能更新、重新记录或修复任一侧。

## Update path（briefing-driven）

Briefing-driven path 以很低成本达到 guidance-corpus quality；archived [briefed-updates Agent Note](../../notes/archived/process/2026-07-26-briefed-minimal-translation-updates.md) 记录 benchmark evidence。

1. **生成 briefing：** `pnpm run gen-translation-brief <any file of the pair>`（无 argument 时为所有 out-of-sync pair 生成 briefing）。Briefing 以可安全对齐的最小 granularity 映射变更——changed Markdown unit（paragraph、table row、list item、heading），然后是完整 heading section，再到完整 document——并包含 authored side 相对最后 confirmed-consistent state 的 diff、每个 changed unit 的 last-confirmed source、current source 和 current counterpart text（带 line number）、涉及的 terminology row、first-occurrence movement note，以及 binding update rule digest。
2. **只有 mechanical diff？使用 `--apply`。** Pair 中每项 change 都位于双方 byte-identical 的 code fence 内时，briefing 会说明；`pnpm run gen-translation-brief --apply <pair>` 将 edited fence 拼接到 counterpart，并在写入前验证 structure——无需 subagent 或 hand-editing。
3. **Prose diff？将 briefing（或生成 command）交给 subagent。** Briefing 是 translator 的完整 working set——subagent 不重读 guidance corpus（rule digest、terminology row 和每个 changed unit 的 three-way context 都已 inline），也不重新推导 diff。只有 briefing 留下具体且确实无法回答的 decision 时，才升级到 whole-document path source of truth——周边 text 无 precedent 的 unlisted term，或 whole-document briefing（`BOTH sides changed`，或者 unit 与 section 均无法对齐）；后者始终需要按 [translation-rules.md](../../../docs/i18n/translation-rules.md) 手动 reconcile。
4. **覆盖 diff 的最小 edit。** 保留 diff 未触及内容已经 reviewed 的 phrasing，再根据 source 逐 clause 验证 changed hunk：不添加、不遗漏；terminology 符合 inline row；code span verbatim。
5. **按 scope 记录并验证：** 先运行 `pnpm run verify-translation-pairing --write <pair>`，再运行 `pnpm run verify-translation-pairing <pair>`。`--write` 要确切命名已确认 pair——它拒绝空参数，因此 bulk re-record 必须显式 `--all`。Corpus-wide check 仍在 `doc-sync`/CI 运行；每次 update 不单独运行。

## Whole-document path（new pair）

需要从头翻译时，orchestrating agent 不亲自翻译：spawn subagent 执行翻译。Translator 先阅读下列 source of truth，再把整个 file 翻译成另一语言——长 document 按 section 处理，过程中保持每个 section structure 与 source 锁定，不要最后再修结构。

### Source of truth（阅读，不要重新概括）

- **[docs/i18n/README.md](../../../docs/i18n/README.md)** —— pairing contract：三文件 pair（`foo.md`、`foo.zh.md`、`foo.i18n.yaml`）、consistency record 两侧 blob hash、language-switcher line、scope 和 exclusion。
- **[docs/i18n/translation-rules.md](../../../docs/i18n/translation-rules.md)** —— 如何翻译：faithfulness、structure preservation、terminology discipline、typography（MUST/SHOULD level）。
- **[docs/i18n/terminology.md](../../../docs/i18n/terminology.md)** —— 双向 binding terminology table。翻译前加载，不要等某个 term 看起来不确定才加载；未注意到的 term 才最容易 drift。
- **[docs/i18n/translation-prompt.md](../../../docs/i18n/translation-prompt.md)** —— automated pipeline 校准后的 machine-consumed template。使用本 skill 的 agent 不 render 它；automated renderer 只 inject terminology table，而本 skill 与 `translation-rules.md` 对 agent-authored translation 仍有约束力。
- **[dsh-prose-standard](../dsh-prose-standard/SKILL.md)** —— required prose coverage 与编辑判断。两侧都应用，不添加或遗漏 source proposition。

### 翻译

- **Pass 1——写作，不要转置。** 阅读 semantic unit，再以最接近 [style sample](../../../docs/i18n/style-samples.md) register 的 native technical author 口吻重述。保留 required frame，不强迫逐 sentence 对应。
- **Pass 2——逐 clause 对照 source 验证。** Fidelity 在此检查，不是在写作时获得：确认没有添加或遗漏，每个 term 遵循 table，每个 code span verbatim。通过自然重写 sentence 修复，不要逐词打补丁。
- **单独阅读完成的 counterpart。** 对照 source 后，不并排查看 source，独立阅读 translated file，重写只有脱离 source 才显出 awkwardness 的 phrasing。
- File 只写 final text，绝不写 draft 或 note。
- [terminology.md](../../../docs/i18n/terminology.md) 中每个 term 都按规定 render。Chinese target 使用 Chinese 与 first-occurrence column；unlisted term 需要可引用的 Chinese OSS/vendor precedent，否则以 English 保留，并列为「待定术语」。English target 使用 English column 与 established English technical term；ambiguous source term 保留并加最短 gloss，列为 pending。绝不能 inline 发明译法。
- Pair 两侧 code block byte-identical，包括 comment。Repository-relative document link 保持相同 semantic target 和精确 query/fragment suffix：active bilingual corpus 中的 target 在 English 侧使用 `.md`，Chinese 侧使用 `.zh.md`；缺失 in-scope counterpart 是 error；corpus 外 target 保持 authored path；switcher 是 cross-locale 例外。
- Pairing gate 检查 heading depth、fenced block、table row/column count、list kind、ordered-list start、list item count、link locale 和 semantic target。Pass 2 手动验证 list/table order、noncanonical list numbering、inline code、emphasis、meaning、terminology 和 tone。

## 查找工作项

- `pnpm run verify-translation-pairing --list` 将每个 in-scope document 打印为 missing / out-of-sync / ok。Missing 和 out-of-sync row 是 contract violation；常规 check 会拒绝。
- 无 argument 的 `pnpm run gen-translation-brief` 为每个 out-of-sync pair 打印 briefing。
- PR 编辑 paired doc 时，diff 本身就是 work list：pair 任一侧发生变更，都要在同一 PR 更新 counterpart 并重新记录 pair；遗漏时 gate 变红。

## 完成 pair

1. Switcher：Chinese file 的 H1 后紧跟 `[English](foo.md) | 中文`；English file 的 H1 后紧跟 `English | [中文](foo.zh.md)`——new pair 两侧都添加。例外是 generator-owned English source 保持与 generator output byte-identical 并省略 switcher，Chinese counterpart 仍链接回去。
2. 记录 consistency：`pnpm run verify-translation-pairing --write <pair>` 重新计算并记录两侧完整 blob hash 到 `foo.i18n.yaml`。PR 中的 yaml diff 是可评审声明「我确认二者表达相同内容」——只有真正确认后才运行。
3. 普通 document 不需要 manifest entry：每个 in-scope source 都要求 pair。只有 owning policy 记录真正 generated、instructional 或 bilingual-by-construction exclusion 时，才修改 [scripts/translation-pairing.manifest.json](../../../scripts/translation-pairing.manifest.json)。
4. PR 前：touched pair 通过 scoped check；`pnpm run doc-sync`（包含 corpus-wide pairing check 及 `verify-md-wrap`/`verify-md-links`）按 [dsh-pre-push-checks](../dsh-pre-push-checks/SKILL.md) 在 PR level 运行一次，不在每个 translation task 内运行。
5. 保持 PR 可评审：说明哪些 pair 是 new，哪些是 minimally updated，并醒目列出「待定术语」。

## 回应 translation review

遵循 [code-review reporting guidance](../dsh-code-review/SKILL.md#reporting-findings)：逐项判断 comment；对于 terminology comment，要记住 terminology table 才是 contract——将 reviewer 的 rendering decision 应用到 [terminology.md](../../../docs/i18n/terminology.md)，而不只改单个 file。
