 ---
 title: Process 适配器
 summary: 通用 shell 进程适配器
 ---

 `process` 适配器用于执行任意 shell 命令。可以用它来跑简单脚本、一次性任务，或基于自定义框架实现的 agents。

 ## 适用场景（When to Use）

 - 运行调用 Paperclip API 的 Python 脚本
 - 执行自定义的 agent 循环
 - 任何可以通过 shell 命令启动的运行时

 ## 不适用的场景（When Not to Use）

 - 需要在多次运行之间保持会话状态（推荐使用 `claude_local` 或 `codex_local`）
 - 需要在心跳之间保留对话上下文的场景

 ## 配置（Configuration）

 | 字段 | 类型 | 必填 | 描述 |
 |------|------|------|------|
 | `command` | string | 是 | 要执行的 shell 命令 |
 | `cwd` | string | 否 | 工作目录 |
 | `env` | object | 否 | 环境变量 |
 | `timeoutSec` | number | 否 | 进程超时时间 |

 ## 工作原理（How It Works）

 1. Paperclip 以子进程的形式启动配置好的命令
 2. 注入标准的 Paperclip 环境变量（`PAPERCLIP_AGENT_ID`、`PAPERCLIP_API_KEY` 等）
 3. 子进程运行直到结束
 4. 退出码用于判断成功或失败

 ## 示例（Example）

 下面是一个运行 Python 脚本的 agent：

 ```json
 {
   "adapterType": "process",
   "adapterConfig": {
     "command": "python3 /path/to/agent.py",
     "cwd": "/path/to/workspace",
     "timeoutSec": 300
   }
 }
 ```

 脚本中可以使用注入的环境变量来完成对 Paperclip API 的认证并执行实际工作。
*** End Patch
