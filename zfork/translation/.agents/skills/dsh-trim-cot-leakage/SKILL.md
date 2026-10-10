---
name: dsh-trim-cot-leakage
description: 审计或修复看起来像 reasoning transcript 泄漏的 prose 时使用——例如已失效 design-session citation（decision N）、audit item code、未 commit draft 的 §N；"used to"、"no longer"、"this cut" 等 change narration；stack/review 视角（"a later PR in this stack"、"rejected in review"）；面向 reviewer 的 justification；control-flow narration；或 comment、JSDoc、doc、Agent Note 中带 hedge 的 planning residue。
---

# 精简 Chain-of-Thought 泄漏

Chain-of-thought leakage 是从 authoring session 而非 repository 视角写成的 prose：它引用只有该 session 可见的 artifact、叙述 change 而非 state，或与已经离开的 reviewer 争论。Passage 包含 factual clause 时，修复绝不能只是删除——将每项重述为在 HEAD 独立成立，再删除周围 transcript；不含 proposition 的 passage（audit code、control-flow narration）直接删除。**必读背景：** [dsh-prose-standard](../dsh-prose-standard/SKILL.md) 负责本 skill 应用的 complete-proposition rule；[committed-artifact-citations note](../../notes/implemented/process/2026-08-09-committed-artifact-citations.md) 负责 citation rule rationale。它是指导原则，不是 script。

## 唯一测试

对每个可疑 passage 询问：**一个位于 HEAD、无法访问 session transcript、PR thread 或 uncommitted draft 的 reader，能否解析每项 reference 并验证每项 claim？** 如果不能，从 repository 视角重述 surviving fact，并删除其余内容。如果能，则无论听起来多么 historical，都不属于 leakage——但 resolvability 只通过本 skill 的门槛：在 current-state surface（README、doc、JSDoc）上，可解析的 change story 仍是 change narration，class 3 会将其路由到允许的 owner。

## Taxonomy

1. **Dead design-session citation** —— `(decision 7)`、`(audit C2)`、`design §4.7`、`plan §1.4`、phase label（`T4`、`W3`、`P-I`）、"the design ledger"、"(B ruling)"。Decision 有 committed owner 时，按 name 和 path 引用；否则删除 citation，并重述其 factual clause 使其独立成立。
2. **Stack 与 PR vantage** —— "a later PR in this stack"、"this PR adds"、"the previous commit"。陈述 shipped mechanism 或 extension point；deferred work 移到 `TODO` marker 或 issue reference。
3. **Change narration 与 version stamp** —— "used to"、"no longer"、"the old X"，以及 indexical stamp（"v1"、"this cut"、"today"、"now" 与过去 state 对比）。陈述 present behavior；fixed regression 改为 present-tense counterfactual（"without X, Y happens"），绝不写 repo history（"used to Y"）。
4. **Review choreography** —— "Rejected in review:"、"the reviewer confirmed"、draft ordinal（"v5 of this note"）、round attribution。把 surviving decision 和 rationale 保留为普通事实；删除谁在何时说过。
5. **面向 reviewer 的 justification** —— "the cast is safe — it simply…"、"this is correct because…"。为自身 correctness 辩护的 comment 面向 reviewer，不面向 maintainer。陈述使 code 安全的 invariant；code 已展示时删除 comment。
6. **Restatement 与 derivation transcript** —— control-flow narration（"first we X, then we Y"）、test walkthrough、显然 branch 的 proof。删除；只保留 non-obvious contract 或 invariant。
7. **Hedge 与 planning residue** —— "probably fine for now"、"should be enough"、没有 marker 的 deferral。提升为 `TODO`/`FIXME` 或重述 actual bound；删除 hedge。
8. **Authoring-language slip** —— prose 其余部分为 English，却含未翻译 working-language fragment（端、设计稿、`---- 私有 ----` separator），或 zh counterpart 中相反情况。翻译或删除。

