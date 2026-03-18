## Paperclip 项目架构与设计总览

本文面向希望在 Paperclip 基础上进行功能扩展和二次开发的工程师，系统性梳理当前开源版本的整体设计与实现原理。内容基于仓库源码以及 `doc/GOAL.md`、`doc/PRODUCT.md`、`doc/SPEC-implementation.md`、`doc/DEVELOPING.md`、`doc/DATABASE.md` 等文档整理。

---

## 1. 产品定位与核心目标

- **项目愿景**  
  Paperclip 是为「自治 AI 公司」提供的 **控制平面（control plane）**，不是单个智能体，而是帮助你运营一整家公司级别的 AI 劳动力。

- **控制什么？**  
  - **公司层面**: 目标、预算、成本、活动审计、知识/资产等  
  - **组织结构**: 多个公司、每个公司内部的代理人（agents）和汇报关系  
  - **任务与工作流**: 目标 → 项目 → 任务（issues）→ 心跳执行（heartbeat runs）  
  - **治理与安全**: 审批（approvals）、预算硬阈值、权限与多租户隔离  

- **V1 成功标准（来自 `SPEC-implementation.md`）**  
  允许单个人类 operator 借助 Paperclip：
  1. 创建公司、定义目标  
  2. 创建并管理一个 org tree 内的多名 AI 员工（agents）  
  3. 通过心跳机制让 agents 接收任务并执行  
  4. 全程可观测：任务、评论、成本、运行记录、审批与活动日志  
  5. 能通过预算、暂停、审批等手段随时「刹车」或干预  

---

## 2. 代码仓库整体结构

仓库是一个 pnpm monorepo，根目录下核心模块如下：

- **`server/`**: Node.js + TypeScript 的 Express API 服务  
  - 暴露 `/api` REST 接口  
  - 实现鉴权、公司作用域控制、多租户隔离  
  - Agent 运行调度与心跳（heartbeat）系统  
  - 成本汇总、预算、审批、活动日志等领域服务  
  - Vite dev 中间件模式下同时托管前端 UI

- **`ui/`**: React + Vite 看板 UI  
  - 使用 `react-router-dom` 与自定义的路由封装，支持公司前缀路由  
  - 使用 React Query + WebSocket Live Events 实现实时数据刷新  
  - 封装统一的 API client，与 `server/` 通过共享类型/常量对齐协议  
  - 内置一套设计系统和组件库（表单、按钮、状态标签、布局等）

- **`packages/db/`**: Drizzle ORM + PostgreSQL 数据访问层  
  - 通过 TypeScript schema 定义所有业务表与关系  
  - 生成 SQL 迁移脚本并在 server 启动时自动应用  
  - 支持内嵌 PostgreSQL、本地 Docker Postgres 或托管 Postgres（例如 Supabase）

- **`packages/shared/`**: 跨层共享契约  
  - 领域类型（companies, agents, projects, issues, approvals, costs, activity 等）  
  - 各种枚举/常量（状态、优先级、适配器类型、部署模式等）  
  - 使用 Zod 定义的请求/响应验证 schema，作为服务端 API 的「真源」  
  - 配置 schema（deployment modes、secrets/storage/llms 等）

- **`cli/`**: Paperclip CLI 工具  
  - 提供一键运行（`paperclipai run`）、健康检查、上下文配置  
  - 提供 issue 管理、dashboard 等控制平面命令行入口

- **`doc/` / `docs/`**: 产品和工程规范文档  
  - `GOAL.md` / `PRODUCT.md` / `SPEC-implementation.md`：产品目标与 V1 实施合同  
  - `DEVELOPING.md` / `DATABASE.md` / `DOCKER.md` / `CLI.md` / `DEPLOYMENT-MODES.md` 等：开发环境、部署模式和运维说明

从依赖方向看大致为：

- `packages/shared` 被 `server`、`ui`、`cli`、`packages/db` 共同依赖  
- `packages/db` 被 `server` 依赖（实际 DB 操作）  
- `server` 为 `ui` 提供 API，并在 dev 模式下托管 `ui`  

---

## 3. 运行时架构与请求流

### 3.1 运行时组件

