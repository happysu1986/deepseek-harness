# 历史简化 pattern

以下示例用于校准无法通过 unused-symbol search 确立价值的 candidate。它们描述已记录 decision，不是当前 removal opportunity。Archived source 是冻结 snapshot：只阅读被引用 note 本身，绝不跟随或修复其 outbound link。应用 pattern 前验证 current owner 和 released-data obligation。

## 要求完整 effect path

Declaration、serializer branch 和 test 可以形成看似可信的支持，但仍没有 effective feature。追踪两端：必须有 producer 产生 value，也必须有 implementation 接受它。[Producerless-variants decision](../../../notes/archived/simplification/2026-07-04-prune-producerless-vocabulary-variants.md) 删除了没有 emitter 的 reserved union member 和 cache hint。[Inert-request-knobs decision](../../../notes/archived/simplification/2026-07-04-drop-inert-request-knobs.md) 区分所有 adapter 都拒绝的 knob 与 hook 可用的 working request field。删除范围包含 declaration、forwarding、rejection-only test 和 generated documentation。

零 fixed caller 是比 unreachability 更弱的证据。[Skill-registration rejection](../../../notes/rejected/simplification/2026-07-12-prune-unused-skill-registry-api.md) 保留了 deliberate third-party registration path。对于 catalogued service，在判断为 dead 前检查 generic plugin invocation 和 returned value。Working extension point 与所有 provider 都无法接受的 field 应得到不同结论。

## 在 consumer 决策处减少 distinction

将每个 public variant 映射到它支持的 consumer action。[Model agent-availability decision](../../../notes/implemented/simplification/2026-09-15-model-agent-availability-and-team-targets.md) 只给 model 一个可用 Team target，将多个 residency state 投影为相同 availability，同时 internal routing 保留 Session identity 和 residency。[Capability-neutral sandbox context](../../../notes/archived/simplification/2026-07-31-capability-neutral-sandbox-policy-context.md) 用真实 conditional policy wording 替换两个近似 capability registry；tool schema 继续承载精确 tool availability。收益包括 registration method、disposal state、invalidation 和 combination test。

不要仅因一个 renderer 忽略 distinction 就删除不同 durable fact。[Durable step-boundary rejection](../../../notes/rejected/simplification/2026-06-20-drop-durable-step-boundaries.md) 保留 explicit start 与 completion fact，因为 empty failed step、crash repair 和 inspection 需要它们。Underlying distinction 仍必要时，只简化 projection。

## 用更小的 explicit promise 换取 general behavior

Used feature 仍可能是满足 required outcome 的高成本方式。[Fixed-rate Schedule decision](../../../notes/archived/simplification/2026-08-09-bounded-fixed-rate-schedule.md) 放弃 calendar recurrence 和 missed-occurrence replay，从而移除 parser、evaluator、cross-record gate、durable variant 和 backlog state。[Non-transactional Loader decision](../../../notes/implemented/simplification/2026-09-09-nontransactional-loader.md) 放弃 automatic plugin rollback，同时保留 failure reporting 和 consumer-owned activation check。把 generic rollback 移到其他 package 仍会保留其成本。

像陈述删除一样清楚地陈述 loss。[Interrupted-turn truncation rejection](../../../notes/rejected/simplification/2026-06-20-truncate-interrupted-turns.md) 拒绝更简单的 recovery algorithm，因为 unfinished turn 可能包含大量真实工作。Convenience、更小 union 或更少 test 本身不足以证明可以丢失 required data 或 behavior。Survey 可以提出 trade-off，但不暗示实施授权。

## 在消费 operation 处明确 assumption

