 ---
 title: Secrets 管理（Secrets Management）
 summary: 主密钥、加密与严格模式
 ---

 Paperclip 使用本地主密钥在静态存储时对 secrets 进行加密。agent 环境变量中的敏感值（API key、token 等）会以加密 secret 引用的形式存储。

 ## 默认提供方：`local_encrypted`

 secrets 使用本地主密钥加密，主密钥文件位置：

 ```
 ~/.paperclip/instances/default/secrets/master.key
 ```

 该密钥会在 Onboard 过程中自动创建，不会离开本机。

 ## 配置（Configuration）

 ### 通过 CLI 设置

 Onboard 时会写入默认的 secrets 配置：

 ```sh
 pnpm paperclipai onboard
 ```

 更新 secrets 设置：

 ```sh
 pnpm paperclipai configure --section secrets
 ```

 验证 secrets 配置：

 ```sh
 pnpm paperclipai doctor
 ```

 ### 环境变量覆盖（Environment Overrides）

 | 变量 | 描述 |
 |------|------|
 | `PAPERCLIP_SECRETS_MASTER_KEY` | 32 字节密钥（base64、hex 或原始字符串） |
 | `PAPERCLIP_SECRETS_MASTER_KEY_FILE` | 自定义主密钥文件路径 |
 | `PAPERCLIP_SECRETS_STRICT_MODE` | 设为 `true` 时启用严格模式 |

 ## 严格模式（Strict Mode）

 在严格模式下，所有敏感环境变量（匹配 `*_API_KEY`、`*_TOKEN`、`*_SECRET`）必须使用 secret 引用，不能直接内联明文：

 ```sh
 PAPERCLIP_SECRETS_STRICT_MODE=true
 ```

 推荐在所有非 `local_trusted` 部署中启用。

 ## 迁移内联 secrets（Migrating Inline Secrets）

 如果已有 agents 在配置中内联明文 API key，可以通过命令迁移到加密 secret 引用：

 ```sh
 pnpm secrets:migrate-inline-env         # 预览（dry run）
 pnpm secrets:migrate-inline-env --apply # 实际应用迁移
 ```

 ## 在 Agent 配置中引用 secrets

 Agent 环境变量通过 secret 引用使用 secrets：

 ```json
 {
   "env": {
     "ANTHROPIC_API_KEY": {
       "type": "secret_ref",
       "secretId": "8f884973-c29b-44e4-8ea3-6413437f8081",
       "version": "latest"
     }
   }
 }
 ```

 服务器会在运行时解析并解密这些引用，将真实值注入到 agent 进程环境变量中。
*** End Patch```} ***!
