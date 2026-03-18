# Agent 认证与入驻（Agent Authentication & Onboarding）

## 问题（Problem）

Agents 需要 API key 才能与 Paperclip 认证。当前做法（在应用中生成 key 再手动配到环境变量）繁琐且难以扩展。不同适配器类型有不同信任模型，我们希望支持从「零配置本地」到「agent 自主注册」的谱系。

## 设计原则（Design Principles）

1. **认证复杂度与信任边界匹配。** 本地 CLI 适配器不应与远程 webhook agent 需要同等仪式。
2. **Agents 应能自行入驻。** 当 agent 有能力完成时，不应由人类在 agent 环境中复制粘贴凭证。
3. **默认审批闸门。** 自主注册必须经显式审批（用户或授权 agent）后，新 agent 才能在公司内行动。

---

## 认证层级（Authentication Tiers）

### Tier 1：本地适配器（claude-local、codex-local）

**信任模型：** 适配器进程与 Paperclip 服务在同一机器运行（或由服务直接调用），无实质网络边界。

**做法：** Paperclip 在调用时生成 token 并通过参数/环境变量直接传给 agent 进程，无需手动配置。

**Token 形式：** 每次心跳（或每会话）签发的短效 JWT。服务签发后通过适配器调用传入，API 请求时验证。

**生命周期：** 编码型 agent 可能运行数小时，token 不宜过短；无限期 token 即使本地也不理想。采用较长有效期（如 48h）的 JWT 与重叠窗口，使临近过期启动的心跳仍能完成。服务无需存储这些 token，仅验证 JWT 签名。

**状态：** 部分已实现。本地适配器已传入 `PAPERCLIP_API_URL`、`PAPERCLIP_AGENT_ID`、`PAPERCLIP_COMPANY_ID`；需增加 `PAPERCLIP_API_KEY`（JWT）。

### Tier 2：CLI 驱动的 Key 交换（CLI-Driven Key Exchange）

**信任模型：** 开发者配置远程或半远程 agent 并有 shell 访问。做法类似 `claude setup-token`：运行 Paperclip CLI 命令打开浏览器确认，然后将 token 写入 agent 配置。Token 为长效 API key（服务端哈希存储）。**状态：** 规划中，待远程适配器需求时实现。

### Tier 3：Agent 自主注册（邀请链接）（Agent Self-Registration）

（详见英文原文。）
