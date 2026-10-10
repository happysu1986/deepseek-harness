# 精炼的 prose 示例

使用这些示例识别 governing principle，不要把它们当作 text template。「Balanced」以当前位置所需的最少 explanation 保留每个 load-bearing proposition。

## 保留每个 factual clause

**Original：**「Coordinator 对每个 session 串行写入，在 disposal resolve 前 flush buffered event，并向 caller 报告 backend failure。」

**Over-trimmed：**「Coordinator 串行化 persistence。」

**Balanced：**「Coordinator 对每个 session 串行写入，在 disposal resolve 前 flush buffered event，并向 caller 报告 backend failure。」

删除装饰和重复，不删除 proposition。Actor、per-session scope、disposal ordering 和 failure visibility 是不同事实。

## 显式 skill scope 具有功能意义

**Over-trimmed：**「阅读 source 并自行判断。」

**Balanced：**「本 skill 是指导原则，不是完整 checklist。除具名检查外还要自行判断；已记录 requirement 仍然适用。」

**Over-detailed：** 用数个 paragraph 论证 list 为何不能取代独立 reasoning。

保留显式 limitation，因为它改变 agent 应用 workflow 的方式。删除重复 persuasion，不删除 guardrail。

## Cookbook 保留 action 与 verification

**Over-trimmed：**「为 tool 添加 test。」

**Balanced：**「在 unit level 测试 registration 与 disposal，通过真实 loader path 运行 tool，并在 rendered output 变化时添加 snapshot。确认 assertion 观察 external result，而不是 model report。」

**Over-detailed：** 逐个讲解 example code 已经展示的 fixture file 和 assertion。

保留 test tier、required action、real entry path 和 observable verification。删除 fixture narration。

## 保留 ownership 与 timing

**Over-trimmed：**「Teardown 时取消 provider work。」

**Balanced：**「Runtime 在释放 child scope 前请求 provider cancellation；provider 仍负责在 disposal resolve 前 join worker。」

**Over-detailed：** 按时间叙述实现 teardown 的每个 promise 和 callback。

Actor、ordering、ownership change point 和 completion guarantee 是不同 factual clause。

## Event JSDoc 保留 boundary timing

**Over-trimmed：**「组合并缓存 session prefix。」

**Balanced：**「在第一次 pre-step 和 model request 前组合一次 session prefix。Listener append 加入当前 request，pre-step pressure accounting 接收组合后的 prefix。」

**Over-detailed：** 讲解实现 ordering 的 loop helper、cache field 和 promise callback。

Event order 及其 current-request consequence 是 caller-visible behavior，不是 implementation narration。

## 定位复杂 code，但不叙述它

**Over-trimmed：**「Worker realm support。」

**Balanced：**「拥有 worker realm 及其 host bridge。Realm initialization 只能执行一次；disposal 终止 worker，并拒绝后续 call。Protocol rationale 见 worker-isolation Agent Note。」

**Over-detailed：** 逐 paragraph 预告下方 class 与 helper function。

保留 module role、dependency、responsibility 和非显然 lifecycle behavior。链接 architecture rationale，让 code 自己展示 local control flow。

## Public JSDoc 包含 failure

**Over-trimmed：**「返回 realm global。」

**Balanced：**「返回已初始化 realm global。Initialization 尚未完成或 realm 已 dispose 时 throw。」

**Over-detailed：** 导致每项 throw 的 internal state-machine branch 和确切 helper call。

Throw 和 state precondition 是 caller-visible contract fact。

## 保留精简 implementation mapping

**Over-trimmed：**「由 external API 支持的 search provider。」

**Balanced：**「将每个 provider result 映射到 shared search-result field，保留 title、URL 和 text，同时省略 provider-only ranking metadata。」

**Over-detailed：** 逐 field 复述 mapping code，包括同名 field 和显然 assignment。

保留解释 adapter 在哪里丢弃或修改信息的 mapping detail。

## 链接 rationale，同时保留 local contract

**Over-trimmed：**「Disposal 记录在 lifecycle Agent Note 中。」

**Balanced：**「Disposal abort run，并等待 provider quiescence。Ownership 和 race handling 见 lifecycle Agent Note。」

**Over-detailed：** 在每个 disposer 旁重复 Agent Note 的 promise choreography 和 rejected ownership model。

