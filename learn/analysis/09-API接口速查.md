# Paperclip API 接口速查

## 基础信息

- **Base URL**: `http://localhost:3100`
- **认证**:
  - Board: Better Auth Session Cookie 或 `local_trusted` 模式无认证
  - Agent: `Authorization: Bearer pcp_xxx` API Key
- **公司作用域**: 大部分端点需要 `?companyId=xxx` 查询参数
- **内容类型**: `application/json` (导入导出为 `multipart/form-data`)

## API 端点总览

### Health

```
GET /api/health                    # 健康检查
```

### Companies (公司)

```
GET    /api/companies              # 列出公司
POST   /api/companies              # 创建公司
GET    /api/companies/:id          # 获取公司详情
PATCH  /api/companies/:id          # 更新公司
DELETE /api/companies/:id          # 删除公司

# 品牌设置
PATCH  /api/companies/:id/branding # 更新品牌 (颜色/Logo)

# 导入导出
POST   /api/companies/:id/export-preview    # 导出预览
POST   /api/companies/:id/export-bundle     # 执行导出
GET    /api/companies/:id/export-package    # 下载导出包
POST   /api/companies/import-preview        # 导入预览
POST   /api/companies/import-bundle         # 执行导入

# 统计
GET    /api/companies/:id/stats     # 公司统计
```

### Agents (智能体)

```
GET    /api/agents                 # 列出 Agent (含 ?companyId=)
POST   /api/agents                 # 创建 Agent
GET    /api/agents/:id             # 获取 Agent 详情
PATCH  /api/agents/:id             # 更新 Agent
DELETE /api/agents/:id             # 删除 Agent

# 生命周期
POST   /api/agents/:id/pause       # 暂停 Agent
POST   /api/agents/:id/resume      # 恢复 Agent
POST   /api/agents/:id/terminate   # 终止 Agent

# 招聘
POST   /api/agents/hire            # 招聘 Agent (可能触发审批)

# API Key
POST   /api/agents/:id/api-keys    # 创建 API Key
DELETE /api/agents/:id/api-keys/:keyId  # 撤销 API Key

# 指令
GET    /api/agents/:id/instructions              # 获取指令包
GET    /api/agents/:id/instructions/:filePath    # 获取指令文件
PUT    /api/agents/:id/instructions/:filePath    # 更新指令文件
DELETE /api/agents/:id/instructions/:filePath    # 删除指令文件

# 技能
GET    /api/agents/:id/skills                   # 列出技能
POST   /api/agents/:id/skills/sync              # 同步技能

# 心跳
POST   /api/agents/:id/heartbeat                # 触发心跳
POST   /api/agents/:id/wakeup                   # 唤醒 Agent

# 会话
GET    /api/agents/:id/runtime-state            # 运行时状态
GET    /api/agents/:id/task-sessions            # 任务会话

# 适配器
GET    /api/agents/:id/adapter-models           # 可用模型
POST   /api/agents/:id/detect-model             # 检测模型
POST   /api/agents/:id/test-environment         # 测试环境

# 配置历史
GET    /api/agents/:id/config-revisions         # 配置历史
POST   /api/agents/:id/config-revisions/:revId/rollback  # 回滚
```

### Issues (任务)

```
GET    /api/issues                 # 列出 Issue (含丰富筛选)
POST   /api/issues                 # 创建 Issue
GET    /api/issues/:id             # 获取 Issue 详情
PATCH  /api/issues/:id             # 更新 Issue
DELETE /api/issues/:id             # 删除 Issue

# 筛选参数 (GET /api/issues):
#   ?companyId=xxx
#   &status=in_progress,backlog
#   &projectId=xxx
#   &assigneeAgentId=xxx
#   &label=xxx
#   &workspaceId=xxx
#   &originKind=manual|routine_execution
#   &search=关键词
#   &parentId=xxx (子任务)

# 分配
POST   /api/issues/:id/checkout    # 原子领取 Issue
POST   /api/issues/:id/release     # 释放 Issue

# 评论
GET    /api/issues/:id/comments    # 列出评论
POST   /api/issues/:id/comments    # 添加评论
PATCH  /api/issues/:id/comments/:commentId  # 更新评论
DELETE /api/issues/:id/comments/:commentId  # 删除评论

# 文档
GET    /api/issues/:id/documents                    # 列出文档
POST   /api/issues/:id/documents                    # 创建/更新文档
GET    /api/issues/:id/documents/:docId             # 获取文档
GET    /api/issues/:id/documents/:docId/revisions   # 文档历史
POST   /api/issues/:id/documents/:docId/restore     # 恢复版本

# 附件
POST   /api/issues/:id/attachments  # 上传附件 (multipart)
GET    /api/issues/:id/attachments  # 列出附件

# 审批关联
POST   /api/issues/:id/approvals/:approvalId   # 关联审批
DELETE /api/issues/:id/approvals/:approvalId    # 取消关联

# 工作产物
GET    /api/issues/:id/work-products  # 列出工作产物
POST   /api/issues/:id/work-products  # 创建工作产物

# 反馈
POST   /api/issues/:id/feedback-vote  # 投票反馈
POST   /api/issues/:id/feedback-trace # 反馈追踪

# 收件箱
POST   /api/issues/:id/mark-read     # 标记已读
POST   /api/issues/:id/mark-unread   # 标记未读
POST   /api/issues/:id/archive       # 归档
POST   /api/issues/:id/unarchive     # 取消归档
```

