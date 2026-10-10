# Template：package-library

此 template 用于没有 plugin surface 的 package：其 entry export plain module API，且不向 composition 注册任何内容。例如 `boot/app-boot`、`util/*`、`sdk/protocol`、`typert/generator`。`boot/app-boot` README pair 是完整示例。

Library README 与 package reference 有三点不同：没有「install into a profile」指导（library 是 dependency，不是 layer）；没有 mount configuration（不存在 `cordis.yml` row）；Model Experience section 只使用 gate 分配的 audited form（多数 library 为 `none` 或 `indirect`）。

## Frontmatter

```yaml
---
description: "What the library lets a caller build, in one or two concrete sentences with the consuming packages or searchable domain terms."
kind: "package-library"
---
```

## Skeleton

```markdown
# @deepseek-ai/dsh-<name>

English | [中文](README.zh.md)

## Summary

Three to five sentences and at most 100 `wc -w`-style words: what a caller can DO with the library, who consumes it, the smallest entry point, and the main boundary. Apply the [Summary voice rules](../SKILL.md#voice-rules).

## Table of Contents

- [Use this package](#use-this-package)
- [Understand the implementation](#understand-the-implementation)
- [Further Exploration](#further-exploration)
- [Model Experience](#model-experience)
- [Known Limitations and Deferred Work](#known-limitations-and-deferred-work)
- [Dev Note](#dev-note)

-----

<a id="use-this-package"></a>
## Use this package

### When to use it

Name the consuming call sites (which bins, packages, or runtimes import it) and when a caller should reach for it instead of a plugin.

### Entry point

The smallest import-plus-call that works, in a `text` or `ts` fence, followed by what success and failure look like. Link the owning contracts in `src/index.ts` for exact detail instead of restating them.

-----

<a id="understand-the-implementation"></a>
## Understand the implementation

<details>
<summary>Implementation internals — click to expand</summary>

Design notes and a source-map table. No API catalogs.

</details>

-----

<a id="further-exploration"></a>
## Further Exploration

Adjacent pages, closest prerequisite first.

-----

<a id="model-experience"></a>
## Model Experience

Only the form the verify-package-readme-model-experience gate assigns this package (`none`, `indirect`, or the canonical blocks). A library never invents model effects it does not have.

## Known Limitations and Deferred Work

<a id="known-limitations-and-deferred-work"></a>

Current package constraints as top-level bullets; allowlist the package in scripts/verify-package-readme-limitations.ts when none exist.

<a id="dev-note"></a>
### Dev Note

<details>
<summary>Working context for maintainers — click to expand</summary>

None.

</details>
```

## 规则

- **根据 entry 分类，而不是 folder。** 选择此 template 前阅读 `src/index.ts`：`export default` service class 或 `apply` export 使 package 成为 `package-reference`，`package.json` 中的 `dsh.bundle.patch` 使其成为 `package-bundle`。不含这些项的 plain module API 才是 library。
- **绝不编写 profile-install guidance。** `dsh plugin --profile <name> add <package>` 会安装任意 npm dependency，但只为声明 `dsh.bundle` 的 package 激活 profile layer；对于 library，它至多是 no-op dependency，绝不能作为 install path 出现。
- 编辑 pair 后重新运行 `pnpm run verify-translation-pairing --write packages/<group>/<pkg>/README.md`。
