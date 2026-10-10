# Template：persistence-change

此 template 用于 `docs/persistence-changes/YYYY-MM-DD-slug.md` 及其 Chinese sibling。Folder README 是 index，不是 record。Record 只确认机械检测到的 persistence-type change。其 generated JSON sibling 保留完整 after schema，因此 verification 不需要 Git baseline。

## Frontmatter

```yaml
---
description: "The persisted types changed and the compatibility decision this record explains."
kind: persistence-change
---
```

## Skeleton

Generator 填充 machine declaration、schema companion 和 consistency record，并从固定规则推导最小 version decision。通过 `--prose` JSON 提供 `en` 和 `zh` object，每个都包含 authored `summary`、`compatibility` 和 `verification` string；[cookbook](../../../../docs/cookbook/reviewing-persistence-type-changes.md) 展示完整 operation。下列 value 只演示 document field；请使用实际 generated root name、record id 和 digest。

````markdown
# Persistence change: <specific persisted change>

English | [中文](YYYY-MM-DD-slug.zh.md)

## Summary

Describe the mechanically detected change and its version decision.

## Table of Contents

- [Declaration](#declaration)
- [Compatibility](#compatibility)
- [Verification](#verification)
- [Dev Note](#dev-note)

<a id="declaration"></a>
## Declaration

```yaml persistence-change
schemaVersion: 1
id: YYYY-MM-DD-slug
baseline: false
changes:
  - root: event:example/message
    previous: YYYY-MM-DD-predecessor
    after: <generated-64-character-sha256-hex>
    decision: same-version
```

<a id="compatibility"></a>
## Compatibility

Explain the declared decision for the detected paths. For an optional addition, explain why existing records can omit it and older readers can ignore it without changing replay. For required-to-optional changes, explain how readers handle absence. For a version bump, link the adjacent migration and its verification owner.

<a id="verification"></a>
## Verification

Record the commands actually run and their outcomes. Link focused tests or preserved evidence that support the compatibility explanation.

<a id="dev-note"></a>
## Dev Note

None.
````

## 规则

- 只保留一个 `yaml persistence-change` block。English 与 Chinese block byte-identical；verifier 只解析一次 English record，再检查 counterpart。
- Filename stem 与 `id` 一致。Root 的 `previous` 标识 predecessor record；`null` 引入 root。其 `after` 匹配 generated companion schema digest；`null` 记录删除。
- 一个初始 `baseline: true` record 覆盖完整 inventory。Routine change 使用 `baseline: false` 并保留 accepted predecessor。Version-bump record 携带自身递增的 `SessionHeader.version` transition。
- 普通 authoring flow 省略 `--decision`。显式 value 表示断言 decision，不能绕过 classification 或 required header increase。Command 不提供 source change 或 validation claim。
- 只解释检测到的 persistence-type change。此 kind 允许该 transition 所需 before/after fact 和 historical evidence；behavior-only change 不在 scope 内。
- 创建 record 和更新 terminal record 会生成 catalog pair 与 consistency sidecar。评审两种语言并保留 generated companion。直接 prose edit 仍需重新记录 pairing；不得手动维护 schema node 或 digest。
- 只有 unaccepted terminal record 才使用显式 `--update`。它拒绝 baseline 和被依赖 record；`--prose` 替换 authored explanation，省略则保留。Accepted record 使用 successor。
- [Folder reference](../../../../docs/persistence-changes/README.md) 负责 compatibility rule；[cookbook](../../../../docs/cookbook/reviewing-persistence-type-changes.md) 负责 command 与 recovery。
