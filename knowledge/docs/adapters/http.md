 ---
 title: HTTP 适配器
 summary: HTTP webhook 适配器
 ---

 `http` 适配器会向外部 agent 服务发送一个 webhook 请求。agent 在外部运行，Paperclip 只负责触发。

 ## 适用场景（When to Use）

 - agent 以外部服务形式运行（云函数、独立服务器等）
 - 采用 fire-and-forget 的触发模型
 - 与第三方 agent 平台集成

 ## 不适用的场景（When Not to Use）

 - agent 直接运行在同一台机器上（此时更适合使用 `process`、`claude_local` 或 `codex_local`）
 - 需要捕获 stdout 并在运行视图中实时查看输出

 ## 配置（Configuration）

 | 字段 | 类型 | 必填 | 描述 |
 |------|------|------|------|
 | `url` | string | 是 | 要 POST 的 webhook URL |
 | `headers` | object | 否 | 额外的 HTTP 请求头 |
 | `timeoutSec` | number | 否 | 请求超时时间 |

 ## 工作原理（How It Works）

 1. Paperclip 向配置好的 URL 发送 POST 请求
 2. 请求体包含执行上下文（agent ID、任务信息、唤醒原因等）
 3. 外部 agent 处理请求，并通过 Paperclip API 回写结果
 4. webhook 的 HTTP 响应会被作为本次运行的结果记录

 ## 请求体（Request Body）

 Webhook 会收到如下 JSON 负载：

 ```json
 {
   "runId": "...",
   "agentId": "...",
   "companyId": "...",
   "context": {
     "taskId": "...",
     "wakeReason": "...",
     "commentId": "..."
   }
 }
 ```

 外部 agent 需要使用 `PAPERCLIP_API_URL` 与 API key 来调用 Paperclip。
*** End Patch
