---
name: dsh-prose-standard
description: 在 deepseek-harness 仓库中编写、评审、恢复、精简或审计 prose 时使用，包括判断 Markdown、JSDoc、code/test comment、prompt、description、diagnostic 和 CLI/UI string 中何处需要文档或 comment。
---

# DeepSeek Harness Prose Standard

写下足以保留 contract 的内容，再删除 reasoning transcript、重复和装饰。Contract 是 caller、callee、implementer、producer 或 consumer 所依赖的 obligation、invariant、precondition、postcondition 或 compatibility promise。本 skill 负责编辑判断和必需 prose coverage；placement、budget、双语 pair 和文档 gate 使用 [dsh-doc](../dsh-doc/SKILL.md)，查找并修复 reasoning-transcript leakage 使用 [dsh-trim-cot-leakage](../dsh-trim-cot-leakage/SKILL.md)。它是指导原则，不是 script。

将 `contract`、`boundary`、`shape`、`surface`、`seam`、`gate` 和 `vocabulary` 视为使用前要检查的 term，不是 banned word。先问 exact rule、API、field set、type、validation、timing point、component split 或 failure 是否能更准确地表达事实。Term 能命名确切 technical subject 时保留，包括 caller/callee contract 和 security/process boundary。

Comment 描述 code 无法表达的非显然 contract 或 rationale；不复述 code 已经显然表达的内容。

## Input 与排除项

要求显式 `scope`。缺失时报告所需 input 并停止；不得推断 repository-wide scope 或开始访谈。

接受 `mode: automatic | interactive`；默认为 `automatic`。只有用户明确要求提问或校准时才进入 interactive mode。

`mode` 控制提问，不控制 write authority。Review 和 audit task 只报告 finding，不编辑；明确要求 write、fix 或 trim 时应用清晰变更。

即使请求 scope 是整个仓库，也始终从 discovery、review 和 edit 中排除 `vendor/`。不得跟随 symlink 进入其中。Exclusion glob 放在 inclusion glob 之后，避免之后的 include 重新纳入，例如让 ripgrep command 以 `--glob '!vendor/**'` 结束，并为 Git command 提供显式 `:(exclude)vendor/**` pathspec。请求 scope 只包含 `vendor/` 时，报告没有 eligible file。

Prose review 和 edit 还要排除 `.agents/notes/archived/`。Archived Agent Note 是冻结 snapshot；只有为理解 historical inbound citation 时才检查确切 target，绝不更新其 prose 或 outbound link。

