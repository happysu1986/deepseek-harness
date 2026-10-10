# Template：persistence-format

此 kind 用于 `docs/persistence-changes/historical-formats/vN.md` 及其 Chinese sibling。每个 record 描述一个显式选择、带完整 declared type 的 historical Session format checkpoint。它是 format reference，不是 release record 或 compatibility acknowledgement。Checkout writer 使用已有 current catalog。

## Frontmatter

```yaml
---
description: "The historical Session format, source checkpoint, and persistence types documented here."
kind: persistence-format
---
```

## Section

1. **Summary：** 标识 format 及其区别性 persisted field。
2. **Table of Contents：** 使用 shared bilingual anchor 链接后续 section。
3. **Source evidence：** 命名 tag 或 PR，并提供足以选择 source tree 的 context。不得把 PR intermediate tree 描述成 final merge 或 product release。Historical source 使用不带 line number 的 file path，并保持为 text，除非链接到 exact source。
4. **Format characteristics：** 解释 header、event 和 physical encoding 区别；超出 declared-type inventory 的 behavior 链接 codec owner。
5. **Declaration：** 每种语言包含一个相同 `yaml persistence-format` block，其中有 `schemaVersion: 1`、数字 `sessionFormatVersion`、只包含一个 `tag` 或数字 `pullRequest` 的 `source`，以及从每个 root key 到 captured SHA-256 digest 的 `roots` mapping。链接 sibling `vN.schema.json`。
6. **Complete schemas：** 保留一个 `persistence-format-schema:start` / `persistence-format-schema:end` comment region。Format verifier 在此 render root digest 和所有 reachable definition。
7. **Verification and limitations：** 陈述实际 extraction evidence、coverage limit 和 offline verifier。保留 historical optional field 与 opaque declaration，不声称 codec admission 或 migration safety。
8. **Dev Note：** 保留 non-authoritative working context，或写 `None.`。

[Format index](../../../../docs/persistence-changes/historical-formats/README.md) 负责 coverage 和 maintenance。`pnpm run verify-persistence-formats --write` 在刷新 generated region 与 pairing record 前验证 schema。它不能创建 historical evidence 或修复 machine data。Current writer 前进时，为每个旧 integer 保留文档；不要以 historical filename 复制 current catalog。
