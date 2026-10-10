# AGENTS.md — Harness Package

这些 package 专用规则是 repo-wide [约定](../AGENTS.md#conventions)的补充。

- **Plugin export：** service package 默认导出其 service class；function plugin 以 named export 导出 `name` / `inject` / `Config` / `apply`，且没有 default export。混用两种形式会使 Loader 丢弃 function plugin 的 namespace（[postmortem](../docs/postmortem/0001-acp-default-export-drops-inject.md)）。
- **Optional service 使用 `ctx.get(name)`。** `ctx.<name>` 仅用于已声明的 injection；property proxy 对 topology 敏感，而 strict `ctx.get` 读取 global service store（[postmortem](../docs/postmortem/0001-acp-default-export-drops-inject.md)）。
- **产品可见 plugin 需要非 unit 的 REAL-composition test。** 手工构建的 `ctx.plugin(...)` suite 不够充分。通过 Loader 和 app/process 启动仅用于测试的 `cordis.yml`；只 mock external service 或 nondeterministic input，并断言 model-visible、durable 或 user-visible output。不要将 opt-in 放入 shipped default。[Policy](../docs/testing.md)。
- **由 initiator 所有的 private chain 先派生，再捕获。** 在 `ctx.agents.withInitiator()` 下，每次进入 orchestration 时恢复 Agent，派生 `agent.session`，并让 operation-local helper 通过 closure 捕获它。在 lifecycle、session-log、service、authority、worker/process、persistence 和 wire interface 上显式保留 `Agent` 和 `Session`；不要仅为隐藏一个 parameter 就把 leaf helper 从 `Session` 扩大为 `Context`（[rationale](../.agents/notes/implemented/architecture/2026-07-15-agent-initiator-scope.md)）。
- **一个 asynchronous operation 由一个 lifecycle controller 或 transaction 表示。** 独立的 readiness、cancellation、disposal、reservation 或 sentinel state 需要独立 owner 或 settlement point；否则应在保留 rollback、callback containment 和 quiescence 的前提下合并。
- **为所有当前 Consumer 设计 Service Definition。** Tool-schema、Loader、UI、transport 和 provider-specific behavior 留在 Consumer 或 provider 中；不要让一个 Consumer 决定 service contract（[capability-seam rationale](../.agents/notes/implemented/architecture/2026-06-13-capability-seams.md)）。反向坏味道：public service method 只有一个 internal caller——应改为传递 private capability closure（`RunCodeBridgeOptions`）。
- **要求当前 owner 和 need。** 每个 abstraction、state machine、option、defensive copy 和 compatibility path 都应对应当前 contract 或 production consumer，并将 behavior 保留在负责它的 plugin 或 service 中。
- **Public choice 需要 evidence。** Configurability 不能证明 unsupported default、public operation set、format 或引入的 external concept 合理。使用 current-consumer evidence 或相关 prior art；否则要求 explicit value，或推迟该 choice。
- **从 model 的视角编写 model-facing contract。** Prompt、tool schema、result 和 diagnostic 只包含与任务相关的 concept，不包含 UI、transport 或 implementation vocabulary。稳定的 model-visible text 使用逐字 snapshot 固定，dynamic behavior 使用 snapshot 或 end-to-end coverage 固定。
- **在做出 decision 的 operation 中强制执行它。** 当 direct 或 alternate caller 可以绕过时，schema omission、prompt filtering、facade、wrapper 和 listener order 都不算 enforcement；通过 executor 测试 denial。
- **只在 commit point 发布 state。** Operation 成功后才 emit notification 并更新 derived state；cache、prompt、UI echo、replay 和 query view 从同一个 authoritative source 派生。
- **对完整 result 应用 bound。** 在能够获知完整 emitted 或 retained value（包括 wrapper 和 metadata）的位置强制 byte、token、item 和 time limit；测试 tiny limit、exact limit、oversized single chunk 和 multibyte byte limit。
- **Registry contribution 要证明 disposal**，使用 [testing policy](../docs/testing.md) 要求的 HMR-safety test：dispose fiber，并观察 contribution 被移除。
- **Spec 在 forked worker 中与其他 gate process 并发运行。** 通过 teardown 管理每个取得的 port、path 和 child process；只在单独运行时通过的 spec 是 spec defect（[execution model](../docs/testing.md#how-specs-execute)）。
- **只有 observation 可能 divergence 时才发布 `./invariant`。** 使用 manifest name 检查由自身负责的 relation。否则省略 wiring，并在 package-specific README 中说明原因。空 companion 和被忽略的 reporter 无法通过 [`verify-package-invariants`](../.agents/notes/implemented/simplification/2026-08-28-omit-unneeded-invariant-companions.md)。

[命名规则](../docs/cookbook/adding-a-package.md#name-the-role-that-exists)：

- **Package tsconfig：** extends `tsconfig.base.json`（Client：`tsconfig.base.client.json`），设置 `rootDir: src` 和 `outDir: lib/types`，引用 workspace dependency，仅当 package 发布 `./invariant` 时引用 `runtime-diagnostics/invariants`，并在一个 aggregate 中注册。具有不同 Host 和 Client compiler face 的 package 使用 `tsconfig.host.json` 和 `tsconfig.client.json` leaf，以及仅作为 solution 的 root；普通双 entry Client plugin 不拆分（[layout](../docs/development.md#typescript-project-layout)）。
- `src/types.ts` 只包含 type，不包含 runtime code。
- Test 位于 package 层级的 `tests/` 下，而不是 `src/__tests__/`。
- Behavior 变更时在同一 commit 中更新 package README 和 JSDoc contract，并使用 [dsh-prose-standard](../.agents/skills/dsh-prose-standard/SKILL.md) 对照 code 验证。Group README 通过 canonical English page link 声明 subsystem ownership，或给出合理的 [exemption](../scripts/verify-subsystem-pages.ts)。
- Package README 使用 [canonical Model Experience format](../docs/cookbook/adding-a-package.md#4-write-the-package-readme)记录 model、token 和 KV-cache effect。
- Package README 将 durable consumer gap 和非显然 maintainer constraint 放入 `## Known Limitations and Deferred Work`；普通 cleanup 留在其 TODO 或 Agent Note 中。没有此类内容的 package 使用有理由的 [allowlist entry](../scripts/verify-package-readme-limitations.ts)。
