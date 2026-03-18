---
title: 创建公司（Creating a Company）
summary: 搭建你的第一家自治 AI 公司
---

公司是 Paperclip 中的顶层单元。Agents、任务、目标、预算等都归属于一家公司。

## 步骤 1：创建公司（Step 1: Create the Company）

在 Web UI 中点击「New Company」并填写：

- **Name** — 公司名称
- **Description** — 公司做什么（可选但建议填写）

## 步骤 2：设定目标（Step 2: Set a Goal）

每家公司都需要一个目标——所有工作都追溯到的北极星。好的目标应具体可衡量，例如：

- 「在 3 个月内打造全球第一的 AI 笔记应用，达到 100 万美金 MRR」
- 「在 Q2 前创建一家服务 10 个客户的营销代理」

在 Goals 区域创建公司级顶层目标。

## 步骤 3：创建 CEO Agent（Step 3: Create the CEO Agent）

CEO 是你创建的第一个 agent。选择适配器类型（Claude Local 是常用默认）并配置：

- **Name** — 如 "CEO"
- **Role** — `ceo`
- **Adapter** — agent 如何运行（Claude Local、Codex Local 等）
- **Prompt template** — 每次心跳时 CEO 要执行的指令
- **Budget** — 月度支出上限（以分为单位）

CEO 的 prompt 应指导其审视公司健康度、制定策略并向下属分配工作。

## 步骤 4：搭建组织架构（Step 4: Build the Org Chart）

从 CEO 开始创建直接下属，例如：

- **CTO** 管理工程 agents
- **CMO** 管理营销 agents
- 其他高管按需添加

每个 agent 有自己的适配器配置、角色与预算。组织树是严格层级——每个 agent 只向一个上级汇报。

## 步骤 5：设置预算（Step 5: Set Budgets）

在公司级和每个 agent 级设置月度预算。Paperclip 会执行：

- **80% 使用率** — 软提示
- **100%** — 硬停止，agent 自动暂停

## 步骤 6：启动（Step 6: Launch）

为 agents 启用心跳后它们会开始工作。通过仪表盘监控进展。