## 不属于 leakage 的内容

机械 citation pass 会同时误删 durable reference 和保留 dead reference。按下列 keep rule 执行；[示例](references/examples.md) 对每项进行校准：

- **Issue reference** —— `#1470`、`TODO(name):`、"issue #N owns the follow-up" 在 HEAD 可解析；任何 surface（包括 README）都保留。不要移到 Agent Note。
- **Agent Note 和 postmortem 中 merged-PR 与 issue citation** —— 按[文档标准](../../../docs/AGENTS.md)的 change-story routing，属于允许 evidence。
- **Suppression justification** —— `oxlint-disable … -- reason`、coverage-ignore reason、empty-catch explanation 是 required prose；false reason 要修复，绝不能删除。
- **Counterfactual-present regression pin** —— "without X, Y happens"、"a naive X would…"。
- **Measured bound** —— 校准 constant 的 "(measured: 512 nests ≈ 0.15s)"；evidence word "measured" 是 load-bearing。
- **Runtime old/new state** —— "the old connection drains before the new one accepts" 描述 runtime lifecycle，不是 change history。
- **Note change-story section 中的 historical stage name** —— "the first cut shipped X" 在那里 current-state-safe；indexical stamp（"this cut"）在所有位置都禁止。
- **按设计在 repo 外解析的 external reference** —— standards section（RFC 9110 §10.1.5）、Figma frame name；§ 禁令针对 uncommitted internal draft，不针对 external standard 或拥有 § 编号的 committed doc。
- **Project voice 与 genre form** —— Project voice 的 "we"；note 的 Alternatives-considered section。

## Workflow

1. Scope 与 exclusion 遵循 [dsh-prose-standard](../dsh-prose-standard/SKILL.md)：要求 explicit scope；绝不触碰 `vendor/` 或 `.agents/notes/archived/`。Recorded fixture 和 snapshot 是 derivative，不是 prose target：只有 authorized behavior change 需要新 evidence 时，修改 owning source/scenario 并重新生成。
2. 先做 read-only audit：运行 [recall battery](references/recall-batteries.md)（带 `--hidden`，使 `.agents/` 被搜索），每个 probe 都先根据 known positive 和 near-miss negative 校准，再信任 output；然后逐个做 semantic 判断。Battery 是 probe，不是 definition——原始 purge 的每轮 review 都发现 battery 漏掉的 case，因此还要不带 pattern 阅读 scope 内 prose 最密集部分（module JSDoc、README、Agent Note）。
3. 按 surface owner-first 修复：generated catalog → 追踪每个 consumer，修复 source JSDoc 或 generator template，再重新生成所有 derivative；type-equivalence fence → 修复 source JSDoc，再重新粘贴两个双语 page（`verify-type-equiv` 固定它们）；双语 prose → 最小化更新 counterpart，并通过 [lightweight routine](../../../docs/AGENTS.md#writing-rules) 重新记录；双语 fence → 按 [dsh-doc](../dsh-doc/SKILL.md) 将 corrected verbatim block byte-for-byte 复制到两侧，再重新记录 pair；model/user-visible string → 通过 [dsh-prose-standard](../dsh-prose-standard/SKILL.md) 路由，只有 owning behavior evidence 时才修改，否则保持不变并报告 defer。
4. 删除前枚举 passage 的 proposition（prose-standard），并检查 [overcorrection trap](references/examples.md#overcorrection-traps)：避免把 obligation 翻转为 endorsement、将 hypothetical 提升为 shipped feature、删除 true fact 或丢失 evidence source。
5. 验证：重新运行 battery，预期只剩 allowed keep、本 skill 自身 directory 和 owning note 引用的 evidence；确认每个 remaining citation 在 HEAD 可解析；运行 touched surface 对应 gate（doc 使用 `doc-sync`、`verify-type-equiv`、`verify-translation-pairing`）。
