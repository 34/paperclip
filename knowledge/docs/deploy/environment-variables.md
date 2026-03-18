 ---
 title: 环境变量（Environment Variables）
 summary: 环境变量完整参考
 ---

 本文列出了 Paperclip 用于服务器配置的所有环境变量。

 ## 服务器配置（Server Configuration）

 | 变量 | 默认值 | 描述 |
 |------|--------|------|
 | `PORT` | `3100` | 服务器端口 |
 | `HOST` | `127.0.0.1` | 服务器绑定主机 |
 | `DATABASE_URL` | （嵌入式） | PostgreSQL 连接字符串 |
 | `PAPERCLIP_HOME` | `~/.paperclip` | Paperclip 所有数据的根目录 |
 | `PAPERCLIP_INSTANCE_ID` | `default` | 实例标识（支持多本地实例） |
 | `PAPERCLIP_DEPLOYMENT_MODE` | `local_trusted` | 运行模式覆盖 |

 ## Secrets（密钥）

 | 变量 | 默认值 | 描述 |
 |------|--------|------|
 | `PAPERCLIP_SECRETS_MASTER_KEY` | （从文件读取） | 32 字节加密密钥（base64/hex/raw） |
 | `PAPERCLIP_SECRETS_MASTER_KEY_FILE` | `~/.paperclip/.../secrets/master.key` | 密钥文件路径 |
 | `PAPERCLIP_SECRETS_STRICT_MODE` | `false` | 对敏感环境变量强制使用 secret 引用 |

 ## Agent 运行时变量（Agent Runtime）

 以下变量在调用 agent 进程时由服务器自动注入：

 | 变量 | 描述 |
 |------|------|
 | `PAPERCLIP_AGENT_ID` | agent 唯一 ID |
 | `PAPERCLIP_COMPANY_ID` | 公司 ID |
 | `PAPERCLIP_API_URL` | Paperclip API 基础 URL |
 | `PAPERCLIP_API_KEY` | 用于 API 认证的短效 JWT |
 | `PAPERCLIP_RUN_ID` | 当前心跳运行 ID |
 | `PAPERCLIP_TASK_ID` | 触发本次唤醒的 issue ID |
 | `PAPERCLIP_WAKE_REASON` | 唤醒原因 |
 | `PAPERCLIP_WAKE_COMMENT_ID` | 触发唤醒的评论 ID |
 | `PAPERCLIP_APPROVAL_ID` | 关联审批 ID |
 | `PAPERCLIP_APPROVAL_STATUS` | 审批决策状态 |
 | `PAPERCLIP_LINKED_ISSUE_IDS` | 关联 issue IDs（逗号分隔） |

 ## LLM 提供方密钥（LLM Provider Keys）

 | 变量 | 描述 |
 |------|------|
 | `ANTHROPIC_API_KEY` | Anthropic API key（用于 Claude Local 适配器） |
 | `OPENAI_API_KEY` | OpenAI API key（用于 Codex Local 适配器） |
