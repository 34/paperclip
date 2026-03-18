 ---
 title: Codex Local
 summary: OpenAI Codex 本地适配器的安装与配置
 ---

 `codex_local` 适配器会在本机运行 OpenAI 的 Codex CLI。它通过 `previous_response_id` 链接来保持会话连续性，并通过全局 Codex skills 目录注入 Paperclip 技能。

 ## 前置条件（Prerequisites）

 - 已安装 Codex CLI（确保可以执行 `codex` 命令）
 - 在环境变量或 agent 配置中设置好 `OPENAI_API_KEY`

 ## 配置字段（Configuration Fields）

 | 字段 | 类型 | 必填 | 描述 |
 |------|------|------|------|
 | `cwd` | string | 是 | agent 进程的工作目录（绝对路径；如权限允许，缺失时会自动创建） |
 | `model` | string | 否 | 要使用的模型 |
 | `promptTemplate` | string | 否 | 用于所有运行的提示词模板 |
 | `env` | object | 否 | 环境变量（支持 secret 引用） |
 | `timeoutSec` | number | 否 | 进程超时时间 |
 | `graceSec` | number | 否 | 超时前的宽限期 |
 | `dangerouslyBypassApprovalsAndSandbox` | boolean | 否 | 绕过安全检查（仅开发环境使用） |

 ## 会话持久化（Session Persistence）

 Codex 使用 `previous_response_id` 来维持会话上下文。适配器会在心跳之间序列化并恢复该字段，使 agent 能够在多次运行之间保持对话状态。

 ## 技能注入（Skills Injection）

 适配器会将 Paperclip 的技能通过符号链接挂载到 Codex 的全局技能目录 `~/.codex/skills` 中。现有的用户技能不会被覆盖。

 ## 环境测试（Environment Test）

 环境测试会检查：

 - Codex CLI 是否已安装且可访问
 - 工作目录是否为绝对路径且可用（在权限允许的情况下会自动创建）
 - 认证信号（是否存在 `OPENAI_API_KEY`）
 - 通过 `codex exec --json -` 和提示词 `Respond with hello.` 的实时探测，以确保 CLI 实际可以运行
*** End Patch