查找已经获得 shared storage、synchronization 或 confirmation machinery 的 local default。[Explicit Schedule time-zone decision](../../../notes/implemented/simplification/2026-08-09-explicit-schedule-time-zone.md) 将 browser fact 留在 originating request，并要求 scheduling operation 明确提供 zone，从而移除 Session default 和 cross-tab conflict state。[Injected-content decision](../../../notes/implemented/simplification/2026-07-20-unwrap-injected-content-envelopes.md) 让 caller 负责 framing，并移除穿过不相关 layer 的 envelope control。两者都减少了解释一项 assumption 的 owner 数量。

不要把 requirement 移过必须 enforce 它的位置。[Host-only Remote input-validation decision](../../../notes/implemented/simplification/2026-09-15-host-only-remote-input-validation.md) 删除 duplicate Client schema execution，但保留 lookup 与 business invocation 前的 Host validation，以及 local arity 和 Context binding check。其记录成本是 malformed input 更晚失败，并失去 Client parsing 产生的 accidental redaction。

## 在使用时派生或复用 delivery owner

对每个 cache 或 event subscription，询问 consumer 何时需要 value，以及 authoritative source 是否已保留它。[Feedback telemetry](../../../notes/archived/simplification/2026-08-06-buffer-free-feedback-telemetry.md) 在 feedback 时读取 canonical Session prefix，删除持续复制的 pre-feedback buffer 及其 listener。[Workspace recency](../../../notes/implemented/simplification/2026-09-10-derived-workspace-recency.md) 对 current summary 排序，不再持久化 promotion history；manual ordering 仍是独立 user state。[Web subagent projection](../../../notes/implemented/simplification/2026-09-08-web-subagent-catalog-projections.md) 复用 shared stream 和 projection store，移除 feature notification、repeated read 和第二个 membership cache。

Timing promise 决定此方法是否成立。Feedback-time redaction 不等于 capture-time redaction；manual placement 不能从 activity timestamp 派生；stream baseline 不存在时可能仍需初始 historical read。应命名这些 residual obligation，不要宣称所有 cache 都冗余。

## 移除 accidental composition owner

信任 demo、support 或 example 等名称前，检查真实 launch 与 resolution path。[Shared-base-config decision](../../../notes/archived/simplification/2026-07-29-shared-base-config-overlays.md) 移除 duplicate composition 和 pass-through setting；实际 Loader behavior 决定 overlay placement。[Owner-local profiles and guides decision](../../../notes/archived/simplification/2026-08-24-owner-local-profile-tests-and-guides.md) 将 test、guide 和 shipped asset 分配给 owner，从而移除 resolver-only workspace。[Merged subagent service decision](../../../notes/archived/simplification/2026-07-26-merge-subagent-control-service.md) 使用 explicit deployment policy，不再从 provider 或 sibling-tool presence 推断，同时在内部保留 optional dependency。

Package 数量减少本身不充分。不同 caller intent 可能有不同 ownership 和 completion timing；保留的 Service Definition 可能服务 independent provider。证明 replacement 的真实 composition 与 dependency behavior，并计入所有剩余 configuration 和 lifecycle glue。

## 使用 rejected experiment 检验 parity claim

[SSE parser replacement](../../../notes/archived/simplification/2026-07-26-eventsource-parser-for-deepseek-sse.md) 删除 generic framing 及其 conformance test，同时保留 provider-specific termination shim，并记录更严格的 truncated-tail behavior。相反，[timer-promises rejection](../../../notes/rejected/simplification/2026-07-26-builtin-timer-promises-for-hand-rolled-sleeps.md) 记录 nominally equivalent builtin 为少量删除破坏了 deterministic fake-clock test。[Dependency-swap audit](../../../notes/rejected/simplification/2026-07-26-dependency-swaps-rejected-by-nih-audit.md) 区分 replaceable generic code，以及 library 无法实现的 configured bound、security rule、lifecycle ownership 和 deployment behavior。

过去的 rejection 是 scoped falsifier，不是永久 package blacklist。只有新证据处理了失败 premise 时才重新评估，并一起测量 implementation、glue、test、dependency 和 supported behavior。
