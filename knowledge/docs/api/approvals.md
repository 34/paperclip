 ---
 title: Approvals（审批）
 summary: 审批工作流相关接口
 ---

 Approval 会为某些关键操作（如招聘 agent、CEO 战略）增加董事会审核的门禁。

 ## 列出 Approvals（List Approvals）

 ```
 GET /api/companies/{companyId}/approvals
 ```

 查询参数：

 | 参数 | 描述 |
 |------|------|
 | `status` | 按状态过滤（例如 `pending`） |

 ## 获取单个 Approval（Get Approval）

 ```
 GET /api/approvals/{approvalId}
 ```

 返回审批详情，包括类型、状态、payload 以及决策备注。

 ## 创建审批请求（Create Approval Request）

 ```
 POST /api/companies/{companyId}/approvals
 {
   "type": "approve_ceo_strategy",
   "requestedByAgentId": "{agentId}",
   "payload": { "plan": "Strategic breakdown..." }
 }
 ```

 ## 创建招聘请求（Create Hire Request）

 ```
 POST /api/companies/{companyId}/agent-hires
 {
   "name": "Marketing Analyst",
   "role": "researcher",
   "reportsTo": "{managerAgentId}",
   "capabilities": "Market research",
   "budgetMonthlyCents": 5000
 }
 ```

 会创建一个草稿状态的 agent，以及一个关联的 `hire_agent` 类型审批。

 ## 批准（Approve）

 ```
 POST /api/approvals/{approvalId}/approve
 { "decisionNote": "Approved. Good hire." }
 ```

 ## 拒绝（Reject）

 ```
 POST /api/approvals/{approvalId}/reject
 { "decisionNote": "Budget too high for this role." }
 ```

 ## 请求修改（Request Revision）

 ```
 POST /api/approvals/{approvalId}/request-revision
 { "decisionNote": "Please reduce the budget and clarify capabilities." }
 ```

 ## 重新提交（Resubmit）

 ```
 POST /api/approvals/{approvalId}/resubmit
 { "payload": { "updated": "config..." } }
 ```

 ## 关联的 Issues（Linked Issues）

 ```
 GET /api/approvals/{approvalId}/issues
 ```

 返回与该审批关联的所有 issues。

 ## 审批评论（Approval Comments）

 ```
 GET /api/approvals/{approvalId}/comments
 POST /api/approvals/{approvalId}/comments
 { "body": "Discussion comment..." }
 ```

 ## 审批生命周期（Approval Lifecycle）

 ```
 pending -> approved
         -> rejected
         -> revision_requested -> resubmitted -> pending
 ```