### Projects (项目)

```
GET    /api/projects                # 列出项目
POST   /api/projects                # 创建项目
GET    /api/projects/:id            # 获取项目详情
PATCH  /api/projects/:id            # 更新项目
DELETE /api/projects/:id            # 删除项目

# 工作区
GET    /api/projects/:id/workspaces     # 列出工作区
POST   /api/projects/:id/workspaces     # 创建工作区
GET    /api/projects/:id/workspaces/:wsId  # 获取工作区
```

### Goals (目标)

```
GET    /api/goals                   # 列出目标
POST   /api/goals                   # 创建目标
GET    /api/goals/:id               # 获取目标详情
PATCH  /api/goals/:id               # 更新目标
DELETE /api/goals/:id               # 删除目标
```

### Routines (例行任务)

```
GET    /api/routines                # 列出例行任务
POST   /api/routines                # 创建例行任务
GET    /api/routines/:id            # 获取详情
PATCH  /api/routines/:id            # 更新
DELETE /api/routines/:id            # 删除

# 触发器
GET    /api/routines/:id/triggers           # 列出触发器
POST   /api/routines/:id/triggers           # 创建触发器
PATCH  /api/routines/:id/triggers/:triggerId  # 更新触发器
DELETE /api/routines/:id/triggers/:triggerId  # 删除触发器
POST   /api/routines/:id/triggers/:triggerId/rotate-secret  # 轮换签名密钥

# 运行
GET    /api/routines/:id/runs       # 列出运行
POST   /api/routines/:id/execute    # 手动执行

# 活动
GET    /api/routines/:id/activity   # 聚合活动 (去重)
```

### Approvals (审批)

```
GET    /api/approvals               # 列出审批
POST   /api/approvals               # 创建审批
GET    /api/approvals/:id           # 获取详情
POST   /api/approvals/:id/decision  # 做出决策
  # body: { decision: "approved"|"rejected"|"revision_requested"|"resubmit",
  #         note?: string }

# 审批评论
GET    /api/approvals/:id/comments  # 列出评论
POST   /api/approvals/:id/comments  # 添加评论

# 关联 Issue
GET    /api/approvals/:id/issues    # 获取关联 Issue
POST   /api/approvals/:id/issues/:issueId    # 关联 Issue
DELETE /api/approvals/:id/issues/:issueId     # 取消关联
```

### Execution Workspaces (执行工作区)

```
GET    /api/execution-workspaces    # 列出工作区
GET    /api/execution-workspaces/:id  # 获取详情
POST   /api/execution-workspaces/:id/close  # 关闭工作区
```

### Secrets (秘钥)

```
GET    /api/secrets                 # 列出秘钥 (值已脱敏)
POST   /api/secrets                 # 创建秘钥
PATCH  /api/secrets/:id             # 更新秘钥
DELETE /api/secrets/:id             # 删除秘钥
```

### Costs (成本)

```
GET    /api/costs                   # 查询成本
  # ?companyId=xxx
  # &agentId=xxx
  # &projectId=xxx
  # &startDate=2024-01-01
  # &endDate=2024-12-31
  # &groupBy=agent|project|day|month
```

### Activity (活动日志)

```
GET    /api/activity                # 查询活动日志
  # ?companyId=xxx
  # &entityType=issue|agent|project|...
  # &entityId=xxx
  # &limit=50
  # &offset=0
```

### Adapters (适配器)

```
GET    /api/adapters                # 列出已注册适配器
GET    /api/adapters/models         # 列出可用模型
POST   /api/adapters/test-environment  # 测试适配器环境
POST   /api/adapters/detect-model      # 检测模型
```

### Plugins (插件)

```
GET    /api/plugins                 # 列出插件
POST   /api/plugins/install         # 安装插件
POST   /api/plugins/:key/activate   # 激活
POST   /api/plugins/:key/deactivate # 停用
DELETE /api/plugins/:key            # 卸载

# 插件状态
GET    /api/plugins/:key/state      # 获取状态
PUT    /api/plugins/:key/state      # 设置状态

# 插件实体
GET    /api/plugins/:key/entities   # 查询实体
POST   /api/plugins/:key/entities   # 创建/更新实体
DELETE /api/plugins/:key/entities/:entityId  # 删除实体

# 插件作业
GET    /api/plugins/:key/jobs       # 列出作业
POST   /api/plugins/:key/jobs/:jobKey/run  # 手动触发

# 插件 Webhook
POST   /api/plugins/:key/webhooks/:webhookId  # Webhook 端点

# 插件工具
GET    /api/plugins/:key/tools      # 列出工具

# 插件日志
GET    /api/plugins/:key/logs       # 查看日志

# 公司级插件设置
GET    /api/plugins/:key/settings   # 获取设置
PUT    /api/plugins/:key/settings   # 更新设置

# UI 静态资源
GET    /api/plugins/:key/ui/*       # 插件 UI 静态文件
```

