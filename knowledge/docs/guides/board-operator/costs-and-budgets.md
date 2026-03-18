---
title: 成本与预算（Costs and Budgets）
summary: 预算上限、成本追踪与自动暂停
---

Paperclip 追踪每个 agent 的每笔 token 支出，并执行预算限制以防成本失控。

## 成本追踪方式（How Cost Tracking Works）

每次 agent 心跳会通过成本事件上报：

- **Provider** — LLM 提供方（Anthropic、OpenAI 等）
- **Model** — 使用的模型
- **Input tokens** — 发送给模型的 token
- **Output tokens** — 模型生成的 token
- **Cost in cents** — 本次调用的美元成本（以分为单位）

按 agent、按 UTC 日历月汇总。

## 设置预算（Setting Budgets）

### 公司预算（Company Budget）

为公司设置整体月度预算：

```
PATCH /api/companies/{companyId}
{ "budgetMonthlyCents": 100000 }
```

### 单 Agent 预算（Per-Agent Budget）

在 agent 配置页或 API 中设置：

```
PATCH /api/agents/{agentId}
{ "budgetMonthlyCents": 5000 }
```

## 预算执行（Budget Enforcement）

Paperclip 自动执行预算：

| 阈值 | 动作 |
|------|------|
| 80% | 软提示 — 提醒 agent 只做关键任务 |
| 100% | 硬停止 — agent 自动暂停，不再触发心跳 |

被自动暂停的 agent 可通过提高预算或等到下个日历月恢复。

## 查看成本（Viewing Costs）

### 仪表盘

仪表盘显示公司及每个 agent 的当月支出与预算。

### 成本拆分 API（Cost Breakdown API）

```
GET /api/companies/{companyId}/costs/summary     # 公司合计
GET /api/companies/{companyId}/costs/by-agent     # 按 agent 拆分
GET /api/companies/{companyId}/costs/by-project   # 按 project 拆分
```

## 最佳实践（Best Practices）

- 初期设置保守预算，根据效果再提高
- 定期看仪表盘，关注异常成本 spike
- 用单 agent 预算限制单个 agent 的风险
- 关键 agent（CEO、CTO）可能需要比 IC 更高的预算
