---
name: dsh-ci-test-reliability
description: 设计、评审和诊断可能因 CI concurrency、共享 host resource、clock、process-global state、subprocess、network listener 或 asynchronous teardown 而非确定性失败的 DeepSeek Harness test 与 fixture。新增或修改具有这些风险的 test、调查 flaky CI 或评审 test isolation 时使用；另行使用 dsh-pre-push-checks 选择对外运行的命令。
---

# 可靠的 DSH CI test

构建在仓库真实 CI topology 下仍然正确的 test，而不是只在安静的 workstation 上单独运行时正确。本 skill 负责 isolation 和 reliability 决策；它不取代仓库的 test-tier policy，也不为 push 选择所有命令。

## 阅读所属规则

- 使用[测试 policy](../../../docs/testing.md)选择 unit、coverage、expected-output、snapshot、browser 或 real-API 证据。
- lifecycle、subprocess、cancellation 和 teardown 行为遵循 [defensive pattern](../../../docs/defensive-patterns.md)。
- 当 active Vitest config 和 GitHub workflow 的 worker 或 job topology 会影响 test 时，阅读这些配置。
- 对于 recorded-session scenario，还要遵循 [snapshot 说明](../../../snapshots/AGENTS.md)。
- Test 设计可靠后，使用 [dsh-pre-push-checks](../dsh-pre-push-checks/SKILL.md) 选择对外 validation。

## 建模 execution topology

除非 active configuration 证明不会重叠，否则假定以下各层可以重叠：

1. 同一个 Vitest file 中的 test。
2. 不同 Vitest file 或 worker process。
3. 同一个 job 中独立的 Vitest 或 repository-gate process。
4. Runner 共享一台 host 的不同 Actions job。

Process isolation 不会隔离 host port、可预测的 filesystem path、external service、database、socket 或继承的 child process。对每个已获取 resource，确定其 owner、atomic allocation mechanism、observable readiness signal、已注册 cleanup 和 quiescent completion signal。

不要仅因一个 fixture 缺少 isolation 就串行化整个 suite。应先缩小 exclusive scope 或修改 resource allocation。Sequential Vitest block 无法保护 host resource，使其免受其他 file、process、job 或 runner 影响。

## 原子分配 resource

使用 resource owner 的 allocator，而不是先检查是否可用、稍后再占用。

- Network fixture 使用 `listen(0)` 绑定 loopback，只在 server 报告已 listening 后读取分配的 address。绝不能扫描 free port 后再绑定。
- 使用 `mkdtemp` 创建每个 test 私有的 temporary root；不得占用可预测的共享 path。
- 为共享 database、socket、session 和 output location 提供每个 test 唯一的 namespace。
- 当 path 必须尚不存在时，使用 exclusive creation。
- 将稳定的 recorded identifier 与 ephemeral transport address 分开。在 fixture 内转换，不要强迫 live resource 使用 recorded value。

只用作 parser input 或 expected value 的 literal path 和 URL 不是已获取的 resource。不要仅因它们看起来固定就重写。

## 约束 process-global state

将 `process.env`、`cwd`、fake timer、locale 和 timezone、module mock、registry、console hook、`globalThis` 以及 global `fetch` interception 视为 exclusive mutable resource。

优先使用 injected dependency 或 instance-local adapter。必须进行 mutation 时：

- 记录原始 value 是 absent 还是 present；
- 精确恢复该状态；
- 立即注册 restoration；
- 在最小 mutation scope 外使用 `try/finally`；
- 如果 local `finally` 前可能失败，保留 `afterEach` fallback；
- 只 intercept fixture 自身负责的最窄、精确 request 或 call。

## 尊重 platform-owned semantic

CI 在 Windows 和 POSIX host 上运行同一 suite，operating system 所有的 value 不一定以 test 写入时的形式返回。

- 只有 assertion 能容忍 write-back 失败时，写回 value 才安全。恢复 file `mtime` 来证明 fingerprint 无论如何都会 invalidate，在所有平台都成立；恢复它以证明 record 保持 valid，则假定了无损 round trip，而 NTFS 的 100-nanosecond tick 无法表示 fractional millisecond。当 assertion 依赖恢复结果时，从 fresh read 获取 expected value，不要使用记忆中的 value。
- Windows 对 environment variable name 进行 case-insensitive 匹配，因此同时设置 `http_proxy` 和 `HTTP_PROXY` 的 fixture 在 Windows 上只保留一个 entry。
- Windows 异步释放 file handle，因此在 POSIX host 上立即完成的 rename 或 removal，需要按实际 contention 设置有界 retry。
- Windows 不具备 POSIX permission 或 signal semantic。依赖这些 semantic 的 case 应明确按 platform skip 并说明原因，不要在所有平台弱化 assertion。

