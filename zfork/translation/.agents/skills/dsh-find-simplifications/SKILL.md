---
name: dsh-find-simplifications
description: 在 DeepSeek Harness code、API、configuration、test 和 prose 中寻找有证据支持的简化；编写或合并 proposal、识别小型 inline cleanup，或评估其他 branch 的简化。用于移除 dead、duplicated、speculative 或维护成本不必要的 behavior 与 infrastructure。
---

# 寻找 DeepSeek Harness 简化机会

寻找能够移除维护 obligation 的变更：API、representation、lifecycle state、configuration path、dependency、test 或 documentation。宁要少量证据充分的 candidate，也不要追求删除数量。本 skill 是指导原则，不是 checklist；保持用户指定的 scope，并区分 survey 与实施 proposal 的权限。

## 确定 scope 与 constraint

阅读 `AGENTS.md`；判断 package 前阅读 [architecture](../../../docs/architecture.md)、[defensive pattern](../../../docs/defensive-patterns.md) 和 [testing policy](../../../docs/testing.md)。查阅相关 active Agent Note 了解 intentional decision；新 record 使用 [note rule](../../notes/README.md)。Archived note 是历史示例，不是当前 authority：绝不检查、验证或修复其中的 outbound link。

两个 LLM adapter 是有意保留的。JSONL 是唯一 first-party Session persistence provider，backend-neutral Service Definition 则支持 out-of-tree provider。除非用户明确覆盖此 constraint，否则不得提议删除任一 LLM twin 或 persistence seam。Protected design 内未使用的 member 在不破坏其用途时仍是 candidate。

对于广泛请求，将独立 domain 分给 subagent：loop 与 persistence；model/tool assembly；Host、Client 与 SDK；subprocess 与 execution provider；composition、script、test 与 documentation。为每个 agent 指定 bounded domain，并要求 consumer evidence 与 rejected alternative。既检查显眼 unused symbol，也检查实质 production machinery；不要找到第一个可删除项就停止。

## 搜索可移除 obligation

使用以下问题指导 discovery。Candidate 需要校准时，阅读 [historical pattern](references/historical-patterns.md) 的相关 section；示例说明判断方式，不是当前删除 inventory。

- **Declared feature 是否有完整 effect path？** 追踪 producer、transformation、provider 或 endpoint support 和 observable result。一个 field 即使处处复制，也可能没有 setter、reader，或所有 implementation 都拒绝它。除 caller 外，还要搜索 constructor 与 discriminant emitter。
- **哪些 distinction 会改变 consumer action？** 多个 internal state 可能只需要一个 model-facing status 或一个 usable identifier。保留控制 authorization、residency、durability 或 ownership 的 internal distinction；删除只会邀请无效选择的 public distinction。
- **更小的 explicit behavior 能否移除整个 subsystem？** Fixed interval、caller-supplied time zone 或 reported activation failure 可以消除 generalized calendar、persisted default 或 automatic rollback。说明放弃的 capability 和删除的 machinery。有 production caller 时，这是 behavior decision，而非自动 rejection；implementation 必须在用户授权内。
- **Consumer 能否在需要时读取 authoritative value？** 查找 copied history、promotion ledger、先 invalidation event 后 read 的模式，以及 shared projection stream 旁的 feature cache。Derivation 或 reuse 可能同时删除 subscription、retained data、stale-response handling 和 recovery path。先确定 required observation time。
- **是否把 composition 误当成 policy？** Optional service presence、provider method 或 sibling tool 可以表达 availability，却不决定 deployment behavior。Explicit configuration 或 caller-intent operation 可能消除 registry 和 dependency check。Operation 的 ownership 或 completion timing 不同时要保持区分。
- **谁承担完整 maintenance cost？** 追踪 pass-through configuration、duplicate application tree、product 使用的 demo launcher、resolver-only workspace，以及有独立 upgrade lifecycle 的 installation helper。比较被删除系统与 replacement 的全部成本，包括 residual glue；移动 complexity 或添加 gate 维持 duplicate definition 一致，不算删除 complexity。

## 证明 reachability 与 trade-off

从 `rg` 开始，再阅读 match。搜索 exact symbol、property read/write、discriminant、event/wire string、config key、package name，以及 `.method(` 和 `method(` 两种形式。Test 和 declaration 可以证明 contract，但不能证明 shipped producer 或 consumer。

覆盖 `packages/*/*/src`、`apps/`、`python/`、runtime script、shipped profile 与 overlay、package manifest 与 export、loader resolution、generated runtime asset 和 installed-package consumer。根据真实 entry point 分类 example 与 fixture，不要根据 directory name。追踪 generated catalog 的 consumer：model 用于编写 plugin mount 的 catalog 是 dynamic product API；仅文档 listing 不是 fixed caller。Public third-party extension path 即使仓库内零 call site，也可能是 intentional。