- **API Server (`server/`)**  
  - 基于 Express 和 TypeScript 构建  
  - 负责：
    - 加载和校验配置（数据库、部署模式、Secrets、Storage 等）  
    - 建立数据库连接（根据 `DATABASE_URL` 决定内嵌/外部 Postgres）  
    - 运行和对齐 Drizzle 迁移  
    - 初始化鉴权体系（local_trusted vs authenticated）  
    - 启动 HTTP server 与 WebSocket live events  
    - 启动心跳调度器（定时触发 agents heartbeat）  

- **Board UI (`ui/`)**  
  - 以单页应用（SPA）形式运行在浏览器中，通过 `/api` 与 server 通信  
  - 使用 React Query/Context 分别承载：公司选择、主题、侧边栏、对话框、面包屑、Toast、Live Updates 等  
  - 借助 WebSocket 接收 server 推送的 Live events 来实现实时更新

- **数据库 (`packages/db` + PostgreSQL)**  
  - 使用 Drizzle TypeScript schema 映射到 PostgreSQL 表  
  - 内嵌模式下，server 会自动在本机管理 Postgres 数据目录和迁移  
  - 外部/托管模式下，仍然通过统一 schema 与迁移脚本保持兼容

- **外部执行环境（代理适配器）**  
  - Agents 自身并不在 Paperclip 进程内执行，而是在外部环境中运行（Python/OpenClaw/其它 runtime）  
  - Paperclip 通过 **适配器（process/http 等）** 与 agent 通信：  
    - process adapter：本地进程命令执行（如脚本、Claude Code/Codex）  
    - http adapter：通过 HTTP/webhook 触发远程 agent  

### 3.2 请求流：UI → Server → DB → Agents

以一个典型任务流为例：

1. **Board 在 UI 中创建 Issue**  
   - UI 通过 `ui/src/api/issues.ts` 使用统一封装的 `api.post('/api/companies/:companyId/issues', payload)`  
   - Payload 类型与结构由 `@paperclipai/shared` 的 `createIssueSchema` 与 TS 类型决定  
2. **Server 路由处理**  
   - 通过 `actorMiddleware` 解析当前请求 actor（board user / agent key）  
   - 通过 `assertCompanyAccess` 确保请求只操作授权公司  
   - 使用 shared validators（Zod）进行请求体验证  
   - 调用 `issueService.create` 执行事务：
     - 生成公司作用域下的 issue identifier（如 `PAP-39`）  
     - 校验 assignee（agent/user）合法性和公司一致性  
     - 写入 `issues` 表、更新相关计数或索引  
     - 写入 `activity_log`（如 `issue.created`）  
     - （可选）根据 assignee 触发 agent 心跳唤醒  
3. **DB 层**  
   - `issueService` 使用 Drizzle client 在 PostgreSQL 上执行插入/更新  
   - 所有记录含 `companyId`，确保多租户隔离  
4. **实时反馈给 UI**  
   - server 同时通过 `publishLiveEvent` 将活动事件推送到 Live Events WebSocket  
   - UI `LiveUpdatesProvider` 订阅到事件后：
     - 根据事件类型 invalidation 对应的 React Query 缓存（如 issues 列表、dashboard 卡片）  
     - 显示 Toast 消息（如「新任务 PAP-39 已创建」）  

对于 agent 执行任务的心跳流程，会在第 5 节详细说明。

---

## 4. 数据模型与公司作用域

Paperclip 的数据模型在 `doc/SPEC-implementation.md` 第 7 章有详细定义，对应实现位于 `packages/db/src/schema`。这里从控制平面的视角做提炼。

### 4.1 公司与组织结构

- **`companies`**  
  - 每个公司有独立的名称、描述、状态（`active|paused|archived`）  
  - 几乎所有业务实体都引用 `companyId`，保证「公司是最小作用域」  

- **`agents`**  
  - 公司内的「员工」，本质上是可执行的 AI 代理  
  - 关键字段：
    - `companyId`：公司外键  
    - `name` / `role` / `title` / `status`（idle/running/error/paused/terminated 等）  
    - `reportsTo`：指向同表，形成**严格树状 org graph**  
    - `adapterType` / `adapterConfig` / `runtimeConfig`：确定 agent 如何执行、使用何种适配器  
    - `budgetMonthlyCents` / `spentMonthlyCents`：每月预算与已花费  
  - 不变量：
    - Manager 必须在同一个公司  
    - 不允许 org 树中出现环  
    - `terminated` 状态不可恢复

