# Page structure 与 hierarchy

## 概要

每个 page 在展示 operation 或 implementation 深度前，为 newcomer 提供简短入口。跨 package 的学习和工程材料位于有意设计的 `docs/` hierarchy 下，package contract 则保留在 code 旁。Small rule file 负责一项可独立搜索的 requirement，但 arbitrary fragmentation 不是目标。最后的 Dev Note 将 active working context 与上方 stable explanation 隔离。

## 目录

- [Page order](#page-order)
- [Section progression](#section-progression)
- [Documentation hierarchy](#documentation-hierarchy)
- [Small rule file](#small-rule-files)
- [Further Exploration](#further-exploration)
- [Dev Note ownership](#dev-note-ownership)
- [Dev Note](#dev-note)

## Page order

Format owner 允许时，authored human-facing page 使用以下顺序。Generated artifact 可以生成相同 entry section，Agent Note 和 postmortem 则保留仓库定义的 skeleton。

1. YAML metadata。
2. H1 title。
3. 双语 page 的 language switcher。
4. `## Summary`：三到五个 explanatory sentence，说明 reader 能做什么或观察什么、为何值得关注、主要 operating model 和最重要的 boundary。English package README Summary 保持在 gate 所有的 100-word limit 内。
5. `## Table of Contents`：链接 page 的 H2 section；保持 navigational，不写 description。
6. Stable content，从 user-facing use 排到 developer-facing design 和 operational detail。
7. 可选 `## Further Exploration`，为 newcomer 提供 adjacent subject link。
8. 最后的 `## Dev Note`，存放 non-authoritative active working context。

不得强迫 tiny machine-owned file、generated fragment 或由 executed parser 定义其他 header 的 format 套用 Summary/Table of Contents wrapper。应在 format owner 中声明例外，而不是创建 invalid output。Package README gate 要求最后两个 H2 section 是 `Model Experience` 和 `Known Limitations and Deferred Work`：将 `Further Exploration` 放在它们之前，并以 limitations H2 内最后的 `### Dev Note` 结束。

## Section progression

每个实质 H2 都先用短 paragraph 说明方向，再放 table、code 或 H3 subsection。解释 section 的 subject 与影响决策的重点；不要重复其完整内容。

Page 内按 reader depth 排列 content：

1. Basic use：何时选择 feature、required input、最短安全 example、observable success 和 likely recovery。
2. Advanced use：configuration choice、limit、operation 和 integration behavior。
3. Developer detail：值得维护的 ownership、lifecycle、data model、failure、performance、security 和 extension point。

使用 `<details>` block 折叠大量 developer detail 和最后的 Dev Note，同时保持 section title 可见（mechanic 见 [style.md](style.md)）。

Folded developer detail 按要求停留在 concept level：整体 design concept、main component architecture 和概略 dataflow——足以理解 package 如何工作——再加 source-map table 和精确 code link。它绝不能变成 exhaustive catalog：不得包含完整 API inventory、column list、event-payload enumeration 或 JSDoc restatement。只有 Dev Note 可以保存 partial idea、scratch 和 undecided direction；其他所有内容，包括 fold，都是 polished current-state prose。

将 exhaustive generated type、schema 或 catalog 放在 compact entry paragraph 和 stable index 后。一次 lookup 需要扫描 unrelated group 时，按已有 domain owner 拆分。

## Documentation hierarchy

Package-local README 用于 package contract，并留在 source 旁。跨 package Markdown 按 audience 与 learning intent 组织，不要把不相关 page 平铺在 `docs/`。

```text
docs/
  learn/
    overview/
    cordis/
    practices/
  user/
  developer/
    discussion/
  scratch/
  subsystems/
```

将此视为 target map，不是 opportunistic mass move 的许可。每次移动一个 coherent topic，原子修复所有 inbound link 和 website mapping，保留 public route 或 alias；`subsystems/` 保持扁平，因为其 page 在逻辑上平行。

`docs/scratch/` 包含必须跨 handoff 保留的 tracked、expiring discussion。每个 scratch page 命名 owner、creation date、expiry 和 promotion target。Local disposable note 保持 ignored 且 uncommitted。

## Small rule file

当一项可独立搜索的 rule、practice、example family 或 decision 有自己的 owner、change cadence、inbound link 或 validation 时，为其创建一个 small file。相关 file 放在描述性 hierarchy 下，例如 `docs/developer/code-quality/`。如果拆分会迫使 reader 打开多个 file 才能理解一项 obligation，则将紧密耦合的 rule 保持在一起。

Index page 用三到五个 sentence 说明 folder，并按用途链接 direct child。它不复述每个 child 的 rule。

## Further Exploration

此可选 section 服务完成当前 page 后想了解相邻内容的 newcomer。链接三到七个直接相关 page，按最近 prerequisite 到更深入 exploration 排序，并以短语说明每项新增内容。不得把它变成完整 site index。

## Dev Note ownership

Authored page 以 Dev Note 结束，但明确保持 non-authoritative。Active hypothesis、compatibility concern、rough alternative、progress pointer 和 unresolved question 可以放在这里；stable behavior、required limitation 和 accepted rationale 放在普通 owner 中。

Dev Note 可以映射或链接 task progress，但不能变成第二个 writable queue。工作结束时，提升 durable conclusion，将 reusable rationale 移到 Agent Note，将 incident chronology 放入 postmortem，并删除 resolved chatter。Git history 保存旧 iteration。

## Dev Note

强制的 final section 有意成为 authored page 中 polish 最少的部分，但仍遵循 lifecycle 纪律。空 Dev Note 应写 `None.`，不要累积 placeholder prose；generated 与 parser-owned format 可以通过具名例外省略。
