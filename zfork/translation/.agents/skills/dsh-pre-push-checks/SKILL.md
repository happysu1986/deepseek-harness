---
name: dsh-pre-push-checks
description: 在 push、force-push、标记 ready for review 或声称 deepseek-harness branch 检查通过前，以及 gh stack sync 发布 rewritten branch 后立即使用；选择覆盖即将发布或刚发布 diff 的最小 test 与 check，不反射性运行完整 repository suite。
---

# DSH Pre-Push Check

使用本 skill 在 `deepseek-harness` push 前运行一次相关 local evidence。唯一顺序例外是 `gh stack sync`：它可能在 rewritten layer 验证前发布 cascading rebase；随后立即验证，evidence 通过前不得 merge。Git hook 有意保持狭窄：pre-commit 修复 staged lint、检查 staged whitespace 并保护 vendored-source metadata；pre-push 只运行 incremental repository typecheck。CI 负责 exhaustive coverage 和 platform matrix。

## 检查 outgoing change

1. 确认 checkout 和 branch。

```sh
git status --short --branch
git rev-parse --show-toplevel
```

2. 验证 live PR base 或 stack parent，fetch 该 ref，并检查相对它的完整 scope。

```sh
pnpm --silent run change-scope --base <verified-base-ref>
```

该 command 绝不猜测或 fetch base。提供从当前 remote 或 stack state 验证的 ref；检查 `HEAD` 以外 commit 时使用 `--head <ref>`。其 versioned JSON 记录相对 resolved merge base 的 committed path；staged、unstaged 和 untracked path 描述当前 worktree。Merge changed base 后重新运行报告，重新评估 combined scope 可能影响的 behavior，并只重跑被 merge invalidate 的 check。

## 选择相关证据

除 hook 外，没有 universal local baseline。每项 behavior change 都需要最窄的现有 test 或 purpose-built check，使其 regression 会失败；只有 diff 实际触及的 surface 才增加更广检查。

Outgoing change 新增或修改 resource-owning/asynchronous test、fixture、helper 或 CI execution path 时，先使用 [dsh-ci-test-reliability](../dsh-ci-test-reliability/SKILL.md) 判断是否需要 restoration、negative-control、quiescent-teardown 或 concurrent-process 证据。本 skill 仍负责选择 command，并避免重复已通过证据。

