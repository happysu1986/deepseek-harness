---
name: fork-sync-upstream-release
description: 将官方 deepseek-ai/deepseek-harness 的 master branch 同步到本 fork 的 master，再把 master merge 进 release。用于维护 fork；push 任一 branch 前必须取得用户明确授权。
---

# 将上游 `master` 同步到 `release`

在不重写 branch history 的前提下，将 `https://github.com/deepseek-ai/deepseek-harness.git` 合入当前 fork。固定顺序为官方 `master` → fork `master` → fork `release`。

用户明确要求执行同步时，即授权完成所需的本地 fetch、branch switch 和 merge；该请求不授权 push、force-push、删除 branch、丢弃修改、变更 repository hosting 设置，或创建、修改、merge PR。只有用户明确要求时才可 push。

## 前置检查

从仓库根目录运行，并在变更状态前检查 `git status --short --branch`、进行中的 Git 操作、branch、worktree 和 remote。必须满足：

- 没有可能被 branch switch 或 merge 覆盖的 tracked 或 untracked 修改；
- 没有正在进行的 merge、rebase、cherry-pick 或 revert；
- 本地存在 `master` 和 `release` branch；
- `origin` 指向用户的 fork，而不是官方仓库；
- 已有 fetch remote 的 URL 能识别 `deepseek-ai/deepseek-harness`，或当前环境能够访问官方 HTTPS URL。

不得通过 stash、reset、clean、detach、删除或重写 branch 来满足前置条件。如果其他 worktree 已 checkout `master` 或 `release`，条件允许时使用对应的干净 worktree，否则停止并报告冲突。记录两个 branch 的起始 OID 和当前 branch，作为结果核对与恢复位置。

仅当 remote 的规范化 HTTPS 或 SSH URL 指向 `github.com/deepseek-ai/deepseek-harness` 时，才将其视为官方 remote；不得只因名称为 `upstream` 就信任它。没有官方 remote 时，直接从 HTTPS URL fetch 到名称明确的 remote-tracking ref，不得新增或改写 repository configuration。merge 前 fetch 官方 `master`，以及 fork 已发布的 `master` 和 `release` ref；如果 `release` 只存在于本地，则允许 fork 不存在 `origin/release`。

## 更新 `master`

切换到本地 `master`。如果存在 `origin/master`，本地 `master` 必须已经包含它，或者能够 fast-forward 到它。本地与 fork remote 发生 divergence 时停止，不得自行选择 history policy。

使用普通 merge 将已 fetch 的官方 `master` 合入本地 `master`，保留 fork 自有 commit。禁止把本地 `master` reset 到官方 ref、rebase 已发布 commit 或 force update。发生 conflict 时逐项检查；只有当前代码和文档能够明确表明预期行为时才解决，否则保留停止中的 merge，并报告需要 owner 决定的确切文件。

记录更新后的 `master` OID，并说明本次操作是 fast-forward、创建 merge commit，还是已经 up to date。

## 将 `master` 合入 `release`

切换到本地 `release`。如果存在 `origin/release`，本地 `release` 必须已经包含它，或者能够 fast-forward 到它；本地与 fork remote 发生 divergence 时停止。

使用普通 merge 将更新后的本地 `master` 合入 `release`。保留 `release` 独有 commit，禁止 reset 或 rebase `release`。conflict 处理规则与 `master` 相同。

## 验证与发布

检查新的 commit graph，并要求以下两个 ancestry 关系成立：

```sh
git merge-base --is-ancestor <fetched-official-master> master
git merge-base --is-ancestor master release
```

确认 worktree 干净，并检查从已记录起始 OID 到新 OID 的完整范围。根据 fetch 引入的变更选择相称的检查；执行任何已授权 push 前，加载并遵循 [`dsh-pre-push-checks`](../dsh-pre-push-checks/SKILL.md)。只报告实际运行的检查。

用户明确要求发布时，在 push 前立即再次 fetch `origin`；如果任一 remote branch 已前进到尚未合入的 ref，则停止。禁止 force-push；优先以一次 atomic push 发布 `master` 和 `release`，避免 fork 暴露只发布一半的同步状态。如果 server 不支持 atomic push，必须先询问用户，再分别发布两个 branch。

如果起始 branch 仍存在且切换安全，完成后返回该 branch。报告官方 fetch OID、`master` 与 `release` 的新旧 OID、merge 结果、conflict 或未发布原因，以及实际运行的验证命令。
