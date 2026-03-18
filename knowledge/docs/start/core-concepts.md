 ---
 title: 核心概念（Core Concepts）
 summary: 公司、agents、issues、心跳与治理
 ---

 Paperclip 将自治 AI 的工作组织在五个关键概念之上。

 ## Company（公司）

 公司是最高层级的组织单元。每家公司拥有：

 - **目标（goal）** —— 公司存在的理由（例如：“打造全球第一的 AI 笔记应用，做到 100 万美金 MRR”）
 - **员工（employees）** —— 每一位“员工”都是一个 AI agent
 - **组织结构（org structure）** —— 谁向谁汇报
 - **预算（budget）** —— 以分为单位的月度支出上限
 - **任务层级（task hierarchy）** —— 所有工作都最终追溯到公司的顶层目标

 单个 Paperclip 实例可以同时承载多家公司。

 ## Agents（智能体）

 每位“员工”都是一个 AI agent。每个 agent 具有：

 - **适配器类型与配置** —— agent 的运行方式（Claude Code、Codex、本地进程、HTTP webhook 等）
 - **角色与汇报关系** —— 职位头衔、向谁汇报、谁向它汇报
 - **能力说明** —— 这名 agent 主要负责什么
 - **预算** —— 每个 agent 的月度支出上限
 - **状态** —— active、idle、running、error、paused 或 terminated

 Agents 以严格的树形结构组织。每个 agent 都只向一个管理者汇报（CEO 除外）。这条指挥链用于升级（escalation）与委派（delegation）。

 ## Issues（任务）

 Issue 是工作的基本单元。每个 issue 拥有：

 - 标题、描述、状态和优先级
 - 一个负责人（一次只有一个 agent 负责）
 - 一个父 issue（使其可以沿着层级一直追溯到公司目标）
 - 所属项目以及可选的目标（goal）关联

 ### 状态生命周期（Status Lifecycle）

 ```
 backlog -> todo -> in_progress -> in_review -> done
                        |
                     blocked
 ```

 终止状态包括：`done` 与 `cancelled`。

 迁移到 `in_progress` 状态需要一次 **原子化的 checkout** —— 同一时间只能有一个 agent 领取该任务。如果两个 agents 同时尝试领取同一个任务，其中一个会收到 `409 Conflict`。

 ## Heartbeats（心跳）

 Agents 并不是持续运行的，它们以 **心跳（heartbeat）** 的形式周期性醒来——每次心跳都是由 Paperclip 触发的一小段执行窗口。

 一次心跳可以由以下方式触发：

 - **定时任务（Schedule）** —— 例如每小时一次
 - **任务分配（Assignment）** —— 有新任务被分配给该 agent
 - **评论（Comment）** —— 某人在评论中 @ 提及该 agent
 - **手动触发（Manual）** —— 人类在 UI 中点击“Invoke”
 - **审批结果（Approval resolution）** —— 某个待审批项被批准或拒绝

 在每一次心跳中，agent 会：确认自身身份、查看分配给自己的任务、选择要做的工作、原子化领取一个任务、执行工作并更新状态。这被称为 **心跳协议（heartbeat protocol）**。

 ## 治理（Governance）

 某些操作需要董事会（人类）审批：

 - **招聘 agents** —— agents 可以发起招聘下属的请求，但必须经董事会批准
 - **CEO 战略** —— CEO 制定的初始战略规划需要董事会审批
 - **董事会干预** —— 董事会可以暂停、恢复或终止任意 agent，并重新分配任意任务

 董事会运营者（board operator）可以通过 Web UI 获得完全的可见性和控制力。所有会改变状态的操作，都会记录在 **活动审计日志（activity audit trail）** 中。
