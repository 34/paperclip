# Agent 配置与活动 UI（Agent Configuration & Activity UI）

## 背景（Context）

Agents 是 Paperclip 公司中的员工。每个 agent 有适配器类型（`claude_local`、`codex_local`、`process`、`http`）决定其运行方式、在组织图中的位置（汇报对象）、心跳策略（如何/何时唤醒）以及预算。`/agents` 的 UI 需要支持创建与配置 agents、查看组织层级、以及查看其行为——运行历史、实时日志与累计成本。

本 spec 覆盖三个界面：

1. **Agent 创建对话框** — “New Agent” 流程
2. **Agent 详情页** — 配置、活动与日志
3. **Agents 列表页** — 对现有列表的改进

---

## 1. Agent 创建对话框（Agent Creation Dialog）

沿用现有 `NewIssueDialog` / `NewProjectDialog` 模式：带展开/收起、公司徽章面包屑和 Cmd+Enter 提交的 `Dialog` 组件。

### 字段（Fields）

**身份（始终可见）：**

| 字段 | 控件 | 必填 | 默认 | 说明 |
|------|------|------|------|------|
| Name | 文本输入（大号、自动聚焦） | 是 | -- | 如 "Alice"、"Build Bot" |
| Title | 文本输入（副标题样式） | 否 | -- | 如 "VP of Engineering" |
| Role | Chip 弹出选择 | 否 | `general` | 来自 `AGENT_ROLES`：ceo、cto、cmo、cfo、engineer、designer、pm、qa、devops、researcher、general |
| Reports To | Chip 弹出（agent 选择） | 否 | -- | 公司内现有 agents 下拉。若为第一个 agent，自动设 role 为 `ceo` 并置灰 Reports To；否则除 role 为 `ceo` 外必填。 |
| Capabilities | 文本输入 | 否 | -- | 该 agent 能做什么的自由文本描述 |

**适配器（可折叠，默认展开）：**

| 字段 | 控件 | 默认 | 说明 |
|------|------|------|------|
| Adapter Type | Chip 弹出选择 | `claude_local` | `claude_local`、`codex_local`、`process`、`http` |
| Test environment | 按钮 | -- | 运行适配器诊断，对当前未保存配置返回 pass/warn/fail |
| CWD | 文本输入 | -- | 本地适配器工作目录 |
| Prompt Template | 多行文本 | -- | 支持 `{{ agent.id }}`、`{{ agent.name }}` 等 |
| Model | 文本输入 | -- | 可选模型覆盖 |

**按适配器类型显示的字段**（略，见英文原文表格）

**运行时 / 心跳策略**（可折叠，默认收起）：Context Mode、Monthly Budget、Timeout、Grace Period、Extra Args、Env Vars、Enabled、Interval、Wake on Assignment/On-Demand/Automation、Cooldown 等。

### 行为（Behavior）

- 提交时调用 `agentsApi.create(companyId, data)`，`data` 将身份字段放顶层，适配器相关字段放入 `adapterConfig`，心跳/运行时放入 `runtimeConfig`。
- 创建后跳转到新 agent 的详情页。
- 若公司尚无 agent，预填 role 为 `ceo` 并禁用 Reports To。
- 切换适配器类型时，适配器配置区更新可见字段，保留共享字段（cwd、promptTemplate 等）的值。

---

## 2. Agent 详情页（Agent Detail Page）

重组现有标签布局。保留头部（名称、角色、标题、状态徽章、操作按钮），并丰富标签内容。

### 头部（Header）

```
[StatusBadge]  Agent Name                    [Invoke] [Pause/Resume] [...]
               Role / Title
```

`[...]` 溢出菜单包含：Terminate、Reset Session、Create API Key。

### 标签（Tabs）

#### Overview 标签

两列布局：左列摘要卡片，右列组织位置。

**摘要卡片**：适配器类型与模型、心跳间隔（如「每 5 分钟」）或「已禁用」、最近心跳时间、会话状态、当月支出/预算与进度条。

**组织位置卡片**：Reports to（可点击跳转）、Direct reports 列表。

#### Configuration 标签

与创建对话框相同的区块（Adapter、Runtime、Heartbeat Policy），预填当前值。采用行内编辑——点击值编辑，Enter 或失焦时通过 `agentsApi.update()` 保存。

#### Runs 标签

心跳运行历史列表，分页，按时间倒序。每行：状态图标、run ID 缩写、来源（timer/assignment/on_demand/automation）、相对时间、token 汇总、成本、结果摘要。点击运行可展开详情或侧滑面板：完整状态时间线、会话前后、token 与成本拆分、错误信息、退出码等。**日志查看器**：流式展示 `heartbeat_run_events`，支持自动滚动与「View full log」。

#### Issues 标签

保持现状：分配给该 agent 的 issues 列表，可点击进入 issue 详情。

#### Costs 标签

扩展：累计总量、月度预算进度条、按运行的成本表格、可选简单柱状图（近 30 天每日支出）。

### 属性面板（Properties Panel，右侧边栏）

保留 `AgentProperties`，增加：Session ID（截断+复制）、Last error（若有，红色）、「View Configuration」链接。

---

## 3. Agents 列表页（Agents List Page）

### 当前状态

扁平列表，显示状态徽章、名称、角色、标题、预算条。

### 改进（Improvements）

- **「New Agent」按钮**（Plus 图标 + 「New Agent」），打开创建对话框。
- **视图切换**：List 与 Org Chart。
- **Org Chart 视图**：树形汇报层级，节点显示名称、角色、状态；CEO 在顶，使用 `agentsApi.org(companyId)`；点击节点进入详情。
- **List 视图**：每行增加适配器类型 chip、最近活跃时间、若有运行中心跳则显示动效点。
- **筛选**：Tab 筛选 All / Active / Paused / Error。

---

## 4. 组件清单（Component Inventory）

需要的新组件：`NewAgentDialog`、`AgentConfigForm`、`AdapterConfigFields`、`HeartbeatPolicyFields`、`EnvVarEditor`、`RunListItem`、`RunDetail`、`LogViewer`、`OrgChart`、`AgentSelect`。复用：`StatusBadge`、`EntityRow`、`EmptyState`、`PropertyRow` 及 shadcn 组件。

---

## 5. API 表面（API Surface）

V1 所需端点均已存在，无需新增服务端工作。列表、创建、更新、暂停/恢复/终止、Reset session、Create API key、Get runtime state、Invoke、List runs、Cancel run、Run events、Run log 等见英文原文表格。

---

## 6. 实现顺序（Implementation Order）

1. New Agent Dialog
2. Agents List 改进（New Agent 按钮、Tab 筛选、adapter chip、运行指示）
3. Agent Detail：Configuration 标签
4. Agent Detail：Runs 标签（列表）
5. Agent Detail：Run Detail + Log Viewer
6. Agent Detail：Overview 标签
7. Agent Detail：Costs 标签扩展
8. Org Chart 视图
9. Properties 面板更新

步骤 1–5 为核心，6–9 为打磨。
