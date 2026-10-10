---
name: dsh-code-review
description: 评审 deepseek-harness 仓库的 pull request 时使用，使 reviewer 了解本代码库的标准（AGENTS.md 约定、defensive pattern、ADR、quality gate）以及无法仅从 code 看出的 review-specific 检查。
---

# 评审 DeepSeek-Harness PR

**本 skill 是指导原则，不是完整 checklist。** 验证并 fetch PR 的 live base 和确切 head，然后在阅读 diff 及足以理解设计的周边 code 前，运行 `pnpm --silent run change-scope --base <verified-base-ref> --head <verified-head-ref>`。报告会识别 path 和 dirty layer，但不能取代 semantic review。Retarget 或 merge 后重新确定 base 并再次运行。Correctness、lifecycle、security 和被破坏的 required behavior 优先于 style；只有一个证据充分 blocker 的简短 review，优于一串 nit。

## 权威来源

- [AGENTS.md](../../../AGENTS.md) 和 [packages/AGENTS.md](../../../packages/AGENTS.md)：长期有效的 repository 与 package authoring rule。
- [docs/defensive-patterns.md](../../../docs/defensive-patterns.md)：subprocess、callback、async-state 和 disposal bug class。
- [docs/AGENTS.md](../../../docs/AGENTS.md)：文档位置和 prose 纪律。
- [dsh-prose-standard](../dsh-prose-standard/SKILL.md)：comment、doc、prompt 和 visible string 的必需覆盖范围与编辑判断。
- [dsh-ci-test-reliability](../dsh-ci-test-reliability/SKILL.md)：resource-owning、asynchronous 或 flaky test/fixture 的 isolation 与 regression-proof 规则。
- [dsh-client-ui-ux](../dsh-client-ui-ux/SKILL.md)：product-user-visible GUI 变更的 feedback-surface 选择、overlay safety、platform window 适配和 visual token 纪律。
- [docs/testing.md](../../../docs/testing.md) 和 [quality-gates Agent Note](../../notes/implemented/process/2026-06-11-quality-gates.md)：必需 test tier 与 gate。
- [Agent Note](../../notes/README.md)：design rationale。与 Agent Note 的分歧应视为 design discussion，不自动构成 veto。
- 对双语变更，阅读 [translation-rules.md](../../../docs/i18n/translation-rules.md) 和 [terminology.md](../../../docs/i18n/terminology.md)；扩展 translation skill 不属于自动 review，仅在用户明确调用时运行。

## Blocking requirement

