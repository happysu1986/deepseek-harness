---
name: agent-experience
description: 设计 agent tool、skill、context loading 或多步骤 workflow 时使用，使信息易于发现并高效使用 context。
metadata:
  date: "2026-09-11"
---

- **从最小 context 开始：** 首先公开用途、可用 action 和 constraint；需要时再加载详细说明。
- **明确 discovery 方式：** 每个延迟加载的 resource 都需要清楚描述，并提供可靠的获取方式。
- **保持关键 constraint 可见：** permission、destructive effect 和必需 validation 应出现在相关 action 之前。
- **优先使用有界 output：** 返回精简结果以及可用于获取详情的 identifier 或 path；避免倾倒完整 log 或 document。
- **利用 locality：** 在 operation result 中附带有限量、很可能需要的相邻 context。例如，提供 thread reply 时附带前几条 message。明确标记省略与截断，并提供获取更多内容的方式。
- **评估总工作量：** 只有不会导致更多 search、重复 read 或 mistake 时，节省 context 才有价值。
