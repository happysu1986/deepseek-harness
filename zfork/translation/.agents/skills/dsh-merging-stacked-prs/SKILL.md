---
name: dsh-merging-stacked-prs
description: 将 dependent GitHub PR stack（A ← B ← C，每层以其下层为 base）落到 master、merge base 为另一个 open PR branch 的 PR，或请求提到 stacked PR、PR stack、dependent PR 或按顺序 merge 多个相关 PR 时使用。要求同一 repository 的每条 dependency chain 在落地前使用 GitHub 官方 stacked-PR feature，使 GitHub 负责整个 stack 的 rule、CI、ordering、retargeting 和 merge state。
---

# 落地官方 GitHub PR stack

通过 GitHub native stack object 和 `gh stack merge` 落地 dependent PR。不得使用 `gh pr merge` 和 `gh pr edit` 逐个 merge、retarget PR 来模拟 stack semantic。根 [AGENTS.md](../../../AGENTS.md) 负责允许的 merge-forward 与 rebase history；[stack review guide](../../../docs/cookbook/responding-to-pr-review-on-a-stack.md) 负责传播 review fix。

## 要求 native stack support

更改 GitHub state 前运行 `gh stack --version`。官方 extension 或 server-side stack feature 不可用时 hard-stop；不得回退到逐个手动 merge 和 retarget PR。GitHub stack 要求所有 head branch 位于同一 repository，因此 cross-fork chain 必须 hard-stop。

使用干净的 dedicated worktree。Fetch 当前 PR metadata 和精确 head OID，不要信任 branch name 或旧 report：

```sh
gh pr view <pr> --json number,author,baseRefName,baseRefOid,headRefName,headRefOid,isCrossRepository,state,isDraft,reviewDecision,mergeStateStatus,statusCheckRollup
```

对每条疑似 chain 中至少一个 PR 查询 `PullRequest.stack` 和 `stackEntry.position`；官方 GitHub object 才是 stack-membership authority，不能只根据 base branch 推断。`size` 超过返回 page 时 paginate `entries`：

```sh
gh api graphql -F owner=<owner> -F name=<repo> -F number=<pr> -f query='
query($owner: String!, $name: String!, $number: Int!) {
  repository(owner: $owner, name: $name) {
    pullRequest(number: $number) {
      number
      author { login }
      baseRefName
      headRefName
      stackEntry { position }
      stack {
        number
        baseRefName
        size
        entries(first: 100) {
          nodes {
            position
            pullRequest { number author { login } baseRefName headRefName state isDraft }
          }
        }
      }
    }
  }
}'
```

根据 live PR base 确定预期 bottom-to-top 顺序：bottom 指向 trunk，每个更高 PR 指向紧邻下层的 head branch。

## 链接缺失的 stack member

先将 existing stack entry 与 expected chain 比较。一个 existing stack 可以包含请求 chain 中保持顺序的 subset；多个 stack number、unexpected entry 或 conflicting order 都要求在 mutation 前由用户决定。

任一 dependent PR 尚未加入 official stack 时：

1. 精确比较每个 `author.login`。
2. 所有 author 相同时，按 bottom-to-top 自动链接 chain：

```sh
gh stack link --base <trunk> <bottom-pr> <next-pr> ... <top-pr>
```

3. Author 不同或任一 author 不可用时，更改 GitHub state 前询问用户是否链接。
4. 重新查询 GraphQL，并要求一个 stack number、expected trunk、完整 PR set，以及 expected position 和 base chain。

绝不能自动 dissolve、reorder 或 rebuild existing stack；`gh stack link` 是 additive，merged 或 queued entry 不能 unstack。

## 只在需要时 refresh

不要仅因存在 refresh mechanism 就重写 branch。Live merge state 或 repository rule 要求更新 trunk 时，选择一种允许的 history：

- **Native cascading rebase：** Remote stack 未在本地 tracking 时使用 `gh stack checkout <pr-or-stack>` checkout，再运行 `gh stack sync`。该 command 可能在 local validation 前 rebase 并以 lease-protected force-push 发布每个 active layer。立即检查 rewritten scope，为每个 affected layer 运行相关检查；通过前不得 merge 或声称 ready。Sync 检测到 rebase conflict 时，使用 `gh stack rebase`，解决并验证后以 `gh stack push` 发布。Checkout 或 sync 报告 local/remote stack composition divergence 时取消并询问，不得自动删除或重建 remote stack。
- **Incremental merge-forward：** 将 trunk merge 到最下方 affected branch，再按 bottom-to-top 将每个 updated parent 传播到 child，并正常 push。进行中的 merge 期间 base 前进时，先保留该 checkpoint，再 merge 新 tip。

Review 后允许 history rewrite，但会使 commit-OID assumption 失效。Push 后重新 fetch exact head，并重新审计 unresolved review thread、approval、mergeability 和 check。绝不能使用裸 `--force` 或覆盖并发前进的 remote head。

## Merge range 前置检查

Merge 前立即重新查询 official stack。要求每个 selected PR 为 open、non-draft、顺序正确，并满足 repository review 与 check requirement。每个 PR state 独立判断；top layer ready 不能证明 dependency ready。

「Land the stack」选择整个 stack。Partial landing 必须明确 boundary PR，并包含从 bottom 到该 boundary 的每层。

## 通过 stack API merge

使用 official stack number merge 整个 stack：

```sh
gh stack merge <stack-number> --yes --merge
```

明确请求 partial landing 时，merge 到 boundary PR：

```sh
gh stack merge <boundary-pr> --yes --merge
```

不得传 `--delete-branch`、手动 retarget dependent，或发出 per-PR merge command。GitHub 从 bottom 向上 merge selected range，并 retarget/rebase 其余 upper layer。Direct stack merge 是 all-or-nothing；trunk 使用 merge queue 时，GitHub 会把 selected range 一起入队，但可能分组落地。

不得绕过 merge requirement。Native merge 报告 blocker 时，通过所属 PR 检查并解决，或停止报告；绝不能回退到 `gh pr merge`。

## 验证 landed state

等待每个 selected PR 报告 `MERGED`；queued request 不代表完成 landing：

```sh
gh pr view <pr> --json number,state,mergedAt,mergeCommit,baseRefName,headRefName
```

Partial landing 后重新查询 official stack，确认每个 remaining PR 仍按 expected order 链接，并指向 stack trunk 或其下层。GitHub 可能 rebase remaining layer，因此重新检查 current head、review state 和 CI。

只有对应 PR 报告 `MERGED` 后，才能在独立 final pass 删除 branch。删除每个 branch 前，要求 GitHub 报告没有 open PR 仍以其为 base：

```sh
gh pr list --state open --base <branch> --json number --jq length
```

任何非 `0` 结果都阻止删除。

## Checklist

- [ ] Native `gh stack` support 可用；每个 PR branch 位于同一 repository。
- [ ] Live PR base 与 exact head 建立一条 bottom-to-top dependency chain。
- [ ] GraphQL 报告一个包含 expected trunk、entry 和 order 的 official stack；符合条件的 same-author unstacked chain 已自动链接。
- [ ] 所有 rewritten layer 通过相关 validation，之后已重新审计 review thread、approval、mergeability 和 check。
- [ ] 整个 stack 或明确限定的 prefix 已通过 `gh stack merge --yes --merge` 提交。
- [ ] 每个 selected PR 报告 `MERGED`；remaining upper layer 仍组成 expected official stack。
- [ ] Branch 只在 merged-state 与 zero-dependent 验证后删除。
