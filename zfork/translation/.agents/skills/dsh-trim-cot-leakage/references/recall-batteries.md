# Recall battery

这些 probe 针对 [taxonomy](../SKILL.md#taxonomy)，在 2026-08 purge 期间调校。每个 hit 都需要 semantic judgment——battery 按设计会 over-match，也天然会 under-match：purge 的每轮 review 都找到 battery 未捕获的 case，因此要同时无 pattern 阅读 scope 内 prose 最密集部分。

## 调用规则

- 添加 `--hidden --glob '!.git/**'`，使 `.agents/` 被搜索；ripgrep 默认跳过 dot-directory，而 purge 最大遗漏风险是 Agent Note。
- Exclusion 放在最后，避免之后 include 重新纳入：`--glob '!vendor/**' --glob '!node_modules/**' --glob '!.agents/notes/archived/**' --glob '!.agents/skills/dsh-trim-cot-leakage/**'`（skill 自己引用 leaked wording 作为校准），再加 scope 内 recorded fixture 和 snapshot directory。[Owning note](../../../notes/implemented/process/2026-08-09-committed-artifact-citations.md) 也会因引用 evidence 而 self-hit；将其判断为 evidence，不是 usage。
- Natural-language line 使用 `-i`，以匹配 sentence-initial capital（"This PR adds…"、"Probably fine…"）；匹配 code pattern 的第一行保持 case-sensitive——`-i` 会把 `\bT\d\b` 和 `\bP-I\b` 变成 noise。
- 限定完整 phrase。`\bthis PR\b` 必须匹配 "this PR adds"，不能匹配 "this project"、"this process" 或 "this provider"。
- Zero-hit pattern 在匹配 known positive 前不能证明任何事；noisy pattern 在拒绝 near-miss negative 前也不能证明任何事。信任 corpus result 前先校准二者。
- 将 authoring-language probe 指向相反语言 surface：在 otherwise-English Markdown 和 code comment/JSDoc 中搜索 Chinese residue，在 `*.zh.md` 中搜索 Chinese change narration。在 Chinese prose 中 generic ASCII 搜索 English residue 会因 code 和 identifier 产生太多 noise；应将 prose addition 与 counterpart 比较。

## English battery

```sh
rg -n --hidden '\(decision \d|\(audit [A-Z]\d|design §|plan §|design ledger|\(B ruling|\bP-I\b|\bW\d\b|\bT\d\b' ...
rg -n --hidden -i '\bthis PR\b|\bthis branch\b|\bthis stack\b|\blater PRs?\b|\bprevious commits?\b|\bthis commit\b' ...
rg -n --hidden -i '\bused to\b|\bno longer\b|\bpreviously\b|\bthe old\b|\bwas renamed\b|\bwas moved\b' ...
rg -n --hidden -i '\bv1\b|this cut|\bcut \d|\btoday\b|\bfor now\b|roadmap' ...
rg -n --hidden -i 'rejected in review|review round|reviewer|as of v\d' ...
rg -n --hidden -i 'probably |should be enough|should suffice|it simply|is safe —|is safe --' ...
rg -n --hidden '§\d' ...
```

## Chinese battery

```sh
# Change or review narration in Chinese counterparts.
rg -n --hidden '评审|上一?轮|旧版|老的|不再|以前|本版|遗留' --glob '*.zh.md' ...

# Chinese authoring-language slips in English Markdown.
rg -n --hidden '设计稿|评审|上一?轮|旧版|老的|不再|以前|本版|遗留|私有|(^|[^a-zA-Z])端([^a-zA-Z]|$)' --glob '*.md' --glob '!*.zh.md' ...

# Chinese authoring-language slips in English code comments and JSDoc.
rg -n --hidden '(^[[:space:]]*(//|/\*|\*)|//|/\*)[^\r\n]*(设计稿|评审|上一?轮|旧版|老的|不再|以前|本版|遗留|私有|端)' --glob '*.{ts,tsx,js,jsx,mjs,cjs,css}' ...
rg -n --hidden '#[^\r\n]*(设计稿|评审|上一?轮|旧版|老的|不再|以前|本版|遗留|私有|端)' --glob '*.py' ...
```

## Known false-positive family

以下内容在 purge 中经过判断并保留；预期会再次出现：

- **Instrumental "used to"** —— "the key used to sign requests" 中是工具性表达，不是 temporal。Temporal form 的主语描述先前 state（"colors used to come from…"）。
- **Runtime old/new** —— "the old connection drains before the new one accepts" 命名 handover 中 live object，不是 repo state。
- **Process doc 中的 "This PR"** —— 关于 PR workflow 的文档（"the PR body should…"、template、本 repo process note）可以合法使用 "PR"；禁止的是 doc 从某个 PR 视角描述 code。
- **`v1` 作为 protocol 或 path segment** —— `/v1/chat` endpoint 和 wire-format name 是 identifier，不是 version stamp。
- **带 committed owner 的 `§N`** —— External standard（RFC 9110 §10.1.5）和拥有 § 编号的 committed doc 可以按 section 引用。
- **Contrastive "actually" 与 noun "wait"** —— 普通 English，不是 hedge；既有 pattern 不 probe 它们，只有扩展 battery 使用更广 hedge pattern 时才出现。
- **Runtime "today" 与 recorded timestamp** —— 询问当前日期的 prompt/test 使用 natural time，不是 repository version stamp；recorded CLI output 保留其 voice。到达 model/user 的 wording 在修改前仍遵循 behavior-evidence rule。
- **zh prose 中的 本版本** —— 在 versioned-artifact context 中是 "this release" 的合法译法；禁止的是作为 "this cut" bare stamp 的 本版。
- **Alternatives-considered section** —— Agent Note genre slot 中的 "rejected" 是允许的归属位置，不是 review choreography。
