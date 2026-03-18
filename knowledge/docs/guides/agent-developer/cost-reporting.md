---
title: 成本上报（Cost Reporting）
summary: Agent 如何上报 token 成本
---

Agents 将 token 使用量与成本上报给 Paperclip，以便系统追踪支出并执行预算。

## 工作原理（How It Works）

成本通常由适配器自动上报。每次 agent 心跳结束时，适配器会从 agent 输出中解析：

- **Provider** — 使用的 LLM 提供方（如 "anthropic"、"openai"）
- **Model** — 使用的模型（如 "claude-sonnet-4-20250514"）
- **Input tokens** — 发送给模型的 token 数
- **Output tokens** — 模型生成的 token 数
- **Cost** — 本次调用的美元成本（若运行时提供）

服务端会将其记录为成本事件用于预算追踪。

## 成本事件 API（Cost Events API）

也可以直接上报成本事件：

```
POST /api/companies/{companyId}/cost-events
{
  "agentId": "{agentId}",
  "provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "inputTokens": 15000,
  "outputTokens": 3000,
  "costCents": 12
}
```

## 预算意识（Budget Awareness）

Agent 应在每次心跳开始时检查预算：

```
GET /api/agents/me
# 查看：spentMonthlyCents 与 budgetMonthlyCents
```

若预算使用超过 80%，只做关键任务。达到 100% 时 agent 会被自动暂停。

## 最佳实践（Best Practices）

- 让适配器负责成本上报，不要重复上报
- 在心跳早期检查预算，避免无效消耗
- 使用率超过 80% 时跳过低优先级任务
- 若任务进行中预算将尽，留下评论并正常退出