1. **新 prose 接受 semantic review。** 使用 [dsh-prose-standard](../dsh-prose-standard/SKILL.md) 严格评审每段新增或修改的 Markdown、JSDoc、comment、prompt、description、diagnostic 和 visible string。对照所属 code 或 behavior 验证 required coverage、accuracy、placement 和 editorial quality；automated check 无法证明这些属性。
2. **Doc 与 code 一致。** Config、default、error、wire field、event 和 public behavior 要在同一 diff 中更新 package README 与 JSDoc。Comment 陈述非显然 contract；标记 implementation narration、test walkthrough、review history 和 duplicated rationale，要求删除或链接到唯一归属位置。
3. **Core type doc 一致。** Spine 或 seam vocabulary 变更要更新相应 [subsystems](../../../docs/subsystems/README.md) page 和所有 `type-equiv` entry。Internal type 不需要 catalog entry。
4. **Registration 完成 cleanup。** 验证每个新 registry contribution 都通过 [packages/AGENTS.md](../../../packages/AGENTS.md) 要求的 disposal test。
5. **Invariant companion 具有语义。** 对每个触及的 `./invariant`，要求在该 package 可观察的位置存在由 owner 管理的 event-stream 或 mutable-data relationship，且 observation 相互独立；service 或 method 是否存在、plugin metadata 或 effect、固定 pure example，以及调用自己声称验证的同一 operation 的 probe，应放入 load、behavior 或 unit test。不存在合理 relationship 时，要求 package 省略 companion 及 publication wiring，并在 README 中记录 package-specific 原因。拒绝 empty installer 和凭空发明的检查（[repository rule](../../../AGENTS.md#conventions)；[package invariant rule](../../../packages/AGENTS.md)）。
6. **存在 required evidence。** 验证 author 已针对 diff 运行[相关 local check](../../../AGENTS.md#run-relevant-checks-locally)，并由 CI 覆盖 exhaustive matrix；评审二者都无法发现的 semantic gap。
7. **Client UI copy 归 locale 所有。** 拒绝嵌入 JSX、template、helper return、accessibility attribute 或 primitive default 的 product text。要求 typed dictionary key、标准 `t` seat 或显式 localized prop、`verify-client-ui-i18n`，以及每个受影响 locale 的 behavior 证据；user/model/wire data 和 code token 保持 verbatim。

## 手动检查

- **Intent 与 interface contract：** 追踪每个 changed interface 的两侧。确认 implementation 与 PR 及所有 Agent Note 一致，包括 error、cancellation、ownership 和 disposal。
- **Lifecycle 与 concurrency：** 对 async setup、callback、process 或 teardown，应用 [defensive-patterns.md](../../../docs/defensive-patterns.md)。检查 publication 前 race、await 期间 cancellation、独立 error reporting、callback containment、reentry 前 ownership、完整 detach cleanup 和 quiescent disposal。
- **Capability 与 consumer 匹配：** 追踪每个 current consumer，然后根据 [package rule](../../../packages/AGENTS.md) 标记泄漏到 interface 中的 consumer-specific behavior。也标记反向问题：generic service（registry、session、agent）新增 public method，但唯一 caller 是一个 internal consumer，属于不必要的 API expansion——要求改为在 construction 时交给该 consumer 的 private capability closure。
- **Scope、ownership 与 necessity：** 将每个 abstraction、state machine、option、defensive copy 和 compatibility path 映射到其 current contract、production consumer 和 owning plugin/service。质疑 unrelated feature 和 speculative generality，再根据 [root rule](../../../AGENTS.md#conventions) 检验 PR。
- **Configuration 与 public choice：** 询问哪些 current-consumer evidence 或 prior art 支持每个 default、public operation set、format 或引入的 external concept。缺少证据时，要求明确选择或 defer。
- **Model perspective：** 检查受影响 mode 中 model 收到的确切 prompt、tool schema、result 和 diagnostic。标记超出 model task 的 concept，然后通过 snapshot 或 end-to-end coverage 验证 stable text verbatim 和 dynamic behavior。
- **Enforcement：** 沿每条 denial path 追踪到实际执行 operation 的位置；测试可绕过 schema、prompt、facade、wrapper 或 listener ordering 的 direct 与 alternate caller。
- **Borrowed 与 derived state：** 根据 package contract 确定每个 retained value 是 borrowed 还是 owned，然后追踪 notification 以及每个 cache、prompt、UI echo、replay 和 query view，直到有文档说明的 success point 与 authoritative source。
- **Bound 覆盖最终 operation：** 定位完整 emitted 或 retained result 的 owner，包括 wrapper 和 metadata。对 byte limit 测试极小值、精确 limit、oversized single chunk 和 multibyte text。
- **真实 entry path：** Test 在适用时运行 shipped Loader、bin、worker、ACP bridge 或 subprocess。Hand-mounted plugin 无法发现 invalid Loader export；function plugin 必须 named-export 其 namespace，且不能有 default export。
- **Test strength：** Assertion 针对目标 regression 失败，并验证 external state、log、event 或 disposal，而不是复述 implementation 或相信 agent report。Coverage 是必要条件，但不能证明 scenario 正确。
- **Test reliability：** 对 resource-owning、asynchronous、platform-sensitive 或 flaky test，使用 [dsh-ci-test-reliability](../dsh-ci-test-reliability/SKILL.md) 检查真实 worker/job topology、resource allocation、global-state restoration、synchronization、timeout budget 和 quiescent teardown。
- **Invariant lifecycle 与 negative control：** 可行时验证 candidate observation 在 publication 前被拒绝；session-backed check 在 late loading 或 HMR 后能够重建 durable history；刻意 invalid case 通过真实 runner 因目标规则而失败。
- **Implemented Agent Note 与 shipped reality 一致：** PR 实现 proposed Agent Note 时，在同一 diff 中移动并重写为 present-tense shipped state，然后根据 implementation 验证 path、name 和 mechanism。
- **Transcript 变更：** Editor-visible 或 model-visible 变更要更新 snapshot，或解释为何不适用 snapshot。将 expected-output diff 作为 behavior change 评审，而不是 formatting noise。
- **双语变更：** 比较两侧含义与术语；pairing hash 为 green 不能证明 translation quality。

## 报告 finding

说明 defect、location、impact 和 evidence。将局部 defect 以内联方式放在最紧密的相关 diff range；cross-cutting architecture、scope 或 review-wide synthesis 使用 PR-level comment。Blocker 与 suggestion 分开，并省略 green gate 已强制执行的问题。Reply 使用现有 GitHub review thread。收到 review 时，验证每项 claim，并基于技术理由修复或反驳，不做表演式认同。