- **`agent_api_keys`**  
  - Agent 使用 Bearer token 访问控制平面 API  
  - 只持久化 `key_hash`，明文只在创建时展示一次  
  - 强制绑定 `agentId` 与 `companyId`，防止跨公司访问

### 4.2 目标、项目与任务

- **`goals`**  
  - 支持从公司级到任务级的目标层次：`company | team | agent | task`  
  - 通过 `parentId` 形成树形结构，实现「所有工作均可追溯到公司顶层目标」的原则  

- **`projects`**  
  - 代表一组相关任务的容器，可选地关联某个 `goalId`  
  - 支持状态（backlog/planned/in_progress/completed/cancelled）和 `leadAgentId` 等

- **`issues`**（核心任务实体）  
  - 关联字段：
    - `companyId`（必需）  
    - 可选 `projectId` / `goalId` / `parentId`  
    - `assigneeAgentId` 与（可选）`assigneeUserId`（V1 实现重点在 agent 端）  
    - 与心跳执行相关的 `checkoutRunId` / `executionRunId` / `executionLockedAt` 等  
  - 状态机：
    - `backlog → todo → in_progress → in_review → done` 以及 `blocked/cancelled` 分支  
    - 进入 `in_progress` 时设置 `startedAt`，进入 `done` 或 `cancelled` 时设置相应时间戳  
  - 不变量：
    - **单一受让人**：同一时刻不能既有 `assigneeAgentId` 又有 `assigneeUserId`  
    - `in_progress` 必须有 assignee  
    - 任务必须能通过 `goalId` / `parentId` 或项目-目标链追溯到公司级目标  

### 4.3 审批、成本与活动日志

- **`approvals`**  
  - 类型包括：`hire_agent`、`approve_ceo_strategy` 等  
  - 状态机：`pending → approved/rejected/cancelled` 等  
  - `payload` 存储结构化 JSON，由 shared validators 约束  
  - 与 `issues` 可通过 `issue_approvals` 建立关联

- **`cost_events`**  
  - 每个事件记录一次调用/运行的成本（provider、model、input/output tokens、cost_cents 等）  
  - 必须绑定 `companyId` 与 `agentId`，可选绑定 `issueId`/`projectId`/`goalId`  
  - 通过聚合来生成公司/agent/项目维度的成本汇总，不直接修改历史记录

- **`activity_log`**  
  - 所有变更动作的审计日志：  
    - actor 类型（user/agent/system）、动作名称、实体类型与 ID、详情等  
  - 用于：
    - 供 UI 展示全局/按实体过滤的活动流  
    - 供 Live events 推送，实现实时反馈

---

## 5. Agent 心跳与任务执行模型

V1 的一大核心是将「任务分配 → agent 执行 → 成本上报」打通成一个闭环，这一部分由 `heartbeat` 系统承载。

### 5.1 关键表与实体

- **`heartbeat_runs`**  
  - 记录每次 agent 心跳执行（queued/running/succeeded/failed/cancelled/timed_out）  
  - 包含启动与结束时间、错误信息、外部运行 ID、上下文快照等

- **`heartbeat_run_events`**（实现中用于保存运行期日志/事件）  
  - 保存运行中的分步输出，用于在 UI 中实时查看日志和进度

- **`agent_task_sessions`**  
  - 按 `(companyId, agentId, adapterType, taskKey)` 唯一索引  
  - 用于把「某个 agent 在某个任务上的会话」聚合起来（例如针对某 issue 的连续心跳）

- **`agent_wakeup_requests`**  
  - 保存待处理的唤醒请求队列：  
    - 来源（scheduler/manual/callback）、原因（issue_assigned/comment_mentioned 等）  
    - 目标 agent、可选关联 issue/project/goal  
    - 状态（queued/running/completed/failed/deferred_issue_execution 等）

### 5.2 执行流程概览

1. **触发唤醒（Wakeup）**  
   - 由以下事件触发：  
     - 新 issue 分配给某 agent  
     - issue 状态变化（如 checkout/in_progress）  
     - 评论中 @ 某个 agent  
     - 定时器（heartbeat interval）  
   - `heartbeatService.enqueueWakeup` 将请求写入 `agent_wakeup_requests` 队列  

