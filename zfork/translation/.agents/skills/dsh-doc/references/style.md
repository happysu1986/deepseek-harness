# Page style

## 概要

使 DSH page 易扫描且不易误读的 page-level style 偏好：短 Summary、controlled technical English、major part 间的 `-----` separator、保持 section title 可见的 `<details>` fold，以及有节制的 emphasis。Template 是 `session-persistence-jsonl` README pair。

## 目录

- [短 Summary](#short-summary)
- [Controlled technical English](#controlled-technical-english)
- [Section separator](#section-separators)
- [Foldable content section](#foldable-content-sections)
- [Emphasis discipline](#emphasis-discipline)
- [Dev Note](#dev-note)

## 短 Summary

每个 authored page 以短 `Summary` 开头：一个 paragraph，三到五个 sentence，说明 reader 能做什么或观察什么、为何值得关注、operating model 和最重要的 boundary。Table of Contents 与各 section 承载细节；placement 和 section order 见 [structure-hierarchy.md](structure-hierarchy.md)。English package README Summary 还由 `verify-package-readme-summaries` 限制为 100 个 `wc -w` 风格 word。

## Controlled technical English

对于需要 agent、translator 或 non-native reader 解析的 English prose，执行一次受 [ASD-STE100](https://www.asd-ste100.org/) 启发的 review pass。这是 clarity discipline，不是经认证的 ASD-STE100 compliance。仓库不复制或验证该标准的 controlled dictionary。

- 命名 actor 和 action。Actor 重要时优先 active voice。
- 每个 concept 使用一个 stable term。不要为追求变化轮换 synonym。
- 优先 direct verb。有精确 verb 时，替换 nominalization 和 ambiguous phrasal verb。
- 每个 sentence 放一项 instruction。三个及以上 step 或 condition 使用 list。
- 拆分 semicolon 和长 clause chain。每个 paragraph 只讨论一个 topic。
- 移除无依据的 quality adjective 和堆叠 hedge。保留 source 中每项 fact 和 uncertainty degree。

将 instruction 20 words、description 25 words 视为 review prompt，而不是 mechanical gate。如果拆分会隐藏 condition 或 relationship，保留较长 sentence。绝不能为满足 length target 而删除或加强 `must`、`may`、`never`、timing、exception、number 或其他 contract term。[Prose standard](../../dsh-prose-standard/SKILL.md) 负责 complete-proposition rule。

## Section separator

使用单独一行的 `-----` horizontal rule 分隔 page 的 major part，前后各留一个 blank line。紧跟 paragraph 的 rule 会被解析为 Setext heading，package README 最后两个 H2 section 内的 rule 会破坏 Model Experience gate。Template 分隔：front matter → use section → folded developer section → Further Exploration → Model Experience。

## Foldable content section

使用 GitHub-native `<details>`/`<summary>` block 折叠 developer-facing detail 和最后的 Dev Note。保持 section title（H2 或 H3）及其 `<a id>` anchor 可见；只折叠 title 下的 content。Block 内在 `<summary>` 后留 blank line，所有 Markdown line 从 column 0 开始（indented content 会成为 code block），并在一个 blank line 后以 `</details>` 关闭。Fold 内的 heading、list、table 和 link 正常解析并保留 anchor。`session-persistence-jsonl` README pair 展示了 implementation section 与 Dev Note 两种 fold。

Package README 的最后两个 H2 heading 保持为 `## Model Experience` 和 `## Known Limitations and Deferred Work`。Limitations anchor 紧跟 H2，以免成为前一个 Model Experience body 的一部分。最后的 Dev Note 作为带 anchor 的 H3 放在 limitations section 下。明确免除 limitations section 的 package 可以使用 H2 Dev Note。

## Emphasis discipline

Bold 只用于改变 behavior 的 clause 或重要 comparison。Benchmark table 中，将 column header 和每行 best value 加粗，与 reference example 一致。

## Dev Note

无。
