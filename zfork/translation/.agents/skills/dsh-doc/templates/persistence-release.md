# Template：persistence-release

此 kind 用于 `docs/persistence-changes/releases/dsh-v<version>.md` 及其 Chinese sibling。这些 record 重建 tagged persistence type，以供 human comparison 和 format validation。它们是 observation，不是 compatibility acknowledgement，也不参与 current-source history。

## Frontmatter

```yaml
---
description: "The tagged persistence types and adjacent-release differences recorded here."
kind: persistence-release
---
```

## Section

1. **Summary：** 解释与前一个可用 tag 相比的具体 type difference 或无差异，并在相关时陈述 Session writer version。
2. **Table of Contents：** 使用 shared bilingual anchor 链接后续 section。
3. **Release evidence：** 命名 source tag 和 tagged source path，再以 relative path 链接 preceding record 与 schema companion。标明 reconstruction limit。
4. **Declaration：** 只包含一个 `yaml persistence-release` block，带有 `schemaVersion: 1`、`tag`、`previous`、`sessionFormatVersion` 和 `changes`。每项 change 命名 `root`、`before` 和 `after`；null 表示 absence。两种语言包含 identical machine block。
5. **Structural changes：** 列出机械检测的 path 和 kind。Current-rule classification 仅供参考；它们不能证明 historical release safety，也不暗示 observed writer version 已增加。
6. **Verification：** 陈述已执行 reconstruction 和 validation evidence。`pnpm run verify-persistence-releases` 在不需要 source checkout、Git 或 network access 的情况下验证 committed archive。
7. **Dev Note：** 此 section 保留 non-authoritative working context，或写 `None.`。

## Snapshot 与 history rule

[Archive reference](../../../../docs/persistence-changes/releases/README.md) 负责 pinned release manifest 和 extraction limit。首个 record 包含每个 extracted root；之后的 schema companion 只包含仍然存在的 changed root 及其所有 reachable type definition。Empty change 和 snapshot 保留 type 未改变的 release。每个 record 引用紧邻的前一 manifest entry。

Index table、inventory cell 和 structural-change fact 使用 `persistence-release-index`、`persistence-release-inventory` 与 `persistence-release-changes` start/end comment marker。Human explanation 放在这些 generated region 外。`pnpm run verify-persistence-releases --write` 在刷新 factual region 和 pairing record 前验证所有 machine data；它保留 authored prose、machine declaration 和 schema file。Default verification 拒绝 stale generated fact 和缺失的 fixed index file。

保持 source `number` declaration 和 historical optional field 不变。单独报告实际 writer version constant。Historical parsing 允许 surface event 使用 optional `surfaceOp`；live persistence parser 和 acknowledgement rule 仍保持严格。绝不能把这些 retrospective record 插入 current-source acknowledgement chain，也不得为适应旧 release 而重写 committed baseline。