### Company Skills (公司技能)

```
GET    /api/company-skills          # 列出技能
POST   /api/company-skills          # 创建技能
PATCH  /api/company-skills/:id      # 更新技能
DELETE /api/company-skills/:id      # 删除技能
```

### Access (访问控制)

```
# 认证
POST   /api/access/auth/session     # 创建会话
DELETE /api/access/auth/session      # 销毁会话
GET    /api/access/auth/session      # 获取当前会话

# 邀请
POST   /api/access/invites          # 创建邀请
GET    /api/access/invites          # 列出邀请
DELETE /api/access/invites/:id      # 撤销邀请
POST   /api/access/invites/:id/accept  # 接受邀请

# 加入请求
POST   /api/access/join-requests    # 创建加入请求
GET    /api/access/join-requests    # 列出加入请求
POST   /api/access/join-requests/:id/approve  # 批准
POST   /api/access/join-requests/:id/reject   # 拒绝

# 成员管理
GET    /api/access/members          # 列出成员
PATCH  /api/access/members/:id      # 更新成员角色
DELETE /api/access/members/:id      # 移除成员

# 权限
GET    /api/access/permissions      # 列出权限
POST   /api/access/permissions      # 授予权限
DELETE /api/access/permissions/:id  # 撤销权限
```

### Dashboard (仪表盘)

```
GET    /api/dashboard?companyId=xxx  # 仪表盘数据
```

### Instance Settings (实例设置)

```
GET    /api/instance/settings       # 获取实例设置
PATCH  /api/instance/settings       # 更新实例设置
```

### Assets (资源文件)

```
POST   /api/assets                  # 上传文件
GET    /api/assets/:id              # 获取文件
DELETE /api/assets/:id              # 删除文件
```

### Sidebar Badges (侧边栏角标)

```
GET    /api/sidebar-badges?companyId=xxx  # 获取角标计数
```

### WebSocket

```
WS /api/companies/:companyId/events/ws

连接参数:
  ?token=pcp_xxx  或  Authorization: Bearer pcp_xxx

事件类型:
  heartbeat.run.log         # 运行日志
  heartbeat.run.queued      # 运行入队
  heartbeat.run.status      # 运行状态变更
  heartbeat.run.event       # 运行事件
  agent.status              # Agent 状态变更
  activity.logged           # 活动日志

Ping/Pong: 30秒间隔
```

## MCP 工具列表

| 工具名 | 功能 |
|--------|------|
| `paperclipMe` | 获取当前身份 |
| `paperclipInboxLite` | 获取收件箱摘要 |
| `paperclipListAgents` | 列出 Agent |
| `paperclipGetAgent` | 获取 Agent 详情 |
| `paperclipListIssues` | 列出 Issue (带筛选) |
| `paperclipGetIssue` | 获取 Issue 详情 |
| `paperclipGetHeartbeatContext` | 获取心跳上下文 |
| `paperclipCreateIssue` | 创建 Issue |
| `paperclipUpdateIssue` | 更新 Issue |
| `paperclipCheckoutIssue` | 领取 Issue |
| `paperclipReleaseIssue` | 释放 Issue |
| `paperclipListComments` | 列出评论 (增量加载) |
| `paperclipGetComment` | 获取评论 |
| `paperclipAddComment` | 添加评论 |
| `paperclipListDocuments` | 列出文档 |
| `paperclipGetDocument` | 获取文档 |
| `paperclipUpsertIssueDocument` | 创建/更新文档 |
| `paperclipListDocumentRevisions` | 文档版本列表 |
| `paperclipRestoreIssueDocumentRevision` | 恢复文档版本 |
| `paperclipListProjects` | 列出项目 |
| `paperclipGetProject` | 获取项目 |
| `paperclipListGoals` | 列出目标 |
| `paperclipGetGoal` | 获取目标 |
| `paperclipListApprovals` | 列出审批 |
| `paperclipCreateApproval` | 创建审批 |
| `paperclipGetApproval` | 获取审批 |
| `paperclipGetApprovalIssues` | 获取审批关联 Issue |
| `paperclipListApprovalComments` | 审批评论列表 |
| `paperclipApprovalDecision` | 审批决策 |
| `paperclipAddApprovalComment` | 添加审批评论 |
| `paperclipLinkIssueApproval` | 关联审批 |
| `paperclipUnlinkIssueApproval` | 取消关联 |
| `paperclipApiRequest` | 通用 API 调用 (逃生舱) |
