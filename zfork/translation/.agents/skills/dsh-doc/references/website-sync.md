# Website publication

## 概要

文档网站是 repository Markdown 经过测试的 projection，绝不是第二份副本。[website/docs.ts](../../../../website/docs.ts) 是显式 public allowlist，[scripts/project-doc-site.ts](../../../../scripts/project-doc-site.ts) 将映射 source 重写到 disposable `website/.generated/` tree，VitePress 构建该 tree。Build 还从同一 manifest 为每条 route 生成 raw-Markdown twin 和根 `llms.txt` index。本 reference 负责 manifest field、projector link rule、preview 与 validation command，以及 deployment boundary。

## 目录

- [Manifest ownership](#manifest-ownership)
- [分类变更](#classify-the-change)
- [DocsPage field](#docspage-fields)
- [保持 link behavior](#preserve-link-behavior)
- [Preview 与 validation](#preview-and-validate)
- [保持 deployment 独立](#keep-deployment-separate)
- [Dev Note](#dev-note)

## Manifest ownership

修改 manifest 前，阅读 [docs/AGENTS.md](../../../../docs/AGENTS.md) 和 [website/docs.ts](../../../../website/docs.ts) 中当前 `DocsPage` type 与 entry；不要依赖记忆中的 field set。新增 section、sidebar collection、locale 或 top-level navigation item 前，阅读 [website/.vitepress/config.ts](../../../../website/.vitepress/config.ts)。对于 edited bilingual source，遵循 [docs/AGENTS.md](../../../../docs/AGENTS.md#writing-rules) 中的 lightweight routine path 和 [pairing contract](../../../../docs/i18n/README.md)；绝不能自动调用扩展 translation skill。

绝不能编辑或 commit `website/.generated/`、`website/.cache/` 或 `website/.dist/`。除 `website/AGENTS.md` 外，不得在 `website/` 下新增 Markdown；`website/zh-CN/`、`website/en/` 和 `website/api/` 等 locale 或 route directory 都是 invalid source layout。Generated catalog 保留在 `docs/` 下，在那里接受 freshness gate，并通过 manifest 发布。

## 分类变更

- **编辑已发布 page：** 只修改 canonical Markdown source。除非 route 或 navigation metadata 改变，否则不触碰 manifest。
- **发布新 page：** 在所属 `docs/` tier 创建，再添加一条 manifest entry。
- **Rename、move 或 remove page：** 原子更新 canonical file、manifest entry 和 inbound repository link。移除 stale manifest entry；`docs:check` 会拒绝 missing source。
- **发布 generated catalog：** 映射 generated `docs/` file，但修改其 generator 或 source metadata，不要手动编辑 catalog。
- **修改 site structure：** 普通 page 更新 manifest；只有已有 sidebar、section 或 locale model 无法表达变更时才更新 VitePress configuration。

Manifest 保持为显式 public allowlist。不要仅因 RFC、postmortem、testing guide、`AGENTS.md` 或 maintainer workflow 位于 `docs/` 下就发布；只有用户明确扩大 site 发布内容时才添加 internal material。

## DocsPage field

有意设置每个 `DocsPage` field。Canonical field set 和 `DocsSidebar` union 位于 [website/docs.ts](../../../../website/docs.ts)——直接读取，不要把 value 复制到 prose；section 由该 file 中的 `sections` record 负责，VitePress config 不另设 order list。

- `source`：repository-relative canonical Markdown path。完整 bilingual pair 通过 `pairedPages()` 添加 English `.md` path；它会推导 sibling `.zh.md`、content locale 和 counterpart alias。
- `route`：包含 `.md` suffix 的 public VitePress path。
- `label`：sidebar label，不一定等于 document H1。
- `sidebar`：复用已有 `DocsSidebar` collection，除非 information architecture 确实需要另一个。
- `section`：尽量复用 existing section。新增时也在 `sections` record 定义。
- `order`：section 内 stable order。
- `sourceAliases`：可选的 additional repository path，在投影 link 时解析到此 page。它不创建另一条 public route。

只有 source 有意在两个 route tree 中回退到同一种可用语言时才使用 `mirroredPages()`。添加 counterpart 后，将 entry 转成 `pairedPages()`。Site route tree 独立于 source layout：`foo.zh.md` 投影到 root route，`foo.md` 投影到对应 `/en/` route。

## 保持 link behavior

Canonical doc 使用普通 repository-relative Markdown link。Projector 应用以下规则：

- Manifest 中存在的 target 转换为 site-relative route。
- Manifest 外存在的 target 转换为 GitHub source link，包括受支持 line suffix。
- Image 是例外：其 file 被复制到 generated tree 并从那里引用，因此无论 repository visibility 如何，site 都会提供它。它必须是 repository 内的 regular file。
- External URL、site-absolute URL、email link 和 fragment-only link 保持不变。
- 缺失的 repository-relative target 会使 projection 失败，而不是静默生成 broken link。
- Cross-page fragment 使用 English GitHub heading id 作为 canonical id。如果 authored heading 生成不同 VitePress id，在它前面紧邻放置显式 `<a id="..."></a>`；generated alias 添加到所属 generator。

不得仅为满足 VitePress 而在 canonical Markdown 中写 website-specific route。对于应解析到 mapped index page 的 directory-style repository link，使用 `sourceAliases`。

## Preview 与 validation

编辑时运行 local preview：

```sh
pnpm docs:dev
```

Dev server 会 watch mapped source file 并重新 projection。Manifest 变更后，如果新 source 未自动拾取，则 restart。

将 mapping 视为 valid 前，运行 focused website gate：

```sh
pnpm docs:check
```

如果 Markdown link check 通过，但 site build 报告 missing fragment，沿 `verify-doc-site-fragments` 的 source 和 target path 检查。在 authored Markdown 或所属 generator 中以显式 alias 保留 English GitHub id。

Commit documentation-site change 前运行：

```sh
pnpm run test:docs
pnpm run doc-sync
pnpm run lint
git diff --check
```

Push 前使用 [dsh-pre-push-checks](../../dsh-pre-push-checks/SKILL.md)。报告 changed canonical file、新增或移除的 manifest entry、受影响 public route 和实际运行的检查。

## 保持 deployment 独立

将 content 同步进 VitePress build 不会发布到 internet。除非用户明确请求 deployment 并确认 hosting policy，否则不得添加 GitHub Pages permission、deployment workflow、custom domain 或 public hosting。

## Dev Note

无。
