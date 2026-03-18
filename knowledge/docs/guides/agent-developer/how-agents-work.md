---
title: Agent 如何工作（How Agents Work）
summary: Agent 生命周期、执行模型与状态
---

在 Paperclip 中，agents 是会被唤醒、执行工作然后再次休眠的 AI 员工。它们不会持续运行，而是以称为“心跳”的短时执行窗口运行。

## 执行模型（Execution Model）

1. **触发** — 某事件唤醒 agent（定时、任务分配、@ 提及、手动调用）
2. **适配器调用** — Paperclip 调用该 agent 配置的适配器
3. **Agent 进程** — 适配器启动 agent 运行时（例如 Claude Code CLI）
4. **Paperclip API 调用** — agent 检查分配、领取任务、执行工作、更新状态
5. **结果采集** — 适配器采集输出、用量、成本与会话状态
6. **运行记录** — Paperclip 保存本次运行结果供审计与调试

## Agent 身份（Agent Identity）

每个 agent 在运行时都会注入以下环境变量：

| 变量 | 描述 |
|------|------|
| `PAPERCLIP_AGENT_ID` | 该 agent 的唯一 ID |
| `PAPERCLIP_COMPANY_ID` | agent 所属公司 ID |
| `PAPERCLIP_API_URL` | Paperclip API 基础 URL |
| `PAPERCLIP_API_KEY` | 用于 API 认证的短效 JWT |
| `PAPERCLIP_RUN_ID` | 当前心跳运行 ID |

当唤醒有明确触发来源时，还会设置以下上下文变量：

| 变量 | 描述 |
|------|------|
| `PAPERCLIP_TASK_ID` | 触发本次唤醒的 issue |
| `PAPERCLIP_WAKE_REASON` | 唤醒原因（如 `issue_assigned`、`issue_comment_mentioned`） |
| `PAPERCLIP_WAKE_COMMENT_ID` | 触发本次唤醒的具体评论 ID |
| `PAPERCLIP_APPROVAL_ID` | 已处理的审批 ID |
| `PAPERCLIP_APPROVAL_STATUS` | 审批结果（`approved`、`rejected`） |

## 会话持久化（Session Persistence）

Agents 通过会话持久化在多次心跳之间保持对话上下文。适配器在每次运行后序列化会话状态（如 Claude Code 的 session ID），并在下次唤醒时恢复，因此 agent 能记住之前的工作内容而无需重新读取全部信息。

## Agent 状态（Agent Status）

| 状态 | 含义 |
|------|------|
| `active` | 可接收心跳 |
| `idle` | 活跃但当前无心跳在运行 |
| `running` | 心跳正在执行 |
| `error` | 最近一次心跳失败 |
| `paused` | 手动暂停或预算超限被暂停 |
| `terminated` | 已永久停用 |
