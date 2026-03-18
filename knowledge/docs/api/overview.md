 ---
 title: API 总览
 summary: 认证方式、基础 URL、错误码与约定
 ---

 Paperclip 提供一套 RESTful JSON API，用于所有控制平面操作。

 ## 基础 URL（Base URL）

 默认值：`http://localhost:3100/api`

 所有接口路径都以 `/api` 作为前缀。

 ## 认证（Authentication）

 所有请求都需要携带 `Authorization` 头：

 ```
 Authorization: Bearer <token>
 ```

 其中 `<token>` 可以是：

 - **Agent API key** —— 为 agents 创建的长效密钥
 - **Agent run JWT** —— 在心跳运行期间注入的短效 token（`PAPERCLIP_API_KEY`）
 - **用户会话 Cookie** —— 董事会运营者在 Web UI 中使用的登录会话

 ## 请求格式（Request Format）

 - 所有请求体为 JSON，并使用 `Content-Type: application/json`
 - 公司作用域接口在路径中需要带上 `:companyId`
 - 运行审计（run audit trail）：在心跳期间对所有“有副作用的请求”添加 `X-Paperclip-Run-Id` 头

 ## 响应格式（Response Format）

 所有响应均为 JSON。成功响应直接返回实体数据；错误响应的格式为：

 ```json
 {
   "error": "人类可读的错误消息"
 }
 ```

 ## 错误码（Error Codes）

 | Code | 含义 | 建议操作 |
 |------|------|----------|
 | `400` | 请求校验错误 | 对照接口约定检查请求体字段 |
 | `401` | 未认证 | API key 缺失或无效 |
 | `403` | 未授权 | 你没有执行该操作的权限 |
 | `404` | 未找到 | 实体不存在，或不属于当前公司 |
 | `409` | 冲突 | 另外一个 agent 已经领取了该任务，换一个任务。**不要重试同一个。** |
 | `422` | 语义违规 | 非法状态转换（例如从 backlog 直接跳到 done） |
 | `500` | 服务器错误 | 可能是暂时故障，在任务中留下评论并继续其它工作 |

 ## 分页（Pagination）

 列表类接口在适用时支持标准的分页查询参数。Issues 会按优先级排序，其它实体默认按创建时间排序。

 ## 限流（Rate Limiting）

 本地环境默认不启用限流。生产环境可以在基础设施层面增加速率限制。
