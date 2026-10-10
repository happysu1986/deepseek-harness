---
name: dsh-client-ui-ux
description: 设计和评审 DeepSeek Harness client UI 变更，包括 visual token 纪律、先复用再新增、feedback surface（toast、in-place notice 或 empty state）、overlay 和 menu 安全性、platform window 适配、loading state 与 error copy 位置。在 packages/client 中新增或修改 product-user-visible GUI behavior，或评审此类 PR 时使用。
---

# DeepSeek Harness Client UI/UX

本 skill 是指导原则，不是完整 checklist。它覆盖 lint、typecheck 和 i18n gate 无法判断的取舍：feedback 出现在哪里、复用什么，以及 surface 在 window edge 和不同 platform 上如何表现。Copy quality 遵循 [dsh-prose-standard](../dsh-prose-standard/SKILL.md)；locale ownership 遵循 [locale-owned copy decision](../../notes/implemented/architecture/2026-08-23-locale-owned-client-ui-copy.md)；styling ownership 和 token 规则见 [docs/web-styling.md](../../../docs/web-styling.md)与 [ui-primitives component catalog](../../../packages/client/ui-primitives/README.md#component-catalog)。本 skill 在这些规则之上提供判断，不复制第二份规则。

## Visual foundation

- **Color、radius 和 spacing 使用现有 token。** 引入 literal value 前，搜索目标 page 及最接近的成熟 sibling（full-page surface 通常参考 Plugins page），找到已在使用的 token 或 value 并沿用。完整 treatment 同样如此——toast、menu、in-message artifact preview、sidebar tab layout、color、clearance rule、page padding：只要已有成熟 component 或 rule，就复用，不得发明平行方案。
- **在 light 和 dark mode 中验证每种 color。** Ink/background 组合在两个 theme 下都必须可读才能交付。Dimmed tone 是长期反例：适合 decorative surface，但用于 text 时经常在某个 theme 下可读性不足；未检查两个 theme 时，绝不能把 dimmed color 用于 text。
- **新增或修改的 feature CSS 中 font weight 最高为 500。** 不得用 600+ 强调；改用 size 或 ink 区分。由 ui-theme 所有的 markdown typography token 是固定例外。
- **已有 role 匹配时，从 theme typography variable 选择 font size，否则使用 page 的既有 scale。** Parallel content 使用同一 size；绝不能为单个 element 发明 size，并保持 page 上 distinct size 数量较少（13px 是既有 row/detail size）。Page 自身没有先例时，借用最相似现有 page 或 element 的 size 与 spacing ratio。
- **Icon、text 和相邻 element 保持比例与对齐。** 每个 element 都沿明确 axis（baseline、vertical center 或 shared edge）与 sibling 对齐，icon size 与旁边 text 保持比例；绝不能交付没有对齐对象或在视觉上压过 neighbour 的 element。

## 新增前先复用

- **创建新 component、container 或 interaction 前先扩展现有实现。** 新 capability 能放进已有 surface 时（增加一个 menu item、为 existing primitive 增加 prop），应优于堆在旁边的 parallel element。
- **Icon 来自现有 icon library。** 选择语义最接近的现有 icon；新增 glyph 必须先获得 designer approval——评审添加 glyph 的 PR 时，建议请求 designer review。
- **Right-sidebar content 在现有 sidebar 注册 slot**，绝不能另建 sidebar。每个 sidebar tab 都声明符合其含义的 tab icon。
- **含义不明显的 icon-only action 要有 Tooltip。** Pointer 必须停留或需要选择的 informational content 使用 HoverCard。优先使用这些 shared primitive，不要使用临时 title attribute。

## Feedback surface

根据 message 相对于触发它的 surface 的生命周期选择承载位置：

- **Transient operation outcome 使用 app-wide Toast primitive**，不使用 in-place notice；toast 的 state 和 rendering 放在比报告 surface 生命周期更长的 host 中（例如 RowActionToast 所用的 `shell.overlay` entry）——由 panel 自己 render 的 toast 会随 panel unmount。Deletion 可能关闭发起操作的 panel 或 tab；只有由该 surface 外部持有的 toast 能在关闭后保留。报告 success 和 failure，不报告仍 pending 的 state。
- **Operation 失败时保留 data 可见。** 删除失败要保留 row，由 toast 宣告 failure。不得为显示 transient operation error 而清空 content。
- **In-place notice 用于与 surface 本身绑定的 state**：带 Retry action 的 query failure、拥有 field 的 form 上的 field validation。有内容的 panel 在保留 content 旁显示 compact notice；只有 empty panel 才把完整 error state 居中。
- **Error copy 使用简短 plain language。** 除非必须暴露 error code 供 debug，否则不用 internal jargon 或 technical noun。优先一句清楚的话；能用两句说明的内容不得扩成长段。最多两句的中文 notice 不加末尾句号（。）。Notice 不得破坏现有 layout——为它预留空间或使用 overlay，绝不能推移相邻 element。

## Loading state

- List 使用对应 skeleton；其他 page-level load 居中显示纯 spinner。绝不能把 spinner 放在 page 角落。
- 每个 page/component 只使用一种 loading treatment——并发 region 不得各自闪现不同 style。
- 除非 context 确实需要，否则不显示 loading text（狭窄 sidebar 几乎从不需要）。

## Overlay 与 menu

每个 menu、popover 和 tooltip 在 merge 前都必须验证以下三项：

- **Dismissable**：outside click（可 focus 时还包括 Escape）能够关闭。
- **Viewport-fitting**：通过 slide 或 flip 保持在 window 内并留出 edge margin；只有另一侧确实容得下时才 flip（两侧都放不下时不得 oscillate）。
- **Unclipped**：避开 `overflow` ancestor 和 containing block（需要时 portal 到 body），且绝不被相邻 element 遮盖。

Scrollbar 保持在 container 内：使用 shared scrollbar style，不使用 component-specific selector；验证 bar 既不溢出 container，也不会被 rounded corner 或 edge clip——保留少量 inset，不要紧贴 container edge。

## Platform window 适配

- 任何新 header element 或 full page 都必须考虑 macOS 和 Windows window chrome。在 macOS 上，titlebar clearance 是 `--dsh-frame-top-clearance`，由 Menu、Modal 和 overlay primitive 作为 top safe margin 使用——**绝不能为修复一个 page 修改 global variable**。应在 page 自身 inset 上添加 scoped `[data-platform='darwin']` override，并在 page stylesheet spec 中断言该 override。
- 评审时说明预期 total offset（clearance + page inset），使两个按「相同 top spacing」对齐的 page 对总和达成一致，而不只是其中一个加数。

## Spacing review

- 检查每个 changed region 的 gap：除非有意如此，否则任何内容都不能紧贴 neighbour；相邻 icon 和 element 保持该 region 的既有 spacing。
- Parallel element 优先保持 symmetry；无法解释的 one-off gap 或 offset 通常意味着遗漏了 shared value。

## 升级 design review

- 包含大量 UI 变更的 PR，或任何用户明显可感知的 UI/interaction 变更，都必须接受 designer 或 product review。没有 approver 担任该角色时，建议从 design/product reviewer 中请求评审（2026-09 roster：yx zhang、yifffan、zhangziya、gaokaige；成员变化时更新此列表）。
