# AGENTS.md

DeepSeek Harness 是完全由 plugin 构成的 Cordis agent harness。修改 `packages/` 前请阅读 [docs/architecture.md](../../docs/architecture.zh.md)；文档工作遵循 [docs/AGENTS.md](docs/AGENTS.md)。

## 尚未稳定的 API 与已发布的 Session 数据

Public API 尚未稳定；变更时要更新所有 consumer。遵循[版本与状态](../../docs/session-format-status.zh.md)以及[类型变更确认规则](../../docs/cookbook/reviewing-persistence-type-changes.zh.md)。[相邻迁移](../../.agents/notes/implemented/architecture/2026-08-31-released-session-format-migrations.zh.md)可以新增以版本命名的后继版本，但绝不能移动、覆盖或删除已经 commit 的代际；存在前序版本不代表支持 fallback 或 downgrade。SQLite 使用单调递增的 `SCHEMA_VERSION`。

确认所有[已声明的 persistence type 变更](../../docs/cookbook/reviewing-persistence-type-changes.zh.md)。

**应用启动。** 只有 `dsh` profile 可以启动受支持的 Node 应用；禁止使用 package bin、demo 和 public SDK argv 逃逸路径（[规则](../../docs/architecture.zh.md#应用启动)）。

## 仓库布局

```
vendor/      Vendored Cordis (vendor/README.md)
packages/    @deepseek-ai/dsh-<pkg> workspaces at packages/<group>/<pkg>/
  core/                 agent/session API
  api/                  remote BFF
  typert/               type graphs
  llm/                  model providers
  shell/                command execution
  subprocess/           child-process management
  ssh/                  SSH execution providers
  terminal/             persistent terminals
  ptc-runtime/          PTC execution
  sandbox/              process confinement
  deliverables/         turn deliverables
  fs/                   filesystem access
  lsp/                  language servers
  skill/                skill loading
  web/                  search/fetch tools
  computer-use/         computer interaction
  browser-use/          browser interaction
  compaction/           context compaction
  context/              request context
  subagent/             delegated agents
  jobs/                 background jobs
  bundle/               profile bundles
  workflow/             workflow execution
  webhook/              webhook ingress
  todo/                 todo_write tool
  plan/                 logged planning
  goal/                 session goals
  schedule/             scheduled follow-ups
  preset/               agent composition
  guard/                loop/tool guards
  extensions/           runtime self-modification
  hooks/                Claude Code/Codex bridges
  session/              durable sessions
  session-query/        browsing/search/export
  attachment/           binary attachments
  spill/                output spill
  storage/              non-session storage
  workspace/            workspace entities
  feedback/             human feedback
  identity/             anonymous identity
  settings/             user settings
  credentials/          credentials/authorization
  acp/                  automation-only ACP
  interaction/          human interaction
  boot/                 application boot
  sdk/                  JSON-RPC SDK
  host/                 GUI host
  client/               GUI client
  mcp/                  external tools
  experimental/         pre-stable prototypes; public by default with explicit private exceptions
  test-support/         test infrastructure
  runtime-diagnostics/  runtime invariants
  util/                 zero-dependency utilities
python/      Python SDK/runtime (python/README.md)
native/      @deepseek-ai/node-addon-system source (native/README.md)
benchmarks/  performance gates
.agents/     Agent workflows/notes
docs/        Documentation (docs/AGENTS.md)
scripts/     gates and generators
website/     VitePress documentation projection
```

Package group 见 [packages/README.md](../../packages/README.zh.md)。

## 命令

```sh
pnpm install            # pnpm workspaces, node ^22.19 || >=24
pnpm run clean           # remove build outputs and safe residue from deleted packages
pnpm run test           # unit tests
pnpm run test:coverage  # CI coverage gate: per-file 100% on packages/*/*/src
pnpm run test:e2e       # real-API tests; self-skip without DEEPSEEK_API_KEY
pnpm run test:expected  # owner-local process expectations
pnpm run test:snapshot  # keyless recorded-session replay through shipped profiles; filter: -t <name>
pnpm run test:snapshot:record  # re-record expected outputs (needs key)
pnpm run typecheck
pnpm run lint
pnpm run duplication    # cross-file TypeScript clone detection
pnpm run build          # tsc emits lib/types, tsdown bundles runtime
pnpm run hygiene        # publint + workspace/package/dependency checks + NodeNext consumer check
pnpm run check:windows-wine  # ONLY when diagnosing a known Windows failure (needs wine); CI owns this signal
pnpm run doc-sync       # documentation gates (scripts/run-gates.ts)
pnpm run test:docs      # quick documentation checks (no build; doc-quick aggregate)
pnpm run website:build  # VitePress build (doubles as dead-link check)
pnpm dsh --profile headless "task"  # run one task from source (needs DEEPSEEK_API_KEY)
pnpm run demo:ptc -- "task"  # headless PTC mode run (needs key)
pnpm run dev:web | dev:desktop  # build, then launch; Web also rebuilds client bundles on edits. start:web | start:desktop skip the build
make web|dev-web|desktop|dev-desktop|build  # the same commands; ARGS='--no-open' forwards options
```

### Host sandbox 失败

如果必需的 `gh`、`pnpm`、build、test 或 generator 命令因 sandbox 阻止 credential、network、IPC、watching 或嵌套 `sandbox-exec` 而失败，请保持命令不变，并以最小范围进行 host escalation。必须有 sandbox 证据；绝不能绕过测试失败或产品 sandbox。

### 在本地运行相关检查

push 前遵循 [dsh-pre-push-checks](.agents/skills/dsh-pre-push-checks/SKILL.md)；只报告实际运行的命令。执行 `gh stack sync` 后立即验证；检查通过前不得 merge。

- 让证据与变更面对应：使用聚焦的行为测试、model/user 输出 snapshot、文档使用 `doc-sync`、发布路径使用构建产物 smoke test、provider 使用 real-API e2e。
- 不得默认运行完整 test suite，也不得仅为 commit 或 push 重复已经通过的检查。CI 负责穷尽式 coverage 和 platform matrix；只有用户明确要求、诊断 CI，或变更不可约地影响整个仓库时，才在本地完整演练。
- CI coverage gate 是 `test:coverage`，而不是 `test`（[原因](../../docs/testing.zh.md)）。
- **Web browser automation 与 GIF 录制：** 使用 `pnpm dsh web --patch apps/web/tests/pin-browse-picker.overlay.yml` 启动，以使用[页面内目录选择器](../../apps/web/tests/pin-browse-picker.overlay.yml)；只有明确测试 native picker 行为时才省略此 override。

## Secret / `.env`

Windows packaging/signing：[必读内容](../../apps/desktop/README.zh.md#windows-ev-签名)。

Real-API test/demo 读取 `DEEPSEEK_API_KEY`、可选的 `DEEPSEEK_BASE_URL` 和根目录 `.env`。cordis.yml 只允许在 plugin `config` 和 entry `disabled` 下使用 `!!js`（绝不能用 `!js`）；其他 metadata 保持 literal，因此条件组合也使用 overlay（[入门说明](../../docs/cordis-primer.zh.md#loader-配置)）。绝不能 commit credential。没有 key 时 CI e2e 会自行跳过；key policy 由 [testing.md](../../docs/testing.zh.md) 负责。

## 约定

- Package 使用 `@deepseek-ai/dsh-<name>`；vendor 已[重新划定 scope](../../docs/rescope.zh.md)，并设为 `private: true`。Harness package 在 `peerDependencies`/`devDependencies` 中声明 `@deepseek-ai/cordis`。Workspace dependency section 对 DSH 使用 `workspace:*`，对 vendor/native 使用 `workspace:~`（[规则](../../.agents/notes/implemented/process/2026-09-22-workspace-release-ranges.zh.md)）。
- 全部使用 ESM（`"type": "module"`）。跨 package 使用 package name，本地相对 import 使用 `.ts`。Config subprocess 在普通 Node 下运行已构建的 `lib/`；source regression 使用其声明的 launcher（[测试 policy](../../docs/testing.zh.md#测试子进程启动模式)）。`dsh` CLI source launch 通过 tsx 的 ESM-only hook（`node --import tsx/esm`）运行；它可达的 module 必须保持 ESM（不得有 CJS-only export）——支持的 engine 范围内不可使用 Node native TypeScript mode（[source launch contract](../../.agents/notes/implemented/architecture/2026-07-29-dsh-source-launch-tsx-esm.zh.md)）。Raw/Web `cordis.yml` 中的 bare plugin 必须出现在 resolver manifest 的 `dependencies` 中；`verify-cordis-config` 会强制执行。
- **Registration 是 effect**：所有 contribution 都通过 `ctx.effect()` / `ctx.on()`；registry 的 `register()` 返回 disposer。
- **Runtime invariant 断言由自身负责的关系。** 只有独立 observation 可能发生 divergence 时才发布 `./invariant`。否则省略其 source 和 wiring，并在 README 中记录原因；空 installer 以及检查 service 是否存在、plugin metadata、effect 或固定示例的做法均无效（[package invariant 规则](packages/AGENTS.md)）。
- **Typed event 使用 declaration merging** 和 merge-extensible map。Event JSDoc 需要 `@mode` 与 payload `@param`；payload 中不存在的 scoped key 需要 `@dshScopeScan unsupported`。Public service method 要记录 parameter 和非 void return。默认读取 `SessionEventMap` member 时必须识别其类型——如果 build 不知道某类型，则拒绝该 log，除非 event 的 envelope 带有 `ignorable: true`；只有结构格式变更才增加 `SESSION_FORMAT_VERSION`（[机制](../../.agents/notes/implemented/architecture/2026-08-10-session-log-version-mechanism.zh.md)）。
- **对 discriminant tag 使用 switch。** Closed union 以 `assertNever` 结束；merge-extensible union 落入有文档说明的 default。
- **Waterfall listener 必须调用 `next()`** 以委托后续处理；不调用便 return 会短路整个 chain（[语义](../../docs/cordis-primer.zh.md#cordis-waterfall-语义)）。
- **Model-visible ⟺ 已记录**：任何进入 model request 的内容都必须能从 session log 重建；新增 model-visible input 需要 session event。
- **使用 plugin，不修改 loop**：新行为放在有文档说明的 extension point 上；修改 `agent-loop` 必须同步更新 docs/architecture.md。
- **Capability seam 包含 Service Definition / Service Provider / Consumer 三种角色。** 它必须完整，不能只有一种角色；只有角色独立演化时才拆分（[术语表](../../docs/glossary.zh.md#capability-seam)）。
- 当 maintained dependency 能真正删除自有代码和测试时，**优先使用 dependency，而不是手写实现**（[policy](../../.agents/notes/implemented/process/2026-07-26-dependencies-over-hand-rolling.zh.md)）。
- **Package boundary 上 explicit > implicit**：defaulting 是所属 implementation 中显式的 `resolve(request): Spec` 步骤，绝不能隐藏在 `run()` 内部的 `?? default` 中（以 `dsh-shell` 的 request/spec 拆分为模板）。
- **Plugin 中不得硬编码 tunable**：随 deployment 变化的选项必须是经过验证、可由 cordis.yml 修改的 `Config` field；`DEFAULT_*` constant 或 test hook 不代表可配置。Protocol constant、external spec 和 security invariant 保持固定。
- **Misconfiguration 要显式失败**：能够独立判断时在 load 阶段失败，否则在最早可解析点失败；绝不能静默跳过缺失的 referent。
- **跨 boundary 的 opaque id 使用 branded type**（来自 `dsh-brand` 的 `Branded<B>`），绝不能是普通 `string`。
- **在 typed same-process boundary 信任 TypeScript。** 不得仅为 static interface 已要求的 value 添加 runtime validation、fallback behavior 或 hostile-input test；应在 parser/config、queued、model/tool JSON、durable/file、worker、process 和 wire boundary 进行验证。
- **不得新增到 `unknown` 的 assertion**（`as unknown` 或 `<unknown>`）。保持或减少既有 baseline；替换实现应使用 typed value 或 validation（[规则](../../.agents/notes/implemented/process/2026-09-19-no-unknown-casts.zh.md)）。
- **Source plane 与 artifact plane 绝不能混用。** Static gate 和 test 通过 tsconfig `paths` 将 workspace import 解析到 `src`，并在干净目录树上通过；消费已构建 `lib/` 的 gate 必须声明该 dependency（[布局](../../docs/development.zh.md#typescript-项目布局)）。
- **Compiler face 必须明确。** 同时包含 Host 与 Client program 的 package 提供 face-specific leaf config 和只作为 solution 的 root；repo-wide program 以某个 face config 为入口，绝不能以 root solution 为入口（[布局](../../docs/development.zh.md#typescript-项目布局)）。
- **空 `catch` 要命名 error 并说明原因**；其 `try` 只能包含一条 statement。
- **Comment 保持局部性。** 不得复述 code、扩展无关 comment，也不得在缺少局部必要性的情况下解释远处行为（[理由](../../.agents/notes/implemented/process/2026-08-09-concrete-prose-names-actors-and-recorded-facts.zh.md)）。
- **禁止 `prove` + `nance`**（[规则](../../.agents/notes/implemented/process/2026-08-26-ban-ambiguous-origin-label.zh.md)）。
- **Parallel value 优先保持对称**；无法解释的不对称通常意味着遗漏了 extraction。
- **Test 描述行为，不描述 correctness。** 修改过时行为时同步修改其 test；在 PR 中说明原因。
- **只有持久的 decision rationale 才创建 Agent Note；** mechanical/local edit 可以豁免，包括局部 UI 变更（[范围](../../.agents/notes/README.zh.md#何时需要写一份)）。Archived note 是冻结内容：绝不能编辑，也不能视为当前 authority（[archive policy](../../.agents/notes/README.zh.md#归档与删除)）。
- **Client UI 文案归 locale 所有。** Product text 通过 typed dictionary 和 `t` 或 localized primitive prop 传递；`verify-client-ui-i18n` 拒绝 hardcoded copy（[决定](../../.agents/notes/implemented/architecture/2026-08-23-locale-owned-client-ui-copy.zh.md)）。
- **测试 policy** — [docs/testing.md](../../docs/testing.zh.md)。每个非 trivial 的 model-visible 或 product-user-visible 变更都要更新 keyless recorded-session snapshot；[snapshot ownership](snapshots/AGENTS.md) 将顶层目录树留给 session-driven case，其他 expected output 保留在 owner 本地。Fixture 在 macOS/Linux 上 replay；修复 fixture，不要修复 normalizer。
- **预先设计每个 tool 的 UI presentation。** Host presenter 保持 pure；Web card 从 raw event 和持久化 result metadata 派生（[cookbook](../../docs/cookbook/adding-a-tool.zh.md)）。
- **为 capability seam、lifecycle path 和 transcript output 规划 unit、e2e 与 snapshot coverage**；缺少 snapshot harness support 时，在同一变更中补齐。
- **两个 SDK 都投影 loop。** Agent-loop、session-lifecycle 和 `SessionEventMap` 变更要在同一 PR 中更新 TypeScript 与 Python SDK expected output；`pnpm run test` 不覆盖二者（[范围](../../docs/testing.zh.md#何时需要快照测试)）。
- **有意识地选择 PR history。** 拆分独立变更，并在传播前修复引入问题的 PR。Standalone/stack branch 可以 merge-forward 或 rebase。重写使用 `--force-with-lease`，remote 发生移动时中止，绝不能使用裸 `--force`；获取更新的 base 前保留进行中的 merge-forward checkpoint（[理由](../../.agents/notes/implemented/process/2026-08-02-native-github-stacks-and-optional-rebases.zh.md)）。
- **Label：** 一个 PR 使用一个 `kind/*`、所有实质相关的 `area/*`，以及 native Issue Type（[分类](../../.agents/notes/implemented/process/2026-08-08-unified-github-label-taxonomy.zh.md)）。
- TODO marker 按紧急程度使用 `FIXME`/`TODO`/`XXX`（[语义](../../docs/development.zh.md)）。
- 文件末尾恰好保留一个 newline；`git diff --cached --check`（pre-commit）会对此执行 gate。

## Defensive pattern

处理 lifecycle、concurrency、subprocess 或 teardown 前阅读 [docs/defensive-patterns.md](../../docs/defensive-patterns.zh.md)。

## Type safety 与文档

所有内容都在 `strict: true` 与 `noImplicitAny` 下编译；每个仍然存在的 `any` 都必须说明无法 narrow 的原因。每个 module 和 export 都要以精简 JSDoc 记录其非显然 contract；function-like export 包含 `@param`/`@returns`，由 `verify-export-jsdoc` 强制执行。通过 heritage 声明的 member、plugin-protocol slot 和 constructor 将文档保留在声明它们的 Service Definition、protocol 或 class 上。

Comment 和文档陈述完整 contract 与上下文，不记录 reasoning transcript。使用直接、具体的术语，不使用 metaphor。写下 `contract`、`boundary` 或 `shape` 前，先判断是否有更精确的词能命名对象：用 `response fields`、`JSON validation` 或 `ESM exports`，而不是 `response shape`、`validation boundary` 或 `module shape`。`contract` 只用于 precondition、postcondition、invariant、compatibility promise，以及 caller、callee、implementer、provider、producer 或 consumer 依赖的其他 obligation。保留字面意义上的 process、wire、security、transaction 或 lifecycle boundary。不要叙述 control flow 或 test，不要保留 review history，也不要复述 code。保留 behavior、failure、timing、ownership 和 safe-use fact；链接到 rationale。判断 prose 时使用 [dsh-prose-standard](.agents/skills/dsh-prose-standard/SKILL.md)。把可机械检查的 invariant 接入实际执行的顶层 gate，并证明每条发生变更的 acceptance path 都会拒绝 invalid case。使用范围窄且理由充分的 exception，不要全局禁用规则。

每次 code change 都要带上相应文档：同时更新受影响的 README 和 JSDoc contract。常规双语工作遵循 [docs/AGENTS.md](docs/AGENTS.md)；只有用户明确调用时才可运行 `dsh-translate-docs`。Current-state prose、每个 paragraph 一行、每个事实一个归属位置，以及 word budget 均由该文件规定。

## 编辑这些说明

根目录和 `packages/` 下的 `CLAUDE.md` 都是指向 `AGENTS.md` 的 symlink；编辑真实文件。每条规则保持独立完整，并链接 high-level 文档。在不损害清晰度时压缩文字；只有所需内容确实需要更多空间时才提高 `verify-doc-budgets` ceiling。

## Vendoring policy

`vendor/` package 是固定版本的 source copy（带有 upstream SHA 的 manifest 位于 [vendor/README.md](vendor/README.md)）。按其中的 sync procedure 更新；重新应用或废弃有记录的 local modification；重新运行 `pnpm run test && pnpm run build`。
