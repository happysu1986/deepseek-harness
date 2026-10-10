# Leakage few-shot 示例

这些示例提炼自 2026-08 repository-wide purge 及其 review round。使用它们识别 governing principle，不要作为 text template。本文件有意引用 leaked wording 作为校准材料——[recall battery](recall-batteries.md) 排除本 skill directory，其 wording 不授权在其他位置使用。

## Dead citation

### 有 committed owner 的 decision ordinal

**Leaked：**「Slash input 根据 visible catalog 解析（decision 21）。」

**Fixed：**「Slash input 根据 visible catalog 解析——plain-text-reference decision 由 [web input-machine note](../../../notes/archived/architecture/2026-07-25-web-input-machine-and-slash-pipeline.md) 负责。」

Ordinal 在 HEAD 无法解析；decision name 和 owning note path 可以。每个 file 至少一次命名 owning note path——surface 支持时使用 link——之后可以只使用可搜索 name。

### 没有 owner 的 decision ordinal

**Leaked：**「Registry 拒绝 duplicate name（decision 7：name 是 flat，不支持 namespacing）。」

**Fixed：**「Registry 拒绝 duplicate name；name 是 flat，不支持 namespacing。」

没有 committed artifact 负责 "decision 7"，因此删除 citation——但 factual clause（flat name）要重述为独立内容，不能一起删除。

### Audit item code

**Leaked：**「Rendering 是 pure：相同 snapshot 产生相同 string（audit R3）。」

**Fixed：**「Rendering 是 pure：相同 snapshot 产生相同 string。」

Repo 中没有 audit document；code 是不承载 proposition 的 session shorthand。

### Uncommitted draft 的 section number

**Leaked：**「Layering 遵循 design（v2 §3.2）：`src/core/` 是 pure core。」

**Fixed：**「Layering：`src/core/` 是 pure core。」

无人 commit 的 draft `§N` 无法解析。对比："escapes per RFC 9110 §10.1.5" 保留——external standard 按设计在 repo 外解析；拥有 § 编号的 committed doc 也可按 section 引用。

### Plan-phase label

**Leaked：**「`src/client/` 是 shell（T4）；P-I migration 负责 adapter。」

**Fixed：**「`src/client/` 是 shell；adapter 位于 `src/client/adapters/`。」

Phase label 索引未落地 plan。以 phase 产出的事实替换 label。

## Stack 与 PR vantage

### Durable prose 中的 stack position

**Leaked：**「Future remote backend 实现此 interface（sandbox backend 是此 stack 中较后的 PR）。」

**Fixed：**「Remote backend 可以实现此 interface，而不改变 render layer。」

Durable prose 看不到 stack。保留 extension-point contract；pending work 放在 PR、`TODO` 或 issue 中。

### README 中的「This PR」

**Leaked：**「This PR 为 session list 添加 cursor-based pagination。」

**Fixed：**「Session list 按 cursor pagination。」

README 比任何 PR 都长寿；把 mechanism 写成 current fact。

## Change narration 与 version stamp

### 带 PR number 的 war story

**Leaked：**「Color 以前来自未定义的 `--widget-*` token，因此始终 render fallback；alias token 修复了它（PR #88）。」

**Fixed：**「Color 来自 alias token；undefined token 会 render fallback。」

两个 live fact 都保留——current mechanism 与长期 failure behavior——并用 present tense 重述。Bug biography 属于 PR 及其 Agent Note。

### Removal narration

**Leaked：**「`probe` field 已随 removal cut 消失；badge 现在使用 generic projection pair。」

**Fixed：**「Badge 使用 generic projection pair。」

从未见过 `probe` 的 reader 无法从其 absence 获益。与 deleted past 对比的 "now" 是 version stamp。

### Fixed regression → counterfactual present

**Leaked：**「这以前会 double-encode multibyte label。」

**Fixed：**「没有 byte-length guard 时，multibyte label 会 double-encode。」

Regression pin 作为命名 guard 的 present-tense counterfactual 保留；"used to" 将其绑定到 repo archaeology。

### Indexical version stamp

