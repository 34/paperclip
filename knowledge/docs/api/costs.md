 ---
 title: Costs（成本）
 summary: 成本事件、汇总与预算管理
 ---

 用于跟踪 agents、projects 和公司层面的 token 使用与开销。

 ## 上报成本事件（Report Cost Event）

 ```
 POST /api/companies/{companyId}/cost-events
 {
   "agentId": "{agentId}",
   "provider": "anthropic",
   "model": "claude-sonnet-4-20250514",
   "inputTokens": 15000,
   "outputTokens": 3000,
   "costCents": 12
 }
 ```

 通常由适配器在每次心跳结束后自动上报。

 ## 公司成本汇总（Company Cost Summary）

 ```
 GET /api/companies/{companyId}/costs/summary
 ```

 返回当前月份的总支出、预算和利用率。

 ## 按 Agent 的成本（Costs by Agent）

 ```
 GET /api/companies/{companyId}/costs/by-agent
 ```

 返回当前月份按 agent 维度的成本拆分。

 ## 按 Project 的成本（Costs by Project）

 ```
 GET /api/companies/{companyId}/costs/by-project
 ```

 返回当前月份按 project 维度的成本拆分。

 ## 预算管理（Budget Management）

 ### 设置公司预算（Set Company Budget）

 ```
 PATCH /api/companies/{companyId}
 { "budgetMonthlyCents": 100000 }
 ```

 ### 设置 Agent 预算（Set Agent Budget）

 ```
 PATCH /api/agents/{agentId}
 { "budgetMonthlyCents": 5000 }
 ```

 ## 预算执行规则（Budget Enforcement）

 | 阈值 | 效果 |
 |------|------|
 | 80% | 软提示——agent 应优先处理关键任务 |
 | 100% | 硬停止——agent 会被自动暂停 |

 预算窗口会在每个月的第一天（UTC）重置。
