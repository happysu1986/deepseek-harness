---
name: dsh-speed-up-perf
description: '调查或优化 DeepSeek Harness performance、设计 realistic synthetic benchmark 或 CI performance gate、profile 长 Session 或 Web responsiveness，或将 performance PR evidence 转化为经过测量且保持 behavior 的 fix 时使用。'
---

# 加速 DeepSeek Harness

将广泛的「make it faster」请求转化为可复现的 user-path measurement 和小型、evidence-backed fix。本 skill 是指导原则，不是 quota 或 script：广泛 survey，追踪 measured cost，并拒绝不能改善用户真实 workload 的诱人变更。

## 确定 scope 与 current authority

阅读 [AGENTS.md](../../../AGENTS.md)、[architecture](../../../docs/architecture.md)、[testing policy](../../../docs/testing.md)、[defensive pattern](../../../docs/defensive-patterns.md)，以及受影响 package 的 instruction 与 Agent Note。Process、clock、browser test 和 asynchronous cleanup 使用 [CI test reliability](../dsh-ci-test-reliability/SKILL.md)。

约定 user-visible endpoint、workload range、resource constraint、可接受的 minor behavior difference 和 stopping rule。Backend 与 browser end-to-end measurement 保持分开：快速 history iterator 或 Client fold 不能证明 transport、paint、scrolling 或 input response 快速。测量 local overhead 时排除 model/network latency，并明确该 exclusion，不得称其为完整 product latency。

检查精确 current base，不只看 running checkout。研究 final merged diff、owning source、test 和 resolved review thread；PR body 可能描述已经放弃的 implementation。区分 merged、closed-unmerged、superseded、estimated 和 newly measured evidence。[Performance workflow decision and evidence](../../notes/implemented/process/2026-09-06-evidence-driven-performance-skill.md) 提供 historical lead，不授权重新引入其 implementation。

## Survey user path，再排序 candidate

广度有帮助时委派独立 domain；要求 measurement 和 production call site，不接受猜测。适合的 domain 包括：

- Cold profile startup、首次 historical read、current-generation reopen 和 writable resume。
- Many-turn 与 tool-heavy history、较大单个 message/result、child Session listing，以及 Session 间 repeated navigation。
- Initial history transport 与 fold、first usable browser paint、older-page loading、scrolling、tool expansion 和 inactive-view activation。
- Live streaming 与 reconnect，包括长 active attempt、interleaved tool work、settlement、cancellation 和 teardown。

分别改变独立 cost driver：byte、durable event、compact record、raw delta、turn、tool、child 和 visible DOM node 是不同 quantity。未检查它压测哪项 user operation 前，不得把大量 tiny identical message 称为 realistic。包含 typical 与 tail workload，但避免没有 decision value 的 combinatorial matrix。

按 observed user latency、CPU/allocation、retained memory、occurrence 和 confidence 排序 candidate。对每项命名 production consumer、repeated work、expected complexity、最小 falsifiable intervention，以及必须保持稳定的 behavior。可疑 loop、unused cache 或 large file 本身不是 bottleneck evidence。

## 先构建 realistic synthetic benchmark

遵循 [benchmarks/AGENTS.md](../../../benchmarks/AGENTS.md) 和 [performance-gate decision](../../notes/implemented/testing/2026-09-04-session-open-performance-gate.md)。扩展 existing required lane，不创建竞争的 calibration 或 reporting infrastructure。Package-local diagnostic 保留在 owner 旁；cross-package required case 放在 `benchmarks/` 中 measured user path 下。

用户授权 local corpus inspection 时，只提取 aggregate workload characteristic。绝不能把 prompt、output、path、identity、ID、credential、recording 或 recognizable snippet 复制到 fixture、log、screenshot、PR 或 artifact。根据 reviewed constant 生成固定 input；benchmark 不依赖用户 home、ambient repository、network service 或 private data。

Implementation 前记录 measurement card：

| Field | Required decision |
|---|---|
| User operation | 确切 action 与 externally observable completion condition |
| Workload | 固定 dimension、distribution、construction seed/constant，以及为何覆盖 ordinary 和 tail use |
| Entry path | Production call/composition 与 built artifact；mock external boundary |
| Clock | 包含的 setup、cold/warm state、timing start/end 和 excluded cost |
| Memory | Reachable endpoint object、baseline、GC policy、retained 与 transient limit |
| Verdict | Raw sample、chosen aggregate、calibrated absolute/ratio/memory limit 和 negative control |
| Behavior | Owning functional test/snapshot 与允许的 minor difference |

CPU worker 在 plain Node 下测量 built JavaScript；source-loader overhead 和 module resolution 不是 shipped path。Browser case 通过 existing test harness 使用 built product asset 和受支持 `dsh` profile。不得仅为 measurement 添加 production export，也不得把 algorithm 复制到「benchmark implementation」。

Cold/process-memory sample 使用 fresh child 和 private temporary root。Warm sample 明确保留 intended cache；绝不能让 fixture setup 暗中 warm cold scenario。两侧保持相同 input、validation、completion condition 和 reachable output。Parse-and-discard baseline 不能与 validated retained history 比较。

