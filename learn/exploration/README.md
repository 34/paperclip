# 功能探索与能力验证

针对 Paperclip 各功能模块的动手探索记录。每个文档聚焦一个具体功能点，回答"这个功能能做什么、怎么用、边界在哪"。

## 建议的探索方向

### 基础功能
- Agent 创建与配置：不同适配器的差异、配置项含义
- Issue 全生命周期：创建 → 指派 → 执行 → 审查 → 完成
- 组织架构搭建：reportsTo 树、角色系统、权限

### 执行与调度
- 心跳机制：触发方式、会话保持、上下文构建
- Routine 系统：Cron 配置、Webhook 触发、并发策略对比
- 执行工作区：git worktree vs project_primary、隔离策略

### 管控与治理
- 预算控制：三级预算的实际效果、hard stop 体验
- 审批流程：触发条件、Board 操作、Agent 请求
- 执行策略：签收阶段配置、normal vs auto 模式

### 集成与扩展
- MCP 服务器：Agent 通过 MCP 自主操作
- 插件安装与使用：示例插件体验、自定义插件开发
- 外部适配器：不修改源码扩展新 Agent 类型

### 数据与运维
- 公司导入导出：实际操作流程、数据完整性
- 成本追踪：计费类型、报表维度
- 备份恢复：流程验证

## 文档命名建议

```
exp-XXX-功能名称.md

例如:
exp-001-agent-adapter-config.md
exp-002-heartbeat-trigger-modes.md
exp-003-routine-cron-and-webhook.md
exp-004-budget-three-tier-control.md
```