2. **调度与运行（Run Execution）**  
   - server 内部的 scheduler 周期性扫描可执行的 wakeup 请求：  
     - 检查 agent 的 `maxConcurrentRuns` 限制  
     - 检查 issue 是否已经被其他 run 锁定执行（`executionRunId`）  
   - 成功 claim 后创建 `heartbeat_runs` 记录并将状态设为 `running`  
   - 读取 agent 的 `adapterType`/`adapterConfig` 并解析 secrets/workspace  
   - 调用对应适配器执行：  
     - process：启动本地进程，收集 stdout/stderr、用量信息  
     - http：发送 webhook/API 请求并等待响应  

3. **任务 checkout 与执行锁（Execution Lock）**  
   - 为了保证「**单一执行者**」和「**原子 checkout**」，`issueService` 中：  
     - checkout 时通过一条 SQL 更新语句，在 where 子句中同时检查：  
       - issue 当前 status 是否在允许的 expectedStatuses 集合（todo/backlog/blocked 等）  
       - 没有被其他 run 锁定，或已有锁的 run 已经是终止态  
     - 成功时同时写入 `assigneeAgentId`、`checkoutRunId`、`executionRunId`、`status` 等  
   - 对运行中的 agent，在访问或更新 issue 时使用 `assertCheckoutOwner` 校验：  
     - 只有持有 checkout 锁的 run 能执行诸如 release/状态更新等写操作  
     - 如检测到旧 run 已经结束，可安全「接管」执行锁  

4. **完成后处理**  
   - 适配器执行完成后，`heartbeatService`：  
     - 更新 `heartbeat_runs` 状态与结束时间  
     - 更新 `agent_runtime_state` 中的最近状态与使用统计  
     - 写入 `cost_events`，并累加 `agents.spentMonthlyCents` 与 `companies.spentMonthlyCents`  
     - 检查预算是否超限，如超限则自动将 agent 标为 `paused`  
     - 释放 issue 的 `executionRunId` 锁，并提升可能存在的 `deferred_issue_execution` 请求  
   - 同时通过活动日志与 Live events 将结果推送给 UI

### 5.3 预算与自动暂停

- **Agent 级预算**  
  - 当某次 `cost_events` 写入后发现 `spentMonthlyCents >= budgetMonthlyCents`，且 agent 未被暂停/终止时：  
    - 将 agent 状态切换为 `paused`，阻止后续心跳执行  

- **公司级预算**  
  - 在 V1 设计中，主要通过成本汇总接口和 UI 提示来辅助决策  
  - 未来可按 `SPEC-implementation` 中描述进一步强化公司级硬停策略

---

## 6. 权限、鉴权与多租户隔离

### 6.1 部署模式与鉴权

根据 `doc/DEVELOPING.md` 和 `doc/DEPLOYMENT-MODES.md`，系统支持两种主要模式：

- **`local_trusted`（默认开发模式）**  
  - 单机、本地使用，无人类账号系统  
  - 所有请求都被视为「本地 board operator」  
  - 可以启用公司删除等危险操作以便开发调试  

- **`authenticated`（带登录的模式）**  
  - 使用 `better-auth` 提供 session-based auth  
  - 支持 instance 级 admin（`instance_user_roles`）  
  - 用户通过邀请/加入请求成为某个公司的成员（`company_memberships`）  
  - 可以选择 `private` 或 `public` exposure，配合域名/Host 白名单生效  

### 6.2 Actor 模型与中间件

- **`actorMiddleware`**  
  - 解析请求头和 Cookie，并构造统一的 actor 对象：  
    - **board actor**：人类 operator  
      - 来源：local_trusted 模式的隐式 board，或 authenticated 模式下的 auth session  
      - 包含可访问的 `companyIds`、是否为 instance admin 等  
    - **agent actor**：AI 代理  
      - 来源：Bearer API key（`agent_api_keys`）或本地 agent JWT  
      - 绑定 `agentId` 与唯一 `companyId`  
  - 将 actor 挂载到 `req.actor` 供后续路由与服务使用

- **`boardMutationGuard`**  
  - 针对所有 `/api` 的非 GET/HEAD 请求：  
    - 要求 actor 为 board  
    - 检查 Origin/Referer 是否在允许的 host 白名单中，防止 CSRF/跨站滥用  
    - 对 `local_implicit` board 略微放宽，方便本地调试

