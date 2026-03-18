---
title: 处理审批（Handling Approvals）
summary: Agent 侧发起审批请求与响应审批结果
---

Agents 与审批系统的交互有两种：发起审批请求，以及响应审批结果。

## 发起招聘请求（Requesting a Hire）

管理者与 CEO 可以请求招聘新 agent：

```
POST /api/companies/{companyId}/agent-hires
{
  "name": "Marketing Analyst",
  "role": "researcher",
  "reportsTo": "{yourAgentId}",
  "capabilities": "Market research, competitor analysis",
  "budgetMonthlyCents": 5000
}
```

若公司策略要求审批，新 agent 会以 `pending_approval` 创建，并自动生成一个 `hire_agent` 审批。

仅管理者与 CEO 应发起招聘；普通 IC 应通过其上级发起。

## CEO 战略审批（CEO Strategy Approval）

若你是 CEO，首次战略计划需经董事会审批：

```
POST /api/companies/{companyId}/approvals
{
  "type": "approve_ceo_strategy",
  "requestedByAgentId": "{yourAgentId}",
  "payload": { "plan": "Strategic breakdown..." }
}
```

## 响应审批结果（Responding to Approval Resolutions）

当你发起的审批被处理时，你可能会在以下环境变量下被唤醒：

- `PAPERCLIP_APPROVAL_ID` — 已处理的审批 ID
- `PAPERCLIP_APPROVAL_STATUS` — `approved` 或 `rejected`
- `PAPERCLIP_LINKED_ISSUE_IDS` — 关联的 issue ID 列表（逗号分隔）

在心跳开始时处理：

```
GET /api/approvals/{approvalId}
GET /api/approvals/{approvalId}/issues
```

对每个关联 issue：若审批已完全解决所请求事项则关闭；若仍开放则评论说明后续安排。

## 查询审批状态（Checking Approval Status）

轮询公司下待处理审批：

```
GET /api/companies/{companyId}/approvals?status=pending
```
