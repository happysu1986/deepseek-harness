# Review criteria

## 概要

评审文档时判断 reader 能否完成 outcome，而不是每个 template heading 是否都存在。根据 code 和 test 验证 prose，保留精确 contract，让 package README 对 consumer 有用，同时为 maintainer 提供足够 implementation detail。运行当前 repository gate；`session-persistence-jsonl` README pair 是格式参考示例。

## 目录

- [Newcomer test](#newcomer-test)
- [Evidence review](#evidence-review)
- [Package README review](#package-readme-review)
- [Reference example](#reference-example)
- [Verification](#verification)
- [Dev Note](#dev-note)

## Newcomer test

没有 repository context 的专业 engineer 在阅读三到五个 linked page 后，应能回答：product 或 feature 做什么、如何安全运行或使用、state 位于哪里、哪个 component 负责、如何失败，以及在哪里修改。如果 reader 必须检查 source 才能发现 public flow，则补回缺失 explanation。如果 reader 必须吸收 unrelated internal，则将这些 detail 移到更深层。

## Evidence review

根据最强 owner 检查每项 material statement。Package metadata 负责 name 和 entry point，public type 与 JSDoc 负责 API contract，runtime code 负责 behavior，test 负责已执行 failure path，generated catalog 负责 exhaustive inventory，active Agent Note 负责 rationale。绝不能认为旧 README、discussion 或 report 比当前 code 和 test 更权威。

对于每项 operational claim——CLI command、config snippet、default value、error message、platform difference——证据来自运行，而不是阅读。在 page 陈述 behavior 前，对当前 checkout 执行确切 command 或 mount 确切 configuration；只能引用观察到的 output、warning 和 failure。依赖 unavailable key 或 network 的 claim 要命名 verification owner，不能直接断言 behavior。对已有 page，与最新 `origin/master` 比较，并根据 code 重新验证 stale statement。

评审 install guidance 前先分类 package：`package.json` 中的 `dsh.bundle.patch` 表示 bundle（通过 `dsh plugin --profile <name> add <package>` 安装，这是该 command 唯一会作为 layer 激活的 shape）；`apply` export 或 default service export 表示 plugin（作为 `cordis.yml` row mount）；plain module API 表示 library（作为 dependency，没有 install path）。拒绝为另一种 shape 编写的 install guidance。

只有 statement 能帮助目标 reader 行动、推理或避免误用时才保留。将 rationale、history、test walkthrough、duplicate catalog 和 unrelated package detail 移到其 owner。

## Package README review

要求以下内容，但不强制统一 internal heading set：

- 可搜索 YAML metadata，包含精确 `description` 和机械推导的 `kind`（`package-group`、`package-reference`、`package-library` 或 `package-bundle`）；
- 三到五句、最多 100 个 `wc -w` 风格 word 的 English Summary，说明 subject 为 user 或 agent reader 做什么，并带有 linked Table of Contents；
- Controlled English，包含 explicit actor、stable term、direct verb、拆开的 instruction 与 condition，以及不变的 modality；
- 何时选择或避免该 package；
- 存在时提供最小安全 configuration 或 usage path——bundle 使用已验证 `dsh plugin` install path，library 使用 consumer entry point；绝不给不支持的 shape 添加 profile-install guidance；
- 与 consumer 相关的 observable behavior、failure、durability、security 和 performance limit；
- 无法从 public type 低成本恢复的 developer-facing ownership 与 concept-level data/lifecycle design——整体 design、architecture、概略 dataflow——并链接 code 获取精确 detail；
- Package policy 要求的 canonical Model Experience 和 Known Limitations section；
- Adjacent doc 确实有帮助时，提供面向 newcomer 的 Further Exploration；
- 最后的 non-authoritative Dev Note，作为 partial idea、scratch 和 undecided direction 的唯一归属位置。

不得复述 JSDoc 或 generated catalog。链接 owner，只解释局部所需 decision 或 relationship。Package Summary 描述 reader-visible capability，不描述 Cordis role、registration 或 internal component；除非 reader 在 configuration、command 或 public API 中直接使用，否则省略 source identifier。拒绝叙述 internal（function subject、event stream、data flow）的 user-facing section，以及枚举 API 而非解释 concept 的 fold。

## Reference example

`session-persistence-jsonl` README pair（[English](../../../../packages/session/session-persistence-jsonl/README.md)、[Chinese](../../../../packages/session/session-persistence-jsonl/README.zh.md)）展示 production 格式：由 `kind` 选择本 package-reference standard 的可搜索 YAML、四句 Summary、linked Table of Contents、覆盖 selection、configuration、layout、durability 和 reading 的 user-facing use section、developer-facing design 与 storage detail 的 GitHub-native `<details>` fold、Further Exploration、canonical Model Experience 和 Known Limitations section，以及最后的 Dev Note。Package README 和 cross-package page 以其 structure、evidence standard 和双语 alignment 为模型；每项 claim 都以所属 source 和 evidence 为依据。

## Verification

迭代时运行最小 focused check，然后运行长期文档检查：

```sh
pnpm run test:docs
pnpm run verify-translation-pairing --write <pair>
pnpm run doc-sync
pnpm run lint
git diff --check
```

Skill 变更还要运行仓库的 skill-invocation metadata check；line-aligned pair 要比较 English/Chinese physical line count。最终 diff 重读两次：一次检查事实完整性，一次检查 brevity、navigation 和 ownership。

## Dev Note

无。
