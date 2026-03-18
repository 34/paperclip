# Paperclip 规范（Paperclip Specification）

Paperclip 控制平面的目标规范。活文档，在规范访谈中逐步更新。

---

## 1. 公司模型（Company Model）[草稿]

公司是一级对象。一个 Paperclip 实例运行多家公司。公司没有独立的「目标」字段——其方向由 Initiatives 集合定义（见任务层级映射）。

### 字段（草稿）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | uuid | 主键 |
| `name` | string | 公司名称 |
| `createdAt` / `updatedAt` | timestamp | |

### 董事会治理（Board Governance）[草稿]

每家公司有一个**董事会**负责高影响决策，董事会是人类监督层。

**V1：单一人类董事会。** 一名人类运营者。

#### 董事会审批闸门（V1）

- 新 agent 招聘（创建新 agents）
- CEO 的首次战略拆解（CEO 提议，董事会批准后执行）
- [待定：其他治理闸门——目标变更、解雇 agents？]

#### 董事会权限（始终可用）

董事会**随时**拥有系统**无限制访问**：设置与修改公司预算、暂停/恢复任意 agent、暂停/恢复任意工作项（任务、项目、子任务树、里程碑）、完整项目管理访问（创建/编辑/评论/修改/删除/重新分配）、覆盖任意 agent 决策、手动调整任意层级预算。董事会不仅是审批闸门，更是实时控制面。

#### 预算委派

董事会设置公司级预算；CEO 可为下属 agents 设预算，各级管理者同理。权限结构支持这种级联委派；具体规则待定。董事会可随时手动覆盖任意层级预算。

（未来治理模型、开放问题等见英文原文。）

---

## 2. Agent 模型（Agent Model）[草稿]

每位员工都是一个 agent；agents 是劳动力。

### Agent 身份（适配器层）（Agent Identity - Adapter-Level）

SOUL.md（身份/使命）、HEARTBEAT.md（循环定义）等概念**不属于 Paperclip 协议**，而是适配器特定配置。例如 OpenClaw 适配器可能使用 SOUL.md 与 HEARTBEAT.md；Claude Code 适配器可能使用 CLAUDE.md；裸脚本可能用命令行参数。Paperclip 不规定 agent 如何定义身份或行为，只提供控制平面；适配器定义 agent 内部运作。

### Agent 配置（Agent Configuration）[草稿]

每个 agent 有**适配器类型**与**适配器特定配置 blob**。协议层面 Paperclip 追踪：身份与汇报、能力描述、预算与成本等。（完整字段与行为见英文原文。）

（其余章节见英文原文。）
