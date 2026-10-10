# Metadata、link 与双语 pair

## 概要

README metadata 是 retrieval 与 template-selection interface，不是微型报告或广告。`kind` field 只选择本 skill 中存在且映射到文档标准的一个 README template；frontmatter 不携带已由 filename convention 或 executed gate 负责的 field。双语 page 保持同等 authority、一对一 structure 和精确 physical line alignment。`*.i18n.yaml` sidecar 记录最后确认的 pair，并支持 automatic merge。Link syntax 必须在 GitHub 和文档网站正确 render，因此 repository link 使用 renderer-valid relative URL。

## 目录

- [README metadata](#readme-metadata)
- [Kind system](#the-kind-system)
- [Description quality](#description-quality)
- [Repository link 与 path mention](#repository-links-and-path-mentions)
- [双语 line alignment](#bilingual-line-alignment)
- [双语 consistency record](#bilingual-consistency-records)
- [Dev Note](#dev-note)

## README metadata

每个 authored README 以 YAML frontmatter 开头。允许 custom field，但 common field 要足够稳定，以供 search 和 indexing。

```yaml
---
description: "Example capability for users and maintainers choosing, configuring, or debugging the package."
kind: "package-reference"
---
```

Package README pair 必须包含 `description` 和 `kind`。Page title 与 package manifest 已经负责 name，而 document job 与 reader path 已表达 audience；在 frontmatter 重复任一项都不增加 retrieval value。Counterpart path 来自 sibling filename（`README.zh.md`），sidecar 负责 pair state，因此 README-local `i18n` metadata 是冗余的。在 repository-owned taxonomy 和 search consumer 证明 `tags` 比 description 与 full-text search 更有价值前，不得添加 `tags`。除非 existing owner 规定另一种拼写，否则 key 使用 lowercase 与 hyphenated 形式；不得把 volatile code inventory 复制到 frontmatter。

## Kind system

`kind` 直接选择 document template；每个 kind 恰好映射到本 skill 中存在的一个 template，并且不存在没有 kind 的 template。名为 `docs/persistence-changes/YYYY-MM-DD-slug.md` 的 file 使用 `persistence-change`；名为 `docs/persistence-changes/releases/dsh-v*.md` 的 file 使用 `persistence-release`；名为 `docs/persistence-changes/historical-formats/vN.md` 的 file 使用 `persistence-format`。它们的 Chinese sibling 使用相同 kind。Folder README 是 index，不是 record。按以下顺序机械推导 package README kind：

1. README 是 `packages/README.md` 或 `packages/<group>/README.md` → `package-group`。
2. Package manifest 声明 `dsh.bundle.patch` → `package-bundle`。
3. Package 位于 `scripts/doc-standard.spec.ts` 的 audited library registry → `package-library`。
4. 其他所有情况——service default export 或 `apply` plugin——均为 `package-reference`。

| `kind` | Repository position | Template | Standard |
|---|---|---|---|
| `package-group` | `packages/README.md`, `packages/<group>/README.md` | [package-group.md](../templates/package-group.md) | Group map：介绍 capability family，映射 direct package，解释 composition relationship，并链接 package-owned detail。 |
| `package-reference` | 带 plugin entry 的 `packages/<group>/<package>/README.md` | [package-reference.md](../templates/package-reference.md) | Package contract：遵循 [package README review standard](review.md#package-readme-review) 和 canonical [package documentation requirement](../../../../docs/cookbook/adding-a-package.md#4-write-the-package-readme)。 |
| `package-library` | 带 plain module entry 的 `packages/<group>/<package>/README.md` | [package-library.md](../templates/package-library.md) | Library contract：consumer entry point 与 boundary；没有 profile-install path 和 mount configuration。 |
| `package-bundle` | 声明 `dsh.bundle.patch` 的 `packages/<group>/<package>/README.md` | [package-bundle.md](../templates/package-bundle.md) | Installable layer：经过验证的 `dsh plugin` install path、layer semantic 和 patch document。 |
| `persistence-change` | `docs/persistence-changes/YYYY-MM-DD-slug.md` 及其 Chinese sibling | [persistence-change.md](../templates/persistence-change.md) | Historical acknowledgement：机械检测的 type change、每个 root 的 predecessor reference、compatibility decision 和 generated after schema。 |
| `persistence-release` | `docs/persistence-changes/releases/dsh-v*.md` 及其 Chinese sibling | [persistence-release.md](../templates/persistence-release.md) | Retrospective observation：固定 tag、相邻 before/after digest、实际 version constant 和 changed after schema；没有 compatibility acknowledgement。 |
| `persistence-format` | `docs/persistence-changes/historical-formats/vN.md` 及其 Chinese sibling | [persistence-format.md](../templates/persistence-format.md) | Selected historical format：source evidence、完整 root schema 与 reachable type；current writer 使用 generated catalog。 |

分配 `package-library` 或 `package-bundle` 前检查事实：读取 `package.json` 的 `dsh.bundle.patch`，以及 `src/index.ts` 的 entry shape（`apply` export 或 default service export 是 plugin；plain module API 是 library）。`dsh plugin --profile <name> add <package>` 会安装任意 npm dependency，但 profile reconcile 只会为声明 `dsh.bundle` 的 package 激活 layer；绝不能把该命令描述成 library 或 plain plugin 的 install path。文档检查从相同事实推导 expected kind，拒绝其他 value，也拒绝 `name`、`audience`、`tags` 和 README-local `i18n` metadata。只有同时提供 distinct template、无歧义 repository position 或 declared owner，以及将 document 映射到该 kind 的 focused check 时，才能添加新 kind。

## Description quality

Agent 先搜索 frontmatter `description` value，再决定加载哪些 page。每个 value 应像 Skill description 一样：说明 page 覆盖什么，以及 reader 何时应打开它。使用一到两个具体 sentence，包含可搜索 domain term，并与相邻 owner 区分。不得概括每个 section、宣称优越、重复 title、含糊宣传、保留 change history 或编写 technical status report。

好：`The shipped JSONL session-persistence backend for deployments and maintainers choosing, configuring, or debugging per-session durable logs with optional Zstandard compression.`

弱：`The best and most advanced session storage implementation with lots of optimizations.`

## Repository link 与 path mention

Link destination 保持 machine-checkable，mention 与 context 相对。当前 page 的 menu 使用 fragment-only link。External resource 使用完整 URL。

理想 internal-link model 从 repository root 命名 target，但以 `/docs/...` 开头的 Markdown URL 在 GitHub 上会解析到 repository 外，在 website projector 中保持不变，并被 `verify-md-links` 跳过。在 repository-owned resolver 支持所有 renderer 的 root path 前，Markdown link 使用当前 renderer-valid relative URL，logical path mention 则按讨论上下文写成 `docs/` 或 `packages/session/`。不得仅为看似 absolute path 而采用未经检查的 leading-slash link。

## 双语 line alignment

English 与 Simplified Chinese 保持同等 authority。Frontmatter key order、heading、blank line、paragraph、list item、table、code fence、link target 和 physical line 总数一一对应。English 侧所有 relative link 指向 `.md` target；当 Chinese counterpart 存在时，Chinese 侧指向 `.zh.md` sibling，否则回退到 `.md` target——pairing gate 将 `.md` 与 `.zh.md` target 视为同一 document。在对应 line 内自然翻译 prose；任一语言都不得 hard-wrap。Code block 保持 byte-identical，并在不改变 line structure 的情况下移动 first-use terminology annotation。

Line equality 是 structure check，不能证明含义忠实。Review 仍负责 completeness、terminology、natural language，以及每条 line 是否表达相同 proposition。

## 双语 consistency record

每个 bilingual pair 保留 `*.i18n.yaml` sidecar。`verify-translation-pairing` 使用其 Git blob hash 恢复最后确认的 text，验证 structure 和精确 line alignment，支持 automatic merge，记录 generated region，并 seal archive。任一语言变更后，使用 `pnpm run verify-translation-pairing --write <pair>` 重新记录。不得将 content hash 复制到 README frontmatter：独立 edit 会修改相同 header line，把原本可 merge 的 prose 变成 owner-file conflict。

## Dev Note

无。
