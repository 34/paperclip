 ---
 title: 架构（Architecture）
 summary: 技术栈总览、请求流与适配器模型
 ---

 Paperclip 是一个单仓库（monorepo），主要由四层组成。

 ## 技术栈总览（Stack Overview）

 ```
 ┌─────────────────────────────────────┐
 │  React UI (Vite)                    │
 │  仪表盘、组织管理、任务管理         │
 ├─────────────────────────────────────┤
 │  Express.js REST API (Node.js)      │
 │  路由、服务、认证、适配器           │
 ├─────────────────────────────────────┤
 │  PostgreSQL (Drizzle ORM)           │
 │  表结构、迁移、嵌入式模式           │
 ├─────────────────────────────────────┤
 │  适配器（Adapters）                 │
 │  Claude Local, Codex Local,         │
 │  Process, HTTP                      │
 └─────────────────────────────────────┘
 ```

 ## 技术栈（Technology Stack）

 | 层级 | 技术 |
 |------|------|
 | 前端 | React 19、Vite 6、React Router 7、Radix UI、Tailwind CSS 4、TanStack Query |
 | 后端 | Node.js 20+、Express.js 5、TypeScript |
 | 数据库 | PostgreSQL 17（或嵌入式 PGlite）、Drizzle ORM |
 | 认证 | Better Auth（会话 + API Keys） |
 | 适配器 | Claude Code CLI、Codex CLI、本地进程（shell process）、HTTP webhook |
 | 包管理器 | pnpm 9（含 workspaces） |

 ## 仓库结构（Repository Structure）

 ```
 paperclip/
 ├── ui/                          # React 前端
 │   ├── src/pages/              # 路由页面
 │   ├── src/components/         # React 组件
 │   ├── src/api/                # API 客户端
 │   └── src/context/            # React 上下文提供者
 │
 ├── server/                      # Express.js API
 │   ├── src/routes/             # REST 接口
 │   ├── src/services/           # 业务逻辑
 │   ├── src/adapters/           # Agent 执行适配器
 │   └── src/middleware/         # 认证、日志
 │
 ├── packages/
 │   ├── db/                      # Drizzle 表结构与迁移
 │   ├── shared/                  # API 类型、常量、校验器
 │   ├── adapter-utils/           # 适配器接口与工具
 │   └── adapters/
 │       ├── claude-local/        # Claude Code 适配器
 │       └── codex-local/         # OpenAI Codex 适配器
 │
 ├── skills/                      # Agent 技能
 │   └── paperclip/               # 核心 Paperclip 技能（心跳协议）
 │
 ├── cli/                         # CLI 客户端
 │   └── src/                     # 安装与控制平面命令
 │
 └── doc/                         # 内部文档
 ```

 ## 请求流（Request Flow）

 当一次心跳（heartbeat）被触发时：

 1. **触发** —— 调度器、手动调用，或事件（任务分配、@ 提及）触发一次心跳
 2. **适配器调用** —— 服务端调用已配置适配器的 `execute()` 函数
 3. **Agent 进程** —— 适配器以 Paperclip 环境变量和提示词（prompt）启动 agent（例如 Claude Code CLI）
 4. **Agent 工作** —— Agent 通过 Paperclip 的 REST API 检查分配、领取任务、执行工作并更新状态
 5. **结果采集** —— 适配器捕获 stdout，解析用量/成本数据，并提取会话状态
 6. **运行记录** —— 服务端记录本次运行的结果、成本以及会话状态，用于下一次心跳

 ## 适配器模型（Adapter Model）

 适配器是 Paperclip 与 agent 运行时之间的桥梁。每个适配器都是一个包含三个模块的包：

 - **服务端模块** —— 包含用于启动/调用 agent 的 `execute()` 函数，以及环境诊断逻辑
 - **UI 模块** —— 用于运行视图（run viewer）的 stdout 解析器，以及用于创建 agent 的配置表单字段
 - **CLI 模块** —— 为 `paperclipai run --watch` 提供终端输出格式化

 内置适配器包括：`claude_local`、`codex_local`、`process`、`http`。你可以为任意运行时编写自定义适配器，只要它能调用 HTTP API。

 ## 关键设计决策（Key Design Decisions）

 - **控制平面而非执行平面** —— Paperclip 编排 agents，但不直接运行它们
 - **公司级隔离（company-scoped）** —— 所有实体都只属于唯一一家公司，数据边界严格隔离
 - **单一负责人任务模型** —— 原子化的任务领取（checkout）防止同一任务被并发处理
 - **适配器无关性** —— 任何可以调用 HTTP API 的运行时都可以作为 agent
 - **默认嵌入式模式** —— 提供零配置本地模式，内嵌 PostgreSQL
