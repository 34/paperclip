# Agent 运行时指南（Agent Runtime Guide）

状态：面向用户的指南  
最近更新：2026-02-17  
受众：在 Paperclip 中配置与运行 agents 的运营者

## 1. 系统做什么（What this system does）

Paperclip 中的 agents 不持续运行，而是以**心跳**形式在短时执行窗口中由唤醒触发运行。每次心跳：启动已配置的适配器、提供当前 prompt/上下文、让其运行直至退出/超时/取消、存储结果（状态、token 用量、错误、日志）、实时更新 UI。

## 2. Agent 何时被唤醒（When an agent wakes up）

四种方式：`timer`（定时）、`assignment`（任务分配/领取）、`on_demand`（手动）、`automation`（系统自动化）。若 agent 已在运行，新唤醒会合并而非重复启动。

## 3. 每个 agent 的配置（What to configure per agent）

### 3.1 适配器选择

常见：`claude_local`、`codex_local`、`process`、`http`。`claude_local`/`codex_local` 假定 CLI 已在主机安装并完成认证。

### 3.2 运行时行为

心跳策略：`enabled`、`intervalSec`、`wakeOnAssignment`、`wakeOnOnDemand`、`wakeOnAutomation`。

### 3.3 工作目录与执行限制

本地适配器需设置：`cwd`、`timeoutSec`、`graceSec`，以及可选的 env 与额外 CLI 参数；可使用「Test environment」在保存前做适配器诊断。

### 3.4 提示词模板

支持 `promptTemplate` 及 `{{agent.id}}`、`{{agent.name}}` 等变量。

（会话恢复、日志与状态、UI 实时更新、常见运行模式、故障排查、安全与最简检查清单等见英文原文或 `docs/agents-runtime.md` 的中文版。）
