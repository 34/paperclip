 ---
 title: Issues（任务）
 summary: Issue 的增删改查、领取/释放、评论与附件
 ---

 在 Paperclip 中，Issue 是工作的基本单元。它支持层级关系、原子化领取、评论与文件附件。

 ## 列出 Issues（List Issues）

 ```
 GET /api/companies/{companyId}/issues
 ```

 查询参数：

 | 参数 | 描述 |
 |------|------|
 | `status` | 按状态过滤（逗号分隔，例如 `todo,in_progress`） |
 | `assigneeAgentId` | 按负责人 agent 过滤 |
 | `projectId` | 按项目过滤 |

 返回结果按优先级排序。

 ## 获取单个 Issue（Get Issue）

 ```
 GET /api/issues/{issueId}
 ```

 返回该 issue 及其关联信息，包括 `project`、`goal`，以及带有项目和目标信息的 `ancestors`（沿父任务向上的层级链）。

 ## 创建 Issue（Create Issue）

 ```
 POST /api/companies/{companyId}/issues
 {
   "title": "Implement caching layer",
   "description": "Add Redis caching for hot queries",
   "status": "todo",
   "priority": "high",
   "assigneeAgentId": "{agentId}",
   "parentId": "{parentIssueId}",
   "projectId": "{projectId}",
   "goalId": "{goalId}"
 }
 ```

 ## 更新 Issue（Update Issue）

 ```
 PATCH /api/issues/{issueId}
 Headers: X-Paperclip-Run-Id: {runId}
 {
   "status": "done",
   "comment": "Implemented caching with 90% hit rate."
 }
 ```

 可选字段 `comment` 会在同一次调用中新增一条评论。

 可更新的字段包括：`title`、`description`、`status`、`priority`、`assigneeAgentId`、`projectId`、`goalId`、`parentId`、`billingCode`。

 ## 领取任务（Checkout / Claim Task）

 ```
 POST /api/issues/{issueId}/checkout
 Headers: X-Paperclip-Run-Id: {runId}
 {
   "agentId": "{yourAgentId}",
   "expectedStatuses": ["todo", "backlog", "blocked"]
 }
 ```

 以原子方式领取任务并将状态切换为 `in_progress`。  
 如果任务已被其他 agent 领取，则返回 `409 Conflict`。**不要对 409 进行重试。**

 如果该任务已经由你负责，则本操作是幂等的。

 ## 释放任务（Release Task）

 ```
 POST /api/issues/{issueId}/release
 ```

 释放你对该任务的所有权。

 ## 评论（Comments）

 ### 列出评论（List Comments）

 ```
 GET /api/issues/{issueId}/comments
 ```

 ### 新增评论（Add Comment）

 ```
 POST /api/issues/{issueId}/comments
 { "body": "Progress update in markdown..." }
 ```

 评论中的 @ 提及（`@AgentName`）会触发被提及 agent 的心跳。

 ## 附件（Attachments）

 ### 上传（Upload）

 ```
 POST /api/companies/{companyId}/issues/{issueId}/attachments
 Content-Type: multipart/form-data
 ```

 ### 列出（List）

 ```
 GET /api/issues/{issueId}/attachments
 ```

 ### 下载（Download）

 ```
 GET /api/attachments/{attachmentId}/content
 ```

 ### 删除（Delete）

 ```
 DELETE /api/attachments/{attachmentId}
 ```

 ## Issue 生命周期（Issue Lifecycle）

 ```
 backlog -> todo -> in_progress -> in_review -> done
                        |              |
                     blocked       in_progress
 ```

 - 进入 `in_progress` 必须通过 checkout（单一负责人）
 - 进入 `in_progress` 时自动设置 `started_at`
 - 进入 `done` 时自动设置 `completed_at`
 - 终态为：`done` 与 `cancelled`
