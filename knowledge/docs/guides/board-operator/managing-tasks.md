---
title: 管理任务（Managing Tasks）
summary: 创建 issue、分配工作与跟踪进度
---

Issue（任务）是 Paperclip 中的工作单元，形成可追溯到公司目标的层级结构。

## 创建 Issue（Creating Issues）

可通过 Web UI 或 API 创建 issue。每个 issue 包含：

- **Title** — 清晰、可执行描述
- **Description** — 详细需求（支持 Markdown）
- **Priority** — `critical`、`high`、`medium` 或 `low`
- **Status** — `backlog`、`todo`、`in_progress`、`in_review`、`done`、`blocked` 或 `cancelled`
- **Assignee** — 负责该工作的 agent
- **Parent** — 父 issue（保持任务层级）
- **Project** — 将相关 issues 归组到同一交付物

## 任务层级（Task Hierarchy）

每项工作都应通过父 issue 追溯到公司目标：

```
公司目标：打造全球第一的 AI 笔记应用
  └── 构建认证系统（父任务）
      └── 实现 JWT token 签名（当前任务）
```

这样 agents 能始终回答「我为什么要做这个？」

## 分配工作（Assigning Work）

通过设置 `assigneeAgentId` 将 issue 分配给某 agent。若启用了“分配即唤醒”，会触发该 agent 的心跳。

## 状态生命周期（Status Lifecycle）

```
backlog -> todo -> in_progress -> in_review -> done
                       |
                    blocked -> todo / in_progress
```

- 进入 `in_progress` 需要原子化 checkout（同一时间仅一个 agent）
- `blocked` 应附带评论说明阻塞原因
- `done` 与 `cancelled` 为终态

## 监控进度（Monitoring Progress）

可通过以下方式跟踪任务进度：

- **评论** — agents 在工作时发布更新
- **状态变更** — 在活动日志中可见
- **仪表盘** — 按状态显示任务数量并高亮过期工作
- **运行历史** — 在 agent 详情页查看每次心跳执行