报告所有 sample 及决定结果的 aggregate。Node lane 使用 existing shared time calibration 和 reviewed variance headroom；不要按 CPU speed 缩放 byte、count 或 dimensionless ratio。Manual browser diagnostic 不设 threshold。Required browser performance case 在采用 timing budget 前，需要 explicit lane decision，并在真实 CI browser/runner 上 repeated measurement；Node machine multiplier 不能校准 browser。Budget 是 source constant，不是 environment override。Measured work 与其他 owned CPU-heavy job 串行；在 comparable condition 下测量 reference 和 candidate。不得放宽 budget 或挑选 lucky run 隐藏 regression。

End-to-end latency 与 component phase 独立测量。Retained memory 在 intended object 仍 reachable 时测量；transient pressure 通过 constrained-heap completion 或合适 peak measurement 单独测量。执行更快但 retention 无界，不自动算胜利。

Browser responsiveness 使用真实 browser input 并观察 resulting UI update。Frame/input measurement 包含 final stall，区分 scheduled timer 与 actual input，并限制 synthetic producer，避免 catch-up burst 制造另一种 workload。说明是否覆盖 first paint、scrolling、paging、live update 和 activated-but-hidden view。Node fold、fake DOM 和 custom heartbeat event 本身不能证明 browser responsiveness。

## 证明 regression，再删除 work

修改 production code 前运行 unoptimized workload。保存 command、revision、runtime/platform、fixture dimension、raw measurement 和 verdict。缩小 failing scenario，直到仍覆盖真实 bottleneck；patch 前排序 falsifiable hypothesis。使用 profile、allocation sample、work count 或 phase timing 区分它们。

值得测试的常见 pattern，不是自动 prescription：

- 让 compact representation 在 downstream reader 中保持 compact；consumer 只需 settled content 或 aggregate 时，避免 per-delta object。
- 只有识别真实 ownership 和 trust transition 后，才删除 duplicate parsing、copying、freezing 和 validation。Typed same-process borrowing 不允许弱化 durable 或 wire parsing。
- Stream artifact transformation 并限制 intermediate state，不要保留每个 generation。User operation 需要时包含 publication、verification 和 writable-readiness obligation。
- 将 read-only preparation 与 write/publication work 分开，但不得把 awaited work 移过 correctness-required endpoint。
- 延迟 inactive-view 和 collapsed-detail work；也测量 first activation 与 retained state。Deferral 不是 deletion，viewport highlighting 不是完整 virtualization。
- 稳定 identity 并缩小 subscription，避免一个 changed node invalidate 整个 history；保留 update ordering 与 immediate-event behavior。
- 使用合适 data structure 代替 repeated shifting、scanning 或 rebuilding。测量完整 consumer path，不只测 isolated container operation。
- Revision-keyed reuse 或 singleflight 只有具备 explicit invalidation、bounded retention、independent waiter cancellation 和 disposal ownership 时才使用。不要缓存 expanded representation 只为让 repeated benchmark 看起来更快。

每次只改变一个 causal factor。重新运行 focused scenario 和其 end-to-end parent。要求 negative control：收紧 assertion 在 original implementation 或 controlled reintroduction of targeted cost 上失败。Regression 仍能通过的宽松 threshold 不提供保护；低于 verified noise floor 的 budget 也不可靠。

## 保留 behavior 与 resource ownership

Performance measurement 补充 functional evidence，不能取代它。按情况运行或添加 output、ordering、paging、stream index、error、cancellation、concurrency 和 disposal 的狭窄 owning test。保留 model-visible/logged equivalence、released-generation immutability、atomic publication、required validation 和 writable readiness。不得为达到数值而静默截断 history、跳过 tool result、禁用 invariant 或改变 lifecycle semantic。

陈述 deliberate minor visible difference，并通过所属 keyless snapshot 验证。Product-visible GUI change 包含 required browser evidence/GIF。Functional expectation 独立于 benchmark internal；benchmark assertion 需要足够证据到达真实 endpoint，不需要第二套 semantic test suite。

Gain 在 end-to-end 消失、typical workload 明显 regression、complexity 超过小幅 gain，或 cancellation/retention/durability 无法解释和测试时，拒绝 optimization。简要记录 rejected hypothesis，不扩大 scope 为其辩护。

## 交付有界、可评审结果

Durable rationale、alternative、calibration、exclusion 和 remaining risk 使用 [Agent Note rule](../../notes/README.md)。检查相关 note 是否 supersede，但不要把 performance work 变成 corpus-wide prose cleanup。Reusable procedure 留在本 skill，scenario-specific truth 留给 benchmark 或 package owner。

任务要求 stacked PR 时，编辑前选择 layer，并使用 official GitHub stack 和 separate worktree。每层保持 mergeable：benchmark infrastructure 可以保护 measured baseline；optimization layer 包含 fix、functional coverage 和更严格 budget。Independent bottleneck 可使用 separate stack。Finding 在所属 layer 修复后再向上传播。

应用 [pre-push check](../dsh-pre-push-checks/SKILL.md)，只报告实际执行 evidence，并检查 CI，不要假定 local timing 能证明 runner stability。标记 ready 后，根据 code 与 executable evidence 评估 review finding；以理由或 fix 回复，并 resolve 已处理 thread。不得仅因 report 来自 bot 就忽略。

每项结果按以下形式总结：workload → before/after absolute value 与 ratio → endpoint 和 memory semantic → behavior evidence → negative control → exact check → exclusion。分开 author-reported historical number、fresh local measurement 和 CI evidence。到约定 scenario/fix scope 即停止；保留简短 ranked follow-up list，不追逐 unrelated opportunity。
