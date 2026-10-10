---
name: record-browser-gif
description: 使用现有 browser-control 工作流、可选的 Playwright Videos（用于提高捕获帧率）以及确定性编码，将浏览器或 Web UI 交互演示录制为经过优化的 GIF；随后使用 `gh --attach` 将 GIF 附加到 pull request，在无法使用 attach 时回退到专用 assets branch。适用于用户要求制作、录制或生成展示浏览器工作流的 GIF，也适用于每一个会改变产品用户可见 GUI 行为的 pull request；这类 pull request 必须包含从该 pull request 的真实 server 和 model 流程录制的 GIF。
---

# 录制浏览器 GIF

生成一段简短、真实的 UI 演示并保存为本地 GIF；仅当任务包含将其附加到 pull request 时，才在本 skill 末尾通过 attach 工作流发布。现有 browser-control 工作流仍为首选。当该工作流支持以更高帧率连续捕获时，使用 [Playwright Videos](https://playwright.dev/docs/videos)；使用随附的 encoder 完成裁剪、播放速度、结尾停留、尺寸和文件大小控制。

[证据链决策](../../notes/implemented/process/2026-08-08-browser-gif-evidence-chain.md)规定了为什么一个 storyboard 必须来自一次隔离运行，以及发布为何要同时重新验证 artifact 和所演示 pull request 的 head。

## 每个 GUI pull request 都必须包含 GIF

凡是改变产品用户可见 GUI 行为的 pull request，都必须包含使用本 skill 录制的演示 GIF，并通过[附加工作流](#publish-the-gif)嵌入 pull request body。

录制本身就是证据的一部分：使用从该 pull request branch tree 启动的真实 server、真实 API key 和真实 model rounds。除非用户明确要求录制 fixture，否则绝不能以 fixture query、mock transport、synthetic event injection 或仅用于测试的 hook 替代。应在嵌入内容旁注明所演示的确切 commit SHA、提供服务的 tree 和 origin、任何 mode flag 或 browser-state 例外，以及是否运行了真实 model round，使 reviewer 清楚录制究竟证明了什么。

## 将录制与发布分开

- 录制只生成本地 video 或 screenshot 以及一个 `.gif` artifact；绝不修改远程状态。
- 发布是独立的最后一步：使用 `gh --attach` 将 GIF 附加到 pull request body；无法使用 attach 时，将其推送到 assets branch 并嵌入对应 URL。只有任务包含将 GIF 附加到 pull request 时才执行发布。发布绝不触碰 pull request 自身的 branch。
- 保留用户要求的录制条件。真实 server 或真实 API demo 不得使用 fixture query、mock transport、synthetic event injection 或仅用于测试的 hook。如果凭据或 server 不可用，应报告该限制，而不是改用 fixture。
- 绝不读取或暴露凭据值。使用应用的正常配置路径和无害的演示 prompt。

## 准备应用

针对特定 pull request 的 GIF 演示的是该 pull request 的 tree，因此需按 pull request 分别准备：

1. 要求 worktree 干净，使用 `git rev-parse HEAD` 记录其确切 commit，然后构建所记录的 tree——此处为 `pnpm run build && pnpm run build:web`。针对另一个 commit 的 build 录制 GIF，会错误归属证据。
2. 从该 tree 为每个 port 启动一个 server，并使用全新的临时 `DSH_HOME`、`DSH_AGENTS_HOME`、workspace 和 session state。browser 也应使用全新的隔离 context 或 profile；如果 browser 工作流无法创建，则在导航前清除该 origin 的 cookie 和 site storage，避免持久化 client state 影响证据。通过应用的正常路径从根目录 `.env` 读取 API key；绝不能输出该 key。
3. 将一个 storyboard 视为一次证据运行：所有发布帧都来自该 server，以及该次运行对应的 state root、workspace、session 和由 model 支持的 scenario。若捕获自动化失败，丢弃其帧并使用全新 root 重新运行；绝不能拼接不同运行的帧。
4. 在 pull request 之间切换时，通过 PID 或其命令行的精确匹配停止旧 server。宽泛的 `pkill -f` pattern 可能匹配并终止启动它的 shell——包括你自己的 shell。

## 录制流程

遵循现有 browser-control 工作流的 setup、interaction 和 cleanup 指令。当它提供 `recordVideo` 时，在同一个受控 context 上启用 video，以捕获更多中间帧。否则，在该工作流中使用[screenshot capture](#screenshot-capture)；video 是否可用不决定采用哪一种 browser-control 工作流。现有用户 browser state 仍属于明确的隔离例外。

只有在 browser control 不可用时，才使用仓库声明的 Playwright dependency 和隔离的 headless browser，并在捕获说明中注明这一 fallback。在本仓库中，它从 `apps/web/package.json` 解析；不要安装另一个 driver，也不要打开用户的 browser。

录制前，确定 origin、built 或 development server、transport 以及任何 mode override。当 production 默认行为会打开自动化无法驱动的 native surface 时，通过正常应用配置选择官方且可由 browser 操作的 production backend，并披露该 override。

将 script、raw video、timing note、QA frame 和 GIF 存放在仓库中被 gitignore 的 `.playwright-mcp/` 目录下。先创建 run directory。

### 捕获 video

明确匹配 `viewport` 和 `recordVideo.size`：否则 Playwright 会缩小 video 以适应 800×800，可能导致 UI 文本无法阅读。

通过所选 browser-control 工作流配置 video。独立 Playwright fallback 使用：

```js
const { chromium } = createRequire(join(repo, 'apps/web/package.json'))('playwright')
const browser = await chromium.launch()
const size = { width: 1440, height: 900 }
const context = await browser.newContext({
  viewport: size,
  recordVideo: { dir: join(runDir, 'videos'), size },
})
try {
  const page = await context.newPage()
  const video = page.video()
  // Navigate and exercise the real application here.
  await context.close()
  await video.saveAs(join(runDir, 'demo.webm'))
} finally {
  await context.close()
  await browser.close()
}
```

从 `node:module` import `createRequire`，从 `node:path` import `join`；在录制 script 中将 `repo` 和全新的 `runDir` 设置为绝对路径。关闭 page 前保留其 video handle。编码或调用 `video.saveAs()` 前等待 `context.close()` 完成；只关闭 browser 不保证 video 已 flush。每个 page 都有自己的 video：明确选择被演示的 page，不要拼接无关 page 或 run。失败的 run 仅用于诊断。

选择一个包含三到六个有意义状态的简短故事。操作前等待唯一的 semantic locator；accessible-name 相等判断使用 `exact: true`，精确文本的完成 predicate 必须确保不会匹配 prompt echo。状态得到验证后，可以使用固定等待提供阅读停留时间，但绝不能用它判断 readiness。捕获 video 时，保留 animation 和 scrolling。

演示 tool call、rejection 或 recovery 时，打开其 detail 或 trajectory，使 video 显示 tool identity、status 或稳定的 error code，以及 downstream result。如果 transient running state 很重要，应通过 prompt 触发缓慢的 foreground operation，并观察其具体 DOM marker；continuous video 会捕获其中间帧。让 model 输出一个简短的最终 sentinel 来标记完成。演示状态可见后，停止不必要地长时间运行的真实 API 操作。

不得捕获 secret、personal data、无关 tab 或 notification。browser video 包含 page content，不包含 browser chrome；避免在应用中渲染包含凭据的 URL。审查整个选定区间，包括所有中间状态。全程使用同一个 viewport。

## 编码 GIF

要求安装 `python3`、`ffmpeg` 和 `ffprobe`。如果缺少 media binary，应报告该 dependency，而不是未经授权安装软件。使用前单独一行 export `GIF_SKILL_DIR`；同一条命令中的 inline assignment 不会影响该命令的参数展开。

```sh
export GIF_SKILL_DIR=/absolute/path/to/this/skill
python3 "$GIF_SKILL_DIR/scripts/encode_gif.py" \
  /absolute/path/to/demo.webm \
  /absolute/path/to/demo.gif \
  --start 2 --end 32 --speed 2 --final-hold 3 \
  --fps 10 --max-width 1200 --colors 128
```

`--start` 和 `--end` 选择一个连续的 source interval，单位为秒。默认保留完整 video，以 1× speed 播放，并增加两秒的最终停留。`--speed` 会改变播放速度；应在 GIF 旁披露该值和所选 interval，避免 demo 暗示测量得到的 response latency。使用观察到的 video 时间，不要猜测 wall-clock offset，并保留被演示行为完整的 cause 和 outcome。最终停留会重复最后一个选定帧。`--fps` 设置编码后 GIF 的 frame rate；提高该值无法恢复 source recording 没有捕获的 motion。保留原始 WebM 用于 QA；不要拼接不同 run，也不要合成缺失状态。

encoder 会探测 WebM container duration，在 palette conversion 前应用 trim 和 speed，并检查编码后的 duration、animation、width 和 byte size。它会拒绝空的或超出范围的 interval、短于两个 output frame 的选择、不适用于当前 mode 的 flag，以及意外 overwrite。对于过大的 artifact，依次降低 `--max-width`、`--colors` 或 `--fps`；同时保持文本可读。只有在解析出确切 output path 后才能使用 `--force`。

### Screenshot capture

当 continuous video 不可用或用户要求 storyboard 时，遵循现有 browser-control 工作流。使用 browser screenshot API，从一次隔离运行中捕获三到六个已验证状态。将返回的 image byte 直接存到一个 run directory 下，并命名为 `00-initial.png`、`01-typed.png` 等；使用一致的尺寸和 crop。对于 transient state，在同一次 browser-script call 中轮询其 DOM marker 并捕获。

```sh
python3 "$GIF_SKILL_DIR/scripts/encode_gif.py" \
  /absolute/path/to/frames /absolute/path/to/demo.gif \
  --durations 1.5,1.5,1.5,3.5 --fps 10 --max-width 1200 --colors 128
```

一个 duration 值会应用于每张 screenshot；否则应为每帧提供一个正 duration，并让 settled state 的停留时间最长。directory input 会拒绝少于两帧、尺寸不匹配或 duration 数量不匹配的输入。video timing flag 仅适用于 video file；`--durations` 和 `--pattern` 仅适用于 screenshot directory。

## 验证 artifact

1. 阅读 encoder 的 JSON summary，确认 output path、source interval 和 speed（或 screenshot count）、编码后的 frame count、dimensions、duration 和 byte size。
2. 目视检查编码后的 GIF 本身，而不只是 source frame。确认 transition 清晰可读，最后状态停留足够长，并且没有 sensitive content。如果 viewer 只渲染第一帧，使用 `ffmpeg` 从编码后的 GIF 解码 representative frame 并检查；编码前的 screenshot 无法证明编码后的顺序、palette 或最终停留。
3. 运行 `git status --short`，确认 raw video、QA frame 和 artifact 只出现在 ignored path 下。
4. 返回 GIF 的绝对路径，在 client 支持 local media 时渲染它，并说明录制使用的是真实 API、fixture 还是其他 transport。当任务不包含把 GIF 附加到 pull request 时，到此停止。

encoder 维护：安装 media prerequisite 后，运行 `python3 -m unittest discover -s "$GIF_SKILL_DIR/scripts" -p 'test_*.py' -v`。这些 local media test 不在仓库 CI 中运行。

## 发布 GIF

仅当任务包含将 GIF 附加到 pull request 时，才执行此步骤。

绝不能将 GIF commit 到 pull request 自身的 branch，或任何会 merge 到 long-lived branch 的 branch：在那里 commit binary media 会让每一次未来 clone 的 repository history 变得臃肿。优先使用 `gh --attach`，它会在一条命令中上传 GIF 并重写 body reference，因此没有 branch 承载 media。

### 使用 gh 附加

`gh --attach` 要求 `gh` v2.99.0 或更高版本（用 `gh --version` 检查）、位于 github.com 的 repository（不支持 GitHub Enterprise Server）、对 repository 有 write access，且 GIF 不超过 10 MB。确认已验证的 artifact 符合该限制；若不符合，应在附加前依次通过 `--max-width`、`--colors` 或 `--fps` 缩小。

在 body file 中将 GIF 写为普通的 local-path reference，使用与传给 `--attach` 相同的路径；`gh` 会原地将 reference 重写为上传后的 URL，同时保留其位置和 alt text：

```markdown
![<alt text>](<path/to/demo.gif>)
```

通常，所演示的 pull request 就是发布目标。tooling pull request 也可以嵌入来自另一个 pull request 的明确标记示例；在下述每项检查中，都要注明该 source PR 并比较其 live head。绝不能把该示例归属于 tooling branch。

附加前一刻，重新读取所演示 pull request 的 live head；对于新的待演示 pull request，则读取已推送的 branch tip。将它与 GIF 旁记录的 commit 比较。如果已移动，停止并重新录制。然后附加：

```sh
gh pr create --body-file <body.md> --attach <path/to/demo.gif>     # new pull request
gh pr edit <pr> --body-file <body.md> --attach <path/to/demo.gif>  # existing pull request
```

`--attach` 可重复指定，但拒绝同一个 file 两次。body 未引用的 GIF 会附加在末尾，此时 flag 上设置的 alt text（`--attach '<path>#<alt text>'`）生效；被重写的 reference 则保留 body 中的 alt text。附加后，再次读取所演示的 live head，并要求它仍位于所记录的 commit。重新读取 live body，确认 reference 现在指向上传后的 URL；通过 GitHub Markdown API 渲染 body 并确认预期的 `<img>`，再 fetch 一次上传后的 URL，确认返回 `200` 和 `image/gif`。

### 回退到 assets branch

只有在 `gh --attach` 无法使用时才采用 assets-branch 工作流：GIF 仍大于 10 MB、`gh` 版本早于 v2.99.0，或 repository 不在 github.com。此时 GIF 位于专用的 orphan assets branch——一个没有 parent commit 且只包含 media 的 branch；一个 assets branch 服务于整个 pull request series（命名为 `<series>-assets`；使用 `git ls-remote --heads origin '*assets*'` 列出现有分支）。

在下述任一工作流 push 前，验证 assets branch 只包含 media，并且 staged GIF 的 checksum 与已验证的 local artifact 一致。

对于现有 assets branch，在 shallow single-branch scratch clone 中工作，避免发布操作触碰当前 working tree：

```sh
git clone --branch <assets-branch> --single-branch --depth 1 <repo-url> /tmp/assets-checkout
cp /absolute/path/to/demo.gif /tmp/assets-checkout/<name>.gif
cd /tmp/assets-checkout
git add <name>.gif
git commit -m "assets: <what it shows> gif (#<pr>)"
git push origin <assets-branch>
```

对于新 series，创建全新的 shallow scratch clone（`git clone --depth 1 <repo-url> /tmp/assets-checkout`），使用 `git switch --orphan <assets-branch>` 创建 orphan branch，然后以相同方式添加 GIF、commit 并 push。

push 后，使用经过身份验证的 GitHub API 或 raw request，确认 remote path、byte size、checksum、`200` response 和 `image/gif` content type。匿名请求返回 `404` 并不能否定 private-repository asset；改用经过身份验证的验证。这证明的是 repository member 的 review path，而不是 public availability。

编辑 pull-request body 前一刻，重新读取所演示 pull request 的 live head，并与 GIF 旁记录的 commit 比较。如果已移动，停止并重新录制。编辑后，再次读取所演示的 live head，并要求它仍位于所记录的 commit。另外，通过 GitHub Markdown API 渲染 body 并确认其中存在预期的 `<img>`。

使用 raw blob URL 将 GIF 嵌入 pull request body；必须添加 `?raw=true` suffix，因为普通 blob URL 渲染的是 GitHub file page，而不是 image：

```markdown
![<alt text>](https://github.com/<owner>/<repo>/blob/<assets-branch>/<name>.gif?raw=true)
```

绝不能删除或重写 assets branch，也绝不能对它 force-push：已 merge 的 pull request body 会永久引用其中的 URL。只能追加新的 commit。