对每个 candidate 记录 current owner、effective producer/consumer path、消失的内容、保留的内容，以及最强保留理由。区分：

- 删除 unreachable 或 unread behavior；
- 收窄 public 或 product behavior，并明确 loss；
- protected obligation 或证据不足。

Candidate 会破坏 retained obligation、只是移动相同 complexity，或没有 meaningful reduction 时应拒绝。按 proposed change 的粒度阅读 rejection reason：stale inventory 或 mixed proposal 中被拒绝的 behavior 不能裁决每个独立 item。重新提出前刷新证据。小型 local improvement 应作为 [urgency rule](../../../docs/development.md) 下可执行的 `TODO`/`FIXME`/`XXX` comment，而不是 standalone design record。

## 保留 ownership 与 failure semantic

对于 copy、freeze、validator 和 callback capture，命名 value origin、next owner 和 trust boundary。Typed same-process call 通常借用 readonly value；parser、config loader、queue、model/tool JSON、durable file、worker、process 和 wire decoder 拥有或验证 input。Hostile-getter 或 callback-replacement test 本身不能建立 requirement。Frozen root 不能证明 descendant immutable，equal id 也不能证明 object identity。

对于 asynchronous machinery，将每个 promise、flag、cancellation path、disposer 和 reservation 映射到 owner 与 transition。只有表达相同事实时才合并 mechanism。需要时保留 synchronous publication 与 rollback、callback containment、first-terminal-outcome arbitration、worker/process ownership 和 dispose-to-quiescence。Durable start/end marker 即使被 transcript renderer 忽略，也可能记录真实事实；丢弃 interrupted work 是 capability loss，不是 redundant-log cleanup。

Invariant companion 只有在比较可能 divergence 的 independently produced observation 时才合理。检查 service presence、plugin metadata、fixed example 或同一 mutation 的 return value 不符合条件。移除 companion 要同时移除 export、build entry、invariant-only compiler reference、dependency 和 test，并在两个 README 中记录 package-specific omission reason。保留比较 independent event producer、durable history 或 mutable data 的 check。

## 只有净减少时才替换 infrastructure

遵循 [dependency policy](../../notes/implemented/process/2026-07-26-dependencies-over-hand-rolling.md)。添加 package 前搜索 existing dependency 和 engine floor 支持的 Node builtin。确定 replacement 删除的确切 implementation 和 dedicated test、其 maintenance 与 transitive cost，以及 glue 必须保留的每项 residual behavior。

测试最可能使 replacement 失败的 mismatch：protocol truncation、cancellation 与 teardown、configured bound、deterministic test clock、platform packaging，以及 source 与 built entry point。只覆盖 framing 却要求重新实现相同 framing 才能强制 size limit 的 library，不是简化。接受的 edge-behavior change 要写入 proposal 和相关 test。不要在遗漏 harness-specific shim test 的同时重新证明 dependency 的 generic specification。

## 记录、合并并验证

Prose 在 scope 内时使用 [dsh-prose-standard](../dsh-prose-standard/SKILL.md)。删除复述 code 或归远端 owner 所有的 explanation，同时保留 local behavior、timing、ownership 和 failure obligation。随所属变更更新受影响 README、JSDoc、model-visible catalog、configuration、snapshot 和 generated file。

实质 proposal 使用必需 note skeleton：`Problem`、`Proposal`、`Alternatives considered`、`Acceptance criteria` 和 `Risks`。包含具体 consumer evidence、移除的 maintenance cost、放弃的 capability 和 observable acceptance condition。Implemented decision 改用 [implemented format](../../notes/README.md#the-body-skeleton)。Decision 相同时更新 existing owner；不得为保留 candidate count 创建 duplicate note。

每份新 note 都要通过 [dsh-archive-agent-notes](../dsh-archive-agent-notes/SKILL.md) 做 scoped supersession check。该 workflow 负责 retention、consolidation、triplet deletion 和 frozen archive mechanic。Code survey 不代表 repository-wide note audit。保留 partial supersession，以及当前 durable、wire、compatibility 或 rejected-alternative obligation。

合并另一 branch 时，将其 independent diff 与 target base 比较，只移植有支持且不 overlap 的 proposal，并合并重叠 rationale。关闭其他 PR 需要授权，或明确拥有该 housekeeping。

仅文档的 Agent Note 工作运行 `pnpm run doc-sync`、`pnpm run lint` 和 `git diff --check`；skill 或 comment edit 还要运行适用 validator。其他检查通过 [dsh-pre-push-checks](../dsh-pre-push-checks/SKILL.md) 选择。报告 surveyed area、supported candidate、有意义的 rejection 或 deferral、添加/合并/保留/删除的 note，以及实际运行的 command。Survey 扩展期间使用 draft PR；不得把未经验证的 search 描述为 exhaustive。
