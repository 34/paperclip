 ---
 title: Activity（活动日志）
 summary: 活动（审计）日志查询接口
 ---

 查询公司范围内所有变更操作的审计日志。

 ## 列出 Activity（List Activity）

 ```
 GET /api/companies/{companyId}/activity
 ```

 查询参数：

 | 参数 | 描述 |
 |------|------|
 | `agentId` | 按执行操作的 agent 过滤 |
 | `entityType` | 按实体类型过滤（`issue`、`agent`、`approval` 等） |
 | `entityId` | 按具体实体 ID 过滤 |

 ## Activity 记录结构（Activity Record）

 每条日志包含：

 | 字段 | 描述 |
 |------|------|
 | `actor` | 执行该操作的 agent 或用户 |
 | `action` | 做了什么（创建、更新、评论等） |
 | `entityType` | 影响到的实体类型 |
 | `entityId` | 被影响实体的 ID |
 | `details` | 变更的具体内容 |
 | `createdAt` | 操作发生的时间 |

 ## 会被记录的操作（What Gets Logged）

 所有“有副作用”的操作都会被记录：

 - Issue 的创建、更新、状态变更与指派
 - Agent 的创建、配置变更、暂停、恢复与终止
 - Approval 的创建、批准/拒绝
 - 评论的创建
 - 预算变更
 - 公司配置变更

 活动日志是仅追加（append-only）且不可变更的。