**Leaked：**「Batch rendering 在 this cut 中是 synchronous；async path 是 roadmap work。」

**Fixed：**「Batch rendering 是 synchronous。」（Deferral 位于 call site 的 `TODO(widget-batch):`。）

"This cut" / "v1" / "today" merge 后立即过时。Agent Note change-story section 中的 historical stage name（"the first cut shipped X"）current-state-safe；indexical form 从不安全。

## Review choreography

### 作为 prose 的 review verdict

**Leaked：**「Rejected in review：缓存 resolved spec。我们保持 per-call resolution。」

**Fixed（Agent Note 的 Alternatives considered 中）：**「**缓存 resolved spec。** Rejected：spec 依赖 per-call cwd，因此按 request keyed 的 cache 会返回 stale root。」

Alternatives-considered genre 是允许位置；reviewer 与 round 不属于 rationale。

### Draft ordinal

**Leaked：**「截至此 note v5，loader 也会验证 manifest。」

**Fixed：**「Loader 验证 manifest。」

Implemented note 陈述 shipped reality；自身 revision history 位于 git。

## 面向 reviewer 的 justification

### 为 cast 辩护

**Leaked：**「Cast 是安全的——SDK 构造 object，只是没有足够严格地声明 optional。」

**Fixed：**「SDK 构造此 object 时填充所有 optional；declared type 比 runtime guarantee 更宽松。」

陈述 maintainer 不得破坏的 invariant。"It simply…" 是在回答 HEAD 上不存在的 objection。Invariant 在 code 中显然时删除 comment。

### 诉诸 review authority

**Leaked：**「这是正确的，因为 reviewer 确认了 wrapping order。」

**Fixed：** 删除；wrapping order 已在 function `@returns` 中陈述。

Correctness claim 引用 invariant 或 test，绝不引用人。

## Restatement 与 derivation

### Control-flow narration

**Leaked：**「先 normalize label，再 truncate，最后 wrap。」

**Fixed：** 删除。

Comment 下方三行 code 已表达相同内容。

### Test walkthrough

**Leaked：**「此 test 创建 session，发送两条 message，等待第二条 reply，再断言 log 有四个 entry。」

**Fixed：**「两个 round-trip 必须产生恰好四个 log entry——projection 对 shared prefix 去重。」

只保留 non-obvious assertion rationale；walkthrough 复述 test body。

## Hedge 与 planning residue

### 无 marker 的 deferral

**Leaked：**「Probably fine to render eagerly for now。」

**Fixed：** 删除；deferral 已有 `TODO(widget-batch):` marker。

没有 owner 的 hedge 是 planning residue。没有 marker 时添加一个（`TODO(name): coalesce per animation frame`），不要保留 hedge。

### Vague sizing

**Leaked：**「64 KiB buffer 对多数 case 应该足够。」

**Fixed：**「64 KiB 能容纳 observed 最大 frame（48 KiB）并留有余量；更大 frame 在 `decode` 中显式失败。」

以 actual bound 和超限 failure behavior 替换 hedge。

## Authoring-language slip

**Leaked：**「The renderer runs on the client 端; see the 设计稿 for spacing. ---- 私有 ----」

**Fixed：**「The renderer runs on the client side; spacing follows the Figma frame `widget-badges`.」

Working-language fragment 和 session separator 是 transcription residue。Figma frame name 保留：它是按设计在 repo 外解析的 external source。

### Paired fence 内的 authoring-language slip

**Leaked（两个 file）：** `// 更新这里 before returning`

**Fixed（两个 file）：** `// Update this before returning.`

修复 block 一次，再把 byte-exact fence 复制到两个 language file。在 Chinese counterpart 中另行翻译 code comment，即使两条 comment 都流畅，也会破坏 pairing contract。

## Behavior-visible candidate

**Suspect：** Exported JSDoc sentence 写 "available for now"，generator 将其复制到 model-visible catalog。

**Wrong：** 在 prose-only purge 中只重写 source sentence，或只手动编辑 generated catalog。

