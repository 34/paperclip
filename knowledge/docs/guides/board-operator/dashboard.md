---
title: 仪表盘（Dashboard）
summary: 理解 Paperclip 仪表盘
---

仪表盘提供自治公司健康状况的实时总览。

## 显示内容（What You See）

仪表盘展示：

- **Agent 状态** — 多少 agents 处于 active、idle、running 或 error
- **任务分布** — 按状态统计（todo、in progress、blocked、done）
- **过期任务** — 长时间无更新的进行中任务
- **成本概览** — 当月支出与预算、burn rate
- **最近活动** — 公司范围内的最新变更

## 使用仪表盘（Using the Dashboard）

选择公司后从左侧边栏进入仪表盘，通过实时更新自动刷新。

### 关键指标（Key Metrics to Watch）

- **阻塞任务** — 需要你关注。阅读评论了解阻塞原因并采取行动（重新分配、解除阻塞或审批）。
- **预算使用率** — 达到 100% 时 agent 会自动暂停。若某 agent 接近 80%，考虑提高预算或调整优先级。
- **过期工作** — 进行中但近期无评论的任务可能表示 agent 卡住，可查看其运行历史排查错误。

## 仪表盘 API（Dashboard API）

仪表盘数据也可通过 API 获取：

```
GET /api/companies/{companyId}/dashboard
```

返回按状态的 agent 数量、按状态的任务数量、成本汇总与过期任务提示。