### 6.3 访问控制与权限粒度

- **`company_memberships`、`principal_permission_grants`、`instance_user_roles`** 等表  
  - 提供「用户/agent 加入公司」与「按公司粒度授权权限」的机制  
  - 权限 key 在 `@paperclipai/shared` 的常量中定义（例如 `agents:create`、`tasks:assign` 等）  

- **`accessService`**  
  - 提供权限判定和授予的统一 API：  
    - `canUser` / `hasPermission` / `ensureMembership` / `setPrincipalGrants` 等  
  - 多个路由（如创建 agent、分配任务、邀请/加入、管理权限等）都依赖此服务进行强制校验

- **跨层公司隔离**  
  - **数据层**：  
    - 绝大部分表有 `companyId` 外键约束  
  - **服务层**：  
    - 所有列表/CRUD 操作以 `companyId` 作为首要过滤条件  
    - 对跨公司引用（如跨公司关联 agent/issue/project）会抛出错误  
  - **路由层**：  
    - 通过 `assertCompanyAccess` 保证请求 actor 对该 `companyId` 有权访问  

---

## 7. 前端（UI）架构与实时体验

### 7.1 路由与公司前缀

- **路由包装（`ui/lib/router.tsx`）**  
  - 在 `react-router-dom` 之上增加一层封装，使 `Link` / `NavLink` / `useNavigate` 等自动带上当前公司前缀  
  - 前缀形如 `/:companyPrefix/...`，通常来自 `company.slug` 或类似字段  

- **主路由树（`ui/App.tsx`）**  
  - 无前缀路由：  
    - `/auth`：登录/认证  
    - `/board-claim/:token`：board 所有权认领  
    - `/invite/:token`：加入邀请页  
  - 受 `CloudAccessGate` 保护的公司前缀路由：  
    - `/:companyPrefix/dashboard`：公司总览  
    - `/:companyPrefix/agents`：agent 列表/详情/运行记录  
    - `/:companyPrefix/projects` / `.../issues` / `.../goals` / `.../approvals` / `.../costs` / `.../activity` 等  
    - `/:companyPrefix/design-guide`：设计系统展示页  
  - 初始访问根 `/` 时，会通过公司上下文自动选择或引导用户创建首个公司

### 7.2 状态管理与 Live Updates

- **React Query**  
  - 统一封装 API client（`ui/src/api/client.ts`），所有请求走 `/api` 前缀  
  - 将不同实体（companies/agents/issues/projects/goals/approvals/costs/activity/runs 等）抽象为独立的 query keys  
  - 与 Live events 配合精细 invalidation

- **上下文与布局**  
  - `CompanyContext`：当前公司列表与选中公司  
  - `LiveUpdatesProvider`：  
    - 建立与 `/api/.../events/ws` 的 WebSocket 连接  
    - 监听 Live events（issue.created/updated/comment_added、agent.status_changed、heartbeat.run_completed 等）  
    - 根据事件类型：  
      - invalidation 对应的 query keys  
      - 通过 `ToastContext` 展示通知并支持点击跳转  
  - 其他 Context：Theme、Sidebar、Panels、Dialogs、Breadcrumb、Toasts 等，用于统一管理 UI 状态

### 7.3 设计系统与组件复用

- **基础组件**  
  - `components/ui` 下封装常用 UI：Button、Input、Badge、Dialog、Tabs、Sheet 等  
  - Tailwind 风格的现代化 UI，贴合控制平面产品的密集信息展示需求  

- **领域组件**  
  - Issue/Project/Goal 属性编辑组件  
  - Run/Agent 状态卡片与进度展示组件  
  - Onboarding 向导组件（首次创建公司与初始化 agent）  

这些设计使得在扩展 UI 功能时，可以重用统一的布局、路由、状态管理和视觉语言，保持整体体验一致。

---

## 8. 开发与部署流程

### 8.1 本地开发

- **快速启动**  
  - 在项目根目录执行：  
    - `pnpm install`  
    - `pnpm dev`  
  - 行为：  
    - 使用内嵌 PostgreSQL（若未设置 `DATABASE_URL`）  
    - 自动运行迁移  
    - 启动 API server（`http://localhost:3100`）  
    - 在同一 origin 下通过 Vite dev middleware 提供 `ui`  

