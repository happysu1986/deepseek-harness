---
name: fork-translate-to-directory
description: 将用户明确指定的 DeepSeek Harness 文件或目录树翻译到项目根目录的 zfork/translation/ 下，保持源文件的项目相对路径，并按需处理译文中一层直接引用。用于集中存放 fork 自己维护的独立翻译副本，不用于 docs/i18n 双语文档配对流程。
---

# 将文件翻译到 `zfork/translation/`

创建翻译副本，不修改源文件。本 skill 用于按路径映射导出翻译，不属于仓库的双语文档配对流程。用户要求创建或更新原位 `foo.md` / `foo.zh.md` 配对时，改用 [`dsh-translate-docs`](../dsh-translate-docs/SKILL.md)。

## 确定范围

要求用户明确指定仓库内的一个或多个文件或目录。使用 `git rev-parse --show-toplevel` 确定仓库根目录；除非用户明确提供其他基准，否则从仓库根目录解析相对路径。

如果请求没有明确目标语言，且无法从上下文可靠判断，必须在写入前询问。不得自行扩大为全仓库翻译。拒绝仓库外路径以及已经位于 `zfork/translation/` 内的输入。

输入为目录时，处理其下所有普通文件，包括 dotfile，但排除其中嵌套的 `zfork/translation/` 输出目录。翻译文本文件；二进制文件按原始 byte 完整复制，使输出目录树保持完整。仅当 symlink 的解析目标仍位于指定源目录树内时才原样保留，否则报告并跳过。

## 映射输出路径

计算每个源文件相对于仓库根目录的路径，并写入：

```text
<repository-root>/zfork/translation/<repository-relative-source-path>
```

例如，`.agents/skills/dsh-archive-agent-notes/SKILL.md` 对应 `zfork/translation/.agents/skills/dsh-archive-agent-notes/SKILL.md`。

创建缺失的父目录，并保持文件名和扩展名不变。不得覆盖当前请求范围外源文件所对应的已有翻译。范围内输出已经存在时，更新该文件以匹配当前源文件，不得另建第二套命名规则。

## 翻译内容

忠实翻译自然语言正文，完整保留原文含义、语气强度、条件、例外和顺序。译文应符合目标语言的技术写作习惯，不得逐词转写。

除非用户明确要求本地化或下文的一层引用规则要求改写，否则以下内容保持不变：

- product、project、library、package、protocol、command、API、type、function、class、field、event、tool 和 configuration 名称；
- 已确立的专业术语和缩写，尤其是翻译后会妨碍检索或 code review 的术语；
- code span、code block、shell command、路径、URL、identifier、environment variable、version string 和字面量示例；
- Markdown link target、YAML key、frontmatter 结构、placeholder、template expression 和其他 machine-readable value。

保持文档结构、heading level、list、table、comment、frontmatter、依赖 whitespace 的 block，以及文件末尾恰好一个 newline。只有 link label、heading、comment 或 frontmatter value 属于面向读者的自然语言时才翻译。不得翻译 code fence 内的内容，包括其中的 comment；只有用户明确把 code comment 纳入范围时才例外。

## 处理一层直接引用

翻译 Markdown 文件时，检查该文件正文中的直接 Markdown link；忽略 code fence、inline code、image、外部 URL 和纯 fragment link。默认只处理这一层，不再递归处理新生成引用文档中的 link，除非用户明确要求更深层级。

相对 link target 应先相对于源文件解析，再按以下顺序处理：

1. 目标 Markdown 文件存在同目录、同 basename 的 `.zh.md` 中文 counterpart 时，不再复制或翻译该目标；将译文中的 link 改为从当前译文到该 `.zh.md` 文件的正确相对路径。
2. link 带 fragment 时，在所选中文文件中找到与英文 heading 对应的中文 heading，并使用该文件实际生成的 Markdown anchor。不得沿用不存在的英文 anchor，也不得仅凭字面翻译猜测 anchor；应考虑标点、大小写和重复 heading 的 suffix。
3. 没有 `.zh.md` counterpart 时，将目标 Markdown 文件翻译到 `zfork/translation/` 下相同项目相对路径。若该镜像译文已经存在，则更新并复用它。只要原 link 从当前译文能够正确解析到该镜像文件，就保持 target 不变；否则改成正确的相对路径。
4. 目标是已经位于当前翻译范围内的镜像文件时，复用该输出，不另建副本。例如 `.agents/skills/dsh-pre-push-checks/SKILL.md` 已映射到 `zfork/translation/.agents/skills/dsh-pre-push-checks/SKILL.md`，从 `zfork/translation/AGENTS.md` 引用时只需确保相对路径确实指向它。
5. 本地非 Markdown target 不生成翻译副本；将 link 改为从当前译文到原文件的正确相对路径。不存在或无法解析的 target 必须报告，不得猜测。

例如，源文件 `AGENTS.md` 的独立翻译位于 `zfork/translation/AGENTS.md` 时：若 `docs/architecture.zh.md` 已存在，原 `docs/architecture.md#application-launch` 应指向从该译文可达的 `../../docs/architecture.zh.md`，fragment 则使用中文文档中“应用启动”heading 的实际 anchor；若 `docs/AGENTS.md` 没有中文 counterpart，则生成 `zfork/translation/docs/AGENTS.md`，原相对 target `docs/AGENTS.md` 仍然正确。

## 验证结果

比较完整的源文件与输出文件清单，确保每个请求范围内的普通文件都有且仅有一个映射输出。确认二进制副本 byte-identical、除一层引用规则外的受保护字面量和 link target 未改变、所有改写后的本地 link 与 fragment 均可解析、格式仍然有效，并且源文件未被修改。对纳入 Git 的输出运行 `git diff --check`，最后报告已翻译路径、复用的 `.zh.md` counterpart、已复制的二进制路径、跳过的 symlink，以及需要用户复核的术语决定。
