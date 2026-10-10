# Template：package-group

此 template 用于 `packages/README.md` 和每个 `packages/<group>/README.md`。Page 是一份 map：介绍 capability family，以一行 role 列出 direct package，并链接 package-owned detail。它绝不复述 package contract。

## Frontmatter

```yaml
---
description: "The <group> package group: what the packages under packages/<group>/ own, for readers choosing or navigating the family."
kind: "package-group"
---
```

## Skeleton

```markdown
# <group>/ — <one-line subject>

English | [中文](README.zh.md)

## Summary

Three to five sentences and at most 100 `wc -w`-style words: what the family provides, what a reader can DO with it, which package owns which half, and the main boundary. Apply the [Summary voice rules](../SKILL.md#voice-rules).

## Table of Contents

- [Packages](#packages)
- [Related documentation](#related-documentation)
- [Dev Note](#dev-note)

-----

<a id="packages"></a>
## Packages

One short orienting sentence, then the package map:

| Package | Role |
|---|---|
| [`<pkg>`](<pkg>/README.md) | One-line role: what it contributes |

<a id="related-documentation"></a>
## Related documentation

- [Adjacent owner](../../<path>.md) — what it adds to this family.

<a id="dev-note"></a>
## Dev Note

None.
```

## 规则

- 每个 direct package 一行；role text 陈述 package 的 contribution，绝不描述 internal。
- 只有 `ctx key`、package shape 或 npm-name 的区别能帮助 reader 在 direct package 间选择时，才添加对应 column。
- Related documentation 链接 adjacent owner（group map、subsystem page、Agent Note），每条 link 附一个短语。
- 不得添加 Model Experience 或 Known Limitations section；group map 不拥有 runtime behavior。
- 编辑 pair 后重新运行 `pnpm run verify-translation-pairing --write packages/<group>/README.md`。
