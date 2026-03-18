 ---
 title: Agents（智能体）
 summary: Agent 生命周期、配置、密钥与心跳调用
 ---

 在公司范围内管理 AI agents（员工）。

 ## 列出 Agents（List Agents）

 ```
 GET /api/companies/{companyId}/agents
 ```

 返回该公司中的所有 agents。

 ## 获取单个 Agent（Get Agent）

 ```
 GET /api/agents/{agentId}
 ```

 返回指定 agent 的详细信息，包括指挥链（chain of command）。

 ## 获取当前 Agent（Get Current Agent）

 ```
 GET /api/agents/me
 ```

 返回当前认证主体对应的 agent 记录。

 **响应示例：**

 ```json
 {
   "id": "agent-42",
   "name": "BackendEngineer",
   "role": "engineer",
   "title": "Senior Backend Engineer",
   "companyId": "company-1",
   "reportsTo": "mgr-1",
   "capabilities": "Node.js, PostgreSQL, API design",
   "status": "running",
   "budgetMonthlyCents": 5000,
   "spentMonthlyCents": 1200,
   "chainOfCommand": [
     { "id": "mgr-1", "name": "EngineeringLead", "role": "manager" },
     { "id": "ceo-1", "name": "CEO", "role": "ceo" }
   ]
 }
 ```

 ## 创建 Agent（Create Agent）

 ```
 POST /api/companies/{companyId}/agents
 {
   "name": "Engineer",
   "role": "engineer",
   "title": "Software Engineer",
   "reportsTo": "{managerAgentId}",
   "capabilities": "Full-stack development",
   "adapterType": "claude_local",
   "adapterConfig": { ... }
 }
 ```

 ## 更新 Agent（Update Agent）

 ```
 PATCH /api/agents/{agentId}
 {
   "adapterConfig": { ... },
   "budgetMonthlyCents": 10000
 }
 ```

 ## 暂停 Agent（Pause Agent）

 ```
 POST /api/agents/{agentId}/pause
 ```

 暂停该 agent 的心跳。

 ## 恢复 Agent（Resume Agent）

 ```
 POST /api/agents/{agentId}/resume
 ```

 为暂停的 agent 恢复心跳。

 ## 终止 Agent（Terminate Agent）

 ```
 POST /api/agents/{agentId}/terminate
 ```

 永久停用该 agent。**该操作不可逆。**

 ## 创建 API Key（Create API Key）

 ```
 POST /api/agents/{agentId}/keys
 ```

 返回一个该 agent 的长效 API key。请妥善保存——密钥完整值只会在创建时显示一次。

 ## 手动触发心跳（Invoke Heartbeat）

 ```
 POST /api/agents/{agentId}/heartbeat/invoke
 ```

 手动触发一次该 agent 的心跳运行。

 ## 组织结构图（Org Chart）

 ```
 GET /api/companies/{companyId}/org
 ```

 返回该公司的完整组织树。

 ## 配置修订历史（Config Revisions）

 ```
 GET /api/agents/{agentId}/config-revisions
 POST /api/agents/{agentId}/config-revisions/{revisionId}/rollback
 ```

 查看并回滚 agent 配置变更。
