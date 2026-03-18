 ---
 title: Secrets（密钥）
 summary: Secrets 的增删改查
 ---

 管理 agents 在其环境配置中引用的加密密钥。

 ## 列出 Secrets（List Secrets）

 ```
 GET /api/companies/{companyId}/secrets
 ```

 返回密钥的元数据（不包含解密后的值）。

 ## 创建 Secret（Create Secret）

 ```
 POST /api/companies/{companyId}/secrets
 {
   "name": "anthropic-api-key",
   "value": "sk-ant-..."
 }
 ```

 `value` 在静态存储时会被加密。响应中只会返回密钥 ID 和元数据。

 ## 更新 Secret（Update Secret）

 ```
 PATCH /api/secrets/{secretId}
 {
   "value": "sk-ant-new-value..."
 }
 ```

 会为该密钥创建一个新的版本。引用 `"version": "latest"` 的 agents 会在下一次心跳时自动使用新值。

 ## 在 Agent 配置中使用 Secrets

 在 agent 的适配器配置中通过引用密钥而不是直接内联值：

 ```json
 {
   "env": {
     "ANTHROPIC_API_KEY": {
       "type": "secret_ref",
       "secretId": "{secretId}",
       "version": "latest"
     }
   }
 }
 ```

 服务端会在运行时解析并解密这些 secret 引用，将真实值注入到 agent 进程的环境变量中。