优先选择所有平台都成立的 observation。某个 case 确实无法跨平台时，明确在该平台排除。

## 根据 lane 确定 timeout budget

`describe` 或 case timeout 会覆盖 runner 的 `--testTimeout`，而不是让位于它，因此低于 lane budget 的值会减少 CI 已授予的时间——同一 literal 在默认值更小的 host 上又像是在放宽限制。受 process creation 限制的 suite 使用 lane budget；更严格的值必须说明为何更严格。

提高 test budget 时同时提高 hook budget。Setup 和 teardown 承担同样的 contention，因此只提高 case budget 会把 contended failure 移到 `afterEach`。

当 timeout 本身是测试对象时，outer wait 必须远大于被测 timeout。证明 20 ms deadline 会触发的 case 不得与 harness 自己的 wait 竞争，否则 load 将决定先报告哪个 deadline。

## 根据 state 同步

固定 sleep 不能证明 setup 已完成或 cleanup 已稳定。

- 等待明确的 readiness event、handshake、state transition、owned promise 或外部可观察 condition。
- 使用 deferred promise 或 barrier 将 race 放到 deterministic point，并证明相关 operation 确实 overlap。
- Timeout 只能限制 wait，绝不能作为使 assertion 正确的 condition。
- 除非 scheduler-dependent ordering 正是被测 product behavior，否则不得断言这种顺序。
- 当 time 本身是测试对象时，inject 或 fake clock，并始终恢复 real timer。

## Dispose 到 quiescence

获取 resource 后立即注册 cleanup，确保 assertion 失败时也能释放。Cleanup 要停止新的 callback 或 request、detach listener、恢复 global hook、终止 owned work，并等待 child exit、server close、worker termination 或等价 completion signal。

调用 `abort()`、`close()` 或 `kill()` 却不等待 owned completion signal，属于不完整 teardown。当可能延迟完成时，证明 disposal 会阻止其修改另一个 test。

## 证明目标 regression

- 可行时，在修复前观察普通 regression 失败。
- 对新的 static 或 corpus guard，暂时引入被拒绝的 case，并观察预期失败。
- 对 race，使用 barrier 证明 overlap；重复执行本身不是 race test。
- 对 port、socket、shared path、subprocess 或其他 host resource，如果修复包含 cross-process isolation，则并发运行独立 test process。
- Fixture 使用自己的 deadline spawn 时，先断言没有 signal 或 timeout 终止 child，再断言 exit status，使被 kill 的 child 报告为 timeout，而不是 status mismatch。
- 验证 external state、event、file、log、exit 或 disposal，不要相信 component 的 self-report。

Stress run 是 deterministic regression 的补充，不能取代后者。

## 拒绝掩盖 flake 的修复

不得把以下做法表述为 deterministic local test 的 root-cause fix：

- 未确定 awaited state 就增加 timeout；
- 添加 retry；
- 将所有 file 设为 serial；
- 吞掉 error 或 unhandled rejection；
- 弱化 assertion；
- normalize 掉 unstable behavior；
- 在 cleanup 或 assertion 前添加 sleep。

对于 real-API policy 下有文档说明的 transient external-provider test，retry 仍然有效。只在 external boundary 保留这一例外。

恢复 budget 并不等于掩盖问题。将 suite 提高到它已经拥有的 lane budget，或根据 runner 上实际测得的 contention 设置有界 retry，都是命名 awaited work，并恢复 lane 授予的资源；两者都没有围绕未经检查的 wait 凭空增加余量。

## 诊断已有 flake

对于已有的 probabilistic CI failure，阅读 [CI flake 诊断 workflow](references/ci-flake-diagnosis.md)。仅诊断的请求保持 read-only：除非用户也要求修复，否则只报告原因和证据。

## 验证并报告

运行覆盖受影响 behavior 的最小 focused regression。只有变更确实涉及相应风险时，才添加 topology-specific 证据：

- global mutation 需要 restoration 证据；
- lifecycle 或 subprocess 工作需要 quiescent teardown 证据；
- port、socket 或 shared path 需要并发独立 process 证据；
- 新 guard 需要 negative control。

Push 前使用 [dsh-pre-push-checks](../dsh-pre-push-checks/SKILL.md)。报告确切命令与观察结果；不得把 retry、skipped test 或 pending CI 描述为通过。
