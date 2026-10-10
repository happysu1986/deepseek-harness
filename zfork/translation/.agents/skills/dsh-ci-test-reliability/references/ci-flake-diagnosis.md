# CI flake 诊断

仅当任务是调查已有 probabilistic test 或 CI failure 时使用此 workflow。保持请求的 read/write scope：诊断不授权修复、重新运行 workflow 或修改 CI configuration。

## 固定证据

记录 repository、workflow、job、commit SHA、runner label、timestamp、确切 failing test 或 command，以及第一个稳定的 failure signature。将 infrastructure message 与 test output 分开。

比较多个 failing 和 passing run。优先比较相同 SHA 的 run；无法做到时，确认相关 test 与 CI configuration 在所比较 commit 间完全相同。一次 rerun 通过不能证明是 infrastructure fault，一次 timeout 也不能证明是 product race。

使用 Actions log 和 metadata 确定 failure 是否在同一 host 或 resource namespace 上 overlap。保留 supporting run 的 link，不要粘贴大量 log。

## 分类 failure

根据已记录证据分类，而不是根据最终 fix 分类：

- **Host-resource collision：** 独立 process 或 job 获取了相同 port、socket、database、可预测 path、cache 或 external namespace。
- **Incomplete lifecycle：** teardown 在 child、worker、stream、server 或 callback 达到 quiescence 前返回；后续 output 或 mutation 出现在另一个 test 中。
- **Process-global contamination：** 结果依赖 test order，或泄漏的 `process.env`、`cwd`、fake timer、global、mock、locale 或 module state。
- **Load-sensitive synchronization：** 使用 sleep、polling interval 或假定的 event-loop turn 代替可观察的 readiness 或 completion。
- **Platform 或 entry-path mismatch：** failure 稳定地随 operating system、shell、filesystem rule、source/build mode 或 executable entry 出现。Timestamp precision、environment variable name case、handle-release timing 和 permission semantic 在 Windows 与 POSIX host 间都不同，因此 macOS 上通过的 case 无法说明 Windows lane 的情况。
- **Product concurrency defect：** Test 控制其 resource，使用显式 overlap 可 deterministic reproduce，并暴露 shipped behavior 中的 race。
- **External-provider transience：** Failure 归 live API 或 network boundary 所有，并符合其已记录的 retry policy。
- **Runner infrastructure：** Checkout、dependency download、disk、host process 或 runner service 独立于 test command 失败。归为此类前必须有直接 runner 证据。当 self-hosted pool 不提供 host metric 时，要明确说明，并根据 log 中已有信息分类：同一 signature 在同一 pool 的不相关 branch 上重复出现，即使无法检查 host，也是 shared-host contention 的证据。

如果证据支持多个独立事实，逐项报告。不得把 timeout、signal、exit code 和 assertion 合并成单一推断结果。

## 复现最小相关 topology

从所属 test file 或 focused test name 开始。只增加 concurrency，直到首次复现 signature 的 topology：

1. 一个 test process；
2. 并发 test 或 file；
3. 多个独立 Vitest process；
4. 使用其已配置 worker count 的所属 repository gate；
5. 共享相关 host resource 的不同 job 或 runner process。

匹配 active Vitest config、environment knob、source/build mode 和 platform。不得通过降低 production timeout 或添加 random load 来制造另一种 failure。

如果 signature 属于可用 host 无法运行的平台，则 ladder 在最后一个可达 rung 停止。记录该限制，不得用另一平台的 passing run 替代；然后使用 CI 复现，每次 run 只修改一个 suspected owner，使结果仍可归因。

对于 suspected race，用 contested transition 上的 barrier 替代 probabilistic timing。对于 suspected host collision，证明同时获取了相同 identifier，或证明 atomic unique allocation 消除了 conflict。

## 在 owner 处修复

获得 implementation 授权时，修复负责 allocation、发布 readiness、修改 global state 或拥有 teardown 的 component。不得把 failure 隐藏在 snapshot normalizer、retry wrapper、更宽 timeout、global serialization setting 或更弱 assertion 中。

将稳定 fixture data 与 live resource allocation 分开。Recorded URL 可以保持稳定，由 fixture 将其 transport 映射到 OS-assigned port；稳定 expected path 可以继续作为 assertion，而不成为 shared writable directory。

## 结束调查

满足以下条件时，证据才完整：

- 原始 signature 已有证据支持的分类；
- 最小相关 topology 能复现，或者 external evidence 已足够且明确说明复现限制；
- 获授权的 fix 在 negative control 或 pre-fix state 下失败，并在之后相同 topology 下通过；
- Resource owner 所要求的 concurrent-process、restoration 或 quiescent-teardown 证明均通过；
- 根据观察状态将其余 Actions check 报告为 passing、pending、skipped 或 failing。

不要一直运行到 test 偶然通过就称其稳定。所选证据足以建立结论后停止，或者报告阻碍分类的缺失事实。