将 generated catalog、snapshot 和 fixture 视为 derivative。编辑前追踪所有 consumer：source JSDoc 也可能生成 model-visible catalog。先编辑 owning source 或 scenario，再重新生成每个 derivative。Generator 从 owner prose 提取 summary 时，使 extracted sentence 对该 surface 自身完整。Bilingual pair 没有永久 owner：任一语言都可以是一次更新的 authored side。遵循 [lightweight routine path](../../../docs/AGENTS.md#writing-rules)，最小化更新 counterpart，并重新记录 pair。

## 保留完整 proposition

编辑前识别 passage 中每个 proposition。保留每项相关内容：

- actor 与 action；
- condition、timing 和 ordering；
- must、may、never 等 modality；
- negative guarantee 与 exception；
- ownership、side effect、failure mode 和 consequence。

只有每个 factual clause 都保留且结果更清楚时，才删除 adjective、重复和 narration。Word count 更小本身不代表改进。

在使用位置保留完整 local contract：caller 或 maintainer 在此需要的 behavior、failure、ownership 和 consequence。Architecture、rationale、algorithm、history 或 extended example 要积极链接所属 document。一个 explanation 只有一个归属位置；essential contract fact 可以在局部重复。

省略 non-obvious rationale 可能导致误用或错误简化时保留。否则陈述 consequence，并链接 rationale owner。

## 按 prose 位置要求 coverage

这不是单向缩短。Code、type 和 structure 未表达以下 required contract 时，新增或恢复 prose。事实在局部已经显然时，不要添加 comment。

- **Public JSDoc：** 记录 caller-visible return distinction、throw/rejection、side effect、ownership、timing、cancellation 和 durability。
- **Internal comment：** 说明 non-local structure 和明显复杂的 local structure，包括 invariant、race ordering、ownership、security boundary 和 surprising failure behavior。删除 control-flow narration 和 code restatement。
- **Module comment：** 陈述 module role、dependency、responsibility 和非显然 architecture choice；将 architecture choice 链接到所属 explanation。
- **Test：** 只解释非显然 test design——为何需要 fixture、assertion、platform accommodation、real entry path 或 indirect observation。删除 walkthrough 和 inventory。
- **Cookbook：** 包含 prerequisite、required action、real entry path、observable verification 和简洁 warning。
- **README：** 包含 consumer contract：configuration、semantic、failure、limitation、extension point 和 model-visible effect。引用 package 所有的 stable model-visible text；链接 generated catalog 和 cross-package owner。保留 durable gap 和 maintainer trap，不保留普通 cleanup inventory。遵循 [package README requirement](../../../docs/cookbook/adding-a-package.md#4-write-the-package-readme)。
- **Agent Note：** 保留 unique rationale、mechanism、alternative、consequence、shipped verification evidence 和具名 coverage gap。Implemented Agent Note 以 present tense 陈述 shipped reality；删除 planning checklist，不删除固定 decision 的 evidence。
- **Postmortem：** 保留 incident sequence、evidence、causal chain、impact 和 prevention。删除不能建立 causality 的重复 persuasion 或 implementation detail。
- **Skill 与 agent instruction：** 陈述 behavioral guardrail 和显式 scope limitation，例如「guidance, not a script/checklist」。保持 workflow 简洁，并链接 source of truth。
- **Example 与 configuration comment：** 解释 access limit、非显然 wiring/load order、security stance、replay behavior、exception 和 likely misuse。不要叙述 configuration 已经展示的 entry。
- **Prompt 与 visible string：** 将 wording 视为 behavior。Client UI copy 属于 typed locale dictionary，并作为显式 localized prop 到达 Cordis-free primitive；一起检查 text、accessibility name、tooltip、placeholder 和 format template，再运行 `verify-client-ui-i18n`。Model-visible text 更新所属 runnable snapshot 和仓库要求的 GUI evidence。Authorized scope 没有 owning scenario 时，保持 wording 不变并报告 defer；不得静默纳入 prose-only edit。
- **Diagnostic：** 在非显然时命名 failing subject 或 path、violated rule 和 correction。删除 internal execution narration。

保留可搜索 mechanism name，以及有意义的 modal、temporal 或 negative emphasis。只 normalize decorative emphasis。

## Workflow

1. 确认 scope、mode、current branch 或 PR base，以及适用 `AGENTS.md` file。不得检查 unrelated branch。
2. 判断 passage 前阅读[文档标准](../../../docs/AGENTS.md)和 owning code/document。需要校准或遇到不熟悉 case 时阅读[精炼示例](references/examples.md)。
3. 检查 requested scope，而不只看 largest file。使用 search 和 word count 找 candidate，再做 semantic 判断。
4. 将 candidate 分类为 keep、add、trim、restore、restructure 或 defer。Task 授权 edit 时只应用清晰变更；不得为了满足 deletion target 凭空制造 edit。
5. 先更新 owner，再更新 derivative artifact。了解新规则后重新检查 analogous passage。
6. 运行狭窄相关 check、documentation gate、`git diff --check` 和 visible string 的 behavior test。确认 final diff 不含 `vendor/` path；意外匹配时如实报告，不得声称 exclusion history 干净。
7. 报告 inspected scope、clear change、deliberate keep、deferred case 和实际运行的 check。

## Borderline decision

只有至少两个版本都满足 complete-proposition rule，但在 accepted principle 间有取舍，且本 skill 尚未解决时，才是 borderline。只有一个保留 proposition 的答案时，不属于 borderline。

Automatic mode 下，有 edit 授权时应用 clear edit，并报告真正 borderline case，不提问。不得为推进工作弱化 proposition。

Interactive mode 下，根据 governing principle 归组 analogous passage。提供两到三个 viable version，推荐其一，并说明 factual 或 structural difference。不得提供 inferior distractor。使用用户要求的 channel；通过 inline comment 校准 PR 时，将 recommended provisional version 放在 diff 中，把 alternative 附到准确 line。

用户决定后，将 principle 与 version 提炼到[示例](references/examples.md)，不带 PR history 或 reviewer narration，并应用到 scope 内每个 analogous passage。
