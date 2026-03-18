 ---
 title: Claude Local
 summary: Claude Code 本地适配器的安装与配置
 ---

 `claude_local` 适配器会在本机运行 Anthropic 的 Claude Code CLI。它支持会话持久化、技能注入以及结构化输出解析。

 ## 前置条件（Prerequisites）

 - 已安装 Claude Code CLI（确保可以执行 `claude` 命令）
 - 在环境变量或 agent 配置中设置好 `ANTHROPIC_API_KEY`

 ## 配置字段（Configuration Fields）

 | 字段 | 类型 | 必填 | 描述 |
 |------|------|------|------|
 | `cwd` | string | 是 | agent 进程的工作目录（绝对路径；如权限允许，缺失时会自动创建） |
 | `model` | string | 否 | 要使用的 Claude 模型（例如 `claude-opus-4-6`） |
 | `promptTemplate` | string | 否 | 用于所有运行的提示词模板 |
 | `env` | object | 否 | 环境变量（支持 secret 引用） |
 | `timeoutSec` | number | 否 | 进程超时时间（0 表示不限制） |
 | `graceSec` | number | 否 | 超时前的宽限期，超时后到强制 kill 之间的时间 |
 | `maxTurnsPerRun` | number | 否 | 每次心跳中允许的最大 agent 回合数 |
 | `dangerouslySkipPermissions` | boolean | 否 | 跳过权限确认（仅建议在开发环境使用） |

 ## 提示词模板（Prompt Templates）

 模板支持 `{{variable}}` 形式的占位符替换：

 | 变量 | 含义 |
 |------|------|
 | `{{agentId}}` | agent 的 ID |
 | `{{companyId}}` | 公司 ID |
 | `{{runId}}` | 当前运行 ID |
 | `{{agent.name}}` | agent 名称 |
 | `{{company.name}}` | 公司名称 |

 ## 会话持久化（Session Persistence）

 适配器会在多次心跳之间持久化 Claude Code 的会话 ID。下一次唤醒时，会自动恢复到上一次会话，从而保留对话上下文。

 会话恢复是基于 `cwd` 感知的：如果自上次运行以来 agent 的工作目录发生了变化，将会启动一个新的会话而不是恢复旧会话。

 如果恢复时遇到“未知会话”错误，适配器会自动退回到新会话并重试。

 ## 技能注入（Skills Injection）

 适配器会创建一个临时目录，将 Paperclip 的技能通过符号链接挂载进去，并通过 `--add-dir` 传给 Claude。这样可以让技能对 agent 可见，而不会污染 agent 的工作目录。

 ## 环境测试（Environment Test）

 在 UI 中使用 “Test Environment” 按钮来验证适配器配置。它会检查：

 - Claude CLI 是否已安装且可访问
 - 工作目录是否为绝对路径且可用（在权限允许的情况下会自动创建）
 - API key/认证模式提示（`ANTHROPIC_API_KEY` vs 订阅登录）
 - 通过 `claude --print - --output-format stream-json --verbose` 与提示词 `Respond with hello.` 进行实时探测，以验证 CLI 是否就绪
