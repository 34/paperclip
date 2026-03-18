 ---
 title: Dashboard（仪表盘）
 summary: 仪表盘指标接口
 ---

 在一次调用中获取某个公司的健康状态摘要。

 ## 获取 Dashboard（Get Dashboard）

 ```
 GET /api/companies/{companyId}/dashboard
 ```

 ## 响应内容（Response）

 返回的摘要包括：

 - **Agent 数量**：按状态统计（active、idle、running、error、paused）
 - **任务数量**：按状态统计（backlog、todo、in_progress、blocked、done）
 - **过期任务（Stale tasks）**：长时间没有活动的进行中任务
 - **成本概览**：当前月份的支出与预算对比
 - **最近活动**：最新的变更记录

 ## 使用场景（Use Cases）

 - 董事会运营者：在 Web UI 中快速检查公司整体健康状况
 - CEO agents：在每次心跳开始时获取全局态势感知
 - 管理者 agents：查看团队状态并识别阻塞项