在 caller 需要的位置保留 behavior 和 completion guarantee。Algorithm 与 rationale 应积极链接；link 不能替代 local contract。

## Implemented Agent Note 保留 verification contract

**Over-trimmed：** 因 Agent Note 已经 shipped 而删除整个 Testing section。

**Balanced：**「Unit test 覆盖 publication 前后 cancellation、disposal quiescence 和 provider reload。Built-entry smoke 覆盖真实 loader path；snapshot coverage 因 transport 为 process-specific 而 defer。」

**Over-detailed：** 逐 file 讲解 fixture 和 assertion，却没有增加 behavioral distinction。

删除 migration task 和 test narration。保留 test tier、固定的 behavior、real entry path 和具名 coverage gap。

## Security boundary 可能需要一个具体示例

**Over-trimmed：**「Mounted plugin 共享 host authority。」

**Balanced：**「Mounted plugin 共享 host authority；例如，访问 `ctx.shell` 可以使用 host executor 的 privilege 执行 command。」

**Over-detailed：** 列出 plugin 可能滥用的每个 service 和每个 hypothetical exploit。

一个示例能让抽象 security limit 在操作上清晰时，保留一个示例。

## 完全删除 reasoning transcript

**Over-detailed：**「Loop 首先检查 value 是否 absent。如果 absent，下一个 branch 提前 return。否则继续，因此 final assertion 是安全的。」

**Balanced：** Code 已表达这些 branch 时不写 comment。如果 early return 保护非显然 invariant，只陈述该 invariant。

不要把 reasoning transcript 压缩成更短 narration；应删除它。

## Configuration comment 解释 tree 无法表达的内容

**Over-detailed：**「此 entry 加载 local filesystem provider，然后加载 policy plugin，再加载 read、write 和 edit tool」，而相邻 entry 已显示该顺序。

**Balanced：**「在 model-facing tool 前加载 policy，使其 write 和 edit call 经过 read-before-mutation gate。」

保留 ordering consequence、surprising scope rule 或 security boundary。让 configuration 自己展示 inventory。

## 不得仅为 word count 精简

**Current：**「Adapter 将 provider error 转为 shared error type，使 caller 统一处理 authentication、rate-limit 和 transient failure。」

**Shorter but worse：**「Adapter normalize provider error。」

**Balanced decision：** 除非 link 或 surrounding contract 已列出 failure category，否则保留 current sentence。Shorter version 丢失 consequence 和 distinction，却未改善 structure。

## Model-visible text 遵循 ownership

**Over-trimmed：**「Call 失败时 tool 返回 error。」

**Over-detailed：** 把另一 package 的 schema 和 renderer string 复制到本 backend README。

**Balanced：** 引用本 package 所有的 stable prompt、result 和 error text。Schema 链接 generated tool catalog；其他 package 所有的 text 链接 consumer README；局部只陈述本 package 的 condition 或 delta。

到达 model 的 wording 是 behavior，但重复仍会 drift。Exactness 属于 owner。

Prose-only audit 可以识别可疑 wording，但没有 owning runnable snapshot 时不得静默修改。保持不变并报告 defer，或扩展 authorized change，包含 owner scenario 和 required GUI evidence。

## Generated summary 必须独立完整

**Over-trimmed：**「Approval request and policy service。」Owner 在后文解释 policy order 和 audit logging，但 catalog 只 export 首句。

**Over-detailed：** 将 service 的完整 lifecycle 和 prompt-notice behavior 移进 extracted sentence。

**Balanced：**「Approval service 在 answerer 前应用 session policy，并把每组 ask/outcome 记录到 requesting session。」Non-catalog detail 留在后续 sentence。

要知道 generator 提取什么。该 fragment 必须保留其 generated output 所需 contract。

## Limitation 是 contract，不是 debt inventory

**Over-trimmed：** 省略一个 process-lifetime cache，导致 configuration change 需要 plugin reload。

**Over-detailed：** 列出对 caller 或 maintainer 没有 consequence 的 private helper cleanup 和 unused test-only accessor。

**Balanced：**「Provider selection 在 plugin lifetime 内缓存；安装或修复 provider 后需要 reload。」普通 cleanup 留在 TODO 或 Agent Note。

保留影响使用或安全维护的 gap 和 non-obvious constraint。Package README 不是 backlog dump。
