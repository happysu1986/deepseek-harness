---
name: dsh-doc
description: 使用 audience-first hierarchy、kind-mapped YAML metadata、双语 line alignment、summary/contents navigation、从 user 到 developer 逐步深入的细节、实际执行操作的 fact-check 和仓库 validation，创建、重构、评审、审计或迁移 DeepSeek Harness Markdown 文档、package README 与文档网站。用于新增或修订 DSH 文档、组织 docs tree、审计文档质量与 budget、发布 website page，以及修改双语文档结构。
---

# DeepSeek Harness 文档

## 概要

DeepSeek Harness 文档标准：使每个 page 都可搜索、新人可读，并且足够精确，可供 agent 与 maintainer 使用；同时让文档网站保持为经过测试的 repository Markdown projection。首先应用仓库 `AGENTS.md` 文件和实际执行的 gate，再使用本 workflow 处理 kind-mapped metadata、progressive detail、line-aligned bilingual page、corpus audit 和 website publication。每个事实只保留一个 owner：source、test、generated catalog、package README、guide、Agent Note 和 scratch 各自保存其类别的 truth。`session-persistence-jsonl` README pair 是格式参考示例。

## 目录

- [Workflow](#workflow)
- [Fact-check procedure：测试，不要假设](#fact-check-procedure-test-do-not-assume)
- [Kind system 与 template](#kind-system-and-templates)
- [Voice rule](#voice-rules)
- [Quality criteria](#quality-criteria)
- [审计 corpus](#audit-the-corpus)
- [Wordcount budget](#wordcount-budgets)
- [Website publication](#website-publication)
- [详细 reference](#detailed-references)
- [Validation](#validation)
- [Dev Note](#dev-note)

## Workflow

对每个请求 scope 遵循以下顺序。保持常见 reader path 简短，但不得仅为减少字数而删除 failure、ownership、limitation 或其他 required contract。

1. 阅读根目录及更具体的 `AGENTS.md`、[文档标准](../../../docs/AGENTS.md)、目标 page、其 source/test、navigation owner 和 bilingual record。
2. 按一个主要任务与 reader 对 page 分类：product quick start、user task guide、contributor tutorial、architecture overview、package/subsystem reference、generated reference、agent instruction、decision record 或 scratch。
3. 将 page 放在最近的 owner 处。Package contract 放在 package code 旁；跨 package 的 learning、user、developer、architecture、discussion 和会过期的 scratch material 放在 `docs/`。
4. 写细节前，定义 reader 的 starting state、observable outcome、likely failure、recovery path 和下一层有用深度。
5. 新增或修改 YAML metadata——分配与该文档任务 template 对应的 `kind`——然后在 document type 允许时依次编写 `Summary`、`Table of Contents`、user-facing content、developer-facing content、可选 `Further Exploration` 和最后的 `Dev Note`。
6. 在同一 pass 更新双语 counterpart。保持 heading、list、table、code、link、frontmatter layout 和 physical line count 对齐。
7. 根据 code、test、generator、package metadata 或 current decision owner 验证每项 claim，并按下述 fact-check procedure 实际运行 page 指示的 operation。先更新 owner，再更新 derivative artifact。
8. 运行 focused check，再运行 `pnpm run test:docs`、`pnpm run doc-sync`、`pnpm run lint` 和 `git diff --check`；完整重读 diff，先检查 correctness，再检查 brevity 与 repository fit。

## Fact-check procedure：测试，不要假设

文档陈述 product 当前 behavior；operation claim 唯一可接受的证据是实际运行过它。每份新文档，以及每个声称某种 operation、command、default、error 或 platform difference 的新 paragraph，都必须遵循此 procedure。

1. **编写 install guidance 前先分类 subject。** 阅读事实，不要根据 folder name 判断：检查 `package.json` 是否声明 `dsh.bundle.patch`，检查 entry file 中的 plugin shape（`apply` export 或 default service export 是 plugin；plain module API 是 library）。Bundle 通过 `dsh plugin --profile <name> add <package>` 安装，并且只有这种 package shape 会让该命令激活 profile layer；plugin 作为 `cordis.yml` row mount；library 是 dependency，没有独立 install path。具有特殊 status（library、bundle）的 package 使用自己的 README template——绝不能套用带有不适用 install guidance 的 plugin README。
2. **在当前 checkout 上运行每个声称的 operation。** 完全按照文档将展示的方式执行每条 CLI command、config snippet 及 profile 或 patch example；只记录观察到的内容，包括确切 output、warning 和 failure mode。如果 claim 依赖当前没有的 key 或 network，明确说明并命名 verification owner，不要断言 behavior。
3. **删除无法复现的内容。** 绝不能从 memory、analogy 或相邻 package README 搬用 command、field、default value 或 behavior。Claim 无法复现时，修复 claim，而不是 test。
4. **根据最新 master 检查旧 doc。** 修改已有 page 前，执行 `git fetch origin` 并将相关 section 与 `origin/master` 比较；pairing sidecar 可以恢复任一侧最后确认的 text。Master 上的 stale statement 仍然错误：根据 code 修正，不要根据旧 prose 修正。
5. **每次编辑后重新记录 pair。** 每次 paired edit 都重新运行 `pnpm run verify-translation-pairing --write <pair>`，使 sidecar 跟踪已确认 pair。

## Kind system 与 template

Frontmatter 的 `kind` field 只选择一个 README template。[Metadata reference](references/metadata-links-i18n.md#the-kind-system) 中的每个 kind 都映射到 [`templates/`](templates/) 中的一个 template file，每个 template 也只支持一个 kind；文档检查根据相同 mechanical fact 推导 expected kind。

- `package-group` → [templates/package-group.md](templates/package-group.md)：group map（`packages/README.md`、`packages/<group>/README.md`）——介绍 family，映射其 direct package，并链接 package-owned detail。
- `package-reference` → [templates/package-reference.md](templates/package-reference.md)：Cordis plugin 或 service package——mount configuration、config table、折叠 implementation、采用 gate-owned 形式的 Model Experience 和 Known Limitations。
- `package-library` → [templates/package-library.md](templates/package-library.md)：没有 plugin surface 的 package——consumer entry point、没有 profile-install path、没有 mount configuration。
- `package-bundle` → [templates/package-bundle.md](templates/package-bundle.md)：声明 `dsh.bundle.patch` 的 package——经过验证的 `dsh plugin` install path、layer semantic 和 patch document。
- `persistence-change` → [templates/persistence-change.md](templates/persistence-change.md)：`docs/persistence-changes/` 中带日期的 record——使用每个 root 的 predecessor 和 generated after schema 确认检测到的 type transition。
- `persistence-release` → [templates/persistence-release.md](templates/persistence-release.md)：`docs/persistence-changes/releases/` 中固定的 tag——比较重建的 historical type，但不声称 compatibility acknowledgement。
- `persistence-format` → [templates/persistence-format.md](templates/persistence-format.md)：`docs/persistence-changes/historical-formats/` 中包含完整 schema 的 historical Session format checkpoint；current writer 使用已有 catalog。

写作前打开 template 并遵循其 skeleton 和 rule；它说明 kind 是什么、page 如何组织，以及每个 section 必须完成哪些 fact check。只有同时提供不同的 template file、有文档记录的 repository position 或 declared owner，以及把 document 映射到该 kind 的 focused check 时，才能添加新 kind。

## Voice rule

以下规则决定 section 可以说什么。它们适用于每个 authored human-facing page，对 package README 尤其严格。

- **Summary 说明 subject 能做什么。** 开头 `Summary` 和 user-facing section 描述 user 或 agent 可以用 subject 做什么——outcome、benefit、何时选择、主要 cost——绝不描述其 role、type 或 internal identity。Package Summary 中的「是什么」只指 reader-visible capability，不指 Cordis role、registration 或 internal component。除非 reader 在 configuration、command 或 public API 中直接使用，否则省略 source identifier。"The seam registers `ctx.x` and appends `x/event` records" 是 identity narration；"you can save a note per message and it survives restarts" 才是它能做什么。
- **Developer section 负责解释，绝不罗列。** Folded implementation content 介绍整体 design concept、architecture 和概略 dataflow——足以理解 package 如何工作——并链接 code 获取确切细节。Fold 内不得有完整 API catalog、穷尽 column list、event-payload enumeration 或 JSDoc restatement。
- **Dev Note 是唯一 slop zone。** Partial idea、scratch、undecided direction、measured artifact 和 working hypothesis 只放在最后的 Dev Note 中，并明确标为 non-authoritative。其他 section 都是 polished current-state prose。
- **只写当前 state。** 普通文档描述当前 codebase。专用 `persistence-change`、`persistence-release` 和 `persistence-format` record 保留 historical type evidence；release comparison 和 format checkpoint 描述 observed history，但不声称 compatibility。Agent Note 和 postmortem 保留自己的 historical scope。
- **使用 controlled technical English。** 当歧义可能改变 behavior 时，每个 sentence 都给出 explicit actor 和一个 main action。每个 concept 复用同一 term，优先 direct verb，拆分堆叠的 instruction 与 condition，并保留 modality 和 exception。应用 [page-style reference](references/style.md#controlled-technical-english) 中未认证、受 ASD-STE100 启发的纪律。Precision 会下降时，不要强行缩短 sentence。

## Quality criteria

评审时使用以下定义。每个 section 在 subsection 或 exhaustive detail 前先用简短 paragraph 定位内容。

- **Brief：** Common path 只包含实现 outcome 所需的 fact；穷尽 truth 保持在一个 direct link 或 detail layer 之内。
- **Intuitive：** Prerequisite 在 dependent concept 前，一个 next action 清晰可见，heading 使用 reader 会搜索的 term。
- **Friendly：** Reader 能识别 success、在行动前理解 risk、从 likely failure 恢复，并选择是否继续深入。
- **Accurate：** 每个 durable claim 都有一个 owner，以及与 risk 相称的 verification path。
- **Agent-readable：** Metadata、stable heading、anchor、terminology、ownership 和 current/proposed status 支持 targeted retrieval，无需加载整个 corpus。
- **Newcomer-complete：** 没有 repository context 的专业 engineer 可以通过三到五个 linked page 重建相关 architecture 或 feature。

不要对 exhaustive reference 应用统一 word limit。测量 entry-path length、单次 lookup 需要扫描的 unrelated material、largest section、heading count 和 page size；retrieval cost 较高时，按已有 domain owner 拆分。

## 审计 corpus

阅读而不要重新概括所属 contract：[docs/AGENTS.md](../../../docs/AGENTS.md) 负责 hierarchy、tutorial/reference form、taxonomy、budget 和 slop checklist；[.agents/notes/README.md](../../notes/README.md) 负责 Agent Note lifecycle；[docs/i18n/README.md](../../../docs/i18n/README.md) 负责双语 pairing rule；[根 AGENTS.md](../../../AGENTS.md) 负责长期命令。Audit 和 edit 排除 `.agents/notes/archived/`——archived note 是冻结历史。

对 scope 中每个人类可读 document（不含 Agent Note）应用标准的 authoring order：定位 document 并陈述其自身 subject；设定允许的 detail level，把更深解释移到所属 descendant 并链接；根据 intended use 而不是 path 将其分类为 tutorial 或 reference；对 tutorial 按 prerequisite 和 difficulty 排序 concept；拆分实质混合的 form。然后检查 placement constraint：每次编辑 paired doc 都要更新 counterpart 并 `--write` 重新记录；绝不手动编辑 generated catalog；move 必须与所有 inbound link 修复一起原子完成。

Structural pass 后，先使用成本最低的 probe 搜索 slop checklist。使用 [dsh-trim-cot-leakage](../dsh-trim-cot-leakage/SKILL.md) 查找 reasoning-transcript leakage，grep distinctive phrase 查找 duplicated rule，用 authoritative owner 替换 hand-written catalog 和 status inventory，并从 implemented Agent Note 移除 migration plan 和 future-tense spec language。使用 `pnpm run verify-doc-budgets --list` 和 word-count scan 测量 outlier；如果删除 prose 会改变 promised behavior 而不只是 explanation，先提出 behavior change（遵循 [dsh-find-simplifications](../dsh-find-simplifications/SKILL.md)）。保留每条 load-bearing rule，最好压缩成一到三行并链接其 rationale；不要为了安置 disposable reasoning 而创建新 explanation。

## Wordcount budget

`pnpm run verify-doc-budgets` 根据 [scripts/doc-budgets.manifest.json](../../../scripts/doc-budgets.manifest.json) 中的 ceiling 检查 standing document；gate 变红时，遵循 [docs/AGENTS.md](../../../docs/AGENTS.md#wordcount-budgets) 中 relocate-condense-raise 的固定顺序。Ceiling 是 guardrail，不是 reduction target：处于或低于 target 时保留至少 5% headroom；只有内容确实需要空间时才提高 ceiling，并在 PR 中说明 manifest diff。

## Website publication

Website 是经过测试的 projection，绝不是第二份副本：[website/docs.ts](../../../website/docs.ts) 是显式 public allowlist，将 canonical `docs/` source 映射到 route tree；[scripts/project-doc-site.ts](../../../scripts/project-doc-site.ts) 将它们重写到 disposable `website/.generated/` tree；VitePress 构建该 tree。Repository Markdown 始终是唯一可编辑 content source；translation 保持 sibling pair（`foo.md`、`foo.zh.md`、`foo.i18n.yaml`），绝不用 locale directory。编辑已发布 page 时只改 canonical source；新 page 添加一条 manifest entry；移动或移除时原子更新 source、manifest entry 和 inbound link；绝不编辑 `website/.generated/`、`website/.cache/` 或 `website/.dist/`。有意设置每个 `DocsPage` field，并遵循 projector link rule；field、sidebar collection 和 preview command 见 [references/website-sync.md](references/website-sync.md)。将 content 同步进 build 不代表发布；deployment 仍是独立且必须明确请求的步骤。

## 详细 reference

只加载任务所需的 reference。每个 reference 都从本文件直接链接，因此 skill 不产生深 reference chain。

- [Metadata、link 与双语 pair](references/metadata-links-i18n.md)：README frontmatter、kind system 及其推导、description semantic、repository path、line alignment 和 sidecar record。
- [Page structure 与 hierarchy](references/structure-hierarchy.md)：必需 section order、section summary、user-to-developer progression、docs tree placement、small rule file、Further Exploration 和 Dev Note ownership。
- [Page style](references/style.md)：短 Summary、`-----` section separator、foldable content section 和 emphasis discipline。
- [Review criteria](references/review.md)：newcomer test、evidence check、package README review、reference example 和 verification command。
- [Website publication](references/website-sync.md)：manifest field、projector link rule、preview 与 validation，以及 deployment separation。

[`templates/`](templates/) 中的 template 为每种 `kind` 提供一个可用 skeleton；写作前打开 document kind 指定的 template。

使用 [dsh-prose-standard](../dsh-prose-standard/SKILL.md) 判断 sentence-level contract coverage 和编辑质量。`session-persistence-jsonl` README pair（[English](../../../packages/session/session-persistence-jsonl/README.md)、[Chinese](../../../packages/session/session-persistence-jsonl/README.zh.md)）是参考示例：可搜索 YAML、Summary 与 Table of Contents、带有 folded developer section 的 user-to-developer progression、Further Exploration、canonical Model Experience 和 Known Limitations section，以及最后的 Dev Note。

## Validation

验证受影响的 format，不只是 Markdown syntax。Strong promise 需要 focused valid fixture 和证明 top-level gate 能失败的 invalid fixture。

- README metadata：解析 YAML，将 `kind` 映射到其 template 和 document standard，拒绝 `name`、`audience`、不受治理的 `tags` 和 README-local `i18n` metadata，并拒绝缺失或广告式 description。
- 双语 page：验证 structure、精确 line count、terminology、link parity 和 sidecar record。
- Tutorial：运行记录的 entry path，或命名 explicit manual verification owner。
- Generated reference：运行 deterministic freshness check，并报告 retrieval-size measure。
- Package README：运行 Summary gate；它将每个 English Summary 限制为 100 个 `wc -w` 风格 word，并让 failure 返回本 skill 和 kind template；运行 model-experience 与 limitation check，然后在 behavior claim 改变时运行 package-focused test；merge 关于 command 的 claim 前，重新运行 README 指示的每条 command。
- Skill：运行仓库的 skill-invocation metadata check。

在完整 `pnpm run doc-sync` 前，运行 `pnpm run test:docs` 执行快速综合文档检查（pairing、wrap、link、README gate、budget、skill metadata、Agent Note gate）。

## Dev Note

无。
