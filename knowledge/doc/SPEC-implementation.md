# Paperclip V1 实现规范（Paperclip V1 Implementation Spec）

状态：首版（V1）实现契约  
日期：2026-02-17  
受众：产品、工程与 agent 集成作者  
来源：`GOAL.md`、`PRODUCT.md`、`SPEC.md`、`DATABASE.md` 及当前 monorepo 代码

## 1. 文档角色（Document Role）

`SPEC.md` 为长期产品规范。本文档为可落地的 V1 契约；若有冲突，以 `SPEC-implementation.md` 为准。

## 2. V1 结果（V1 Outcomes）

Paperclip V1 须提供完整的自治 agent 控制平面闭环：

1. 人类董事会创建公司并定义目标。
2. 董事会在组织树中创建与管理 agents。
3. Agents 通过心跳调用接收并执行任务。
4. 所有工作通过任务/评论追踪并具审计可见性。
5. Token/成本用量被上报，预算限制可中止工作。
6. 董事会可在任意处干预（暂停 agent/任务、覆盖决策）。

成功标准：一名运营者能端到端运行一家小型 AI 原生公司，并具备清晰可见性与控制力。

## 3. 明确的 V1 产品决策（Explicit V1 Product Decisions）

| 主题 | V1 决策 |
|---|---|
| 租户 | 单租户部署，多公司数据模型 |
| 公司模型 | 公司为一级实体；所有业务实体均以公司为作用域 |
| 董事会 | 每个部署单一人类董事会运营者 |
| 组织图 | 严格树（`reports_to` 根可空）；无多上级汇报 |
| 可见性 | 董事会与同公司所有 agents 完全可见 |
| 沟通 | 仅任务 + 评论（无独立聊天系统） |
| 任务归属 | 单一负责人；进入 `in_progress` 需原子 checkout |
| 恢复 | 无自动重新分配；过期工作被暴露而非静默修复 |
| Agent 适配器 | 内置 `process` 与 `http` 适配器 |
| 认证 | 人类认证依模式（当前代码中 `local_trusted` 为隐式 board；认证模式使用会话）；agents 使用 API key |
| 预算周期 | 月度 UTC 日历窗口 |
| 预算执行 | 软提示 + 硬限制自动暂停 |
| 部署模式 | 规范模型为 `local_trusted` + `authenticated`（含 private/public 暴露策略，见 `doc/DEPLOYMENT-MODES.md`） |

## 4. 当前基线（Repo 快照）（Current Baseline）

截至 2026-02-17，仓库已包含：Node + TypeScript 后端与 `agents`、`projects`、`goals`、`issues`、`activity` 的 REST CRUD；React UI 的 dashboard/agents/projects/goals/issues 列表；通过 Drizzle 的 PostgreSQL schema，未设置 `DATABASE_URL` 时使用嵌入式 PostgreSQL。V1 在此基线上扩展为公司中心、治理感知的控制平面。

## 5. V1 范围（V1 Scope）

### 5.1 在范围内（In Scope）

公司生命周期；与公司使命关联的目标层级；带组织与适配器配置的 agent 生命周期；带父子层级与评论的任务生命周期；原子任务 checkout 与显式状态迁移；招聘与 CEO 战略提议的董事会审批；心跳调用、状态追踪与取消；成本事件接入与汇总（agent/任务/project/公司）；预算设置与硬停止执行；董事会 Web UI（仪表盘、组织图、任务、agents、审批、成本）；面向 agent 的 API 契约（任务读写、心跳上报、成本上报）；所有变更操作的可审计活动日志。

### 5.2 不在范围内（V1）（Out of Scope）

插件框架与第三方扩展 SDK；超出模型/token 成本以外的收入/支出核算；知识库子系统；公开市场（ClipHub）；多董事会治理或人类细粒度角色权限；自动自愈编排（自动重分配/重试规划器）。

## 6. 架构（Architecture）

运行时组件：`server/`（REST API、认证、编排服务）、`ui/`（董事会运营者界面）、`packages/db/`（Drizzle schema、迁移、DB 客户端）、`packages/shared/`（共享 API 类型、校验器、常量）。数据存储：主库 PostgreSQL；本地默认嵌入式 PostgreSQL；可选 Docker/托管 Postgres；文件/对象存储默认本地磁盘，可选 S3。（其余章节见英文原文。）