**Right：** 追踪 source 的 generated fan-out，更新 owner，重新生成每个 derivative，并更新 owning runnable snapshot。GUI string 还要包含仓库要求的 behavior evidence。Authorized scope 没有 owning scenario 时，保持 wording 不变并报告 defer。

## Keep

### Issue reference 在所有 surface 都持久有效

**Keep：**「Cap 应用于完整 rendered value，包括 wrapper（issue #1470 负责 follow-up）。」

Unaided pass 认为 issue citation 属于 Agent Note，因而删除了它，方向错误：issue 可从任何 surface 在 HEAD 解析，"#N owns the follow-up" 是 README 中 deferred work 的允许位置。Agent Note 与 postmortem 额外允许的是引用 *merged PR* 作为 evidence。

### Dead name-drop 不等于「命名 owner」

**Delete：**「Widget seam 上的 badge renderer（见 widget-rendering RFC）。」

Unaided pass 把它当成「按 topic 命名 owning document」而保留。测试标准是 resolvability，不是形式：没有 committed file 叫 "widget-rendering RFC"，因此 pointer 已 dead。有 committed owner 时 retarget，否则删除。

### Suppression justification

**Keep（修复后）：** `// oxlint-disable-next-line no-non-null-assertion -- the one-element literal guarantees index 0.`

Justification clause 是 required prose。Stated reason 为 false 时（原文说 "the loop guard above proves a frame exists"，但附近没有 loop），修复 reason；绝不能删除。

### Measured bound

**Keep：**「Depth cap（measured: 512 nests ≈ 0.15s synchronous；4096 blocks the loop）。」

Measurement 使 constant 不会被无依据地重调，"measured" 表明数据不是猜测。

### Runtime old/new 不是 change history

**Keep：**「Old connection 在 new one 接受前 drain。」

这里的 "old" 和 "new" 命名 handover 中两个 live runtime object，不是 repository state。Change-narration 禁令针对 repo history，不针对 lifecycle vocabulary。

### Runtime natural time 不是 version stamp

**Keep：**「What is today's date?」

Prompt 询问 runtime clock；"today" 不对比 repository state。因为 text 到达 model，任何重写仍需要 owning behavior evidence。

## Overcorrection trap

下列每项在原始 purge 中进入提交，并在 review 时被发现。精简前枚举 passage proposition。

### 将 obligation 翻转为 endorsement

**Original：**「这些 direct registration 是等待迁移到 slot 的 exception。」

**Overcorrected：**「这些 direct registration 是 sanctioned exception。」

**Right：**「这些 direct registration 是等待迁移到 slot 的 exception。」

"Pending migration" 是 obligation；"sanctioned" 认可 status quo。精简在缩短 sentence 时反转了 modality。

### 将 hypothetical 提升为 shipped feature

**Original：**「Future IPC-based shell 继承 executor 并 override `spawn`。」

**Overcorrected：**「IPC-based shell 继承 executor 并 override `spawn`。」

**Right：**「Hypothetical IPC-based shell——当前不存在该 shell——会继承 executor 并 override `spawn`。」

只删除 future marker 会把 design illustration 变成 class 已 shipped 的 claim。明确标记 hypothetical，不要只取消 future 标记。

### 连同 transcript 删除 true fact

**Original：**「Gate notice 叙述 check order；notice text 也是 `verify-doc-typecheck` 编译的内容。」

**Overcorrected：** 整句删除，认为是 narration。

**Right：**「Notice text 是 `verify-doc-typecheck` 编译的内容。」

Sentence 一半是 narration，另一半是 load-bearing coupling。多个 proposition 共用一行时删除 clause，不删除整个 sentence。

### 保留数字但丢失 measurement source

**Original：**「4 MiB ceiling 是 measured：最大 generated `py-types` module 为 3.1 MiB。」

**Overcorrected：**「Ceiling 是 4 MiB；最大 generated `py-types` module 为 3.1 MiB。」

**Right：** 保留 "measured"。

没有 "measured"，3.1 MiB 看起来像 definition 而不是 observation，之后提高 ceiling 时也没人会重新测量。