- **Package 或 script behavior：** 运行所属 Vitest file 或 focused test name。Shared contract 变化时添加 adjacent package test；除非变更确实 cross-cutting 或用户要求，否则 repository-wide coverage 留给 CI。
- **Remote mock typing：** Unbuilt `any` 是显式 local fallback，不是 strict evidence。Handoff Remote/mock 变更前运行 `pnpm run typecheck`；诊断其余 error 前重建 missing、stale 或 partial generated declaration。Exception 只留在 [test proxy](../../../packages/test-support/remote-mock/README.md#remote-proxy)，绝不放入 production Remote type、ambient flag 或 copied signature。
- **Documentation、Agent Note、catalog 或 doc-linked comment：** 运行 `pnpm run doc-sync`；文档 workflow 要求时运行 full lint。
- **Model、editor、CLI 或 terminal visible output：** 运行负责该 output 的 focused keyless snapshot 或真实 runnable-example scenario。
- **Expected-output placement：** Selected recorded Session generation 既是 replay input 又是 expected persisted output 的 test 放在顶层 `snapshots/`，并由 `snapshot.yml` 命名 shipped `dsh` profile 与 composition/header pin。Canonical parent file 是 `session[.vN].jsonl`，child 是 `session.<ordinal>[.vN].jsonl`，harness 为每个 role 选择最高 generation。没有该 Session round trip 的 ARIA、geometry、generator、CLI 和 unit expectation 保留在所属 test 旁的 `tests/expected/`；不得放进 `snapshots/` 或使用 `*.snapshot.ts` owner。使用所属 `test:expected`、`test:web` 或 `test` lane。
- **Profile 与 configuration placement：** Shipped `dsh` profile 的 cross-package behavior 放在 `apps/cli/tests/profiles/`；package-specific Loader composition 放在该 package 的 `tests/fixtures/`。User-facing optional overlay 放在 `apps/cli/config/examples/`，并与 `docs/user/` 下 guide 配对。
- **Package manifest、public export、build configuration、worker/bin entry 或 built runtime path：** 运行 `pnpm run build`、相关 hygiene check 和所属 built-artifact smoke。
- **真实 provider 或 agent behavior：** Credential 可用时运行相关 `pnpm run test:e2e` target；绝不打印 secret。

不得仅因随后要 commit 或 push 就手动重复 passing check。尤其不得只为复制 pre-push hook 而在 push 前立即运行 typecheck。

使用 `pnpm run` 时，将 Vitest file 和 name filter 直接放在 script name 后；不要插入独立 `--`，否则它会传给 Vitest，并可能禁用 `-t` filtering。例如以下命令运行一个 headless snapshot scenario：

```sh
pnpm run test:snapshot snapshots/session/headless.snapshot.ts -t 'cordis-inspect-jsdoc'
```

此规则适用于这些 pnpm script；npm 和其他 launcher 保留各自 argument-forwarding syntax。将 filtered run 视为 focused evidence 前，检查报告的 selected test count。

### 将 unit coverage 聚焦到受影响 source

Test selection 与 coverage selection 相互独立。Vitest file filter 选择运行哪些 test，而 repository configuration 否则会测量所有 `packages/*/*/src/**/*.ts` file。Unit coverage 相关时，同时命名所属 test 和这些 test 必须证明 coverage 的 source file 或 package：

```sh
pnpm exec vitest run packages/<group>/<package>/tests/<behavior>.spec.ts \
  --coverage \
  --coverage.include='packages/<group>/<package>/src/**/*.ts'
```

Behavior 确实只限于一个 module 时使用 exact source file。多个受影响 file 或 package 重复使用 `--coverage.include`，并传入覆盖该 scope 所需的所有 owning test file。Configured per-file 100% threshold 仍适用于 selected source scope。

不清楚 owning test 时，使用 Vitest dependency graph 发现 candidate set，再检查 selected test 后才视为 evidence：

```sh
pnpm exec vitest related packages/<group>/<package>/src/<changed>.ts \
  --run \
  --coverage \
  --coverage.include='packages/<group>/<package>/src/<changed>.ts'
```

`vitest related` 无法发现只能经 configuration、dynamic loading、subprocess、worker、built artifact 或 external provider 到达的 behavior；显式选择这些 owning test。不得使用 `--passWithNoTests`、降低 coverage threshold，或仅为隐藏 uncovered affected file 而缩窄 `--coverage.include`。Selected package scope 因一个 focused test 无法覆盖而失败时，添加其他相关 owning test；只有被排除 module 不可能受变更影响时才缩窄 source scope。

## 完整 local rehearsal

只有用户明确要求、诊断 CI failure，或变更广泛到没有可信的更窄集合时，才运行完整 local approximation。使用当前 workflow 和 package script 作为 inventory；不要重建已移除的 `check:pre-push` aggregate。

## 保护 history-rewriting push

Standalone 和 stacked PR branch 都允许 rebase，包括 review 后。Standalone history rewrite 前 fetch 当前 remote branch 并记录 exact OID；使用 `--force-with-lease=<branch>:<observed-oid>` 发布，使并发 update 中止 push。`gh stack push` 和 `gh stack sync` 为其管理的 branch 提供 lease protection。绝不能使用裸 `--force`。

任何 rewritten push 后，重新 fetch live head，并重新审计 unresolved review thread、approval、mergeability 和 check。Rewrite 前的 commit hash 与 inline-comment anchor 不再是当前 evidence。

### Post-sync validation

`gh stack sync` 将 fetch、cascade-rebase 和 push 合为一个 operation，无法在 rewrite 与 publication 间插入 local validation。运行前要求 clean worktree，并记录 official stack order 与 exact remote head。返回后：

1. 重新查询每个 branch head 和 official GitHub stack order。
2. 根据 live PR base 检查每个 rewritten layer 的 changed scope。
3. 为每个 affected layer 运行本 skill 选择的相关 evidence。
4. 所有 selected check 通过前，保持每个 PR unmerged，并将 validation 报告为 pending。

Post-sync evidence 失败时，保留 lease-protected published head，修复 failure、验证修复并发布 correction。不能仅因 command 成功就声称 sync 使 stack ready。

## 处理 failure

普通 push 前相关 check 失败时停止并修复，或解释 blocker。不得 push 后寄希望于 CI 不同。Post-sync 例外则阻止 merge，并遵循上述 repair procedure。

Failure 看似 environment-specific 时要证明：

- 记录确切 command、failing test 和 platform-specific mismatch。
- 确认相关 non-platform evidence。
- Required check 优先修复 cross-platform nondeterminism。
- 只有用户明确要求或同意时才 bypass local hook，并准确报告失败内容及为何预期 CI 不同。

## Push procedure

普通与 standalone rebase push：

1. 运行一次 selected relevant check。
2. 正常 commit，并在继续前检查 pre-commit fixer 修改的所有 file。
3. 正常 push；或对获授权 rewritten branch 使用 exact lease，使 incremental typecheck hook 运行。
4. 验证 remote ref 与 local `HEAD` 一致。

```sh
git rev-parse HEAD origin/$(git branch --show-current)
```

GitHub PR 在 push 后检查 remote CI：

```sh
gh pr checks
```

Pending check 如实报告为 pending。将 failure 归因于 branch 或 environment 前先检查。

当 `gh pr checks` 报告 "no checks reported"，且 `/actions/runs?head_sha=<sha>` 返回 `total_count: 0` 时，在怀疑 push 或 GitHub event 丢失前读取 mergeability：

```sh
gh pr view <number> --json mergeable,mergeStateStatus
```

PR 为 `CONFLICTING`/`DIRTY` 时，GitHub 不创建 `pull_request` workflow run，因此缺失 signal 就是 conflict，而非 infrastructure。唯一修复方式是解决 conflict；empty commit、`--allow-empty` push、draft/ready toggle 和 revert-and-restore bounce 都会让 `total_count` 保持为零，并增加 junk history。Branch 尚不能 local merge 时，使用 `git merge-tree --write-tree HEAD origin/<base>` 确认 conflicting path。

对于 `gh stack sync`，使用 post-sync validation sequence，不要假装普通顺序可行。
