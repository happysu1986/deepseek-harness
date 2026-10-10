# Template：package-reference

此 template 用于 entry 为 Cordis plugin（service default export 或 `apply` function）并 mount 在 composition 中的 package。这是 `packages/<group>/<pkg>/README.md` 的默认 template。`session-persistence-jsonl` README pair 是此 template 的完整示例。

## Frontmatter

```yaml
---
description: "What the package lets a reader choose, configure, or debug, in one or two concrete sentences with searchable domain terms."
kind: "package-reference"
---
```

## Skeleton

```markdown
# @deepseek-ai/dsh-<name>

English | [中文](README.zh.md)

## Summary

Three to five sentences and at most 100 `wc -w`-style words on what a user or agent can DO with the package: outcomes, when to choose it, main cost, most important boundary. Apply the [Summary voice rules](../SKILL.md#voice-rules); never describe its role, type, or internal identity.

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

One orienting sentence: the common path.

### When to choose it

Choose or avoid the package: one paragraph naming the deciding conditions and the fallback package.

### Minimal configuration

The smallest mount that works, as a `cordis.yml` snippet, plus the config table:

| Field | Default | Meaning |
|---|---|---|
| `<field>` | `<default>` or `required` | One-line meaning |

The generated [configuration catalog](../../../docs/config-catalog.md#deepseek-aidsh-<name>) is the exhaustive source for every accepted field.

-----

<a id="understand-the-implementation"></a>
## Understand the implementation

<details>
<summary>Implementation internals — click to expand</summary>

Design concept, component architecture, and hand-waving dataflow — enough to understand the package. A source-map table links files for exact detail. No API catalogs or JSDoc restatement.

</details>

-----

<a id="further-exploration"></a>
## Further Exploration

Three to seven adjacent pages, closest prerequisite first, one short phrase each.

-----

<a id="model-experience"></a>
## Model Experience

<per the Model Experience contract in docs/cookbook/adding-a-package.md#4-write-the-package-readme; the verify-package-readme-model-experience gate owns the required form>

## Known Limitations and Deferred Work

<a id="known-limitations-and-deferred-work"></a>

One orienting sentence, then top-level bullets naming current package constraints. Packages with none use the allowlist in scripts/verify-package-readme-limitations.ts.

<a id="dev-note"></a>
### Dev Note

<details>
<summary>Working context for maintainers — click to expand</summary>

None.

</details>
```

## 规则

- **写作前 fact-check。** 在 test composition 中 mount package，并运行 README 声称的每条 command、config field、default 和 behavior。删除无法复现的内容；链接 generated config catalog，不要复述 field。
- **Installation guidance。** Plugin package 通过 `cordis.yml` row mount。只有声明 `dsh.bundle.patch` 的 package 才通过 `dsh plugin --profile <name> add <package>` 安装成 profile layer——如果此 package 没有该声明，说明它如何在 composition 中 mount，绝不能写 `dsh plugin add`。
- **Model Experience 和 Known Limitations 归 gate 所有。** 精确匹配两个 gate 强制的 heading 和 per-package form；behavior 使 package 在 form 间移动时，在同一变更中更新 gate 的 audited list。
- 编辑 pair 后重新运行 `pnpm run verify-translation-pairing --write packages/<group>/<pkg>/README.md`。
