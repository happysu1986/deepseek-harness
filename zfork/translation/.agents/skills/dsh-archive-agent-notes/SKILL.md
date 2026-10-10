---
name: dsh-archive-agent-notes
description: 在 deepseek-harness 中新增、审计、精简、归档、恢复或评审 Agent Note 时使用；检查每个新 note 是否取代 active record，删除小型 UI 与纯 mechanical record，按未来 decision value 分类其他 implemented note，删除不再防止诱人错误的 rejected note，并应用冻结的 archived/{kind} triplet 与 manifest 规则。
---

# 归档 DeepSeek Harness Agent Note

在不抹除仍可指导工作的历史信息前提下，缩减 active decision corpus。必须根据语义判断每份 note；word count 和 age 只能用于发现，绝不能作为归档标准。

## 阅读 contract

分类前阅读 [Agent Note 规则](../../notes/README.md)、[archive 说明](../../notes/archived/AGENTS.md)以及适用的 active lifecycle 说明。使用当前 code、configuration、package doc、generated catalog、较新的 Agent Note 和 inbound link，确定某项 rationale 是否仍负责或约束任何内容。

## 新增 note 时检查 supersession

起草前应用[创建标准](../../notes/README.md#when-to-write-one)。每个新 Agent Note 都会触发一次限定范围的审计，检查覆盖相同 decision、mechanism 或 rejected alternative 的 active note。编写新 note 时，对每个完全或部分 supersession 分类：在同一 PR 中归档符合条件的 implemented triplet；保留并交叉链接部分 supersession 或仍具独立价值的 rationale；拒绝 obsolete proposal；删除不再防止 plausible mistake 的 rejected note。新 owner 吸收所有 unique proposition 时，应用 Agent Note consolidation 规则；不得把已经发现的匹配推迟到之后的 corpus audit。

## 按未来价值分类

应用以下 lifecycle-specific 结果：

- **Implemented — 删除：** 删除只描述小型 UI 调整或纯 mechanical change 的完整 triplet，并修复或移除 inbound link。Local bug fix、performance change、new capability，以及实质性的 behavior 或 ownership decision，不会仅因已经 implemented 而符合此条件。在 keep/archive 分类前先应用此标准。
- **Implemented — 保持 active：** 如果 note 的 rationale、alternative、negative guarantee、durable/wire semantic、ownership boundary、security rule 或重新引入条件很可能指导未来变更，则保留。分类对象是 decision，而不是 implementation mechanic：mechanical rename 或 type extraction 仍可能记录持久的 naming、compatibility 或 ownership rule。长度无关紧要。
- **Implemented — 归档：** 实质性的历史 decision 已经完成且不太可能指导未来工作，但其历史 rationale 仍值得保留时，将其归档。不得归档符合直接删除标准的 record。
- **Proposed — 绝不归档：** 保持 live proposal 为 active；如果不再值得推进，则以诚实理由拒绝，并满足 rejected lifecycle format。
- **Rejected — 仅作为 guardrail 保留：** 只有 losing proposal 仍是诱人且有实质意义的错误，并且 note 解释了它为何不可取时，才保留 rejection。
- **Rejected — 删除：** rejected idea 已 obsolete、被 supersede、不再 plausible，或不太可能阻止重新争论时，删除整个 triplet。修复或删除 inbound link。

不得为达成 quota 而归档。检查 scope 内的每份 note，以一项原则分类相似 group，对接近临界的 case 使用最佳判断，并在 handoff 中记录真正 borderline 的决定。

## 校准示例

以下示例确定判断门槛；word count 用于说明大小并非判断标准。

删除以下 implemented note：

- collapsed sidebar control rail — 533 words：已完成的小型 UI behavior；
- issue-policy module ownership — 已完成的 function relocation 和 import rewiring，behavior 未改变。

保留以下 implemented note：

- event-sourced sessions — 248 words：基础 authority 和 durability boundary；
- single Harness-home resolver — 596 words：跨 product ownership rule；
- project session directories — 628 words：durable storage 和 identity policy；
- parallel pre-push gates — 400 words：属于 borderline，但仍指导 gate scheduling 和 resource tuning；
- dropped image content block — 334 words：在 multimodal support 落地前保留，因为它陈述了协同重新引入条件。

对于 rejected note：

- 保留 folding the compaction package split — 426 words：合并这些 package 仍是有现实意义的诱惑；
- 删除 streaming workflow progress through tool calls — 972 words：其 ACP/UI premise 已 obsolete；
- 删除 dropping ACP terminal metadata — 362 words：后续 automation-only ACP decision 已解决该问题。

## 归档一个 implemented triplet

1. 将完整的 `foo.md`、`foo.zh.md` 和 `foo.i18n.yaml` triplet 从 `implemented/<kind>/` 移到 `archived/<kind>/`；archive path 特意不包含 `implemented`。
2. 不得修改正文。仅在两个语言文件的 `Status: implemented` 正下方插入 `Archived: YYYY-MM-DD`，使用归档日期，且两侧值相同。
3. 针对这两处仅 metadata 的修改，机械地重新记录 sidecar hash。不得翻译、重新格式化、更新事实或修复 note 内的 link。既有 title punctuation、blank-line layout 和 language-switcher wording 均保持原样，不作为归档的前置条件。
4. 搜索 active prose 的 inbound link。将其重定向到 current authority；只有明确引用历史 snapshot 时才改到 archived path；否则删除。绝不验证或修复 archived note 指向外部的 link。
5. 运行 `pnpm run verify-archived-agent-notes --write`。其 append-only mode 会先证明所有既有 seal 仍然匹配，再只添加新 triplet hash。之后运行常规 verifier。

Triplet 被 seal 后，绝不能编辑、移动、翻译、重新格式化或删除。Archived note 仍可作为 inbound-link target，但它们是历史 snapshot，不是当前 behavior 的 authority。

## 验证并报告

运行 archive verifier 的 focused test、`pnpm run verify-archived-agent-notes`、`pnpm run doc-sync`、`pnpm run lint` 和 `git diff --check`；通过 [dsh-pre-push-checks](../dsh-pre-push-checks/SKILL.md) 选择其他必要证据。

报告保留的 active implemented note、删除或归档的 implemented note、保留/删除的 rejected note、被拒绝的 proposed note（如有），以及每个真正 borderline case 的 word count 和所选结果。不得声称 archived outbound link 有效：archive verifier 有意不检查这些 link。
