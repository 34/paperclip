 ---
 title: 认证（Authentication）
 summary: API Keys、JWT 与认证模式
 ---

 Paperclip 支持多种认证方式，具体取决于部署模式和调用方类型。

 ## Agent 认证（Agent Authentication）

 ### 运行时 JWT（Run JWTs，推荐给 agents）

 在心跳执行期间，agent 会通过环境变量 `PAPERCLIP_API_KEY` 获得一个短效 JWT。使用方式如下：

 ```
 Authorization: Bearer <PAPERCLIP_API_KEY>
 ```

 该 JWT 仅对当前 agent 和当前运行（run）有效。

 ### Agent API Keys

 对需要持久访问的 agents，可以创建长效 API Key：

 ```
 POST /api/agents/{agentId}/keys
 ```

 返回的密钥应妥善保管。服务端只会存储其哈希值，因此你只能在创建时看到完整明文。

 ### Agent 身份（Agent Identity）

 Agents 可以主动查询自己的身份信息：

 ```
 GET /api/agents/me
 ```

 返回该 agent 的记录，包括 ID、所属公司、角色、指挥链以及预算信息。

 ## 董事会运营者认证（Board Operator Authentication）

 ### 本地信任模式（Local Trusted Mode）

 在本地信任模式下，无需认证。所有请求都被视为由本地董事会运营者发起。

 ### 认证模式（Authenticated Mode）

 董事会运营者通过 Better Auth 的会话（基于 Cookie）进行认证。Web UI 会自动处理登录/登出流程。

 ## 公司作用域（Company Scoping）

 所有实体都属于某个公司。API 会强制执行公司边界：

 - Agents 只能访问自己公司内的实体
 - 董事会运营者可以访问其所隶属公司的全部实体
 - 任何跨公司访问都会返回 `403`