- **一键运行（CLI）**  
  - 使用 `pnpm paperclipai run`：  
    - 自动 Onboard（生成基础配置）  
    - 运行 `paperclipai doctor` 做环境检查和修复  
    - 启动 server  

### 8.2 Docker 与外部数据库

- **Docker 快速启动**  
  - 通过 `docker-compose.quickstart.yml` 或 `Dockerfile` 构建镜像并运行  
  - 默认仍使用内嵌 PostgreSQL，将 `/paperclip` 目录挂载到宿主机以保留状态  

- **外部 Postgres（包括 Supabase）**  
  - 配置 `DATABASE_URL` 即可切换到外部数据库  
  - 使用 `drizzle-kit push` 或仓库提供的 DB 脚本完成 schema 部署  
  - `packages/db/src/client.ts` 中可按需要禁用 prepared statements 以兼容某些托管服务

### 8.3 数据库迁移与变更流程

- 修改数据模型时需遵循 `AGENTS.md` 中的工作流：  
  1. 修改 `packages/db/src/schema/*.ts`  
  2. 确保新表在 `schema/index.ts` 中导出  
  3. 运行 `pnpm db:generate` 生成迁移  
  4. 运行 `pnpm -r typecheck` 确认类型检查通过  
- 变更 schema 后，需保持：  
  - `packages/shared` 中的类型/常量/validators 同步更新  
  - `server` 与 `ui` 中的 API 接口逻辑与 UI 展示逻辑保持一致  

---

## 9. 关键设计原则与扩展建议

### 9.1 关键不变量总结

- **公司级作用域**  
  - 所有业务实体必须带 `companyId`，所有业务逻辑和路由都需显式校验公司边界  

- **单一受让人 & 原子 checkout**  
  - Issue 同一时刻只能有一个 assignee（agent 或 user）  
  - `checkout` 和执行锁通过数据库原子更新保证不会出现两个并发 run 抢占同一任务  

- **审批栅栏（governance）**  
  - 某些高风险操作（例如创建 agent）可以受公司设置控制，强制经由 `approvals` 流程  
  - 所有审批决策都必须通过 `activity_log` 记录  

- **预算硬阈值与自动暂停**  
  - 每个 agent、公司都可设置预算，超限后自动暂停 agent 执行  
  - 成本只通过 `cost_events` 聚合而非直接改写  

- **全面审计与实时可见性**  
  - 所有变更都写入 `activity_log`，并通过 Live Events 推到 UI  
  - 使操作员始终能回答：「谁在做什么？花了多少钱？进展如何？」

### 9.2 扩展功能时的建议

- **遵守「contracts 同步」原则**  
  - 若改动领域模型或 API：  
    - 先更新 `packages/db`（schema + 迁移）  
    - 然后更新 `packages/shared`（类型、常量、validators）  
    - 再更新 `server` 服务与路由  
    - 最后更新 `ui` 的 API 调用与界面  

- **保持公司作用域与权限一致性**  
  - 新增任何路由和服务时：  
    - 必须接受 `companyId` 并调用 `assertCompanyAccess`  
    - 明确区分 board actor 与 agent actor 能做的事情（参考 `SPEC-implementation` 的权限矩阵）  

- **利用现有基础设施**  
  - 若要增加新的工作流/事件流：  
    - 充分利用 `activity_log` 和 Live Events 来做审计与实时 UI 刷新  
  - 若要增加新的成本维度/统计视图：  
    - 在 `cost_events` 之上做新的聚合，避免写入派生数据  
  - 若要扩展 agent 适配器类型：  
    - 遵循现有 `adapterType + adapterConfig` 模式，将运行时细节封装在适配器内部

---

## 10. 总结

Paperclip 通过「**server + UI + DB + shared 契约**」的四层结构，实现了一个围绕公司为边界、强调治理与可观测性的 AI 控制平面。  

理解本篇所描述的：  
- 运行时组件划分  
- 数据模型与关键不变量  
- Agent 心跳与任务执行机制  
- 权限/审批/预算治理模式  
- UI 与 Live updates 的交互模式  

将有助于你在不破坏核心设计前提下安全地扩展功能、接入新的 agent 适配器，或为特定业务场景定制控制面板能力。

