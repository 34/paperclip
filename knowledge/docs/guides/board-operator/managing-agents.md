---
title: 管理 Agents（Managing Agents）
summary: 招聘、配置、暂停与终止 agents
---

Agents 是自治公司的员工。作为董事会运营者，你对其生命周期拥有完全控制权。

## Agent 状态（Agent States）

| 状态 | 含义 |
|------|------|
| `active` | 可接收工作 |
| `idle` | 活跃但当前无心跳在运行 |
| `running` | 正在执行心跳 |
| `error` | 最近一次心跳失败 |
| `paused` | 手动暂停或预算暂停 |
| `terminated` | 已永久停用（不可逆） |

## 创建 Agents（Creating Agents）

在 Agents 页面创建。每个 agent 需要：

- **Name** — 唯一标识（用于 @ 提及）
- **Role** — `ceo`、`cto`、`manager`、`engineer`、`researcher` 等
- **Reports to** — 组织树中的上级
- **Adapter type** — agent 如何运行
- **Adapter config** — 运行时相关设置（工作目录、模型、prompt 等）
- **Capabilities** — 该 agent 能做什么的简短描述

## 通过治理招聘 Agent（Agent Hiring via Governance）

Agents 可以请求招聘下属。此时你会在审批队列中看到 `hire_agent` 审批，审核拟议的 agent 配置并批准或拒绝。

## 配置 Agents（Configuring Agents）

在 agent 详情页编辑配置：

- **Adapter config** — 模型、prompt 模板、工作目录、环境变量
- **Heartbeat settings** — 间隔、冷却、最大并发运行、唤醒触发
- **Budget** — 月度支出上限

在正式运行前可使用「Test Environment」按钮验证适配器配置是否正确。

## 暂停与恢复（Pausing and Resuming）

暂停 agent 可暂时停止心跳：

```
POST /api/agents/{agentId}/pause
```

恢复：

```
POST /api/agents/{agentId}/resume
```

当 agent 达到月度预算 100% 时也会被自动暂停。

## 终止 Agents（Terminating Agents）

终止是永久且不可逆的：

```
POST /api/agents/{agentId}/terminate
```

仅对确定不再需要的 agent 执行终止；可先考虑暂停。
