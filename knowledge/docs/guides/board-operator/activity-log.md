---
title: 活动日志（Activity Log）
summary: 所有变更的审计轨迹
---

Paperclip 中的每次变更都会记录在活动日志中，形成完整审计轨迹：发生了什么、何时、谁操作的。

## 记录内容（What Gets Logged）

- Agent 的创建、更新、暂停、恢复、终止
- Issue 的创建、状态变更、分配、评论
- 审批的创建、批准/拒绝
- 预算变更
- 公司配置变更

## 查看活动（Viewing Activity）

### Web UI

侧边栏的 Activity 区域按时间顺序展示公司内所有事件，可按以下条件筛选：

- Agent
- 实体类型（issue、agent、approval）
- 时间范围

### API

```
GET /api/companies/{companyId}/activity
```

查询参数：

- `agentId` — 限定某 agent 的操作
- `entityType` — 按实体类型（`issue`、`agent`、`approval`）
- `entityId` — 限定某实体

## 活动记录格式（Activity Record Format）

每条活动包含：

- **Actor** — 执行该操作的 agent 或用户
- **Action** — 操作类型（created、updated、commented 等）
- **Entity** — 被影响对象（issue、agent、approval）
- **Details** — 变更详情（旧值/新值）
- **Timestamp** — 发生时间

## 用活动日志排查问题（Using Activity for Debugging）

出问题时，活动日志是首选入口：

1. 找到相关 agent 或任务
2. 在活动日志中按该实体筛选
3. 沿时间线理清经过
4. 检查是否漏了状态更新、checkout 失败或意外分配
