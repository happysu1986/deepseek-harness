# AGENTS.md — Recorded-session snapshot

此目录树只包含以已 commit 的 session JSONL 作为 replay input 和 expected persisted output 的 test。非 session 的 ARIA、geometry、generator、CLI 和 unit expected output 应保留在负责它的 app、script 或 package 中；按其所属 tier 使用 `test:expected`、`test:web` 或 `test`。

每个被测试的 process 都通过 `dsh` CLI、shipped profile 和可选 scenario patch 启动。Test client 可以驱动 public protocol 或 browser interface；不得添加其他 application entrypoint、hidden CLI mode 或 executable scenario driver。

每个 scenario 拥有或显式引用一个 primary Session role 以及连续的 child role。Canonical parent filename 为 `session[.vN].jsonl`；child 为 `session.<ordinal>[.vN].jsonl`；v0 省略 `.v0`，正版本使用小写 `.vN`，并且每个 filename 都与其 header 一致。一个 directory 可以保留同一 role 的多个 generation，但 replay、record 和 refresh 选择数值最高的版本。Record 和 refresh 写入带版本名的 output，不 rename 或删除已 commit generation；经过 review 的 source-tree curation 是独立操作。只有 owner 可以 record 或 refresh 所选 role。对于普通 one-shot case，从所选 JSONL 派生 user task 和 replay script；不要在 `input.json` 中重复它们。Shared reference 只读、无环，指向 owner 所选 parent generation，并且只在另一个 interface 有意渲染同一 recorded behavior 时使用。

必须保留 historical generation 或 retired-tool recording 的 owner，要在 `snapshot.yml` 中声明确切的 `sessionFormat.version` 和闭合的 migration `coverage` name；record 和 refresh 绝不重写该 Session fixture。`retired-tools` 独立于 migration coverage 固定已经移除的 capability，包括当前 format version。没有此声明的 owner 在 record 和 refresh 时写入 current-writer output；replay 接受保留的 V3 baseline 或当前 generation，每个 owner 只选择一个 generation。[corpus policy](../scripts/session-snapshot-corpus-policy.ts) 要求 direct upgrade coverage 使用 V3 input，并显式覆盖 V0–V2 migration；仅增加 writer version 不会 refresh fixture。

已 commit session 是 normalization fixed point。用保留 typed relationship 的 token 替换 volatile identity，用 token 替换 request system prompt 和 tool schema，并且每种 header class 恰好保留一个可读 sidecar owner。绝不能仅因为任意 user 或 tool text 看起来像 identifier 就对其 redact。

只有 `snapshot.yml` 指定 source 时，adapter-local symlink 才能暴露 cross-profile prompt 或 schema sidecar；corpus gate 会解析 link 并检查声明的 target。必需的 snapshot lane 会在 macOS 和 Linux 上运行这些 alias。

Workspace seed 保持 scenario-local。会修改 workspace 的 scenario 设置 `workspace.final: true`，并在 `workspace.expected/` 下 commit 完整 result；空 result 只能使用 ignored `.empty` marker。Record 和 refresh 不重写这个独立 oracle。Model prose 和 tool-result text 不能证明 external effect。

`pnpm run test:snapshot` 以只读方式 replay。Recording 和 refresh 使用显式 snapshot script，并且 commit 前要 review 由此产生的每项 JSONL、prompt、schema、protocol、UI 和 workspace diff。
