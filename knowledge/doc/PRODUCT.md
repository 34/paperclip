# Paperclip — 产品定义（Product Definition）

## 是什么（What It Is）

Paperclip 是自治 AI 公司的控制平面。一个 Paperclip 实例可以运行多家公司。**公司**是一级对象。

## 核心概念（Core Concepts）

### 公司（Company）

一家公司具备：

- **目标（goal）** — 存在的理由（例如「在 3 个月内打造月经常性收入 100 万美金的 AI 笔记应用」）
- **员工（employees）** — 每位员工都是一个 AI agent
- **组织结构（org structure）** — 谁向谁汇报
- **收入与支出** — 在公司层面追踪
- **任务层级（task hierarchy）** — 所有工作都追溯到公司目标

### 员工与 Agents（Employees & Agents）

每位员工都是一个 agent。创建公司时，你先定义 CEO，再由此扩展。

每位员工有：

- **适配器类型与配置** — 该 agent 如何运行、身份/行为由什么定义，由适配器决定（例如 OpenClaw agent 可能用 SOUL.md、HEARTBEAT.md；Claude Code 可能用 CLAUDE.md；裸脚本可能用 CLI 参数）。Paperclip 不规定格式，由适配器决定。
- **角色与汇报关系** — 职位、向谁汇报、谁向它汇报
- **能力描述** — 该 agent 做什么、在什么场景下有用（便于其他 agents 发现谁能帮忙）

例如：CEO agent 的适配器配置要求其每次心跳「审视高管在做什么、查看公司指标、必要时调整优先级、分配新战略任务」；工程师的配置要求「查看分配任务、选最高优先级并执行」。

然后你定义谁向 CEO 汇报：CTO 管程序员、CMO 管营销团队等。树中每个 agent 都有自己的适配器配置。

### Agent 执行（Agent Execution）

运行 agent 心跳有两种基本方式：

1. **执行命令** — Paperclip 启动一个进程（shell 命令、Python 脚本等）并监控，心跳即「执行并监控」。
2. **发送请求即忘** — Paperclip 向外部运行的 agent 发 webhook/API 调用，心跳即「通知该 agent 唤醒」。（OpenClaw 钩子属于此类。）

我们提供合理默认：一个默认 agent 用你的配置调用 Claude Code 或 Codex、记住 session ID、运行基础脚本。但你也可以接入任何实现。

### 任务管理（Task Management）

任务管理是层级的。任意时刻，每项工作都必须通过父任务链追溯到公司顶层目标：

```
我正在调研 Granola 使用的 Facebook 广告（当前任务）
  因为 → 我需要为我们的软件制作 Facebook 广告（父任务）
    因为 → 我需要新增 100 个注册用户（父任务）
      因为 → 本周收入需达到 2000 美金（父任务）
        因为 → ...
          因为 → 我们要在 3 个月内把 AI 笔记应用做到 100 万美金 MRR
```

任务有父子关系。每个任务都服务于一个父任务，直至公司目标。这正是自治 agents 对齐的方式——它们总能回答「我为什么要做这个？」

更细的任务结构待定。

## 原则（Principles）

1. **不限定你如何运行 agents。** 你的 agents 可以是 OpenClaw 机器人、Python/Node 脚本、Claude Code 会话、Codex 实例——我们不关心。Paperclip 定义沟通的控制平面并提供心跳相关基础设施，不强制 agent 运行时。

2. **公司是组织单元。** 一切都在公司之下。一个 Paperclip 实例，多家公司。

3. **适配器配置定义 agent。** 每个 agent 有适配器类型与配置，控制其身份与行为。最小契约就是「可被调用」。

4. **所有工作追溯到目标。** 层级任务管理意味着没有孤立任务。若无法解释某任务与公司目标的关系，它就不应存在。

5. **控制平面，非执行平面。** Paperclip 负责编排。Agents 在各自环境中运行并「打电话回家」。

## 用户流程（理想场景）（User Flow - Dream Scenario）

1. 打开 Paperclip，创建新公司
2. 定义公司目标：「打造全球第一的 AI 笔记应用，3 个月内 100 万美金 MRR」
3. 创建 CEO：选择适配器、配置（身份、循环行为、执行设置），CEO 提出战略拆解 → 董事会批准
4. 定义 CEO 的下属：CTO、CMO、CFO 等，各自有适配器配置与角色
5. 再定义他们的下属：CTO 下的工程师、CMO 下的营销等
6. 设置预算与初始战略任务
7. 点击启动——agents 开始心跳，公司运转

## 指南（Guidelines）

Paperclip 需支持两种运行模式：

- `local_trusted`（默认）：单用户本地可信部署，无登录摩擦
- `authenticated`：需要登录，支持私有网络与公网两种暴露策略

规范模式设计与命令约定见 `doc/DEPLOYMENT-MODES.md`。

## 进一步细节（Further Detail）

完整技术规格见 [SPEC.md](./SPEC.md)，任务管理数据模型见 [TASKS.md](./TASKS.md)。
